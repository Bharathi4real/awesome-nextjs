```ts
import fs from "node:fs";
import path from "node:path";

interface PackageJson {
  dependencies?: Record<string, string>;
  devDependencies?: Record<string, string>;
}

interface OSVQuery {
  package: {
    name: string;
    ecosystem: string;
  };
  version: string;
}

interface OSVResponse {
  results: {
    vulns?: {
      id: string;
      summary?: string;
      details?: string;
      severity?: { type: string; score: string }[];
    }[];
  }[];
}

const ECOSYSTEM = "npm";

async function checkCVEs() {
  const packageJsonPath = path.join(process.cwd(), "package.json");

  if (!fs.existsSync(packageJsonPath)) {
    console.error("Error: package.json not found in the current directory.");
    process.exit(1);
  }

  const packageJsonContent = fs.readFileSync(packageJsonPath, "utf-8");
  const packageJson: PackageJson = JSON.parse(packageJsonContent);

  const dependencies = {
    ...packageJson.dependencies,
    ...packageJson.devDependencies,
  };

  const queries: OSVQuery[] = [];
  const packagesToCheck: string[] = [];

  for (const [name, versionStr] of Object.entries(dependencies)) {
    // Clean version string (remove ^, ~, etc. for simple checking, though OSV handles some)
    // For a more robust check we might need to resolve the installed version from node_modules or lock file.
    // For this utility, we'll try to use the version specified in package.json, stripping basics.
    const version = versionStr.replace(/^[\^~]/, "");

    // Skip local paths or git urls for now
    if (version.startsWith("file:") || version.includes("/")) {
      continue;
    }

    queries.push({
      package: {
        name,
        ecosystem: ECOSYSTEM,
      },
      version,
    });
    packagesToCheck.push(`${name}@${version}`);
  }

  if (queries.length === 0) {
    console.log("No suitable dependencies found to check.");
    return;
  }

  console.log(`Checking ${queries.length} packages for vulnerabilities...`);

  try {
    // OSV.dev batch API
    const response = await fetch("https://api.osv.dev/v1/querybatch", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ queries }),
    });

    if (!response.ok) {
      throw new Error(`API request failed with status ${response.status}`);
    }

    const data = (await response.json()) as OSVResponse;
    const results = data.results;

    let foundVulnerabilities = false;

    results.forEach((result, index) => {
      if (result.vulns && result.vulns.length > 0) {
        foundVulnerabilities = true;
        const pkg = packagesToCheck[index];
        console.log(`\n⚠️  Vulnerabilities found in ${pkg}:`);

        result.vulns.forEach((vuln) => {
          console.log(`  - [${vuln.id}] ${vuln.summary || "No summary"}`);
          if (vuln.severity) {
            vuln.severity.forEach((s) => {
              console.log(`    Severity: ${s.type} ${s.score}`);
            });
          }
          console.log(`    https://osv.dev/vulnerability/${vuln.id}`);
        });
      }
    });

    if (!foundVulnerabilities) {
      console.log("\n✅ No vulnerabilities found in checked dependencies.");
    } else {
      console.log("\n❌ Vulnerabilities found.");
      process.exit(1); // Exit with error code if vulnerabilities found
    }
  } catch (error) {
    console.error("Error querying OSV API:", error);
    process.exit(1);
  }
}

checkCVEs();

```

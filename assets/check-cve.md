# CVE Checker for Node.js Dependencies

This document outlines a TypeScript utility that checks your Node.js project dependencies for known security vulnerabilities using the OSV.dev API.

## Installation

Ensure you have the necessary dependencies installed in your project:

```bash
npm install -D typescript @types/node
```

## Overview

The CVE checker scans both `dependencies` and `devDependencies` from your `package.json` file and queries the OSV.dev API to identify known vulnerabilities.

## Features

- 🔍 Scans both production and development dependencies
- 🚀 Batch API queries for efficient vulnerability checking
- 📊 Displays vulnerability IDs, summaries, and severity scores
- ✅ Clean exit codes for CI/CD integration
- 🔗 Direct links to vulnerability details on OSV.dev

## Usage

### Basic Usage

Run the script from your project root directory (where `package.json` is located):

```bash
# Using ts-node
npx ts-node check-cves.ts

# Or compile and run
tsc check-cves.ts
node check-cves.js
```

### Add as NPM Script

Add to your `package.json`:

```json
{
  "scripts": {
    "check-cves": "ts-node check-cves.ts",
    "security": "npm run check-cves"
  }
}
```

Then run:

```bash
npm run check-cves
```

### CI/CD Integration

The script exits with code `1` if vulnerabilities are found, making it perfect for CI/CD pipelines:

```yaml
# Example GitHub Actions workflow
- name: Check for vulnerabilities
  run: npm run check-cves
```

```yaml
# Example GitLab CI
security_check:
  script:
    - npm run check-cves
```

## How It Works

### 1. Reading package.json

The script reads your project's `package.json` and extracts all dependencies:

```typescript
const packageJsonPath = path.join(process.cwd(), "package.json");
const packageJsonContent = fs.readFileSync(packageJsonPath, "utf-8");
const packageJson: PackageJson = JSON.parse(packageJsonContent);
```

### 2. Preparing Queries

Dependencies are prepared for batch querying, with version string cleanup:

```typescript
const version = versionStr.replace(/^[\^~]/, "");
queries.push({
  package: {
    name,
    ecosystem: ECOSYSTEM,
  },
  version,
});
```

### 3. Querying OSV.dev API

All packages are checked in a single batch request:

```typescript
const response = await fetch("https://api.osv.dev/v1/querybatch", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
  },
  body: JSON.stringify({ queries }),
});
```

### 4. Processing Results

The response is parsed and vulnerabilities are displayed with detailed information including severity scores and direct links to OSV.dev.

## Output Examples

### No Vulnerabilities Found

```
Checking 42 packages for vulnerabilities...

✅ No vulnerabilities found in checked dependencies.
```

### Vulnerabilities Found

```
Checking 42 packages for vulnerabilities...

⚠️  Vulnerabilities found in express@4.17.1:
  - [GHSA-rv95-896h-c2vc] qs vulnerable to Prototype Pollution
    Severity: CVSS_V3 7.5
    https://osv.dev/vulnerability/GHSA-rv95-896h-c2vc

⚠️  Vulnerabilities found in axios@0.21.1:
  - [GHSA-4w2v-q235-vp99] Server-Side Request Forgery in axios
    Severity: CVSS_V3 5.3
    https://osv.dev/vulnerability/GHSA-4w2v-q235-vp99

❌ Vulnerabilities found.
```

## Full Code

```typescript
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
    // Clean version string (remove ^, ~, etc.)
    const version = versionStr.replace(/^[\^~]/, "");
    
    // Skip local paths or git urls
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
      process.exit(1);
    }
  } catch (error) {
    console.error("Error querying OSV API:", error);
    process.exit(1);
  }
}

checkCVEs();
```

## TypeScript Interfaces

### PackageJson

Defines the structure of a package.json file:

```typescript
interface PackageJson {
  dependencies?: Record<string, string>;
  devDependencies?: Record<string, string>;
}
```

### OSVQuery

Structure for querying the OSV.dev API:

```typescript
interface OSVQuery {
  package: {
    name: string;
    ecosystem: string;
  };
  version: string;
}
```

### OSVResponse

Expected response format from the OSV.dev API:

```typescript
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
```

## Limitations

- Version parsing is basic (strips `^` and `~` prefixes only)
- Skips local file paths and git URLs
- Uses versions from `package.json`, not resolved versions from lock files
- Requires internet connection to query OSV.dev API

## Improving Accuracy

For more accurate vulnerability detection, consider:

1. Using versions from `package-lock.json` or `yarn.lock`
2. Reading installed versions from `node_modules`
3. Using npm's built-in `npm audit` for production workflows

## Troubleshooting

### "package.json not found"

Ensure you're running the script from your project root directory where `package.json` is located.

### API Request Failed

- Check your internet connection
- Verify OSV.dev API is accessible at https://osv.dev
- Check for rate limiting (wait and retry if hitting limits)

### TypeScript Errors

Ensure `@types/node` is installed:

```bash
npm install -D @types/node
```

## API Reference

The script uses the [OSV.dev Batch Query API](https://osv.dev/docs/#tag/api/operation/OSV_QueryAffectedBatch):

- **Endpoint**: `https://api.osv.dev/v1/querybatch`
- **Method**: POST
- **Rate Limits**: Subject to OSV.dev API limits
- **Ecosystem**: npm

## Related Tools

- [npm audit](https://docs.npmjs.com/cli/v8/commands/npm-audit) - Official npm security auditing
- [Snyk](https://snyk.io/) - Commercial security scanning platform
- [Dependabot](https://github.com/dependabot) - Automated dependency updates
- [OSV-Scanner](https://github.com/google/osv-scanner) - Official OSV.dev CLI tool

---

## Contribution

Want to improve the CVE checker? Open a pull request with your suggestions!

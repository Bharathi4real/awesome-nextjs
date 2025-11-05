# Complete Biome & Next.js Setup Guide

## 1. Install Dependencies and Initialize Biome
```bash
bun i @biomejs/biome -D  
biome init
```

## 2. Update biome.json
Save the provided `biome.json` configuration in the project root with Next.js specific rules:

```jsonc
{
  "$schema": "https://biomejs.dev/schemas/2.3.3/schema.json",
  "vcs": {
    "enabled": true,
    "clientKind": "git",
    "useIgnoreFile": true
  },
  "files": {
    "ignoreUnknown": true,
    "includes": ["**", "!node_modules", "!.next", "!dist", "!build"]
  },
  "formatter": {
    "enabled": true,
    "indentStyle": "space",
    "indentWidth": 2
  },
  "linter": {
    "enabled": true,
    "rules": {
      "recommended": true,
      "suspicious": {
        "noUnknownAtRules": "off"
      }
    },
    "domains": {
      "next": "recommended",
      "react": "recommended"
    }
  },
  "css": {
    "parser": {
      "tailwindDirectives": true
    }
  },
  "assist": {
    "actions": {
      "source": {
        "organizeImports": "on"
      }
    }
  }
}
```

## 3. Disable ESLint Completely

### Uninstall ESLint Dependencies
```bash
bun remove eslint eslint-config-next @typescript-eslint/eslint-plugin @typescript-eslint/parser
```

### Remove ESLint Configuration Files
Delete the following files if they exist:
- `.eslintrc.json`
- `.eslintrc.js`
- `.eslintrc.yml`
- `.eslintrc.yaml`
- `eslint.config.js`
- `eslint.config.mjs`

## 4. Update package.json Scripts
Replace the scripts section in your `package.json` file with:

```json
{
  "name": "your-project-name",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev --turbopack",
    "build": "next build",
    "start": "next start -p 3005",
    "lint": "biome lint .",
    "typecheck": "tsc --noEmit",
    "format": "biome format --write . && biome check --write .",
    "clean": "node -e \"const {execSync} = require('child_process'); const isWin = process.platform === 'win32'; execSync(isWin ? 'rd /s /q .next && rd /s /q node_modules' : 'rm -rf .next node_modules');\""
  },
  "dependencies": {
    "next": "15.0.0",
    "react": "^18.0.0",
    "react-dom": "^18.0.0"
  },
  "devDependencies": {
    "@biomejs/biome": "^2.1.1",
    "@types/node": "^20.0.0",
    "@types/react": "^18.0.0",
    "@types/react-dom": "^18.0.0",
    "typescript": "^5.0.0"
  }
}
```

**Note**: Remove any existing `eslint` related scripts like `"lint": "next lint"` and replace with the Biome commands above.

## 5. VS Code Integration (Optional)
Install the Biome VS Code extension and add to your `.vscode/settings.json`:

```json
{
  "biome.enabled": true,
  "biome.lsp.bin": "application/node_modules/.bin/biome",
  "biome.lsp.trace.server": "off",
  "editor.defaultFormatter": "biomejs.biome",
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "quickfix.biome": "explicit",
    "source.organizeImports.biome": "explicit"
  },
  "eslint.enable": false
}
```

## Common Commands

```bash
# Development with hot reload
bun dev

# Build for production (ESLint disabled)
bun run build

# Format and fix all issues
bun format

# Lint only
bun lint

# Clean node modules and .next folders
bun clean
```

---

## Contribution
Want to suggest useful Biome configuration options for Next.js? Open a pull request!

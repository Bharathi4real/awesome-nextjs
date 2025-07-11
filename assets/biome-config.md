# Complete Biome & Next.js Setup Guide

## 1. Install Dependencies and Initialize Biome
```bash
pnpm i @biomejs/biome -D  
biome init
```

## 2. Update biome.json
Save the provided `biome.json` configuration in the project root with Next.js specific rules:

```json
{
  "$schema": "https://biomejs.dev/schemas/2.1.1/schema.json",
  "vcs": {
    "enabled": true,
    "clientKind": "git",
    "useIgnoreFile": true
  },
  "files": {
    "ignoreUnknown": false,
    "experimentalScannerIgnores": ["**/components/ui/**"]
  },
  "formatter": {
    "enabled": true,
    "formatWithErrors": false,
    "indentStyle": "space",
    "indentWidth": 2,
    "lineEnding": "lf",
    "lineWidth": 100,
    "attributePosition": "multiline",
    "bracketSameLine": false,
    "bracketSpacing": true,
    "expand": "auto",
    "useEditorconfig": true
  },
  "linter": {
    "enabled": true,
    "rules": {
      "recommended": true,
      "style": {
        "noNonNullAssertion": "off"
      },
      "a11y": {
        "noAutofocus": "off",
        "useAriaPropsForRole": "off",
        "noNoninteractiveElementToInteractiveRole": "off",
        "noRedundantAlt": "off",
        "noSvgWithoutTitle": "off",
        "useAltText": "off",
        "useButtonType": "off",
        "useHeadingContent": "off",
        "useHtmlLang": "off",
        "useKeyWithClickEvents": "off",
        "useKeyWithMouseEvents": "off",
        "useMediaCaption": "off",
        "useSemanticElements": "off",
        "useValidAriaProps": "off"
      }
    },
    "domains": {
      "next": "recommended"
    }
  },
  "javascript": {
    "formatter": {
      "jsxQuoteStyle": "double",
      "quoteProperties": "asNeeded",
      "trailingCommas": "all",
      "semicolons": "always",
      "arrowParentheses": "always",
      "bracketSameLine": false,
      "quoteStyle": "single",
      "attributePosition": "multiline",
      "bracketSpacing": true
    }
  },
  "html": {
    "formatter": {
      "selfCloseVoidElements": "always"
    }
  },
  "assist": {
    "enabled": true,
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
pnpm remove eslint eslint-config-next @typescript-eslint/eslint-plugin @typescript-eslint/parser
```

### Remove ESLint Configuration Files
Delete the following files if they exist:
- `.eslintrc.json`
- `.eslintrc.js`
- `.eslintrc.yml`
- `.eslintrc.yaml`
- `eslint.config.js`
- `eslint.config.mjs`

### Disable ESLint in next.config.js
Create or update your `next.config.js` file to prevent ESLint from running during builds (including Vercel):

```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  eslint: {
    // This will completely disable ESLint during builds on Vercel and locally
    ignoreDuringBuilds: true,
  },
}

module.exports = nextConfig
```

## 4. Update package.json Scripts
Replace the scripts section in your `package.json` file with:

```json
{
  "name": "your-project-name",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev --turbopack",
    "build": "npm run lint && npm run typecheck && next build",
    "start": "next start",
    "lint": "biome lint .",
    "lint:fix": "biome lint --write .",
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
pnpm dev

# Build for production (ESLint disabled)
pnpm build

# Format and fix all issues
pnpm format

# Lint only
pnpm lint
```

---

## Contribution
Want to suggest useful Biome configuration options for Next.js? Open a pull request!

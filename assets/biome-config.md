# Biome & Next.js Setup Guide

## 1. Install Dependencies and init biome
```bash
pnpm i @biomejs/biome -D  
biome init
```

## 2. Update biome.json
Save the provided `biome.json` configuration in the project root.
```json
{
  "$schema": "https://biomejs.dev/schemas/2.0.5/schema.json",
  "vcs": {
    "enabled": true,
    "clientKind": "git",
    "useIgnoreFile": true
  },
  "files": {
    "ignoreUnknown": false
  },
  "formatter": {
    "enabled": true,
    "formatWithErrors": false,
    "indentStyle": "space",
    "indentWidth": 2,
    "lineEnding": "lf",
    "lineWidth": 80,
    "attributePosition": "auto",
    "bracketSameLine": false,
    "bracketSpacing": true,
    "expand": "auto",
    "useEditorconfig": true
  },
  "linter": {
    "enabled": false,
    "rules": {
      "recommended": false
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
      "attributePosition": "auto",
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
(enable linter if you want)

## 3. Add Next.js Scripts
Add the following scripts to your `package.json` file:
```json
{
  "scripts": {
    "dev": "next dev --turbopack",
    "build": "next build",
    "start": "next start -p 3009",
    "lint": "next lint",
    "format": "biome format --write . && biome check --write ."
  }
}
```

## 4. Format Code
Run:
```bash
pnpm format
```

---

## Contribution
Want to suggest useful prettier config options? Open a pull request!

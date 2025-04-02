# Prettier & Next.js Setup Guide

## 1. Install Dependencies
```bash
pnpm install -D prettier prettier-plugin-tailwindcss
```
(Tailwind plugin is optional if you use SCSS instead of Tailwind CSS.)

## 2. Create Prettier Config
Save the provided `.prettierrc` configuration in the project root.
```json
{
  "printWidth": 80,
  "tabWidth": 2,
  "useTabs": false,
  "semi": true,
  "singleQuote": true,
  "trailingComma": "all",
  "bracketSpacing": true,
  "arrowParens": "always",
  "bracketSameLine": false,
  "endOfLine": "lf",
  "plugins": ["prettier-plugin-tailwindcss"],
  "tailwindStylesheet": "./styles/globals.css"
}
```
(Remove "prettier-plugin-tailwindcss and tailwindStylesheet" if using SCSS instead of Tailwind.)

## 3. Format Code
Run:
```bash
pnpm format
```
To check for issues:
```bash
pnpm check
```

## 4. Add Next.js Scripts
Add the following scripts to your `package.json` file:
```json
{
  "scripts": {
    "dev": "next dev --turbopack",
    "build": "next build",
    "start": "next start -p 3004",
    "lint": "next lint",
    "format": "prettier --write .",
    "check": "prettier --check ."
  }
}
```

## 5. Run Next.js with Turbopack
Start development:
```bash
pnpm dev
```
Build for production:
```bash
pnpm build && pnpm start
```

---

## Contribution
Want to suggest usefull prettier config options? Open a pull request!

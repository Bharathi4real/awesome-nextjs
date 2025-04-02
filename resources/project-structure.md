## Project Structure

### This project follows a modular architecture, ensuring scalability and maintainability.

```
/app                      # Next.js frontend
 ├── (auth)               # Authentication pages (login, register)
 ├── (dashboard)          # Protected pages (users, settings)
 │      ├── settings/     # Settings page
 │      └── users/        # Users page (route)
 │        └── page.tsx    # Users page component
 │
 ├── api                  # API routes (server-side logic)
 │      ├── auth/         # Authentication API (login, register)
 │      ├── users/        # Users API (get, create users)
 │      └── products/     # Products API (route)
 │        └── route.ts    # Product API handlers (get, add products)
 │
 ├── components/          # Reusable UI components
 │      ├── ui/           # Common UI elements
 │      ├── login/        # Login page components
 │      └── dashboard/    # Dashboard components
 │
 ├── hooks/               # Custom React hooks (e.g., useAuth, useTheme)
 ├── lib/                 # Utility functions, API configurations
 ├── store/               # Global state management (Redux, Zustand, etc.)
 ├── types/               # TypeScript types
 ├── utils/               # Helper functions (e.g., formatting, storage)
 │
 ├── styles/              # Global styles
 │      ├── globals.css   # Tailwind global styles (or SCSS if needed)
 │      └── scss/         # SCSS styles (variables, components)
 │
 ├── public/              # Static assets
 │      ├── images/       # Image files
 │      ├── videos/       # Video files
 │      └── audios/       # Audio files
 │
 ├── .env                 # Environment variables
 ├── next.config.js       # Next.js configuration
 ├── postcss.config.js    # PostCSS configuration (for Tailwind)
 ├── .prettierrc          # Prettier configuration
 ├── tsconfig.json        # TypeScript configuration
 ├── package.json         # Project dependencies
 └── README.md            # Project documentation

```
---

## Contribution
Want to suggest better structure? Open a pull request!

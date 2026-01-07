# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Development
- `pnpm dev` - Start development server with Vite
- `pnpm build` - Type check with TypeScript and build for production
- `pnpm preview` - Preview production build locally

### Code Quality
- `pnpm lint` - Run ESLint on the codebase
- `pnpm format` - Format code with Prettier
- `pnpm format:check` - Check code formatting without making changes

### Testing
- `pnpm test` - Run tests in watch mode
- `pnpm test:ui` - Run tests with Vitest UI
- `pnpm test:coverage` - Run tests with coverage report
- `pnpm test:watch` - Run tests in watch mode (same as `pnpm test`)

### Package Management
**IMPORTANT**: This project uses **pnpm only** for package management. Never use npm, yarn, or bun.
- `pnpm install` - Install dependencies
- `pnpm add [package]` - Add new dependency
- `pnpm remove [package]` - Remove dependency

## Architecture

### Tech Stack
- **React 19** with TypeScript
- **Vite** as build tool with SWC plugin for fast builds
- **TailwindCSS v4.1** for styling (CSS-first configuration)
- **React Router DOM v7** for routing
- **TanStack Query** for server state management
- **Zustand** with Immer for client state management
- **Vitest** with jsdom for testing
- **Axios** for HTTP requests

### Project Structure
```
src/
├── assets/
│   ├── css/index.css       # Main CSS with Tailwind imports
│   └── images/             # Static images
├── lib/
│   └── utils.ts            # Utility functions (cn helper for class merging)
├── models/
│   └── types.ts            # Shared TypeScript types
├── pages/                  # Page components
│   └── Home.tsx
├── provider/
│   └── queryProvider.tsx   # TanStack Query provider
├── routes/
│   └── routes.tsx          # Route configuration
├── App.tsx                 # Root app component with routing setup
└── main.tsx                # Application entry point
```

### Path Aliases
The project uses `@/` as an alias for the `src/` directory:
- Import example: `import { cn } from '@/lib/utils'`
- Configured in both `vite.config.ts` and `tsconfig.json`

### Routing
Routes are centrally defined in `src/routes/routes.tsx` as an array of route objects:
- Each route has: `href`, `id`, `name`, and `element`
- Routes are rendered in `App.tsx` using `react-router-dom`
- Unknown paths redirect to home (`/`)

### State Management
- **Server state**: Use TanStack Query (provider already set up in `App.tsx`)
- **Client state**: Use Zustand with Immer for immutable updates
- Example type pattern in `models/types.ts` shows separation of state and actions

### Styling with Tailwind v4.1

**Key Changes from v3:**
- Use `@import 'tailwindcss'` instead of `@tailwind` directives
- CSS-first configuration with `@theme` directive (not `tailwind.config.js`)
- Theme variables use `--category-name` format (e.g., `--color-blue-500`)
- Never use `var(--color-name)` directly in JSX - use utility classes like `bg-blue-500` or `text-blue-500`
- Built-in container queries with `@container` and `@sm:`, `@md:`, etc.
- Shadow utilities renamed: `shadow-sm` → `shadow-xs`, `shadow` → `shadow-sm`

**Styling Guidelines:**
- Use theme config colors from `src/assets/css/index.css` when available
- Use ShadCN components for UI (recommended)
- Use `cn()` helper from `@/lib/utils` to merge class names conditionally
- Hover styles only apply on devices that support hover (`@media (hover: hover)`)

### Testing Setup
- Vitest configured with jsdom environment
- Globals enabled (no need to import `describe`, `it`, `expect`)
- Setup file expected at `src/test/setup.ts` (create if needed with `@testing-library/jest-dom`)
- Coverage configured with v8 provider (text, JSON, and HTML reports)

### Environment Variables
- Store in `.env` file (not committed to git)
- Example: `VITE_MIXPANEL_TOKEN=token_here`
- Access in code: `import.meta.env.VITE_MIXPANEL_TOKEN`

### TypeScript Configuration
- Strict mode enabled
- Unused locals and parameters trigger warnings
- Path alias `@/*` maps to `src/*`
- No emit mode (Vite handles bundling)

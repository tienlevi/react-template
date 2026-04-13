# AGENTS.md

Instructions for AI coding agents working in this repository.

## Commands

### Build & Dev

- `pnpm dev` - Start Vite dev server
- `pnpm build` - Type check (`tsc -b`) then build for production
- `pnpm preview` - Preview production build

### Lint & Format

- `pnpm lint` - Run ESLint
- `pnpm format` - Format with Prettier
- `pnpm format:check` - Check formatting (no changes)

### Testing

- `pnpm test` - Run tests in watch mode (Vitest)
- `pnpm test:coverage` - Run tests once with coverage
- `pnpm test:ui` - Open Vitest UI
- **Run a single test**: `pnpm vitest run src/path/to/file.test.ts` or filter by name: `pnpm vitest -t "test name"`
- Globals enabled - no need to import `describe`, `it`, `expect`
- Setup file: `src/test/setup.ts` (create if missing, use `@testing-library/jest-dom`)

### Package Management

**pnpm only** - never use npm, yarn, or bun.

- `pnpm add [package]` / `pnpm remove [package]`

## Tech Stack

React 19, TypeScript (strict), Vite + SWC, TailwindCSS v4.1, React Router DOM v7, TanStack Query, Zustand + Immer, Axios, Vitest, ShadCN UI (new-york style, lucide icons)

## Project Structure

```
src/
├── assets/css/index.css   # Tailwind entry (@import 'tailwindcss')
├── components/ui/          # ShadCN components
├── interfaces/types.ts     # Shared TypeScript types
├── lib/utils.ts            # cn() helper (clsx + tailwind-merge)
├── pages/                  # Page components
├── provider/               # Context providers (QueryProvider)
├── routes/routes.tsx       # Route config array
├── App.tsx                 # Root component
└── main.tsx                # Entry point
```

## Path Aliases

`@/` maps to `src/`. Use `@/lib/utils`, `@/components/...`, etc. Never use relative `../../` for cross-directory imports.

## Code Style

### Imports

- Use `@/` alias for all `src/` imports
- Group: (1) React/library imports, (2) internal imports, (3) types
- Use single quotes per Prettier defaults where applicable; follow existing file conventions

### Components

- Default export for page components: `export default function Home()`
- Named exports for utilities, providers, hooks
- Functional components only, no class components
- Use `ReactNode` for children prop type

### TypeScript

- Strict mode enabled - do not use `any` (ESLint warns on it)
- Use `type` over `interface` for most definitions (see `interfaces/types.ts`)
- Prefix unused params/vars with `_` to suppress lint warnings
- Avoid type assertions (`as`) unless absolutely necessary

### Styling

- Tailwind v4 CSS-first config in `src/assets/css/index.css` using `@theme` directive
- Use utility classes directly, never `var(--color-name)` in JSX
- Use `cn()` from `@/lib/utils` for conditional/merged classes
- ShadCN components recommended for UI primitives
- Tailwind v4 shadow rename: `shadow-sm` -> `shadow-xs`, `shadow` -> `shadow-sm`

### State Management

- Server state: TanStack Query (use `useQuery`, `useMutation`)
- Client state: Zustand stores with Immer middleware for immutable updates

### Routing

- Routes defined in `src/routes/routes.tsx` as array of `{href, id, name, element}` objects
- Unknown paths redirect to `/`

### Naming

- Files: `camelCase.ts` for utilities/providers, `PascalCase.tsx` for components
- Types: `PascalCase`
- Variables/functions: `camelCase`
- Boolean prefixes: `is`, `has`, `should`

### Error Handling

- Use error boundaries for component-level errors
- TanStack Query `onError` for server errors
- Avoid silent catches - log or surface errors

## Environment Variables

Prefix with `VITE_`, access via `import.meta.env.VITE_*`. Store in `.env` (not committed).

## Don'ts

- Don't add comments unless asked
- Don't commit `.env` or secrets
- Don't use npm/yarn/bun
- Don't create files outside existing conventions
- Don't run `pnpm build` or `pnpm dev` unless asked

# React + TypeScript + Vite

A modern React application built with TypeScript, Vite, and Tailwind CSS. This project includes state management with Zustand, UI components from shadcn/ui, and is optimized for deployment on Cloudflare Pages.

## 🚀 Quick Start

### Prerequisites

- Node.js (v18 or higher)
- pnpm (recommended package manager)

### Installation

1. Clone the repository:

```bash
git clone <repository-url>
cd fe-vite
```

2. Install dependencies using pnpm:

```bash
pnpm install
```

3. Start the development server:

```bash
pnpm dev
```

## 📦 Package Management with pnpm

This project uses **pnpm** as the package manager. Here are the essential commands:

### Installing Dependencies

```bash
# Install all dependencies from package.json
pnpm install

# Add a new dependency
pnpm add <package-name>

# Add a development dependency
pnpm add -D <package-name>

# Add a specific version
pnpm add <package-name>@<version>
```

### Managing Dependencies

```bash
# Remove a package
pnpm remove <package-name>

# Update all dependencies
pnpm update

# Update a specific package
pnpm update <package-name>

# Check for outdated packages
pnpm outdated
```

### Scripts

```bash
# Development
pnpm dev          # Start development server
pnpm build        # Build for production
pnpm preview      # Preview production build

# Code Quality
pnpm lint         # Run ESLint
pnpm format       # Format code with Prettier
pnpm format:check # Check code formatting

# Testing
pnpm test         # Run tests
pnpm test:ui      # Run tests with UI
pnpm test:coverage # Run tests with coverage
```

## 🎨 Adding UI Components from shadcn/ui

This project is configured with shadcn/ui for beautiful, accessible UI components.

### Installation

1. Install the shadcn/ui CLI globally:

```bash
pnpm add -g shadcn@latest
```

2. Initialize shadcn/ui in your project:

```bash
npx shadcn@latest init
```

### Adding Components

```bash
# Add a specific component
npx shadcn@latest add button
npx shadcn@latest add input
npx shadcn@latest add card

# Add multiple components
npx shadcn@latest add button input card dialog
```

### Using Components

```tsx
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'

function MyComponent() {
    return (
        <div>
            <Input placeholder='Enter text...' />
            <Button>Click me</Button>
        </div>
    )
}
```

### Available Components

The project includes these pre-configured components:

- Button (`@/components/ui/button`)
- Custom components in `@/components/`

## 🏪 State Management with Zustand

This project uses Zustand for state management with a clean, type-safe approach.

### Store Structure

Stores are located in `src/stores/` and follow this pattern:

```typescript
// src/stores/counter.ts
import { createStore } from 'zustand/vanilla'

export const createCounterStore = (initState) => {
    return createStore((set) => ({
        ...initState,
        incrementCount: () => set((state) => ({ count: state.count + 1 })),
        decrementCount: () => set((state) => ({ count: state.count - 1 })),
    }))
}
```

### Using Stores

1. **With Provider Pattern** (Recommended for React):

```tsx
// Wrap your app with the store provider
import { CounterStoreProvider } from '@/provider/counterProvider'

function App() {
    return (
        <CounterStoreProvider>
            <YourComponents />
        </CounterStoreProvider>
    )
}

// Use the store in components
import { useCounterStore } from '@/provider/counterProvider'

function Counter() {
    const count = useCounterStore((state) => state.count)
    const incrementCount = useCounterStore((state) => state.incrementCount)

    return (
        <div>
            <p>Count: {count}</p>
            <button onClick={incrementCount}>Increment</button>
        </div>
    )
}
```

2. **Direct Store Usage**:

```tsx
import { createCounterStore } from '@/stores/counter'

const store = createCounterStore()
const count = store.getState().count
store.getState().incrementCount()
```

### Creating New Stores

1. Create a new store file in `src/stores/`:

```typescript
// src/stores/user.ts
import { createStore } from 'zustand/vanilla'

interface UserState {
    user: User | null
    setUser: (user: User) => void
    clearUser: () => void
}

export const createUserStore = (initState) => {
    return createStore<UserState>((set) => ({
        ...initState,
        setUser: (user) => set({ user }),
        clearUser: () => set({ user: null }),
    }))
}
```

2. Create a provider in `src/provider/`:

```typescript
// src/provider/userProvider.tsx
import { createContext, useContext, useRef } from 'react'
import { useStore } from 'zustand'
import { createUserStore } from '@/stores/user'

// Provider implementation...
```

## ☁️ Deploying to Cloudflare Pages

This project is optimized for deployment on Cloudflare Pages.

### Build Configuration

The project uses Vite with the following build settings:

- **Build command**: `pnpm build`
- **Build output directory**: `dist`
- **Node.js version**: 18.x or higher

### Deployment Steps

1. **Connect to Cloudflare Pages**:
    - Go to [Cloudflare Pages](https://pages.cloudflare.com/)
    - Click "Create a project"
    - Connect your Git repository

2. **Configure Build Settings**:

    ```
    Framework preset: Vite
    Build command: pnpm build
    Build output directory: dist
    Root directory: / (or your project root)
    ```

3. **Environment Variables** (if needed):
    - Add any environment variables in the Cloudflare Pages dashboard
    - Use `VITE_` prefix for client-side variables

4. **Deploy**:
    - Cloudflare Pages will automatically build and deploy your project
    - You'll get a `*.pages.dev` URL for your deployed app

### Custom Domain (Optional)

1. In Cloudflare Pages dashboard, go to your project
2. Click "Custom domains"
3. Add your domain and follow the DNS setup instructions

### Build Optimization

The project is configured with:

- Tree shaking for smaller bundles
- Code splitting for better performance
- Optimized assets and images
- TypeScript compilation for type safety

### Troubleshooting

If you encounter build issues:

- Ensure Node.js version is 18+ in Cloudflare Pages settings
- Check that all dependencies are in `dependencies` (not `devDependencies`)
- Verify build command and output directory are correct

## 🛠️ Development

### Project Structure

```
src/
├── components/     # React components
│   └── ui/        # shadcn/ui components
├── stores/        # Zustand stores
├── provider/      # Store providers
├── hooks/         # Custom React hooks
├── lib/           # Utility functions
├── models/        # TypeScript types
├── pages/         # Page components
├── routes/        # Routing configuration
└── services/      # API services
```

### Tech Stack

- **React 19** - UI library
- **TypeScript** - Type safety
- **Vite** - Build tool and dev server
- **Tailwind CSS** - Styling
- **Zustand** - State management
- **shadcn/ui** - UI components
- **React Router** - Client-side routing
- **TanStack Query** - Server state management
- **Vitest** - Testing framework

## 📝 Available Scripts

- `pnpm dev` - Start development server
- `pnpm build` - Build for production
- `pnpm preview` - Preview production build
- `pnpm lint` - Run ESLint
- `pnpm format` - Format code with Prettier
- `pnpm test` - Run tests
- `pnpm test:ui` - Run tests with UI
- `pnpm test:coverage` - Run tests with coverage

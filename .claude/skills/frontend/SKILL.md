---
name: frontend
description: Frontend development guidelines for React/TypeScript applications. Modern patterns including Suspense, lazy loading, useSuspenseQuery, file organization, MUI v7 styling, TanStack Router, performance optimization, and TypeScript best practices. Use when creating components, pages, fetching data, styling, routing, or working with frontend code.
---

# Frontend Development Guidelines

## Purpose

Comprehensive guide for modern React development, emphasizing Suspense-based data fetching, lazy loading, proper file organization, and performance optimization.

## When to Use This Skill

- Creating new components or pages
- Fetching data with TanStack Query
- Setting up routing with TanStack Router
- Styling components with MUI v7
- Performance optimization
- Organizing frontend code
- TypeScript best practices

---

## Quick Start

### New Component Checklist

Creating a component? Follow this checklist:

- [ ] Use `React.FC<Props>` pattern with TypeScript
- [ ] Lazy load if heavy component: `React.lazy(() => import())`
- [ ] Wrap in `<SuspenseLoader>` for loading states
- [ ] Use `useSuspenseQuery` for data fetching
- [ ] Import aliases: `@/`, `~types`, `~components`
- [ ] Styles: Inline if <100 lines, separate file if >100 lines
- [ ] Use `useCallback` for event handlers passed to children
- [ ] Default export at bottom
- [ ] No early returns with loading spinners
- [ ] Use `useMuiSnackbar` for user notifications

---

## Import Aliases Quick Reference

| Alias         | Resolves To      | Example                                                       |
| ------------- | ---------------- | ------------------------------------------------------------- |
| `@/`          | `src/`           | `import { apiClient } from '@/lib/apiClient'`                 |
| `~types`      | `src/types`      | `import type { User } from '~types/user'`                     |
| `~components` | `src/components` | `import { SuspenseLoader } from '~components/SuspenseLoader'` |

Defined in: [vite.config.ts](../../vite.config.ts) lines 180-185

---

## Common Imports Cheatsheet

```typescript
// React & Lazy Loading
import React, { useState, useCallback, useMemo } from "react";
const Heavy = React.lazy(() => import("./Heavy"));

// MUI Components
import { Box, Paper, Typography, Button, Grid } from "@mui/material";
import type { SxProps, Theme } from "@mui/material";

// TanStack Query (Suspense)
import { useSuspenseQuery, useQueryClient } from "@tanstack/react-query";

// TanStack Router
import { createFileRoute } from "@tanstack/react-router";

// Project Components
import { SuspenseLoader } from "~components/SuspenseLoader";

// Hooks
import { useAuth } from "@/hooks/useAuth";
import { useMuiSnackbar } from "@/hooks/useMuiSnackbar";

// Types
import type { Post } from "~types/post";
```

---

## Topic Guides

### 🎨 Component Patterns

**Modern React components use:**

- `React.FC<Props>` for type safety
- `React.lazy()` for code splitting
- `SuspenseLoader` for loading states
- Named const + default export pattern

**Key Concepts:**

- Lazy load heavy components (DataGrid, charts, editors)
- Always wrap lazy components in Suspense
- Use SuspenseLoader component (with fade animation)
- Component structure: Props → Hooks → Handlers → Render → Export

**[📖 Complete Guide: resources/component-patterns.md](resources/component-patterns.md)**

---

### 📊 Data Fetching

**PRIMARY PATTERN: useSuspenseQuery**

- Use with Suspense boundaries
- Cache-first strategy (check grid cache before API)
- Replaces `isLoading` checks
- Type-safe with generics

**API Service Layer:**

- Use `apiClient` axios instance
- Centralized methods per resource in `services/`
- Route format: `/form/route` (NOT `/api/form/route`)

**[📖 Complete Guide: resources/data-fetching.md](resources/data-fetching.md)**

---

### 📁 File Organization

**components/:**

- `components/`: Truly reusable (SuspenseLoader, CustomAppBar)

**[📖 Complete Guide: resources/file-organization.md](resources/file-organization.md)**

---

### 🎨 Styling

**Inline vs Separate:**

- <100 lines: Inline `const styles: Record<string, SxProps<Theme>>`
- > 100 lines: Separate `.styles.ts` file

**Primary Method:**

- Use `sx` prop for MUI components
- Type-safe with `SxProps<Theme>`
- Theme access: `(theme) => theme.palette.primary.main`

**MUI v7 Grid:**

```typescript
<Grid size={{ xs: 12, md: 6 }}>  // ✅ v7 syntax
<Grid xs={12} md={6}>             // ❌ Old syntax
```

**[📖 Complete Guide: resources/styling-guide.md](resources/styling-guide.md)**

---

### 🛣️ Routing

**TanStack Router - Folder-Based:**

- Directory: `routes/my-route/index.tsx`
- Lazy load components
- Use `createFileRoute`
- Breadcrumb data in loader

**Example:**

```typescript
import { createFileRoute } from "@tanstack/react-router";
import { lazy } from "react";

const MyPage = lazy(() => import("@/pages/MyPage"));

export const Route = createFileRoute("/my-route/")({
  component: MyPage,
  loader: () => ({ crumb: "My Route" }),
});
```

**[📖 Complete Guide: resources/routing-guide.md](resources/routing-guide.md)**

---

### ⏳ Loading & Error States

**CRITICAL RULE: No Early Returns**

```typescript
// ❌ NEVER - Causes layout shift
if (isLoading) {
    return <LoadingSpinner />;
}

// ✅ ALWAYS - Consistent layout
<SuspenseLoader>
    <Content />
</SuspenseLoader>
```

**Why:** Prevents Cumulative Layout Shift (CLS), better UX

**Error Handling:**

- Use `useMuiSnackbar` for user feedback
- NEVER `react-toastify`
- TanStack Query `onError` callbacks

**[📖 Complete Guide: resources/loading-and-error-states.md](resources/loading-and-error-states.md)**

---

### ⚡ Performance

**Optimization Patterns:**

- `useMemo`: Expensive computations (filter, sort, map)
- `useCallback`: Event handlers passed to children
- `React.memo`: Expensive components
- Debounced search (300-500ms)
- Memory leak prevention (cleanup in useEffect)

**[📖 Complete Guide: resources/performance.md](resources/performance.md)**

---

### 📘 TypeScript

**Standards:**

- Strict mode, no `any` type
- Explicit return types on functions
- Type imports: `import type { User } from '~types/user'`
- Component prop interfaces with JSDoc

**[📖 Complete Guide: resources/typescript-standards.md](resources/typescript-standards.md)**

---

### 🔧 Common Patterns

**Covered Topics:**

- React Hook Form with Zod validation
- DataGrid wrapper contracts
- Dialog component standards
- `useAuth` hook for current user
- Mutation patterns with cache invalidation

**[📖 Complete Guide: resources/common-patterns.md](resources/common-patterns.md)**

---

### 📚 Complete Examples

**Full working examples:**

- Modern component with all patterns
- API service layer
- Route with lazy loading
- Suspense + useSuspenseQuery
- Form with validation

**[📖 Complete Guide: resources/complete-examples.md](resources/complete-examples.md)**

---

## Navigation Guide

| Need to...             | Read this resource                                                   |
| ---------------------- | -------------------------------------------------------------------- |
| Create a component     | [component-patterns.md](resources/component-patterns.md)             |
| Fetch data             | [data-fetching.md](resources/data-fetching.md)                       |
| Organize files/folders | [file-organization.md](resources/file-organization.md)               |
| Style components       | [styling-guide.md](resources/styling-guide.md)                       |
| Set up routing         | [routing-guide.md](resources/routing-guide.md)                       |
| Handle loading/errors  | [loading-and-error-states.md](resources/loading-and-error-states.md) |
| Optimize performance   | [performance.md](resources/performance.md)                           |
| TypeScript types       | [typescript-standards.md](resources/typescript-standards.md)         |
| Forms/Auth/DataGrid    | [common-patterns.md](resources/common-patterns.md)                   |
| See full examples      | [complete-examples.md](resources/complete-examples.md)               |

---

## Core Principles

1. **Lazy Load Everything Heavy**: Routes, DataGrid, charts, editors
2. **Suspense for Loading**: Use SuspenseLoader, not early returns
3. **useSuspenseQuery**: Primary data fetching pattern for new code
4. **Styles Based on Size**: <100 inline, >100 separate
5. **Import Aliases**: Use @/, ~types, ~components
6. **No Early Returns**: Prevents layout shift
7. **useMuiSnackbar**: For all user notifications

---

## Quick Reference: File Structure

```
src/
  components/
    Message/
      MessageThread.tsx       # Component folder per UI area
    ui/                       # Shared UI primitives

  pages/
    Chat.tsx                  # Page components

  services/
    messages.ts               # API service

  hooks/
    useMessages.ts            # Custom hooks

  interfaces/
    message.ts                # TypeScript types

  utils/                      # Utilities

  routes/
    my-route/
      index.tsx               # Route component
      create/
        index.tsx             # Nested route
```

---

## Modern Component Template (Quick Copy)

```typescript
import React, { useState, useCallback } from 'react';
import { Box, Paper } from '@mui/material';
import { useSuspenseQuery } from '@tanstack/react-query';
import { getItem } from '@/services/items';

interface MyComponentProps {
    id: number;
    onAction?: () => void;
}

export const MyComponent: React.FC<MyComponentProps> = ({ id, onAction }) => {
    const [state, setState] = useState<string>('');

    const { data } = useSuspenseQuery({
        queryKey: ['item', id],
        queryFn: () => getItem(id),
    });

    const handleAction = useCallback(() => {
        setState('updated');
        onAction?.();
    }, [onAction]);

    return (
        <Box sx={{ p: 2 }}>
            <Paper sx={{ p: 3 }}>
                {/* Content */}
            </Paper>
        </Box>
    );
};

export default MyComponent;
```

For complete examples, see [resources/complete-examples.md](resources/complete-examples.md)

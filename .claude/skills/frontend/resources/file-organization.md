# File Organization

Proper file and directory structure for maintainable, scalable frontend code in the the application.

---

## Top-Level Directories

| Directory | Purpose |
|-----------|---------|
| `components/` | UI components, one folder per UI area (`Message/`, `Friend/`, `NavRail/`) |
| `components/ui/` | Shared UI primitives |
| `components/layouts/` | Layout components |
| `pages/` | Page components rendered by routes |
| `routes/` | Route definitions |
| `services/` | API service layer, one file per resource |
| `hooks/` | Custom hooks (queries, mutations, utilities) |
| `interfaces/` | TypeScript types and interfaces |
| `contexts/` | React contexts |
| `constants/` | Constants |
| `utils/` | Utility functions |
| `lib/` | Library setup (API client, helpers) |
| `locales/` | Translations |

---

## components/ Directory

**Purpose**: UI components, grouped by UI area

**Organization:**
- One folder per UI area, named in PascalCase
- Truly reusable primitives go in `components/ui/`

**Structure:**
```
components/
  Message/
    MessageThread.tsx
  Friend/
    FriendList.tsx
  ui/
    button.tsx
```

---

## services/ Directory

**Purpose**: Centralized API calls, one file per resource

**Files:**
- `services/messages.ts`, `services/users.ts`, `services/friends.ts`

**Pattern:**
```typescript
// services/items.ts
import apiClient from '@/lib/apiClient';

export const getItem = async (id: number) => {
    const { data } = await apiClient.get(`/items/${id}`);
    return data;
};

export const createItem = async (payload) => {
    const { data } = await apiClient.post('/items', payload);
    return data;
};
```

---

## hooks/ Directory

**Purpose**: Custom hooks, including TanStack Query wrappers around services

**Naming:**
- `use` prefix (camelCase)
- Descriptive of what they do

**Examples:**
```
hooks/
  useMessages.ts                # Query hook
  useAddFriend.ts               # Mutation hook
  useDebounce.ts                # Utility hook
```

---

## interfaces/ Directory

**Purpose**: TypeScript types and interfaces, one file per domain

**Files:**
```
interfaces/
  message.ts
  user.ts
  friend.ts
```

---

## Import Aliases (Vite Configuration)

### Available Aliases

| Alias | Resolves To | Use For |
|-------|-------------|---------|
| `@/` | `src/` | Absolute imports from src root |
| `~types` | `src/types` | Shared TypeScript types |
| `~components` | `src/components` | Reusable components |

### Usage Examples

```typescript
// ✅ PREFERRED - Use aliases for absolute imports
import { apiClient } from '@/lib/apiClient';
import { SuspenseLoader } from '~components/SuspenseLoader';
import { getPosts } from '@/services/posts';
import type { User } from '~types/user';

// ❌ AVOID - Relative paths from deep nesting
import { apiClient } from '../../../lib/apiClient';
import { SuspenseLoader } from '../../../components/SuspenseLoader';
```

### When to Use Which Alias

**@/ (General)**:
- Lib utilities: `@/lib/apiClient`
- Hooks: `@/hooks/useAuth`
- Config: `@/config/theme`
- Services: `@/services/auth`

**~types (Type Imports)**:
```typescript
import type { Post } from '~types/post';
import type { User, UserRole } from '~types/user';
```

**~components (Reusable Components)**:
```typescript
import { SuspenseLoader } from '~components/SuspenseLoader';
import { CustomAppBar } from '~components/CustomAppBar';
import { ErrorBoundary } from '~components/ErrorBoundary';
```

---

## File Naming Conventions

### Components

**Pattern**: PascalCase with `.tsx` extension

```
MyComponent.tsx
PostDataGrid.tsx
CustomAppBar.tsx
```

**Avoid:**
- camelCase: `myComponent.tsx` ❌
- kebab-case: `my-component.tsx` ❌
- All caps: `MYCOMPONENT.tsx` ❌

### Hooks

**Pattern**: camelCase with `use` prefix, `.ts` extension

```
useMessages.ts
useSuspensePost.ts
useAuth.ts
useDebounce.ts
```

### API Services

**Pattern**: camelCase resource name, `.ts` extension

```
messages.ts
posts.ts
users.ts
```

### Helpers/Utilities

**Pattern**: camelCase with descriptive name, `.ts` extension

```
formatDate.ts
validation.ts
constants.ts
```

### Types

**Pattern**: camelCase domain name

```
interfaces/post.ts
interfaces/user.ts
```

---

## When to Create a Reusable Component

- Used in 3+ places
- Generic, no domain logic
- Pure presentation
- Shared pattern

**Example:** `components/SuspenseLoader/`

---

## Import Organization

### Import Order (Recommended)

```typescript
// 1. React and React-related
import React, { useState, useCallback, useMemo } from 'react';
import { lazy } from 'react';

// 2. Third-party libraries (alphabetical)
import { Box, Paper, Button, Grid } from '@mui/material';
import type { SxProps, Theme } from '@mui/material';
import { useSuspenseQuery, useQueryClient } from '@tanstack/react-query';
import { createFileRoute } from '@tanstack/react-router';

// 3. Alias imports (@ first, then ~)
import { apiClient } from '@/lib/apiClient';
import { useAuth } from '@/hooks/useAuth';
import { useMuiSnackbar } from '@/hooks/useMuiSnackbar';
import { getPosts } from '@/services/posts';
import { SuspenseLoader } from '~components/SuspenseLoader';

// 4. Type imports (grouped)
import type { Post } from '~types/post';
import type { User } from '~types/user';

// 5. Relative imports (same folder)
import { MySubComponent } from './MySubComponent';
```

**Use single quotes** for all imports (project standard)

---

## Directory Structure Visualization

```
src/
├── components/                  # UI components
│   ├── Message/
│   ├── Friend/
│   ├── layouts/
│   └── ui/
│
├── pages/                       # Page components
│   ├── Chat.tsx
│   └── Friends.tsx
│
├── routes/                      # Route definitions
│
├── services/                    # API service layer
│   ├── messages.ts
│   └── users.ts
│
├── hooks/                       # Custom hooks
│   ├── useAuth.ts
│   ├── useMessages.ts
│   └── useDebounce.ts
│
├── interfaces/                  # TypeScript types
│   ├── message.ts
│   └── user.ts
│
├── contexts/                    # React contexts
├── constants/                   # Constants
├── lib/                         # Library setup
├── utils/                       # Utilities
├── locales/                     # Translations
│
└── App.tsx                      # Root component
```

---

## Summary

**Key Principles:**
1. **components/** grouped by UI area, primitives in `components/ui/`
2. **services/** for API calls, **hooks/** for queries and mutations
3. **interfaces/** for TypeScript types
4. Import aliases for clean imports (@/, ~types, ~components)
5. Consistent naming: PascalCase components, camelCase utilities

**See Also:**
- [component-patterns.md](component-patterns.md) - Component structure
- [data-fetching.md](data-fetching.md) - API service patterns
- [complete-examples.md](complete-examples.md) - Full examples

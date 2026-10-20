---
name: frontend
description: Frontend development guidelines for the chat-app React 19 + TypeScript frontend (Vite, TailwindCSS v4, shadcn/ui, React Router DOM v7, TanStack Query, axios, react-hook-form + zod, i18next, sonner). Covers the folder structure (components/, pages/, routes/, services/, hooks/, interfaces/, contexts/, constants/, utils/, locales/), component patterns, data fetching, styling, routing, forms, i18n, loading/error states and TypeScript conventions. Use when creating components, pages, hooks, services, routes, forms or working with any code in frontend/.
---

# Frontend Development Guidelines (chat-app)

## Purpose

Guide for writing code in `frontend/` that matches the existing project structure and conventions. Always follow what the existing code does; this skill documents it.

## When to Use This Skill

- Creating a component, page, or layout
- Adding an API call (service) and the hook that wraps it
- Adding a route
- Building a form with validation
- Styling with Tailwind / shadcn
- Adding translations (en / vi)

---

## Tech Stack

| Concern       | Library                                                    |
| ------------- | ---------------------------------------------------------- |
| Framework     | React 19, TypeScript (strict), Vite + SWC                  |
| Styling       | TailwindCSS v4 (CSS-first, `@theme`), shadcn/ui (new-york) |
| Icons         | `lucide-react`                                             |
| Routing       | `react-router-dom` v7 (`BrowserRouter` + route array)      |
| Server state  | `@tanstack/react-query` v5 (`useQuery`, `useMutation`)     |
| HTTP          | `axios` (`instance` in `@/utils/axios`)                    |
| Forms         | `react-hook-form` + `zod` + `@hookform/resolvers/zod`      |
| i18n          | `i18next` + `react-i18next` (`en`, `vi`)                   |
| Notifications | `sonner` (`toast`)                                         |
| Client state  | `zustand` + `immer` (available, use when needed)           |
| Tests         | Vitest + Testing Library                                   |

**Package manager: pnpm only.**

---

## Folder Structure

```
frontend/src/
├── App.tsx                 # Providers + BrowserRouter + Routes + <Toaster />
├── main.tsx                # Entry, imports assets/css/index.css
├── assets/
│   ├── css/index.css       # Tailwind entry, theme tokens, custom utilities
│   └── images/             # SVG/PNG imported in code
├── components/
│   ├── ui/                 # shadcn primitives (button, input, dialog, ...) – lowercase files
│   ├── layouts/            # ChatLayout.tsx
│   ├── Conversations/      # index.tsx + ConversationHeader.tsx, UserMessage.tsx
│   ├── Friend/             # index.tsx + FriendHeader, FriendRequests, SearchFriends, ...
│   ├── Message/            # index.tsx + MessageHeader, MessageThread, MessageBubble, MessageComposer
│   ├── LoginForm/  RegisterForm/  NavRail/  LanguageSwitcher/  ThemeToggle/  Wordmark/
│   └── UserAvatar.tsx      # Shared single-file component
├── constants/
│   ├── index.ts            # baseApi, env values, token keys, defaultAvatar
│   ├── query-key.ts        # QUERY_KEY = { USER, MESSAGE, FRIENDS }
│   └── images.ts
├── contexts/               # QueryContext.tsx, ThemeContext.tsx (+ useTheme)
├── hooks/                  # useAuth, useFriends, useMessages, useLogin, useAddFriend, useDebounce, ...
├── interfaces/             # user.ts, friend.ts, message.ts, chat.ts, routes.ts, token.ts
├── lib/utils.ts            # cn()
├── locales/                # en.json, vi.json
├── pages/                  # Home, Register, Chat, Friends – thin, compose components
├── routes/
│   ├── routes.tsx          # routers: Routes[] = [{ href, id, name, element }]
│   └── AuthGuard.tsx
├── services/               # auth, users, friends, messages, token, cloudinary
└── utils/                  # axios, i18n, validations, format, datetime, message, cropImage
```

Data flow: **service** (axios call) → **hook** (`useQuery`/`useMutation`) → **page** (reads hooks) → **components** (receive props).

**[📖 resources/file-organization.md](resources/file-organization.md)**

---

## Quick Start Checklist

New component:

- [ ] Put it in `components/<Area>/` (PascalCase folder, `index.tsx` is the main component)
- [ ] `function Name({ ... }: Props) { ... }` + `export default Name` at the bottom
- [ ] Props typed with a local `interface Props`
- [ ] Styling with Tailwind classes + `cn()`; use shadcn primitives from `@/components/ui/*`
- [ ] All user-facing text via `const { t } = useTranslation()` and keys in **both** `en.json` and `vi.json`
- [ ] Icons from `lucide-react`
- [ ] Import with `@/` for cross-directory, `./` for siblings in the same folder

New API feature:

- [ ] Function in `services/<resource>.ts` using `instance` from `@/utils/axios`, returns `response.data`
- [ ] Types in `interfaces/<resource>.ts`
- [ ] Hook in `hooks/use<Thing>.ts` with `QUERY_KEY` from `@/constants/query-key`
- [ ] Mutations invalidate the related query keys in `onSuccess`; show `toast` from `sonner`

---

## Import Alias

Only one alias: `@/` → `src/` (vite.config.ts + tsconfig.app.json). There is no `~types` / `~components`.

```typescript
import { useState } from "react";
import { useTranslation } from "react-i18next";
import { useNavigate, useSearchParams, Link } from "react-router-dom";
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";
import { toast } from "sonner";
import { Search } from "lucide-react";

import { cn } from "@/lib/utils";
import { Button } from "@/components/ui/button";
import { QUERY_KEY } from "@/constants/query-key";
import { instance } from "@/utils/axios";
import useAuth from "@/hooks/useAuth";
import type IUser from "@/interfaces/user";
import type { IMessage } from "@/interfaces/message";
```

---

## Topic Guides

| Need to...                      | Read                                                                 |
| ------------------------------- | -------------------------------------------------------------------- |
| Where to put files / naming     | [file-organization.md](resources/file-organization.md)               |
| Write a component               | [component-patterns.md](resources/component-patterns.md)             |
| Call API / query / mutation     | [data-fetching.md](resources/data-fetching.md)                       |
| Tailwind, shadcn, theme, dark   | [styling-guide.md](resources/styling-guide.md)                       |
| Add a route / guard / layout    | [routing-guide.md](resources/routing-guide.md)                       |
| Loading, empty, error, toast    | [loading-and-error-states.md](resources/loading-and-error-states.md) |
| useMemo, debounce, lazy         | [performance.md](resources/performance.md)                           |
| Types & interfaces              | [typescript-standards.md](resources/typescript-standards.md)         |
| Forms, i18n, auth, theme        | [common-patterns.md](resources/common-patterns.md)                   |
| End-to-end feature example      | [complete-examples.md](resources/complete-examples.md)               |

---

## Core Principles

1. **Follow the layers**: services → hooks → pages → components. Components don't call axios directly.
2. **Pages are thin**: they read hooks / URL params and pass data down as props.
3. **One folder per UI area** in `components/`; shadcn primitives stay in `components/ui/`.
4. **Tailwind + `cn()`** for styling, semantic tokens (`bg-card`, `text-muted-foreground`), never inline `style` or `var(--x)` in JSX.
5. **Every string translated** in `en.json` and `vi.json`.
6. **Query keys from `QUERY_KEY`**, invalidate after mutations.
7. **`toast` from `sonner`** for user feedback.
8. **No `any`** in new code; type API responses with `interfaces/`.

---

## Component Template

```tsx
import { useTranslation } from "react-i18next";
import { cn } from "@/lib/utils";
import { Button } from "@/components/ui/button";
import type IUser from "@/interfaces/user";

interface Props {
  user: IUser;
  isActive?: boolean;
  onSelect: (id: string) => void;
}

function FriendItem({ user, isActive, onSelect }: Props) {
  const { t } = useTranslation();

  return (
    <div
      className={cn(
        "flex items-center gap-3 rounded-2xl px-3 py-2",
        isActive && "bg-secondary",
      )}
    >
      <span className="flex-1 truncate font-semibold">{user.name}</span>
      <Button size="sm" onClick={() => onSelect(user._id || "")}>
        {t("friends.message")}
      </Button>
    </div>
  );
}

export default FriendItem;
```

## Commands

`pnpm dev` · `pnpm build` · `pnpm lint` · `pnpm format` · `pnpm test` (run from `frontend/`). Don't run `build`/`dev` unless asked.

# File Organization

Where each kind of file goes in `frontend/src/` and how to name it.

---

## Top-Level Directories

| Directory             | Purpose                                                                | Example                                   |
| --------------------- | ---------------------------------------------------------------------- | ----------------------------------------- |
| `assets/css/`         | Tailwind entry, theme tokens, custom utilities/animations              | `index.css`                               |
| `assets/images/`      | Images imported from code                                              | `chatapp-logo.svg`                        |
| `components/`         | UI components, one PascalCase folder per UI area                       | `Message/`, `Friend/`, `NavRail/`         |
| `components/ui/`      | shadcn/ui primitives (generated, lowercase file names)                 | `button.tsx`, `scroll-area.tsx`           |
| `components/layouts/` | Layout wrappers used by routes                                         | `ChatLayout.tsx`                          |
| `constants/`          | Env-derived config, keys, query keys                                   | `index.ts`, `query-key.ts`, `images.ts`   |
| `contexts/`           | React context providers (+ their consumer hook)                        | `QueryContext.tsx`, `ThemeContext.tsx`    |
| `hooks/`              | Custom hooks: TanStack Query wrappers and generic hooks                | `useFriends.ts`, `useDebounce.ts`         |
| `interfaces/`         | TypeScript types/interfaces, one file per domain                       | `user.ts`, `message.ts`, `friend.ts`      |
| `lib/`                | `cn()` helper (shadcn alias `@/lib`)                                   | `utils.ts`                                |
| `locales/`            | i18n JSON resources                                                    | `en.json`, `vi.json`                      |
| `pages/`              | Route-level components (thin)                                          | `Chat.tsx`, `Friends.tsx`                 |
| `routes/`             | Route config array and guards                                          | `routes.tsx`, `AuthGuard.tsx`             |
| `services/`           | API calls, one file per backend resource                               | `friends.ts`, `messages.ts`, `auth.ts`    |
| `utils/`              | Pure helpers + infrastructure setup (axios, i18n, zod schemas)         | `axios.ts`, `validations.ts`, `format.ts` |
| `public/`             | (outside src) static files served as-is, e.g. `/emoji/like.png`        |                                           |

---

## components/

```
components/
  ui/                       # shadcn – don't hand-edit unless needed; add with `pnpm dlx shadcn@latest add <name>`
  layouts/
    ChatLayout.tsx
  Message/
    index.tsx               # main component of the area (import "@/components/Message")
    MessageHeader.tsx
    MessageThread.tsx
    MessageBubble.tsx
    MessageComposer.tsx
  Friend/
    index.tsx
    FriendHeader.tsx
    FriendRequests.tsx
    FriendsBlocked.tsx
    CommonFriends.tsx
    SearchFriends.tsx
  LoginForm/index.tsx
  RegisterForm/index.tsx + Avatar.tsx
  UserAvatar.tsx            # shared single-file component used across areas
```

Rules:

- A new UI area → new folder `components/<Area>/index.tsx`. Sub-parts sit next to it, named `<Area><Part>.tsx` (e.g. `MessageBubble.tsx`).
- Inside the folder import siblings with `./` (`import MessageThread from "./MessageThread"`).
- A component used by several areas and small enough for one file → `components/<Name>.tsx` (like `UserAvatar.tsx`) or `components/<Name>/index.tsx`.
- Primitives (button, input, dialog...) come from shadcn in `components/ui/`, don't recreate them.

---

## pages/

Pages are thin: read hooks/URL params, pass props into components.

```tsx
// pages/Friends.tsx
import Friend from "@/components/Friend";

function Friends() {
  return <Friend />;
}

export default Friends;
```

`pages/Chat.tsx` reads `receiverId` from `useSearchParams`, calls `useAuth`, `useFriends`, `useMessages` and passes data to `Conversations` and `Message`.

---

## services/ → hooks/ → components

```
services/friends.ts       getFriendsByUserId, searchFriends, addFriend, acceptFriend, blockFriend ...
hooks/useFriends.ts       useQuery wrappers  (QUERY_KEY.FRIENDS)
hooks/useAddFriend.ts     useMutation + invalidateQueries
hooks/useBlockFriend.ts
components/Friend/*       use the hooks / receive data as props
```

- One service file per backend resource (`/auth`, `/users`, `/friends`, `/message`, `/token`), plus `cloudinary.ts` for uploads.
- One hook file per query/mutation concern, default-exported, named `use<Thing>`.

See [data-fetching.md](data-fetching.md).

---

## interfaces/

One file per domain. Models from the backend are `I`-prefixed interfaces; unions/aliases are `type`.

```
interfaces/user.ts      default export IUser
interfaces/friend.ts    IFriend, FriendListItem, type FriendTab
interfaces/message.ts   IMessage, MessageFile, type MessageEmoji, type MessageFileType
interfaces/chat.ts      type PresenceStatus, type ConversationFilter
interfaces/routes.ts    type Routes
interfaces/token.ts
```

---

## constants/

- `constants/index.ts` – `baseApi`, `VITE_*` env values, `ACCESS_TOKEN_KEY`, `REFRESH_TOKEN_KEY`, `defaultAvatar`.
- `constants/query-key.ts` – add a new key here when adding a new resource:

```typescript
export const QUERY_KEY = {
  USER: "user",
  MESSAGE: "message",
  FRIENDS: "friends",
};
```

---

## utils/

| File             | Contents                                                         |
| ---------------- | ---------------------------------------------------------------- |
| `axios.ts`       | `instance` (API, token + refresh interceptors), `cloudinaryApi`  |
| `i18n.ts`        | i18next init, `SUPPORTED_LANGUAGES`                              |
| `validations.ts` | zod schema factories + inferred form value types                 |
| `format.ts`      | formatting helpers (`formatISODateToTimestamp`, ...)             |
| `datetime.ts`    | date/time helpers                                                |
| `message.ts`     | message helpers (`getThread`, ...)                               |
| `cropImage.ts`   | canvas crop for avatar upload                                    |

Pure, framework-free helpers go in `utils/`. Anything with React hooks goes in `hooks/`.

---

## Naming

| Kind                   | Convention                           | Example                         |
| ---------------------- | ------------------------------------ | ------------------------------- |
| Component folder/file  | PascalCase                           | `Message/MessageBubble.tsx`     |
| shadcn primitive       | lowercase kebab                      | `ui/dropdown-menu.tsx`          |
| Hook                   | `use` + PascalCase, camelCase file   | `useAcceptDelineFriend.ts`      |
| Service / util file    | camelCase / lowercase                | `services/messages.ts`          |
| Constants file         | kebab-case                           | `constants/query-key.ts`        |
| Model interface        | `I` + PascalCase                     | `IMessage`, `IFriend`, `IUser`  |
| Constant values        | UPPER_SNAKE_CASE                     | `QUERY_KEY`, `ACCESS_TOKEN_KEY` |
| Boolean                | `is` / `has` / `should` prefix       | `isLoadingUsers`, `isActive`    |

## Don'ts

- Don't create new top-level directories (`features/`, `types/`, `api/`, `store/`...) without being asked.
- Don't put axios calls in components or hooks other than via `services/`.
- Don't use relative `../../` across directories; use `@/`.

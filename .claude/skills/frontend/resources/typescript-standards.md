# TypeScript Standards

`strict: true`, `noUnusedLocals`, `noUnusedParameters` (tsconfig.app.json). `pnpm build` runs `tsc -b` first, so type errors break the build.

---

## Where Types Live

- Domain/API types → `src/interfaces/<domain>.ts`
- Component props → local `interface Props` in the component file
- Form value types → inferred from zod in `utils/validations.ts`

```typescript
// interfaces/message.ts
export type MessageEmoji = "like" | "dislike" | "heart" | "smile" | "sad" | "angry";
export type MessageFileType = "file" | "image" | "video";

export interface MessageFile {
  _id: string;
  type: MessageFileType;
  url: string;
}

export interface IMessage {
  _id: string;
  senderId: string;
  receiverId: string;
  text: string;
  files: MessageFile[];
  isDeleted: boolean;
  isRead: boolean;
  emoji?: MessageEmoji;
  createdAt: string; // ISO string from backend
  updatedAt: string;
}
```

Conventions:

- Backend models: `interface I<Name>` (`IUser`, `IFriend`, `IMessage`). `IUser` is a default export; the others are named.
- Unions / aliases / UI enums: `type` (`FriendTab`, `PresenceStatus`, `ConversationFilter`, `Routes`).
- Mongo ids are `_id: string`; dates are ISO `string`.
- JSDoc on fields that need context (`/** Only sent when registering */`).

---

## Imports

Prefer `import type` for type-only imports:

```typescript
import type IUser from "@/interfaces/user";
import type { IMessage } from "@/interfaces/message";
```

---

## Generics on Queries

```typescript
useQuery<IFriend[]>({ queryKey: [QUERY_KEY.FRIENDS], queryFn: getFriendsByUserId });
instance.get<IUser[]>("/users");
```

---

## Zod-Inferred Types

```typescript
export const loginSchema = () => z.object({ email: z.email(...), password: z.string().min(6, ...) });
export type LoginFormValues = z.infer<ReturnType<typeof loginSchema>>;

useForm<LoginFormValues>({ resolver: zodResolver(schema) });
```

---

## Variant Maps

```typescript
const SIZES = { sm: "size-8", md: "size-10" } as const;
size?: keyof typeof SIZES;

const PRESENCE_RING: Record<PresenceStatus, string> = { ... };
```

---

## Rules

- No `any` in new code (ESLint warns). For axios errors use `AxiosError<{ code: string }>`.
- Avoid `as` assertions; narrow instead. Acceptable for `localStorage` values (`as Theme | null`) and `error.config as RetriableConfig`.
- Prefix intentionally unused params with `_`.
- Use optional chaining + defaults for possibly-missing data: `user?._id || ""`, `friend?.friends?.some(...)`.
- Env vars: `import.meta.env.VITE_*`, read only in `constants/index.ts`.

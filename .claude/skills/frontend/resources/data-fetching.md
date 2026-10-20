# Data Fetching

Layers: `utils/axios.ts` (client) → `services/*.ts` (API calls) → `hooks/use*.ts` (TanStack Query) → pages/components.

---

## Axios Client (`@/utils/axios`)

```typescript
export const instance = axios.create({
  baseURL: baseApi, // PROD: VITE_API_URL, dev: http://localhost:3000/api
  headers: { "Content-Type": "application/json" },
  withCredentials: true,
});
export const cloudinaryApi = axios.create({ baseURL: baseCloudinaryV1 });
```

- Request interceptor attaches `Authorization: Bearer <accessToken>` from `localStorage[ACCESS_TOKEN_KEY]` when no header is set.
- Response interceptor: on 401, refreshes via `services/token.ts` using the `refreshToken` cookie, stores the new tokens, retries once.
- Paths are relative to `/api` → write `/friends`, `/message/sender`, not `/api/friends`.

---

## Services (`services/<resource>.ts`)

Named async functions, return `response.data`.

```typescript
import { instance } from "@/utils/axios";

export const searchFriends = async (name?: string) => {
  const response = await instance.get(`/friends/search`, {
    params: { name },
  });
  return response.data;
};

export const addFriend = async (friendId: string) => {
  const response = await instance.post(`/friends/add`, { friendId });
  return response.data;
};
```

Notes:

- Existing services also pass `headers: { Authorization: ... }` manually; the interceptor already does that, so new code can omit it.
- For `post` without body, the 2nd arg is the body — pass config as the 3rd arg: `instance.post(url, undefined, config)`.
- Type the return when possible: `instance.get<IMessage[]>(...)`.

---

## Query Keys

Always from `@/constants/query-key`:

```typescript
queryKey: [QUERY_KEY.FRIENDS]
queryKey: [QUERY_KEY.FRIENDS, "search", debounce || ""]
queryKey: [QUERY_KEY.MESSAGE, receiverId]
queryKey: [QUERY_KEY.USER, accessToken]
```

The first element is the resource, so `invalidateQueries({ queryKey: [QUERY_KEY.FRIENDS] })` refreshes all friend queries. Add new resources to `QUERY_KEY`.

---

## Query Hooks

Default-exported function. Return the raw query (`useAuth`) or a shaped object with renamed loading flags (`useFriends`, `useMessages`).

```typescript
import { QUERY_KEY } from "@/constants/query-key";
import { IMessage } from "@/interfaces/message";
import { getMessageByReceiverId } from "@/services/messages";
import { useQuery } from "@tanstack/react-query";

function useMessages(receiverId: string) {
  const { data, isLoading: loadingReceiverMessages } = useQuery<IMessage[]>({
    queryKey: [QUERY_KEY.MESSAGE, receiverId],
    queryFn: async () => getMessageByReceiverId(receiverId),
    initialData: [],
    enabled: !!receiverId,
  });

  return { receiverMessages: data, loadingReceiverMessages };
}

export default useMessages;
```

- `enabled` for params that may be empty (`!!receiverId`, `!!accessToken`).
- Provide a default (`initialData: []` or `users || []`) so components can map without null checks.
- Don't mutate query data in place (`data.sort(...)` mutates the cache) — copy first: `[...data].sort(...)`.
- Debounced search: `useDebounce(query)` in the component, pass the debounced value into the hook key.

---

## Mutation Hooks

```typescript
import { QUERY_KEY } from "@/constants/query-key";
import { addFriend } from "@/services/friends";
import { useMutation, useQueryClient } from "@tanstack/react-query";

function useAddFriend() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationKey: ["send-request"],
    mutationFn: async ({ friendId }: { friendId: string }) => addFriend(friendId),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: [QUERY_KEY.USER] });
      queryClient.invalidateQueries({ queryKey: [QUERY_KEY.FRIENDS] });
    },
  });
}

export default useAddFriend;
```

Usage:

```tsx
const { mutate: sendRequest, isPending } = useAddFriend();
<Button disabled={isPending} onClick={() => sendRequest({ friendId: user._id || "" })} />
```

- Side effects (toast, navigate, store tokens) live in the hook's `onSuccess` / `onError` (see `useLogin.ts`).
- Don't call hooks conditionally. Instead of `type === "block" ? useMutation(...) : useMutation(...)`, use one `useMutation` and branch inside `mutationFn`.

---

## Error Messages from the API

Backend errors return a `code`; translate via `errors.<code>`:

```typescript
onError: (error: AxiosError<{ code: string }>) => {
  const code = error.response?.data?.code;
  toast.error(t(`errors.${code}`));
},
```

---

## Cloudinary Upload

Use `cloudinaryApi` from `@/utils/axios` through `services/cloudinary.ts`; config values come from `@/constants` (`cloudinaryUploadPreset`, ...).

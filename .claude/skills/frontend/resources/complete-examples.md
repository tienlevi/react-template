# Complete Example: "Pinned messages" feature

End-to-end walkthrough touching every layer in the order you'd write it. Adapt names to the real feature.

---

## 1. Type — `interfaces/message.ts`

```typescript
export interface IPinnedMessage {
  _id: string;
  messageId: string;
  conversationId: string;
  pinnedBy: string;
  createdAt: string;
}
```

## 2. Query key — `constants/query-key.ts`

```typescript
export const QUERY_KEY = {
  USER: "user",
  MESSAGE: "message",
  FRIENDS: "friends",
  PINNED: "pinned",
};
```

## 3. Service — `services/messages.ts`

```typescript
import type { IPinnedMessage } from "@/interfaces/message";
import { instance } from "@/utils/axios";

export const getPinnedMessages = async (receiverId: string) => {
  const response = await instance.get<IPinnedMessage[]>(`/message/pinned/${receiverId}`);
  return response.data;
};

export const pinMessage = async (messageId: string) => {
  const response = await instance.post(`/message/pin/${messageId}`);
  return response.data;
};
```

## 4. Hooks — `hooks/usePinnedMessages.ts`, `hooks/usePinMessage.ts`

```typescript
import { QUERY_KEY } from "@/constants/query-key";
import type { IPinnedMessage } from "@/interfaces/message";
import { getPinnedMessages } from "@/services/messages";
import { useQuery } from "@tanstack/react-query";

function usePinnedMessages(receiverId: string) {
  const { data, isLoading } = useQuery<IPinnedMessage[]>({
    queryKey: [QUERY_KEY.PINNED, receiverId],
    queryFn: async () => getPinnedMessages(receiverId),
    enabled: !!receiverId,
  });

  return { pinned: data || [], isLoadingPinned: isLoading };
}

export default usePinnedMessages;
```

```typescript
import { QUERY_KEY } from "@/constants/query-key";
import { pinMessage } from "@/services/messages";
import { useMutation, useQueryClient } from "@tanstack/react-query";
import type { AxiosError } from "axios";
import { useTranslation } from "react-i18next";
import { toast } from "sonner";

function usePinMessage() {
  const queryClient = useQueryClient();
  const { t } = useTranslation();

  return useMutation({
    mutationKey: ["pin-message"],
    mutationFn: async (messageId: string) => pinMessage(messageId),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: [QUERY_KEY.PINNED] });
      toast.success(t("chat.pinned.success"));
    },
    onError: (error: AxiosError<{ code: string }>) => {
      toast.error(t(`errors.${error.response?.data?.code}`));
    },
  });
}

export default usePinMessage;
```

## 5. Translations — `locales/en.json` and `locales/vi.json`

```json
"chat": {
  "pinned": {
    "title": "Pinned messages",
    "empty": "No pinned messages yet",
    "pin": "Pin",
    "success": "Message pinned"
  }
}
```

```json
"chat": {
  "pinned": {
    "title": "Tin nhắn đã ghim",
    "empty": "Chưa có tin nhắn nào được ghim",
    "pin": "Ghim",
    "success": "Đã ghim tin nhắn"
  }
}
```

## 6. Component — `components/Message/MessagePinned.tsx`

```tsx
import { Pin } from "lucide-react";
import { useTranslation } from "react-i18next";
import { cn } from "@/lib/utils";
import { Skeleton } from "@/components/ui/skeleton";
import type { IPinnedMessage } from "@/interfaces/message";

interface Props {
  pinned: IPinnedMessage[];
  isLoading: boolean;
  className?: string;
}

function MessagePinned({ pinned, isLoading, className }: Props) {
  const { t } = useTranslation();

  return (
    <div className={cn("border-b border-border px-5 py-3", className)}>
      <div className="flex items-center gap-2 text-xs font-semibold text-muted-foreground">
        <Pin className="size-4" />
        {t("chat.pinned.title")}
      </div>

      {isLoading ? (
        <Skeleton className="mt-2 h-8 rounded-xl" />
      ) : pinned.length === 0 ? (
        <p className="mt-2 text-sm text-muted-foreground">{t("chat.pinned.empty")}</p>
      ) : (
        <ul className="mt-2 flex flex-col gap-1">
          {pinned.map((item) => (
            <li key={item._id} className="truncate rounded-xl bg-secondary px-3 py-1.5 text-sm">
              {item.messageId}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}

export default MessagePinned;
```

## 7. Wire into the area — `components/Message/index.tsx`

```tsx
import MessagePinned from "./MessagePinned";
import usePinnedMessages from "@/hooks/usePinnedMessages";

const { pinned, isLoadingPinned } = usePinnedMessages(activeId || "");

<MessageHeader ... />
<MessagePinned pinned={pinned} isLoading={isLoadingPinned} />
<MessageThread ... />
```

And the action in a leaf (`MessageBubble.tsx`):

```tsx
const { mutate: pin, isPending } = usePinMessage();

<Button variant="ghost" size="icon" disabled={isPending} onClick={() => pin(message._id)}>
  <Pin />
  <span className="sr-only">{t("chat.pinned.pin")}</span>
</Button>
```

---

## Example: New Page + Route

```tsx
// pages/Settings.tsx
import SettingsPanel from "@/components/SettingsPanel";

function Settings() {
  return <SettingsPanel />;
}

export default Settings;
```

```tsx
// routes/routes.tsx
{
  href: "/settings",
  id: "settings",
  name: "Settings",
  element: (
    <AuthGuard>
      <ChatLayout>
        <Settings />
      </ChatLayout>
    </AuthGuard>
  ),
},
```

Files touched for a typical feature: `interfaces/` → `constants/query-key.ts` → `services/` → `hooks/` → `locales/en.json` + `vi.json` → `components/<Area>/` → (`pages/` + `routes/routes.tsx` if new screen).

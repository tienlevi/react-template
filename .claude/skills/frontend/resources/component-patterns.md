# Component Patterns

How components are written in this project.

---

## Declaration Style

Function declaration + default export at the bottom. No `React.FC`, no class components.

```tsx
interface Props {
  query: string;
  setQuery: (query: string) => void;
  tab: FriendTab;
  setTab: (tab: FriendTab) => void;
}

function FriendHeader({ query, setQuery, tab, setTab }: Props) {
  const { t } = useTranslation();
  return <header className="px-5 pt-5 pb-3">{/* ... */}</header>;
}

export default FriendHeader;
```

- Props: local `interface Props` (or a named `<Component>Props` when exported/reused, e.g. `UserAvatarProps`).
- Children: `{ children }: { children: ReactNode }`.
- Optional `className?: string` merged with `cn()` when the parent needs to tweak layout.
- Named exports are fine for helpers living next to the component (`export const getInitials = ...` in `UserAvatar.tsx`).

---

## Order Inside a Component

1. `useTranslation`, router hooks (`useNavigate`, `useSearchParams`)
2. Query/mutation hooks (`useAuth`, `useFriends`, `useAddFriend`)
3. Local state (`useState`)
4. Derived values (`useMemo` / plain consts)
5. Handlers (`function onSubmit(...)` or inline arrows)
6. JSX

```tsx
function Chat() {
  const [searchParams] = useSearchParams();
  const receiverId = searchParams.get("receiverId") || "";
  const { data: user } = useAuth();
  const { users } = useFriends();
  const { receiverMessages, senderMessages } = useMessages(receiverId);
  const [mobilePane, setMobilePane] = useState<"list" | "thread">("list");
  const currentUserId = user?._id || "";

  return (/* ... */);
}
```

---

## Area Component (container) + Parts

The `index.tsx` of an area owns state and data; sub-components receive props.

```tsx
// components/Friend/index.tsx
function Friend() {
  const [query, setQuery] = useState("");
  const debounce = useDebounce(query);
  const [tab, setTab] = useState<FriendTab>("all");
  const { friend, users, isLoadingUsers } = useFriends(debounce);

  const blockedFriends = users.filter((user) =>
    friend?.blockFriends?.some((f) => f === user._id),
  );

  return (
    <section className="flex w-full flex-1 flex-col rounded-3xl border border-border bg-card">
      <FriendHeader query={query} setQuery={setQuery} tab={tab} setTab={setTab} />
      <ScrollArea className="min-h-0 flex-1">
        {tab === "all" && <SearchFriends friend={friend} users={users} isLoading={isLoadingUsers} />}
        {tab === "blocked" && <FriendsBlocked blocked={blockedFriends} isLoading={isLoadingUsers} />}
      </ScrollArea>
    </section>
  );
}
```

Leaf components (e.g. `MessageBubble`, `UserMessage`) stay presentational. They may call a mutation hook for their own action button (accept / block friend).

---

## Conditional Rendering

- Tabs / modes: `{tab === "requests" && <FriendRequests ... />}`
- Empty vs content: ternary inside a stable wrapper (see `components/Message/index.tsx` — the outer card is always rendered, the inside switches between thread and empty state).
- Responsive panes: toggle `hidden md:flex` with `cn()` instead of unmounting.

```tsx
<div className={cn("flex w-full flex-col rounded-3xl border border-border bg-card", mobilePane === "list" && "hidden md:flex")}>
```

---

## shadcn Primitives

Compose from `@/components/ui/*`:

```tsx
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { ScrollArea } from "@/components/ui/scroll-area";
import { Tooltip, TooltipContent, TooltipTrigger } from "@/components/ui/tooltip";

<Tooltip>
  <TooltipTrigger asChild>
    <Button size="icon" className="rounded-2xl">
      <UserPlus />
      <span className="sr-only">{t("friends.add")}</span>
    </Button>
  </TooltipTrigger>
  <TooltipContent>{t("friends.add")}</TooltipContent>
</Tooltip>
```

`TooltipProvider` is already mounted in `ChatLayout`. Available primitives: avatar, badge, button, card, dialog, dropdown-menu, field, form, input, label, scroll-area, separator, skeleton, slider, sonner, switch, textarea, tooltip. Add more with `pnpm dlx shadcn@latest add <name>`.

---

## Variants via Lookup Maps

For size/status variants, use `Record`/`as const` maps instead of long ternaries (from `UserAvatar.tsx`):

```tsx
const SIZES = { sm: "size-8", md: "size-10", lg: "size-12", xl: "size-16" } as const;
const PRESENCE_RING: Record<PresenceStatus, string> = {
  online: "bg-success",
  away: "bg-amber-500",
  offline: "bg-muted-foreground/50",
};

interface UserAvatarProps {
  name: string;
  size?: keyof typeof SIZES;
  status?: PresenceStatus;
}
```

---

## Accessibility

- Icon-only buttons: `<span className="sr-only">` label.
- Toggle buttons: `aria-pressed`.
- Inputs without visible label: `aria-label={t(...)}`.
- Decorative elements: `aria-hidden`.
- Focus ring: `focus-visible:ring-[3px] focus-visible:ring-ring/50 focus-visible:outline-none`.

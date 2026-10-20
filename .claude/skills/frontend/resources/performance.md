# Performance

Keep it simple; optimize where there's a real cost.

---

## `useMemo` for Derived Data

Used for lookups and transforms over lists (`components/Message/index.tsx`):

```tsx
const activeUser = useMemo(
  () => (activeId ? users.find((user) => user._id === activeId) : undefined),
  [users, activeId],
);

const thread = useMemo(
  () => getThread(messages, currentUserId, activeId),
  [messages, currentUserId, activeId],
);
```

Cheap filters (a few dozen items) can stay plain consts.

Also used to rebuild zod schemas when the language changes:

```tsx
const schema = useMemo(() => loginSchema(), [i18n.language]);
```

---

## Don't Mutate Query Data

`Array.prototype.sort` mutates. Copy before sorting query results:

```typescript
const sorted = [...data].sort(
  (a, b) => formatISODateToTimestamp(a.createdAt) - formatISODateToTimestamp(b.createdAt),
);
```

---

## Debounce Search

`hooks/useDebounce.ts` (default 500ms):

```tsx
const [query, setQuery] = useState("");
const debounce = useDebounce(query);
const { users } = useFriends(debounce); // query key includes debounce
```

---

## `useCallback` / `React.memo`

Only when passing handlers to memoized children or long lists (e.g. message bubbles). Inline arrows are the norm in this codebase for simple handlers.

---

## Effects Cleanup

Always clean timers, listeners, sockets:

```typescript
useEffect(() => {
  const timer = setTimeout(() => setDebouncedValue(value), delay);
  return () => clearTimeout(timer);
}, [value, delay]);
```

---

## Scrolling & Lists

- Use `ScrollArea` with `min-h-0 flex-1` inside flex columns so only the list scrolls.
- Stable `key` = `_id` from the backend, never the index for dynamic lists.

---

## Code Splitting

Pages are eagerly imported today. Use `React.lazy` + `Suspense` for heavy, rarely used pieces (e.g. image cropper `react-easy-crop`, big dialogs). See [routing-guide.md](routing-guide.md).

## Query Tuning

`QueryClient` uses defaults (`contexts/QueryContext.tsx`). Set `staleTime` per query when data rarely changes (e.g. current user) instead of changing global defaults.

# Loading, Empty & Error States

The project uses `useQuery` flags (`isLoading`, `isPending`) — not Suspense queries. Feedback goes through `sonner` toasts.

---

## Loading

### Full-page (auth check)

`AuthGuard` is the only place that returns a full-screen loader:

```tsx
if (isLoading) {
  return (
    <div className="flex min-h-svh items-center justify-center bg-background">
      <Loader2 className="size-8 animate-spin text-muted-foreground" />
    </div>
  );
}
```

### Inside a panel

Keep the panel/card mounted and swap only its content, so the layout doesn't jump. Pass `isLoading` down and render `Skeleton` rows:

```tsx
import { Skeleton } from "@/components/ui/skeleton";

function SearchFriends({ users, isLoading }: Props) {
  if (isLoading) {
    return (
      <div className="flex flex-col gap-2 p-5">
        {Array.from({ length: 5 }).map((_, i) => (
          <Skeleton key={i} className="h-14 rounded-2xl" />
        ))}
      </div>
    );
  }
  return (/* list */);
}
```

Early return is fine in a **leaf** list component; avoid it in the container that renders the card shell/header.

### Mutations

Disable the trigger with `isPending`:

```tsx
const { mutate, isPending } = useLogin();
<Button disabled={isPending} type="submit">{t("auth.login.submit")}</Button>
```

---

## Empty States

Centered text inside the existing card, translated (see `components/Message/index.tsx`):

```tsx
<div className="flex flex-1 items-center justify-center px-6 text-center">
  <div>
    <h2 className="font-display text-2xl font-semibold">{t("chat.empty.title")}</h2>
    <p className="mt-2 text-sm text-muted-foreground">{t("chat.empty.subtitle")}</p>
  </div>
</div>
```

---

## Errors & Notifications

`<Toaster position="top-right" />` is mounted in `App.tsx`. Use `toast` from `sonner` (not react-toastify, not MUI snackbar):

```typescript
import { toast } from "sonner";

toast.success(t("auth.login.loginSuccess"));
toast.error(t(`errors.${code}`));
```

- Put toasts in the mutation hook's `onSuccess` / `onError`.
- API error codes map to `errors.<code>` in `locales/*.json` — add new codes to both files.
- Don't swallow errors silently; at least surface a toast or rethrow.

## Form Validation Errors

Shown under the field from `fieldState.error.message` (messages come from zod schemas in `utils/validations.ts`), with `text-destructive`. See [common-patterns.md](common-patterns.md).

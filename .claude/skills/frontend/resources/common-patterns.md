# Common Patterns

Forms, i18n, auth/tokens, theme, contexts.

---

## Forms: react-hook-form + zod + i18n

### 1. Schema in `utils/validations.ts`

Schemas are **factories** so messages are re-translated when the language changes:

```typescript
import i18n from "@/utils/i18n";
import { z } from "zod";

export const loginSchema = () =>
  z.object({
    email: z
      .email(i18n.t("auth.validation.emailInvalid"))
      .min(1, i18n.t("auth.validation.emailRequired")),
    password: z.string().min(6, i18n.t("auth.validation.passwordMin")),
  });

export type LoginFormValues = z.infer<ReturnType<typeof loginSchema>>;
```

Cross-field rules use `.refine(..., { message, path: ["confirmPassword"] })` (see `registerSchema`).

### 2. Form component (`components/LoginForm/index.tsx`)

```tsx
const { t, i18n } = useTranslation();
const schema = useMemo(() => loginSchema(), [i18n.language]);
const { control, handleSubmit } = useForm<LoginFormValues>({
  resolver: zodResolver(schema),
  defaultValues: { email: "", password: "" },
});
const { mutate: handleLogin, isPending } = useLogin();

function onSubmit(values: LoginFormValues) {
  handleLogin(values);
}

<form onSubmit={handleSubmit(onSubmit)} className="flex flex-col gap-5">
  <Controller
    control={control}
    name="email"
    render={({ field, fieldState }) => (
      <div className="grid gap-2">
        <Label htmlFor="email" className={cn(fieldState.error && "text-destructive")}>
          {t("auth.login.emailLabel")}
        </Label>
        <Input id="email" type="email" className="h-12 rounded-xl" {...field} />
        {fieldState.error && (
          <p className="text-sm text-destructive">{fieldState.error.message}</p>
        )}
      </div>
    )}
  />
  <Button disabled={isPending} type="submit">{t("auth.login.submit")}</Button>
</form>
```

Use `Controller` + shadcn `Label`/`Input` (the pattern in Login/Register forms).

---

## i18n

- Setup: `utils/i18n.ts` (imported once in `App.tsx`). Languages: `en`, `vi`; detection from `localStorage["language"]` then navigator.
- Resources: `locales/en.json`, `locales/vi.json`, nested by area: `common.*`, `auth.login.*`, `auth.register.*`, `auth.validation.*`, `friends.*`, `chat.*`, `errors.*`.
- In components: `const { t } = useTranslation(); t("friends.title")`; dynamic keys: `t(\`friends.tabs.${option}\`)`.
- Outside React (schemas, utils): `import i18n from "@/utils/i18n"; i18n.t("...")`.
- **Always add the key to both JSON files** with the same structure.
- Language switching UI: `components/LanguageSwitcher`.

---

## Auth & Tokens

- Login (`hooks/useLogin.ts`): stores `accessToken` in `localStorage[ACCESS_TOKEN_KEY]`, `refreshToken` in a cookie via `js-cookie`, invalidates `QUERY_KEY.USER`, toasts, navigates `/chat`.
- Current user: `const { data: user, isLoading } = useAuth();` (query key `[QUERY_KEY.USER, accessToken]`, disabled without token).
- Logout: `hooks/useLogout.ts`.
- Refresh on 401 is automatic in `utils/axios.ts`.
- Route protection: `routes/AuthGuard.tsx`.

---

## Theme

`contexts/ThemeContext.tsx` toggles the `dark` class on `<html>` and persists to `localStorage["theme"]`.

```tsx
import { useTheme } from "@/contexts/ThemeContext";
const { theme, setTheme } = useTheme();
setTheme(theme === "dark" ? "light" : "dark");
```

UI: `components/ThemeToggle`.

---

## Contexts

Providers live in `contexts/`, default-export the provider, named-export the consumer hook that throws outside the provider:

```tsx
const Ctx = createContext<Value | undefined>(undefined);

function SomeContext({ children }: { children: React.ReactNode }) {
  return <Ctx.Provider value={...}>{children}</Ctx.Provider>;
}

export function useSome() {
  const context = useContext(Ctx);
  if (!context) throw new Error("useSome must be used within SomeContext");
  return context;
}

export default SomeContext;
```

Mount new providers in `App.tsx` around `BrowserRouter`.

---

## Client State (Zustand)

`zustand` + `immer` are installed. For cross-component client state that isn't server data (e.g. UI preferences, socket status), create a store — keep server data in TanStack Query, not Zustand.

---

## Dialogs

Use shadcn `Dialog` from `@/components/ui/dialog` (`Dialog`, `DialogTrigger`, `DialogContent`, `DialogHeader`, `DialogTitle`, `DialogFooter`). Translate title and buttons; use `common.cancel` / `common.save`.

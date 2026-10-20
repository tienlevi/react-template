# Routing Guide

`react-router-dom` v7 in declarative mode: `BrowserRouter` + `<Routes>` generated from a config array. No file-based routing, no TanStack Router.

---

## Setup (`App.tsx`)

```tsx
<ThemeContext>
  <QueryContext>
    <BrowserRouter>
      <Routes>
        <Route>
          {routers.map((route) => (
            <Route key={route.id} path={route.href} element={route.element} />
          ))}
          <Route path="*" element={<Navigate to="/" replace />} />
        </Route>
      </Routes>
    </BrowserRouter>
    <Toaster position="top-right" />
  </QueryContext>
</ThemeContext>
```

Unknown paths redirect to `/`.

---

## Route Config (`routes/routes.tsx`)

```typescript
// interfaces/routes.ts
export type Routes = {
  href: string;
  id: string;
  name: string;
  element: React.ReactNode;
};
```

Add a route by appending an entry. Wrap in `AuthGuard`, and in `ChatLayout` for app screens:

```tsx
import Settings from "@/pages/Settings";

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

Then add the nav entry in `components/NavRail` if it should appear in the sidebar.

---

## AuthGuard (`routes/AuthGuard.tsx`)

- Uses `useAuth()`; shows a centered `Loader2` spinner while loading.
- Logged in + on `/` or `/register` → redirect `/chat`.
- Not logged in + any other page → redirect `/`.

If you add another public page (e.g. `/forgot-password`), add it to the `isAuthPage` check.

---

## Navigation & URL State

```tsx
import { Link, useNavigate, useSearchParams } from "react-router-dom";

<Link to="/register">...</Link>

const navigate = useNavigate();
navigate("/chat", { replace: true });
navigate(`/chat?receiverId=${user._id}`);

const [searchParams] = useSearchParams();
const receiverId = searchParams.get("receiverId") || "";
```

The selected conversation is stored in the `receiverId` search param — keep URL as the source of truth for shareable selection state.

---

## Lazy Loading (optional)

Routes are currently imported eagerly. If a page becomes heavy:

```tsx
const Settings = lazy(() => import("@/pages/Settings"));
element: (
  <AuthGuard>
    <Suspense fallback={<PageLoader />}>
      <Settings />
    </Suspense>
  </AuthGuard>
),
```

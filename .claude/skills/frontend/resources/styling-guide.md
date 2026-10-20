# Styling Guide

TailwindCSS v4 (CSS-first) + shadcn/ui (new-york, neutral base, CSS variables). No MUI, no CSS modules, no styled-components.

---

## Where Styles Live

- `src/assets/css/index.css`
  - `@import "tailwindcss"; @import "tw-animate-css";`
  - `@custom-variant dark (&:is(.dark *));` — dark mode via `.dark` class on `<html>` (set by `ThemeContext`)
  - `:root` / `.dark` CSS variables (`--background`, `--primary`, `--success`, ...)
  - `@theme inline` maps them to Tailwind colors (`bg-background`, `text-primary`, `bg-success`, ...)
  - Fonts: `font-sans` (Plus Jakarta Sans), `font-display` (Fraunces)
  - Custom: `animate-float-slow`, `animate-float-slower`, `bg-grain`
- Components: Tailwind utility classes in `className`.

There is no `tailwind.config.js` — add tokens in `index.css`.

---

## Semantic Tokens (use these, not raw colors)

| Purpose          | Classes                                                   |
| ---------------- | --------------------------------------------------------- |
| Page / surface   | `bg-background`, `bg-card`, `bg-popover`                  |
| Text             | `text-foreground`, `text-muted-foreground`                |
| Brand            | `bg-primary text-primary-foreground`, `text-primary`      |
| Subtle fill      | `bg-secondary`, `bg-muted`, `bg-accent`                   |
| States           | `text-destructive`, `bg-success/10 text-success`          |
| Borders / focus  | `border-border`, `border-input`, `ring-ring`              |

Opacity modifiers are fine: `bg-primary/12`, `ring-ring/50`. Tokens adapt to dark mode automatically; use `dark:` only for special cases.

---

## `cn()` for Conditional Classes

```tsx
import { cn } from "@/lib/utils";

<button
  className={cn(
    "flex items-center gap-1.5 rounded-full px-3 py-1.5 text-xs font-semibold transition-colors",
    tab === option
      ? "bg-foreground text-background"
      : "bg-secondary text-muted-foreground hover:text-foreground",
  )}
/>
```

Accept `className` from the parent and put it last: `cn("base classes", className)`.

---

## Project Look & Feel

- Cards/panels: `rounded-3xl border border-border bg-card`
- Buttons / inputs / avatars: `rounded-2xl` or `rounded-xl`; pills `rounded-full`
- Headings: `font-display text-3xl font-semibold tracking-tight leading-none`
- Inputs on panels: `h-10 rounded-2xl border-transparent bg-secondary`
- Icons: `size-4` / `size-5`, `text-muted-foreground`
- Layout: flex + `gap-*`, `min-h-0 flex-1` for scroll regions inside flex columns, `h-svh` for full height
- Responsive: mobile first, `md:` breakpoint for the two-pane chat layout (`hidden md:flex`)

See `frontend/docs/design-guidelines/extracted-design.md` for the design reference.

---

## Tailwind v4 Notes

- Shadow rename: `shadow-sm` → `shadow-xs`, `shadow` → `shadow-sm`.
- Use `size-*` for equal width/height.
- Arbitrary values allowed when needed: `size-[520px]`, `bg-[oklch(0.90_0.06_260)]`.
- Never `style={{ color: "var(--primary)" }}` in JSX — use the utility class.

---

## shadcn Variants

Use component props before custom classes:

```tsx
<Button variant="ghost" size="icon" />
<Button variant="outline" size="sm" />
<Badge variant="secondary" />
```

Extend a primitive's look via `className`; edit `components/ui/*` (cva variants) only when the change should apply everywhere.

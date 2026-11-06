# Tailwind CSS — Cheat Sheet

Full guide: [README.md](README.md)

## Vite

```bash
npm install tailwindcss @tailwindcss/vite
```

```ts
// vite.config.ts
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({ plugins: [react(), tailwindcss()] });
```

```css
/* src/index.css */
@import "tailwindcss";
```

## Next.js

```bash
npm install tailwindcss @tailwindcss/postcss postcss
```

```js
// postcss.config.mjs
export default { plugins: { "@tailwindcss/postcss": {} } };
```

```css
/* app/globals.css */
@import "tailwindcss";
```

## Theme tokens (v4)

```css
@theme {
  --color-brand-500: oklch(0.62 0.19 250);
  --font-display: "Satoshi", sans-serif;
  --radius-card: 0.875rem;
}
```

```html
<div class="bg-brand-500 font-display rounded-card p-4">...</div>
```

## Custom utilities

```css
@utility btn {
  @apply inline-flex items-center rounded-lg px-4 py-2 font-medium;
}
```

## Dark mode

```css
@custom-variant dark (&:where(.dark, .dark *));
```

```html
<html class="dark">
  <body class="bg-white text-black dark:bg-gray-950 dark:text-white">
```

## CLI

```bash
npm install -D tailwindcss @tailwindcss/cli

npx @tailwindcss/cli -i ./src/input.css -o ./dist/output.css --minify
npx @tailwindcss/cli -i ./src/input.css -o ./dist/output.css --watch
```

## Prettier sorting

```bash
npm install -D prettier prettier-plugin-tailwindcss
```

```json
{ "plugins": ["prettier-plugin-tailwindcss"] }
```

## Reminder

v4 is CSS-first: no `tailwind.config.js` and no `content: []` — configuration lives in `@theme`.

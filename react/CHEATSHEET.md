# React — Cheat Sheet

Full guide: [README.md](README.md)

## Create & run

```bash
npm create vite@latest my-app -- --template react-ts
cd my-app
npm install
npm run dev            # http://localhost:5173
```

```bash
npm run build          # → dist/
npm run preview        # serve the build
npm run lint
npx tsc --noEmit       # type-check
```

## Control

```bash
npm run dev -- --port 3000     # change port
npm run dev -- --host          # expose on LAN

# Windows: kill whatever holds port 5173
netstat -ano | findstr :5173
taskkill /PID <pid> /F
```

## Env

```bash
# .env — only VITE_ vars reach the browser
VITE_API_URL=http://localhost:3000/api
```

```ts
const url = import.meta.env.VITE_API_URL;
const isDev = import.meta.env.DEV;
```

## Common packages

```bash
npm install react-router-dom axios @tanstack/react-query zustand
npm install react-hook-form zod
```

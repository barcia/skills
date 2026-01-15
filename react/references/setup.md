# Project Setup

Complete setup guide for initializing a new React project with the preferred stack.

## Requirements

```json
{
  "engines": {
    "node": ">=24.0.0"
  },
  "packageManager": "pnpm@10.x"
}
```

## Installation

```bash
# Initialize
corepack enable
pnpm init

# Core dependencies
pnpm add react react-dom @tanstack/react-query zustand wouter nuqs

# Dev dependencies
pnpm add -D typescript vite @vitejs/plugin-react
pnpm add -D tailwindcss @tailwindcss/vite lightningcss
pnpm add -D @biomejs/biome zod
pnpm add -D vitest @testing-library/react @testing-library/jest-dom msw
```

## Configuration Files

### vite.config.ts

```typescript
import tailwindcss from "@tailwindcss/vite";
import react from "@vitejs/plugin-react";
import { defineConfig } from "vitest/config";

export default defineConfig({
  plugins: [react(), tailwindcss()],
  resolve: {
    alias: { "@": "/src" },
  },
  css: {
    transformer: "lightningcss",
  },
  build: {
    cssMinify: "lightningcss",
  },
  test: {
    environment: "jsdom",
    setupFiles: ["./src/testing/setup-tests.ts"],
    globals: true,
    include: ["src/**/*.test.{ts,tsx}"],
  },
});
```

### tsconfig.json

```json
{
  "compilerOptions": {
    "strict": true,
    "baseUrl": ".",
    "paths": { "@/*": ["./src/*"] }
  }
}
```

### biome.json

```json
{
  "$schema": "https://biomejs.dev/schemas/2.2.4/schema.json",
  "formatter": {
    "enabled": true,
    "lineWidth": 120
  },
  "linter": {
    "enabled": true,
    "rules": {
      "correctness": {
        "noRestrictedImports": {
          "level": "error",
          "options": {
            "paths": {
              "@tanstack/react-query": "Import from @/lib/react-query instead",
              "zustand": "Import from @/lib/zustand instead",
              "wouter": "Import from @/lib/wouter instead",
              "nuqs": "Import from @/lib/nuqs instead",
              "zod": "Import from @/lib/zod instead"
            }
          }
        }
      },
      "nursery": {
        "useSortedClasses": {
          "level": "warn",
          "fix": "safe"
        }
      }
    }
  },
  "assist": {
    "enabled": true
  }
}
```

### package.json scripts

```json
{
  "scripts": {
    "dev": "vite",
    "build": "tsc -b && vite build",
    "preview": "vite preview",
    "lint": "biome check .",
    "format": "biome format --write .",
    "test": "vitest",
    "test:ci": "vitest run",
    "test:coverage": "vitest run --coverage"
  }
}
```

## CSS Setup

Create `src/index.css`:

```css
@import "tailwindcss";
```

## Entry Point

Create `src/main.tsx`:

```tsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import { App } from "@/app/app";
import "./index.css";

createRoot(document.getElementById("root")!).render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

## HTML Template

Create `index.html`:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>App</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

## Test Setup

Create `src/testing/setup-tests.ts`:

```typescript
import "@testing-library/jest-dom/vitest";
import { server } from "./mocks/server";

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

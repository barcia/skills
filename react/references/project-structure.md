# Project Structure

## Base Structure

```
src/
├── app/                  # App shell (providers, main component)
│   ├── app.tsx          # Main app component
│   └── provider.tsx     # Global providers wrapper
├── components/          # Shared UI components
├── config/              # Global config, env variables
├── features/            # Feature-based modules
├── hooks/               # Shared hooks
├── lib/                 # Re-exports of dependencies
│   ├── react-query.ts
│   ├── zustand.ts
│   ├── wouter.ts
│   ├── nuqs.ts
│   └── zod.ts
├── pages/               # Route pages
├── routes.ts            # Route definitions
├── stores/              # Global state stores
├── testing/             # Test utilities and mocks
├── types/               # Shared TypeScript types
└── utils/               # Shared utility functions
```

## Feature Structure

Each feature is self-contained:

```
features/[feature-name]/
├── api/          # API request declarations and hooks
├── components/   # Feature-specific components
├── hooks/        # Feature-specific hooks
├── stores/       # Feature-specific state
├── types/        # Feature-specific types
└── utils/        # Feature-specific utilities
```

Only include folders needed for the feature.

## Lib Re-exports

All external dependencies are re-exported from `lib/`:

```typescript
// lib/wouter.ts
export { Link, Route, Switch, useLocation, useParams, useRoute } from "wouter";

// lib/react-query.ts
export {
  useQuery,
  useMutation,
  useQueryClient,
  QueryClient,
  QueryClientProvider,
  queryOptions,
} from "@tanstack/react-query";

// lib/zustand.ts
export { create } from "zustand";

// lib/nuqs.ts
export { useQueryState, useQueryStates, parseAsString, parseAsInteger } from "nuqs";

// lib/zod.ts
export { z } from "zod";
export type { ZodType } from "zod";
```

**Why?** Easier refactoring. Change dependency in one place.

## Pages and Routes

```typescript
// routes.ts
import Home from "@/pages/home";
import About from "@/pages/about";

type RouteDef = {
  path: string;
  component: React.ComponentType;
};

const routes: Record<string, RouteDef> = {
  home: { path: "/", component: Home },
  about: { path: "/about", component: About },
};

export default routes;
```

```typescript
// main.tsx
import { Route, Switch } from "@/lib/wouter";
import NotFound from "@/pages/404";
import routes from "@/routes";

createRoot(document.getElementById("root")!).render(
  <Switch>
    {Object.values(routes).map(({ path, component: Component }) => (
      <Route key={path} path={path}>
        <Component />
      </Route>
    ))}
    <Route>
      <NotFound />
    </Route>
  </Switch>
);
```

## Key Rules

### 1. No Cross-Feature Imports

Features must not import from other features. Compose at app/page level.

```typescript
// Bad - feature importing from another feature
import { UserCard } from "@/features/users/components/user-card";
// in features/posts/components/post.tsx

// Good - compose at page level
// pages/user-posts.tsx
import { UserCard } from "@/features/users/components/user-card";
import { PostList } from "@/features/posts/components/post-list";
```

### 2. Unidirectional Flow

Code flows: `lib → shared → features → pages/app`

- `lib/` can be imported anywhere
- `components/`, `hooks/`, `utils/` can be imported anywhere
- `features/` can import from lib/shared, NOT from `pages/` or `app/`
- `pages/` can import from features and shared

### 3. No Barrel Files

Import files directly. Barrel files hurt tree shaking.

```typescript
// Bad
import { Button } from "@/components/ui";

// Good
import { Button } from "@/components/ui/button/button";
```

### 4. Import from lib/, not directly

```typescript
// Bad
import { useQuery } from "@tanstack/react-query";
import { Link } from "wouter";

// Good
import { useQuery } from "@/lib/react-query";
import { Link } from "@/lib/wouter";
```

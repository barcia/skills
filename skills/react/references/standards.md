# Code Standards

Development standards and conventions. For initial setup and configuration files, see [setup.md](setup.md).

## TypeScript

Strict mode always. Never use `any`.

## Absolute Imports

Always use `@/` prefix. Never relative paths beyond `./`.

```typescript
// Good
import { Button } from "@/components/ui/button";
import { useAuth } from "@/lib/auth";
import { useQuery } from "@/lib/react-query";

// Bad
import { Button } from "../../../components/ui/button";
```

## File Naming

**kebab-case** for all files and folders:

```
src/
├── components/
│   └── ui/
│       └── button/
│           ├── button.tsx
│           └── button.test.tsx
├── features/
│   └── user-management/
│       └── components/
│           └── user-profile.tsx
└── pages/
    ├── home.tsx
    └── 404.tsx
```

## Biome Commands

```bash
# Check linting
pnpm biome check .

# Format files
pnpm biome format --write .

# Fix linting issues
pnpm biome check --fix .

# Check and fix everything
pnpm biome check --fix --unsafe .
```

## Import Restrictions

Enforce lib re-exports with Biome (configured in [setup.md](setup.md)):

```typescript
// Bad - direct imports blocked by Biome
import { useQuery } from "@tanstack/react-query";
import { Link } from "wouter";

// Good - use lib re-exports
import { useQuery } from "@/lib/react-query";
import { Link } from "@/lib/wouter";
```

## Git Hooks (Optional)

With Lefthook or Husky:

```yaml
# lefthook.yml
pre-commit:
  commands:
    check:
      run: pnpm biome check --staged
    types:
      run: pnpm tsc --noEmit
```

---
name: react-architecture
description: Personal React architecture patterns combining Bulletproof React principles with preferred stack. Use when working with React projects for: (1) Creating new React applications or features, (2) Structuring or refactoring React code, (3) Setting up project architecture with feature-based modules, (4) Implementing API layers, state management, forms, auth, or testing. Stack: React 19 + Vite + TypeScript + Tailwind 4 + Biome + Wouter + Zustand + React Query + nuqs + Zod + PNPM.
---

# React Architecture

Personal architecture for scalable React applications using feature-based modules and modern tooling.

## Stack

| Category           | Preferred                                  | Also Valid                          |
| ------------------ | ------------------------------------------ | ----------------------------------- |
| Build              | **Vite** + **LightningCSS**                | -                                   |
| Package Manager    | **PNPM**                                   | npm, yarn, bun                      |
| Linting/Formatting | **Biome**                                  | ESLint + Prettier                   |
| Routing            | **Wouter** (minimal)                       | **React Router** (full-featured)    |
| Server State       | **React Query**                            | SWR, RTK Query                      |
| Client State       | **Zustand**                                | Jotai                               |
| URL State          | **nuqs**                                   | useSearchParams                     |
| Forms              | **React Hook Form** + **Zod**              | Formik                              |
| Styling            | **Tailwind CSS 4**                         | **CSS Modules** (per project needs) |
| Testing            | **Vitest** + **Testing Library** + **MSW** | Jest                                |

**Note:** Router and styling choices depend on project needs. Use Wouter for simple SPAs, React Router for complex routing. Use Tailwind for rapid prototyping, CSS Modules for component isolation.

## Quick Reference

| Topic                               | Reference File                                          |
| ----------------------------------- | ------------------------------------------------------- |
| Initial setup & configuration       | [setup.md](references/setup.md)                         |
| Project structure & feature modules | [project-structure.md](references/project-structure.md) |
| Components & styling patterns       | [components.md](references/components.md)               |
| API layer & React Query             | [api-layer.md](references/api-layer.md)                 |
| State management                    | [state-management.md](references/state-management.md)   |
| Testing strategy & MSW              | [testing.md](references/testing.md)                     |
| Code standards & Biome              | [standards.md](references/standards.md)                 |
| Auth, authorization, security       | [security.md](references/security.md)                   |

## Project Structure

```
src/
├── app/              # App shell (providers, main app)
├── components/       # Shared UI components
├── config/           # Global config, env
├── features/         # Feature modules (self-contained)
│   └── [feature]/
│       ├── api/      # API hooks and fetchers
│       ├── components/
│       └── hooks/
├── hooks/            # Shared hooks
├── lib/              # Re-exports of dependencies
├── pages/            # Route pages
├── stores/           # Global Zustand stores
├── testing/          # Test utils, MSW mocks
├── types/            # Shared types
└── utils/            # Utilities
```

See [project-structure.md](references/project-structure.md) for detailed structure and rules.

## Core Principles

1. **Feature-based organization** - Code grouped by feature, not by type
2. **Unidirectional flow** - `shared → features → app`
3. **No cross-feature imports** - Features are independent
4. **Re-export dependencies** - Import from `@/lib/*` not directly
5. **Type safety** - TypeScript strict mode, Zod validation
6. **Absolute imports** - Always use `@/` prefix

## Lib Re-exports

All external dependencies are re-exported from `lib/` for easier refactoring:

```typescript
import { useQuery } from "@/lib/react-query";
import { Link } from "@/lib/wouter";
import { create } from "@/lib/zustand";
```

See [project-structure.md](references/project-structure.md) for full re-export examples.

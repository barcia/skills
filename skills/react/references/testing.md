# Testing

Priority: **Integration tests > E2E tests > Unit tests**

Integration tests provide the best confidence-to-effort ratio.

## Types of Tests

### Unit Tests
Test isolated shared components and utilities.

```tsx
// components/ui/button/__tests__/button.test.tsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { Button } from "../button";

test("calls onClick when clicked", async () => {
  const onClick = vi.fn();
  render(<Button onClick={onClick}>Click me</Button>);

  await userEvent.click(screen.getByRole("button"));

  expect(onClick).toHaveBeenCalledTimes(1);
});
```

### Integration Tests
Test features with mocked API. Most valuable test type.

```tsx
// features/discussions/__tests__/discussions.test.tsx
import { render, screen, waitFor } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { AppProvider } from "@/app/provider";
import { DiscussionsList } from "../components/discussions-list";

test("creates a new discussion", async () => {
  render(<DiscussionsList />, { wrapper: AppProvider });

  await userEvent.click(screen.getByRole("button", { name: /create/i }));
  await userEvent.type(screen.getByLabelText(/title/i), "New Discussion");
  await userEvent.type(screen.getByLabelText(/body/i), "Discussion content");
  await userEvent.click(screen.getByRole("button", { name: /submit/i }));

  await waitFor(() => {
    expect(screen.getByText("New Discussion")).toBeInTheDocument();
  });
});
```

### E2E Tests
Full application tests with Playwright.

```typescript
// e2e/discussions.spec.ts
import { test, expect } from "@playwright/test";

test("user can create discussion", async ({ page }) => {
  await page.goto("/discussions");
  await page.click('button:has-text("Create")');
  await page.fill('[name="title"]', "E2E Test Discussion");
  await page.fill('[name="body"]', "Test content");
  await page.click('button:has-text("Submit")');

  await expect(page.locator("text=E2E Test Discussion")).toBeVisible();
});
```

## MSW for API Mocking

Mock API at network level - works in tests and development.

```typescript
// testing/mocks/handlers/discussions.ts
import { http, HttpResponse } from "msw";
import { db } from "../db";

export const discussionsHandlers = [
  http.get("/api/discussions", () => {
    const discussions = db.discussion.getAll();
    return HttpResponse.json({ data: discussions });
  }),

  http.post("/api/discussions", async ({ request }) => {
    const data = await request.json();
    const discussion = db.discussion.create(data);
    return HttpResponse.json(discussion);
  }),
];
```

```typescript
// testing/mocks/db.ts
import { factory, primaryKey } from "@mswjs/data";

export const db = factory({
  discussion: {
    id: primaryKey(() => crypto.randomUUID()),
    title: String,
    body: String,
    createdAt: () => new Date().toISOString(),
  },
});
```

```typescript
// testing/mocks/server.ts
import { setupServer } from "msw/node";
import { discussionsHandlers } from "./handlers/discussions";

export const server = setupServer(...discussionsHandlers);
```

```typescript
// testing/setup-tests.ts
import "@testing-library/jest-dom/vitest";
import { server } from "./mocks/server";

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

## Test Utils

Custom render with providers:

```tsx
// testing/test-utils.tsx
import { render, RenderOptions } from "@testing-library/react";
import { AppProvider } from "@/app/provider";

const AllProviders = ({ children }: { children: React.ReactNode }) => (
  <AppProvider>{children}</AppProvider>
);

export const renderWithProviders = (ui: React.ReactElement, options?: RenderOptions) =>
  render(ui, { wrapper: AllProviders, ...options });

export * from "@testing-library/react";
export { renderWithProviders as render };
```

## Tooling

| Tool | Purpose |
|------|---------|
| Vitest | Test runner (faster than Jest) |
| Testing Library | DOM testing utilities |
| MSW | API mocking |
| Playwright | E2E testing |

## Vitest Config

Vitest configuration is included in `vite.config.ts`. See [setup.md](setup.md) for the complete configuration.

Key test options:
- `environment: "jsdom"` - DOM environment for React
- `setupFiles` - Points to test setup with MSW
- `globals: true` - Global test functions (describe, test, expect)
- `include` - Test file patterns

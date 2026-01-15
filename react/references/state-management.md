# State Management

Categorize state by type:

## 1. Component State

Local to component. Use React primitives.

```typescript
// Simple state
const [isOpen, setIsOpen] = useState(false);

// Complex state with related updates
const [state, dispatch] = useReducer(reducer, initialState);
```

Start local, lift up only when needed elsewhere.

## 2. Application State

Global UI state (modals, notifications, theme). Keep minimal.

**Zustand (preferred):**

```typescript
// stores/notifications-store.ts
import { create } from "@/lib/zustand";
import { nanoid } from "nanoid";

export type Notification = {
  id: string;
  type: "info" | "warning" | "success" | "error";
  title: string;
  message?: string;
};

type NotificationsStore = {
  notifications: Notification[];
  addNotification: (notification: Omit<Notification, "id">) => void;
  dismissNotification: (id: string) => void;
};

export const useNotifications = create<NotificationsStore>((set) => ({
  notifications: [],
  addNotification: (notification) =>
    set((state) => ({
      notifications: [...state.notifications, { id: nanoid(), ...notification }],
    })),
  dismissNotification: (id) =>
    set((state) => ({
      notifications: state.notifications.filter((n) => n.id !== id),
    })),
}));
```

**Alternatives:** Jotai (atomic), Redux Toolkit

## 3. Server Cache State

Remote data. Use React Query.

```typescript
// Already handled by React Query hooks in api layer
import { useQuery } from "@/lib/react-query";

const { data, isLoading, error } = useDiscussions({ page: 1 });
```

**Alternatives:** SWR, RTK Query

## 4. Form State

Use React Hook Form with Zod validation:

```tsx
import { useForm, FormProvider, SubmitHandler } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { z } from "@/lib/zod";

const schema = z.object({
  title: z.string().min(1, "Required"),
  body: z.string().min(1, "Required"),
});

type FormValues = z.infer<typeof schema>;

const MyForm = () => {
  const form = useForm<FormValues>({
    resolver: zodResolver(schema),
  });

  const onSubmit: SubmitHandler<FormValues> = (data) => {
    console.log(data);
  };

  return (
    <form onSubmit={form.handleSubmit(onSubmit)}>
      <input {...form.register("title")} />
      {form.formState.errors.title && <span>{form.formState.errors.title.message}</span>}

      <textarea {...form.register("body")} />
      {form.formState.errors.body && <span>{form.formState.errors.body.message}</span>}

      <button type="submit">Submit</button>
    </form>
  );
};
```

**Alternatives:** Formik + Yup

## 5. URL State

Use **nuqs** for type-safe URL params:

```typescript
import { useQueryState, parseAsInteger, parseAsString } from "@/lib/nuqs";

const MyComponent = () => {
  // Single param
  const [search, setSearch] = useQueryState("search");

  // With parser
  const [page, setPage] = useQueryState("page", parseAsInteger.withDefault(1));

  // Multiple params
  const [filters, setFilters] = useQueryStates({
    search: parseAsString.withDefault(""),
    page: parseAsInteger.withDefault(1),
    sort: parseAsString.withDefault("date"),
  });

  return (
    <div>
      <input value={search ?? ""} onChange={(e) => setSearch(e.target.value)} />
      <button onClick={() => setPage(page + 1)}>Next Page</button>
    </div>
  );
};
```

### nuqs with Wouter

nuqs works with any router. No special configuration needed.

```typescript
// URL: /products?category=electronics&page=2
const [category] = useQueryState("category"); // "electronics"
const [page] = useQueryState("page", parseAsInteger); // 2
```

**Alternatives:** Manual useSearchParams

## 6. Route Params

Use Wouter for dynamic route segments:

```typescript
import { useParams, useLocation } from "@/lib/wouter";

// Route: /users/:userId
const UserPage = () => {
  const { userId } = useParams<{ userId: string }>();
  const [location, setLocation] = useLocation();

  return <div>User ID: {userId}</div>;
};
```

## Decision Guide

| State Type | Solution |
|------------|----------|
| UI toggle in one component | `useState` |
| Complex local logic | `useReducer` |
| Global UI (modals, toasts) | Zustand |
| Server data | React Query |
| Form inputs + validation | React Hook Form + Zod |
| URL query params | nuqs |
| Route params | Wouter useParams |

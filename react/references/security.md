# Security

## Authentication

### Token Storage

**Prefer HttpOnly cookies** over localStorage (protects against XSS).

```typescript
// API client configured for cookies
api.interceptors.request.use((config) => {
  config.withCredentials = true; // Send cookies
  return config;
});
```

### User State Management

Use React Query for user state:

```typescript
// lib/auth.tsx
import { useQuery, useMutation, useQueryClient } from "@/lib/react-query";
import { api } from "@/lib/api-client";

type User = {
  id: string;
  email: string;
  role: "ADMIN" | "USER";
};

const getUser = async (): Promise<User | null> => {
  try {
    return await api.get("/auth/me");
  } catch {
    return null;
  }
};

export const useUser = () => {
  return useQuery({
    queryKey: ["user"],
    queryFn: getUser,
    staleTime: Infinity,
  });
};

export const useLogin = () => {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (data: { email: string; password: string }) => api.post("/auth/login", data),
    onSuccess: (user) => {
      queryClient.setQueryData(["user"], user);
    },
  });
};

export const useLogout = () => {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: () => api.post("/auth/logout"),
    onSuccess: () => {
      queryClient.setQueryData(["user"], null);
      queryClient.clear();
    },
  });
};
```

### Protected Routes

With Wouter:

```tsx
// components/protected-route.tsx
import { Redirect } from "@/lib/wouter";
import { useUser } from "@/lib/auth";

export const ProtectedRoute = ({ children }: { children: React.ReactNode }) => {
  const { data: user, isLoading } = useUser();

  if (isLoading) return <div>Loading...</div>;
  if (!user) return <Redirect to="/auth/login" />;

  return <>{children}</>;
};
```

With React Router:

```tsx
import { Navigate, Outlet } from "react-router-dom";
import { useUser } from "@/lib/auth";

export const ProtectedRoute = () => {
  const { data: user, isLoading } = useUser();

  if (isLoading) return <div>Loading...</div>;
  if (!user) return <Navigate to="/auth/login" replace />;

  return <Outlet />;
};
```

## Authorization

### RBAC (Role-Based Access Control)

```typescript
// lib/authorization.tsx
import { useUser } from "./auth";

export enum ROLES {
  ADMIN = "ADMIN",
  USER = "USER",
}

type RoleTypes = keyof typeof ROLES;

export const useAuthorization = () => {
  const { data: user } = useUser();

  const checkAccess = ({ allowedRoles }: { allowedRoles: RoleTypes[] }) => {
    if (!user) return false;
    return allowedRoles.includes(user.role);
  };

  return { checkAccess, role: user?.role };
};

type AuthorizationProps = {
  allowedRoles?: RoleTypes[];
  policyCheck?: boolean;
  forbiddenFallback?: React.ReactNode;
  children: React.ReactNode;
};

export const Authorization = ({
  allowedRoles,
  policyCheck,
  forbiddenFallback = null,
  children,
}: AuthorizationProps) => {
  const { checkAccess } = useAuthorization();

  let canAccess = false;
  if (allowedRoles) canAccess = checkAccess({ allowedRoles });
  if (typeof policyCheck !== "undefined") canAccess = policyCheck;

  return <>{canAccess ? children : forbiddenFallback}</>;
};
```

**Usage:**

```tsx
// Only admins see this
<Authorization allowedRoles={["ADMIN"]}>
  <AdminPanel />
</Authorization>
```

### PBAC (Permission-Based Access Control)

For fine-grained control (e.g., owner-only actions):

```typescript
// lib/authorization.tsx
export const POLICIES = {
  "comment:delete": (user: User, comment: Comment) => {
    if (user.role === "ADMIN") return true;
    if (user.role === "USER" && comment.authorId === user.id) return true;
    return false;
  },
};
```

**Usage:**

```tsx
// Only comment author or admin can delete
<Authorization policyCheck={POLICIES["comment:delete"](user, comment)}>
  <DeleteButton onClick={handleDelete} />
</Authorization>
```

## XSS Prevention

Sanitize user-generated HTML:

```tsx
import DOMPurify from "dompurify";
import { marked } from "marked";

export const MdPreview = ({ content }: { content: string }) => {
  const html = marked(content);
  const sanitized = DOMPurify.sanitize(html);

  return <div dangerouslySetInnerHTML={{ __html: sanitized }} />;
};
```

## Error Handling

### API Error Interceptor

```typescript
// Already in api-client.ts
api.interceptors.response.use(
  (response) => response.data,
  (error) => {
    // Show notification
    useNotifications.getState().addNotification({
      type: "error",
      title: "Error",
      message: error.response?.data?.message || error.message,
    });

    // Redirect on 401
    if (error.response?.status === 401) {
      window.location.href = "/auth/login";
    }

    return Promise.reject(error);
  }
);
```

### Error Boundaries

```tsx
import { ErrorBoundary } from "react-error-boundary";

const ErrorFallback = ({ error, resetErrorBoundary }) => (
  <div>
    <p>Something went wrong:</p>
    <pre>{error.message}</pre>
    <button onClick={resetErrorBoundary}>Try again</button>
  </div>
);

// Wrap sections of the app
<ErrorBoundary FallbackComponent={ErrorFallback}>
  <FeatureComponent />
</ErrorBoundary>
```

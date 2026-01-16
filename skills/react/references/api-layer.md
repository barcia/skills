# API Layer

## Single API Client Instance

Create one configured instance for all requests:

```typescript
// lib/api-client.ts
import Axios, { InternalAxiosRequestConfig } from "axios";
import { useNotifications } from "@/stores/notifications-store";
import { env } from "@/config/env";

function authRequestInterceptor(config: InternalAxiosRequestConfig) {
  if (config.headers) {
    config.headers.Accept = "application/json";
  }
  config.withCredentials = true;
  return config;
}

export const api = Axios.create({
  baseURL: env.API_URL,
});

api.interceptors.request.use(authRequestInterceptor);
api.interceptors.response.use(
  (response) => response.data,
  (error) => {
    const message = error.response?.data?.message || error.message;
    useNotifications.getState().addNotification({
      type: "error",
      title: "Error",
      message,
    });

    if (error.response?.status === 401) {
      window.location.href = "/auth/login";
    }

    return Promise.reject(error);
  }
);
```

## Request Declaration Pattern

Every API request has:

1. **Types** - `{Element}Params` and `{Element}Response`
2. **Fetcher function** - calls the endpoint
3. **Hook** - React Query hook (reuses the same types)

### Type Naming Convention

```typescript
// Pattern: {Element}Params, {Element}Response
export type GetDiscussionsParams = { page?: number };
export type GetDiscussionsResponse = { data: Discussion[]; meta: Meta };

export type CreateDiscussionParams = { title: string; body: string };
export type CreateDiscussionResponse = Discussion;
```

### Query Example (GET)

```typescript
// features/discussions/api/get-discussions.ts
import { queryOptions, useQuery } from "@/lib/react-query";
import { api } from "@/lib/api-client";
import type { QueryConfig } from "@/lib/react-query";
import type { Discussion, Meta } from "@/types/api";

// 1. Types
export type GetDiscussionsParams = {
  page?: number;
};

export type GetDiscussionsResponse = {
  data: Discussion[];
  meta: Meta;
};

// 2. Fetcher
export const getDiscussions = (params: GetDiscussionsParams = {}): Promise<GetDiscussionsResponse> => {
  return api.get("/discussions", { params });
};

// 3. Query options
export const getDiscussionsQueryOptions = (params: GetDiscussionsParams = {}) => {
  return queryOptions({
    queryKey: ["discussions", params],
    queryFn: () => getDiscussions(params),
  });
};

// 4. Hook - reuses GetDiscussionsParams
type UseDiscussionsOptions = GetDiscussionsParams & {
  queryConfig?: QueryConfig<typeof getDiscussionsQueryOptions>;
};

export const useDiscussions = ({ queryConfig, ...params }: UseDiscussionsOptions = {}) => {
  return useQuery({
    ...getDiscussionsQueryOptions(params),
    ...queryConfig,
  });
};
```

### Mutation Example (POST/PUT/DELETE)

```typescript
// features/discussions/api/create-discussion.ts
import { useMutation, useQueryClient } from "@/lib/react-query";
import { z } from "@/lib/zod";
import { api } from "@/lib/api-client";
import type { MutationConfig } from "@/lib/react-query";
import type { Discussion } from "@/types/api";
import { getDiscussionsQueryOptions } from "./get-discussions";

// 1. Types + Schema
export type CreateDiscussionParams = {
  title: string;
  body: string;
};

export type CreateDiscussionResponse = Discussion;

export const createDiscussionSchema = z.object({
  title: z.string().min(1, "Required"),
  body: z.string().min(1, "Required"),
}) satisfies z.ZodType<CreateDiscussionParams>;

// 2. Fetcher
export const createDiscussion = (params: CreateDiscussionParams): Promise<CreateDiscussionResponse> => {
  return api.post("/discussions", params);
};

// 3. Hook - reuses CreateDiscussionParams
type UseCreateDiscussionOptions = {
  mutationConfig?: MutationConfig<typeof createDiscussion>;
};

export const useCreateDiscussion = ({ mutationConfig }: UseCreateDiscussionOptions = {}) => {
  const queryClient = useQueryClient();
  const { onSuccess, ...restConfig } = mutationConfig || {};

  return useMutation({
    onSuccess: (...args) => {
      queryClient.invalidateQueries({
        queryKey: getDiscussionsQueryOptions().queryKey,
      });
      onSuccess?.(...args);
    },
    ...restConfig,
    mutationFn: createDiscussion,
  });
};
```

## React Query Types Helper

```typescript
// lib/react-query.ts
export {
  useQuery,
  useMutation,
  useQueryClient,
  QueryClient,
  QueryClientProvider,
  queryOptions,
} from "@tanstack/react-query";
export type { QueryKey, UseQueryOptions, UseMutationOptions } from "@tanstack/react-query";

import type { UseQueryOptions, UseMutationOptions, DefaultError } from "@tanstack/react-query";

export type QueryConfig<T extends (...args: any[]) => any> = Omit<
  ReturnType<T>,
  "queryKey" | "queryFn"
>;

export type MutationConfig<MutationFnType extends (...args: any[]) => Promise<any>> = UseMutationOptions<
  Awaited<ReturnType<MutationFnType>>,
  DefaultError,
  Parameters<MutationFnType>[0]
>;
```

## Data Prefetching

```typescript
// In route loader or component
import { queryClient } from "@/lib/react-query";
import { getDiscussionsQueryOptions } from "@/features/discussions/api/get-discussions";

// Prefetch before navigation
queryClient.prefetchQuery(getDiscussionsQueryOptions({ page: 1 }));
```

## QueryClient Setup

```typescript
// lib/react-query.ts (or app/provider.tsx)
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60, // 1 minute
      retry: 1,
    },
  },
});

// In provider
<QueryClientProvider client={queryClient}>
  {children}
</QueryClientProvider>
```

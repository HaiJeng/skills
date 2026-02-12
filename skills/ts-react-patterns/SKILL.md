---
name: ts-react-patterns
description: TypeScript and React architectural patterns, project structure, and best practices for building scalable, type-safe React applications. Use when setting up project architecture or implementing common patterns.
triggers:
- "project structure"
- "architecture patterns"
- "type-safe react"
- "react typescript"
- "feature-based structure"
- barrel exports
---

# TypeScript + React Patterns

## Project Structure Patterns

### Feature-Based Architecture

```
src/
├── features/                    # Feature modules
│   ├── auth/                   # Auth feature
│   │   ├── components/         # Auth-specific components
│   │   │   ├── LoginForm.tsx
│   │   │   ├── RegisterForm.tsx
│   │   │   └── index.ts
│   │   ├── hooks/              # Auth hooks
│   │   │   ├── useAuth.ts
│   │   │   └── index.ts
│   │   ├── services/           # Auth API calls
│   │   │   └── authApi.ts
│   │   ├── types/              # Auth types
│   │   │   └── auth.types.ts
│   │   └── index.ts           # Public API
│   ├── dashboard/              # Dashboard feature
│   │   ├── components/
│   │   ├── hooks/
│   │   └── types/
│   └── users/                  # Users feature
│       ├── components/
│       ├── hooks/
│       └── types/
├── components/                 # Shared components
│   ├── ui/                    # Basic UI components
│   │   ├── Button/
│   │   │   ├── Button.tsx
│   │   │   ├── Button.test.tsx
│   │   │   ├── Button.stories.tsx
│   │   │   └── index.ts
│   │   ├── Input/
│   │   ├── Modal/
│   │   └── index.ts
│   ├── layout/                # Layout components
│   │   ├── Header/
│   │   ├── Sidebar/
│   │   └── index.ts
│   └── index.ts
├── hooks/                      # Shared hooks
│   ├── useLocalStorage.ts
│   ├── useDebounce.ts
│   └── index.ts
├── services/                   # API services
│   ├── api.ts
│   ├── auth.service.ts
│   └── index.ts
├── types/                      # Global types
│   ├── api.types.ts
│   └── index.ts
├── utils/                      # Utility functions
│   ├── format.ts
│   ├── validation.ts
│   └── index.ts
├── lib/                        # Third-party configs
│   ├── axios.ts
│   ├── react-query.ts
│   └── chakra-theme.ts
├── contexts/                   # React contexts
│   ├── AuthContext.tsx
│   └── ThemeContext.tsx
├── pages/                      # Page components
│   ├── HomePage.tsx
│   ├── DashboardPage.tsx
│   └── index.ts
├── App.tsx
└── main.tsx
```

### Barrel Exports Pattern

```typescript
// features/auth/components/index.ts
export { LoginForm } from './LoginForm';
export { RegisterForm } from './RegisterForm';
export { PasswordReset } from './PasswordReset';

// features/auth/index.ts
export * from './components';
export * from './hooks';
export * from './services';
export * from './types';

// Usage - clean imports
import { LoginForm, RegisterForm, useAuth } from '@/features/auth';
```

## Type-Safe API Layer

### API Client Setup

```typescript
// lib/axios.ts
import axios, { AxiosError, AxiosInstance } from 'axios';

export const apiClient: AxiosInstance = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json',
  },
});

// Request interceptor
apiClient.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem('token');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => Promise.reject(error)
);

// Response interceptor
apiClient.interceptors.response.use(
  (response) => response.data,
  (error: AxiosError) => {
    if (error.response?.status === 401) {
      // Handle unauthorized
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);
```

### Typed API Services

```typescript
// types/api.types.ts
export interface ApiResponse<T> {
  data: T;
  message: string;
  status: number;
}

export interface ApiError {
  message: string;
  code: string;
  field?: string;
}

export interface PaginatedResponse<T> {
  items: T[];
  total: number;
  page: number;
  pageSize: number;
  totalPages: number;
}

// features/users/types/user.types.ts
export interface User {
  id: string;
  email: string;
  name: string;
  role: 'admin' | 'user' | 'guest';
  createdAt: string;
  updatedAt: string;
}

export interface CreateUserRequest {
  email: string;
  name: string;
  password: string;
  role: User['role'];
}

export interface UpdateUserRequest {
  name?: string;
  role?: User['role'];
}

export interface UserListFilters {
  search?: string;
  role?: User['role'];
  page?: number;
  pageSize?: number;
}

// features/users/services/usersApi.ts
import { apiClient } from '@/lib/axios';
import type {
  ApiResponse,
  PaginatedResponse,
} from '@/types/api.types';
import type {
  User,
  CreateUserRequest,
  UpdateUserRequest,
  UserListFilters,
} from '../types/user.types';

export const usersApi = {
  // Get all users with filters
  getUsers: async (
    filters: UserListFilters = {}
  ): Promise<PaginatedResponse<User>> => {
    return apiClient.get('/users', { params: filters });
  },

  // Get single user by ID
  getUser: async (id: string): Promise<User> => {
    return apiClient.get(`/users/${id}`);
  },

  // Create new user
  createUser: async (
    data: CreateUserRequest
  ): Promise<User> => {
    return apiClient.post('/users', data);
  },

  // Update user
  updateUser: async (
    id: string,
    data: UpdateUserRequest
  ): Promise<User> => {
    return apiClient.patch(`/users/${id}`, data);
  },

  // Delete user
  deleteUser: async (id: string): Promise<void> => {
    return apiClient.delete(`/users/${id}`);
  },
};
```

### React Query Integration

```typescript
// features/users/hooks/useUsers.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { usersApi } from '../services/usersApi';
import type { UserListFilters } from '../types/user.types';

export function useUsers(filters: UserListFilters = {}) {
  return useQuery({
    queryKey: ['users', filters],
    queryFn: () => usersApi.getUsers(filters),
  });
}

export function useUser(id: string) {
  return useQuery({
    queryKey: ['users', id],
    queryFn: () => usersApi.getUser(id),
    enabled: !!id,
  });
}

// features/users/hooks/useUserMutations.ts
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { usersApi } from '../services/usersApi';
import type { CreateUserRequest, UpdateUserRequest } from '../types/user.types';

export function useCreateUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (data: CreateUserRequest) => usersApi.createUser(data),
    onSuccess: () => {
      // Invalidate and refetch users list
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
  });
}

export function useUpdateUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ id, data }: { id: string; data: UpdateUserRequest }) =>
      usersApi.updateUser(id, data),
    onSuccess: (_, variables) => {
      // Invalidate specific user and list
      queryClient.invalidateQueries({ queryKey: ['users'] });
      queryClient.invalidateQueries({ queryKey: ['users', variables.id] });
    },
  });
}

export function useDeleteUser() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (id: string) => usersApi.deleteUser(id),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
  });
}
```

## Form Patterns

### Type-Safe Form Hook

```typescript
// hooks/useForm.ts
import { useState, useCallback } from 'react';

export interface FormErrors<T> extends Partial<Record<keyof T, string>> {}

export interface UseFormReturn<T> {
  values: T;
  errors: FormErrors<T>;
  touched: Partial<Record<keyof T, boolean>>;
  handleChange: (field: keyof T) => (value: string) => void;
  handleBlur: (field: keyof T) => () => void;
  handleSubmit: (
    onSubmit: (values: T) => void | Promise<void>
  ) => (e: React.FormEvent) => Promise<void>;
  reset: () => void;
  setErrors: (errors: FormErrors<T>) => void;
  setValue: (field: keyof T, value: any) => void;
}

export function useForm<T extends Record<string, any>>(
  initialValues: T,
  validate?: (values: T) => FormErrors<T>
): UseFormReturn<T> {
  const [values, setValues] = useState<T>(initialValues);
  const [errors, setErrors] = useState<FormErrors<T>>({});
  const [touched, setTouched] = useState<Partial<Record<keyof T, boolean>>>({});

  const handleChange = useCallback(
    (field: keyof T) => (value: string) => {
      setValues((prev) => ({ ...prev, [field]: value }));
      // Clear error when user starts typing
      if (errors[field]) {
        setErrors((prev) => ({ ...prev, [field]: undefined }));
      }
    },
    [errors]
  );

  const handleBlur = useCallback(
    (field: keyof T) => () => {
      setTouched((prev) => ({ ...prev, [field]: true }));
      // Validate on blur if validation function provided
      if (validate) {
        const validationErrors = validate(values);
        setErrors(validationErrors);
      }
    },
    [validate, values]
  );

  const handleSubmit = useCallback(
    async (onSubmit: (values: T) => void | Promise<void>) => async (
      e: React.FormEvent
    ) => {
      e.preventDefault();

      // Validate all fields
      if (validate) {
        const validationErrors = validate(values);
        if (Object.keys(validationErrors).length > 0) {
          setErrors(validationErrors);
          setTouched(
            Object.keys(values).reduce((acc, key) => ({ ...acc, [key]: true }), {})
          );
          return;
        }
      }

      await onSubmit(values);
    },
    [validate, values]
  );

  const reset = useCallback(() => {
    setValues(initialValues);
    setErrors({});
    setTouched({});
  }, [initialValues]);

  const setValue = useCallback((field: keyof T, value: any) => {
    setValues((prev) => ({ ...prev, [field]: value }));
  }, []);

  return {
    values,
    errors,
    touched,
    handleChange,
    handleBlur,
    handleSubmit,
    reset,
    setErrors,
    setValue,
  };
}
```

### Form Validation Schema

```typescript
// utils/validation.ts
export interface ValidationRule {
  required?: boolean;
  minLength?: number;
  maxLength?: number;
  pattern?: RegExp;
  custom?: (value: string) => string | undefined;
}

export interface ValidationSchema<T> {
  [K in keyof T]?: ValidationRule;
}

export function validateField<T>(
  value: string,
  rules?: ValidationRule
): string | undefined {
  if (!rules) return undefined;

  if (rules.required && !value) {
    return 'This field is required';
  }

  if (rules.minLength && value.length < rules.minLength) {
    return `Minimum length is ${rules.minLength}`;
  }

  if (rules.maxLength && value.length > rules.maxLength) {
    return `Maximum length is ${rules.maxLength}`;
  }

  if (rules.pattern && !rules.pattern.test(value)) {
    return 'Invalid format';
  }

  if (rules.custom) {
    return rules.custom(value);
  }

  return undefined;
}

export function validateValues<T extends Record<string, any>>(
  values: T,
  schema: ValidationSchema<T>
): FormErrors<T> {
  const errors: FormErrors<T> = {};

  for (const field in schema) {
    const rules = schema[field];
    const value = values[field];
    const error = validateField(value, rules);
    if (error) {
      errors[field] = error;
    }
  }

  return errors;
}

// Usage
import { validateValues, ValidationSchema } from '@/utils/validation';

interface LoginFormData {
  email: string;
  password: string;
}

const loginSchema: ValidationSchema<LoginFormData> = {
  email: {
    required: true,
    pattern: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
    custom: (value) => {
      if (value.endsWith('@example.com')) {
        return 'Example emails are not allowed';
      }
    },
  },
  password: {
    required: true,
    minLength: 8,
    pattern: /^(?=.*[A-Za-z])(?=.*\d)[A-Za-z\d@$!%*#?&]/,
  },
};

function LoginForm() {
  const { values, errors, handleChange, handleSubmit } = useForm<LoginFormData>(
    { email: '', password: '' },
    (values) => validateValues(values, loginSchema)
  );

  return (
    <form onSubmit={handleSubmit(handleLogin)}>
      <input
        value={values.email}
        onChange={(e) => handleChange('email')(e.target.value)}
      />
      {errors.email && <span>{errors.email}</span>}
    </form>
  );
}
```

## Context Patterns

### Type-Safe Context Factory

```typescript
// utils/createContext.ts
import { createContext, useContext } from 'react';

export function createSafeContext<T>(errorMessage: string) {
  const Context = createContext<T | undefined>(undefined);

  const useSafeContext = (): T => {
    const context = useContext(Context);
    if (context === undefined) {
      throw new Error(errorMessage);
    }
    return context;
  };

  return [Context.Provider, useSafeContext] as const;
}

// contexts/AuthContext.tsx
import { createSafeContext } from '@/utils/createContext';

interface AuthContextValue {
  user: User | null;
  token: string | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  isAuthenticated: boolean;
}

const [AuthProvider, useAuth] = createSafeContext<AuthContextValue>(
  'useAuth must be used within AuthProvider'
);

export { AuthProvider, useAuth };
```

## Component Patterns

### Type-Safe Polymorphic Components

```typescript
// components/ui/Button/types.ts
import { ComponentProps } from 'react';

export type ButtonVariants = 'primary' | 'secondary' | 'ghost';
export type ButtonSizes = 'sm' | 'md' | 'lg';

export interface ButtonOwnProps {
  variant?: ButtonVariants;
  size?: ButtonSizes;
  isLoading?: boolean;
  leftIcon?: React.ReactNode;
  rightIcon?: React.ReactNode;
  children: React.ReactNode;
}

export type ButtonProps<E extends React.ElementType> = ButtonOwnProps &
  Omit<ComponentProps<E>, keyof ButtonOwnProps> & {
    as?: E;
  };

// components/ui/Button/Button.tsx
export function Button<E extends React.ElementType = 'button'>({
  variant = 'primary',
  size = 'md',
  isLoading = false,
  leftIcon,
  rightIcon,
  children,
  as,
  ...props
}: ButtonProps<E>) {
  const Component = as || 'button';

  return (
    <ChakraButton
      as={Component}
      variant={variant}
      size={size}
      isLoading={isLoading}
      leftIcon={leftIcon}
      rightIcon={rightIcon}
      {...props}
    >
      {children}
    </ChakraButton>
  );
}
```

### Generic Table Component

```typescript
// components/ui/Table/Table.tsx
interface ColumnDef<T> {
  id: keyof T | string;
  header: string;
  cell?: (row: T) => React.ReactNode;
  sortable?: boolean;
}

interface TableProps<T> {
  data: T[];
  columns: ColumnDef<T>[];
  onRowClick?: (row: T) => void;
  isLoading?: boolean;
}

export function Table<T>({
  data,
  columns,
  onRowClick,
  isLoading,
}: TableProps<T>) {
  if (isLoading) {
    return <TableSkeleton />;
  }

  return (
    <ChakraTable>
      <Thead>
        <Tr>
          {columns.map((col) => (
            <Th key={String(col.id)}>{col.header}</Th>
          ))}
        </Tr>
      </Thead>
      <Tbody>
        {data.map((row, index) => (
          <Tr
            key={index}
            onClick={() => onRowClick?.(row)}
            cursor={onRowClick ? 'pointer' : 'default'}
          >
            {columns.map((col) => (
              <Td key={String(col.id)}>
                {col.cell ? col.cell(row) : String(row[col.id as keyof T] ?? '')}
              </Td>
            ))}
          </Tr>
        ))}
      </Tbody>
    </ChakraTable>
  );
}

// Usage
interface User {
  id: string;
  name: string;
  email: string;
  role: string;
}

const columns: ColumnDef<User>[] = [
  { id: 'name', header: 'Name' },
  { id: 'email', header: 'Email' },
  {
    id: 'role',
    header: 'Role',
    cell: (user) => <Badge>{user.role}</Badge>,
  },
];

<Table data={users} columns={columns} onRowClick={(user) => navigate(user.id)} />
```

## Utility Types

```typescript
// types/utils.ts

// Make specific props optional
type Optional<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;

// Component props with HTML attributes
type PropsWithHTMLAttributes<T, P> = P & React.HTMLAttributes<T>;

// Async operation state
type AsyncState<T, E = Error> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: E };

// Extract component props
type ComponentProps<T> = T extends React.ComponentType<infer P> ? P : never;

// Omit children from props
type PropsWithoutChildren<P> = Omit<P, 'children'>;

// Union to intersection
type UnionToIntersection<U> = (U extends any ? (k: U) => void : never) extends (
  k: infer I
) => void
  ? I
  : never;

// Deep readonly
type DeepReadonly<T> = {
  readonly [P in keyof T]: DeepReadonly<T[P]>;
};

// Exact types (no extra properties)
type Exact<T, Shape> = T & Record<Exclude<keyof T, keyof Shape>, never>;
```

## Path Aliases Configuration

### TypeScript Configuration

```json
// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"],
      "@/components/*": ["./src/components/*"],
      "@/features/*": ["./src/features/*"],
      "@/hooks/*": ["./src/hooks/*"],
      "@/utils/*": ["./src/utils/*"],
      "@/types/*": ["./src/types/*"],
      "@/lib/*": ["./src/lib/*"]
    }
  }
}
```

### Vite Configuration

```typescript
// vite.config.ts
import path from 'path';

export default defineConfig({
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
      '@/components': path.resolve(__dirname, './src/components'),
      '@/features': path.resolve(__dirname, './src/features'),
      '@/hooks': path.resolve(__dirname, './src/hooks'),
      '@/utils': path.resolve(__dirname, './src/utils'),
      '@/types': path.resolve(__dirname, './src/types'),
      '@/lib': path.resolve(__dirname, './src/lib'),
    },
  },
});
```

## Best Practices

1. **Use absolute imports**: Avoid relative paths like `../../../components`
2. **Barrel exports**: Create `index.ts` files for clean imports
3. **Type-first development**: Define types before implementations
4. **Feature-based structure**: Group by feature, not by type
5. **Colocate related code**: Keep types, components, and hooks together
6. **Strict type checking**: Enable strict mode in tsconfig
7. **Avoid any**: Use `unknown` if type is truly unknown
8. **Use discriminated unions**: Better than optional properties
9. **Extract types**: Don't inline complex types
10. **Use generics**: For reusable components and utilities

## Common Patterns

### Type Guards

```typescript
function isUser(value: unknown): value is User {
  return (
    typeof value === 'object' &&
    value !== null &&
    'id' in value &&
    'email' in value
  );
}

function getUser(value: unknown): User {
  if (isUser(value)) {
    return value;
  }
  throw new Error('Invalid user');
}
```

### Branded Types

```typescript
type UserId = string & { readonly __brand: unique symbol };
type Email = string & { readonly __brand: unique symbol };

function createUserId(id: string): UserId {
  return id as UserId;
}

function createUser(email: Email): User {
  return { id: createUserId(crypto.randomUUID()), email };
}
```

### Discriminated Unions

```typescript
type ApiResponse =
  | { success: true; data: User }
  | { success: false; error: string };

function handleResponse(response: ApiResponse) {
  if (response.success) {
    return response.data; // Type is User
  }
  return response.error; // Type is string
}
```

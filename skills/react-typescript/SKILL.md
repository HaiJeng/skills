---
name: react-typescript
description: React development with TypeScript best practices, component typing, props interfaces, and type-safe patterns. Use when building React components, hooks, or applications with TypeScript.
triggers:
- react
- typescript
- ".tsx"
- ".ts"
- "react component"
- "use client"
- "use server"
---

# React + TypeScript Development Guide

## Project Structure

Standard React + TypeScript project layout:

```
my-react-app/
├── src/
│   ├── components/           # Reusable UI components
│   │   ├── ui/              # Basic UI components (Button, Input, etc.)
│   │   ├── layout/          # Layout components (Header, Footer, Sidebar)
│   │   └── features/        # Feature-specific components
│   ├── hooks/               # Custom React hooks
│   ├── types/               # TypeScript type definitions
│   │   ├── api.ts          # API response types
│   │   ├── components.ts    # Component prop types
│   │   └── index.ts        # Type exports
│   ├── utils/               # Utility functions
│   ├── lib/                # Third-party library configurations
│   ├── services/            # API services and data fetching
│   ├── contexts/            # React contexts
│   ├── pages/               # Page components (if using file-based routing)
│   ├── App.tsx             # Root component
│   └── main.tsx            # Application entry point
├── public/                  # Static assets
├── tsconfig.json           # TypeScript configuration
├── vite.config.ts          # Vite configuration
└── package.json
```

## Component Type Definitions

### Functional Component Props

```typescript
// Good: Define interface for props
interface ButtonProps {
  children: React.ReactNode;
  onClick?: () => void;
  disabled?: boolean;
  variant?: 'primary' | 'secondary' | 'ghost';
  size?: 'sm' | 'md' | 'lg';
  className?: string;
}

// Good: Use React.FC (or function directly)
export const Button: React.FC<ButtonProps> = ({
  children,
  onClick,
  disabled = false,
  variant = 'primary',
  size = 'md',
  className,
}) => {
  return (
    <button
      onClick={onClick}
      disabled={disabled}
      className={`btn btn-${variant} btn-${size} ${className || ''}`}
    >
      {children}
    </button>
  );
};

// Alternative: Function declaration (preferred in modern React)
export function Button({
  children,
  onClick,
  disabled = false,
  variant = 'primary',
  size = 'md',
  className,
}: ButtonProps) {
  return (
    <button
      onClick={onClick}
      disabled={disabled}
      className={`btn btn-${variant} btn-${size} ${className || ''}`}
    >
      {children}
    </button>
  );
}
```

### Advanced Props Patterns

```typescript
// Polymorphic components (as prop)
import { ComponentProps } from 'react';

type AsProp<T extends React.ElementType> = {
  as?: T;
};

type PropsToOmit<T extends React.ElementType, P> = Omit<
  ComponentProps<T>,
  keyof P
>;

type PolymorphicComponentProps<
  T extends React.ElementType,
  P = {}
> = AsProp<T> & P & PropsToOmit<T, P>;

function Button<T extends React.ElementType = 'button'>({
  as,
  ...props
}: PolymorphicComponentProps<T, ButtonProps>) {
  const Component = as || 'button';
  return <Component {...props} />;
}

// Usage
<Button as="a" href="/home">Click me</Button>
<Button onClick={handleClick}>Click me</Button>

// Children with render props
interface ListProps {
  items: string[];
  renderItem: (item: string, index: number) => React.ReactNode;
}

export function List({ items, renderItem }: ListProps) {
  return (
    <ul>
      {items.map((item, index) => (
        <li key={index}>{renderItem(item, index)}</li>
      ))}
    </ul>
  );
}

// Discriminated unions for component variants
type BaseButtonProps = {
  children: React.ReactNode;
};

type PrimaryButton = BaseButtonProps & {
  variant: 'primary';
  onClick: () => void;
};

type SecondaryButton = BaseButtonProps & {
  variant: 'secondary';
  href: string;
};

type ButtonProps = PrimaryButton | SecondaryButton;

function Button(props: ButtonProps) {
  if (props.variant === 'primary') {
    return <button onClick={props.onClick}>{props.children}</button>;
  }
  return <a href={props.href}>{props.children}</a>;
}
```

## Type-Safe Event Handlers

```typescript
// Form events
interface FormProps {
  onSubmit: (data: { email: string; password: string }) => void;
}

function LoginForm({ onSubmit }: FormProps) {
  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    const formData = new FormData(e.currentTarget);
    onSubmit({
      email: formData.get('email') as string,
      password: formData.get('password') as string,
    });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input name="email" type="email" />
      <input name="password" type="password" />
      <button type="submit">Submit</button>
    </form>
  );
}

// Input events
interface InputProps {
  value: string;
  onChange: (value: string) => void;
  placeholder?: string;
}

function TextInput({ value, onChange, placeholder }: InputProps) {
  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    onChange(e.target.value);
  };

  return (
    <input
      type="text"
      value={value}
      onChange={handleChange}
      placeholder={placeholder}
    />
  );
}

// Click events
interface CardProps {
  onClick: (id: string) => void;
  id: string;
  title: string;
}

function Card({ onClick, id, title }: CardProps) {
  const handleClick = () => {
    onClick(id);
  };

  return <div onClick={handleClick}>{title}</div>;
}
```

## Custom Hooks with TypeScript

```typescript
// Good: Generic hook
function useLocalStorage<T>(key: string, initialValue: T) {
  const [storedValue, setStoredValue] = useState<T>(initialValue);

  const setValue = (value: T | ((val: T) => T)) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error(error);
    }
  };

  return [storedValue, setValue] as const;
}

// Usage
const [name, setName] = useLocalStorage('name', 'John');

// Good: API data fetching hook
interface UseFetchResult<T> {
  data: T | null;
  loading: boolean;
  error: Error | null;
  refetch: () => void;
}

function useFetch<T>(url: string): UseFetchResult<T> {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  const fetchData = async () => {
    setLoading(true);
    setError(null);
    try {
      const response = await fetch(url);
      const json = await response.json();
      setData(json);
    } catch (err) {
      setError(err as Error);
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    fetchData();
  }, [url]);

  return { data, loading, error, refetch: fetchData };
}

// Usage
interface User {
  id: number;
  name: string;
  email: string;
}

const { data: user, loading, error } = useFetch<User>('/api/user/1');

// Good: Form hook
interface UseFormReturn<T> {
  values: T;
  handleChange: (field: keyof T) => (value: string) => void;
  handleSubmit: (onSubmit: (values: T) => void) => (e: React.FormEvent) => void;
  reset: () => void;
}

function useForm<T extends Record<string, any>>(initialValues: T): UseFormReturn<T> {
  const [values, setValues] = useState<T>(initialValues);

  const handleChange = (field: keyof T) => (value: string) => {
    setValues(prev => ({ ...prev, [field]: value }));
  };

  const handleSubmit = (onSubmit: (values: T) => void) => (e: React.FormEvent) => {
    e.preventDefault();
    onSubmit(values);
  };

  const reset = () => setValues(initialValues);

  return { values, handleChange, handleSubmit, reset };
}

// Usage
interface LoginFormData {
  email: string;
  password: string;
}

const { values, handleChange, handleSubmit } = useForm<LoginFormData>({
  email: '',
  password: '',
});
```

## Context with TypeScript

```typescript
// Good: Type-safe context
interface AuthContextType {
  user: User | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  loading: boolean;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // Check auth status
    checkAuth().then(setUser).finally(() => setLoading(false));
  }, []);

  const login = async (email: string, password: string) => {
    const user = await apiLogin(email, password);
    setUser(user);
  };

  const logout = () => {
    setUser(null);
  };

  return (
    <AuthContext.Provider value={{ user, login, logout, loading }}>
      {children}
    </AuthContext.Provider>
  );
}

// Custom hook to use context
export function useAuth() {
  const context = useContext(AuthContext);
  if (context === undefined) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
}

// Usage in component
function Profile() {
  const { user, logout } = useAuth();

  if (!user) return <div>Please login</div>;

  return (
    <div>
      <h1>Welcome, {user.name}</h1>
      <button onClick={logout}>Logout</button>
    </div>
  );
}
```

## Type Guards and Narrowing

```typescript
// Discriminated union
interface LoadingState {
  status: 'loading';
}

interface SuccessState {
  status: 'success';
  data: User[];
}

interface ErrorState {
  status: 'error';
  error: Error;
}

type DataState = LoadingState | SuccessState | ErrorState;

function DataList({ state }: { state: DataState }) {
  switch (state.status) {
    case 'loading':
      return <div>Loading...</div>;
    case 'success':
      return (
        <ul>
          {state.data.map(user => (
            <li key={user.id}>{user.name}</li>
          ))}
        </ul>
      );
    case 'error':
      return <div>Error: {state.error.message}</div>;
  }
}

// Type guard
function isValidEmail(email: string): email is `${string}@${string}` {
  return email.includes('@');
}

function EmailInput({ email }: { email: string }) {
  if (isValidEmail(email)) {
    return <div className="valid">{email}</div>;
  }
  return <div className="invalid">{email}</div>;
}
```

## Generic Components

```typescript
// Generic list component
interface ListProps<T> {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
  keyExtractor: (item: T) => string | number;
}

function List<T>({ items, renderItem, keyExtractor }: ListProps<T>) {
  return (
    <ul>
      {items.map(item => (
        <li key={keyExtractor(item)}>{renderItem(item)}</li>
      ))}
    </ul>
  );
}

// Usage
interface User {
  id: number;
  name: string;
}

<List
  items={users}
  renderItem={user => <span>{user.name}</span>}
  keyExtractor={user => user.id}
/>

// Generic table component
interface TableProps<T> {
  data: T[];
  columns: {
    key: keyof T;
    header: string;
    render?: (value: any, item: T) => React.ReactNode;
  }[];
}

function Table<T>({ data, columns }: TableProps<T>) {
  return (
    <table>
      <thead>
        <tr>
          {columns.map(col => (
            <th key={String(col.key)}>{col.header}</th>
          ))}
        </tr>
      </thead>
      <tbody>
        {data.map((row, i) => (
          <tr key={i}>
            {columns.map(col => (
              <td key={String(col.key)}>
                {col.render ? col.render(row[col.key], row) : String(row[col.key])}
              </td>
            ))}
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

## Utility Types

```typescript
// Make specific props optional
type Optional<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;

interface RequiredProps {
  id: string;
  name: string;
  email: string;
}

type UserFormProps = Optional<RequiredProps, 'email'>;

// Extract props from component
type ButtonProps = React.ComponentProps<typeof Button>;

// Extend with additional props
type PrimaryButtonProps = ButtonProps & {
  variant: 'primary';
};

// Async operation type
type AsyncResult<T, E = Error> =
  | { status: 'pending'; }
  | { status: 'success'; data: T }
  | { status: 'error'; error: E };

// Component props inference
function withLoading<P extends object>(
  Component: React.ComponentType<P>
) {
  return function LoadingComponent(props: P & { loading: boolean }) {
    if (props.loading) return <div>Loading...</div>;
    return <Component {...(props as P)} />;
  };
}
```

## Best Practices

1. **Always type component props**: Define interfaces for all component props
2. **Avoid `any`**: Use `unknown` if type is truly unknown, or generics
3. **Use discriminated unions**: For components with variant props
4. **Type custom hooks**: Return types should be explicit
5. **Use type assertions sparingly**: Only when you're certain of the type
6. **Enable strict mode**: Set `"strict": true` in `tsconfig.json`
7. **Use utility types**: `Partial`, `Pick`, `Omit`, `Record`, etc.
8. **Type events properly**: Use `React.MouseEvent`, `React.ChangeEvent`, etc.
9. **Use generics**: For reusable components and hooks
10. **Avoid prop drilling**: Use context or composition

## TypeScript Configuration

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

## Common Patterns

### Compound Components

```typescript
interface CardContextValue {
  variant: 'primary' | 'secondary';
}

const CardContext = createContext<CardContextValue>({
  variant: 'primary',
});

interface CardProps {
  variant?: 'primary' | 'secondary';
  children: React.ReactNode;
}

function Card({ variant = 'primary', children }: CardProps) {
  return (
    <CardContext.Provider value={{ variant }}>
      <div className={`card card-${variant}`}>{children}</div>
    </CardContext.Provider>
  );
}

function Header({ children }: { children: React.ReactNode }) {
  return <div className="card-header">{children}</div>;
}

function Body({ children }: { children: React.ReactNode }) {
  return <div className="card-body">{children}</div>;
}

Card.Header = Header;
Card.Body = Body;

// Usage
<Card>
  <Card.Header>Title</Card.Header>
  <Card.Body>Content</Card.Body>
</Card>
```

### Render Props

```typescript
interface DataSourceProps<T> {
  fetch: () => Promise<T[]>;
  children: (data: T[], loading: boolean, error: Error | null) => React.ReactNode;
}

function DataSource<T>({ fetch, children }: DataSourceProps<T>) {
  const [data, setData] = useState<T[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    fetch()
      .then(setData)
      .catch(setError)
      .finally(() => setLoading(false));
  }, [fetch]);

  return <>{children(data, loading, error)}</>;
}

// Usage
<DataSource fetch={() => fetch('/api/users').then(r => r.json())}>
  {(users, loading, error) => {
    if (loading) return <div>Loading...</div>;
    if (error) return <div>Error!</div>;
    return <ul>{users.map(u => <li key={u.id}>{u.name}</li>)}</ul>;
  }}
</DataSource>
```

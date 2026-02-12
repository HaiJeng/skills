---
name: react-components
description: React component design patterns, composition strategies, and best practices for building reusable, maintainable UI components. Use when designing component architecture or building component libraries.
triggers:
- "component composition"
- "compound components"
- "render props"
- "higher-order component"
- "component library"
- "reusable component"
---

# React Component Design Patterns

## Component Composition

### Container/Presentational Pattern

```typescript
// Presentational Component (dumb, just renders)
interface UserCardProps {
  user: User;
  onEdit: () => void;
  onDelete: () => void;
}

function UserCard({ user, onEdit, onDelete }: UserCardProps) {
  return (
    <div className="user-card">
      <h3>{user.name}</h3>
      <p>{user.email}</p>
      <button onClick={onEdit}>Edit</button>
      <button onClick={onDelete}>Delete</button>
    </div>
  );
}

// Container Component (smart, handles logic)
interface UserCardContainerProps {
  userId: string;
}

function UserCardContainer({ userId }: UserCardContainerProps) {
  const [user, setUser] = useState<User | null>(null);
  const { delete, update } = useUserMutations();
  
  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, [userId]);
  
  if (!user) return <div>Loading...</div>;
  
  return (
    <UserCard
      user={user}
      onEdit={() => update(userId)}
      onDelete={() => delete(userId)}
    />
  );
}
```

### Compound Components Pattern

```typescript
// Context for sharing state
const TabsContext = createContext<{
  activeTab: string;
  setActiveTab: (tab: string) => void;
} | undefined>(undefined);

function useTabs() {
  const context = useContext(TabsContext);
  if (!context) throw new Error('Must be used within Tabs');
  return context;
}

// Parent component
function Tabs({ children, defaultTab }: { 
  children: React.ReactNode; 
  defaultTab: string; 
}) {
  const [activeTab, setActiveTab] = useState(defaultTab);
  
  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      <div className="tabs">{children}</div>
    </TabsContext.Provider>
  );
}

// Child components
function TabList({ children }: { children: React.ReactNode }) {
  return <div className="tab-list">{children}</div>;
}

function Tab({ children, value }: { 
  children: React.ReactNode; 
  value: string; 
}) {
  const { activeTab, setActiveTab } = useTabs();
  
  return (
    <button
      className={activeTab === value ? 'active' : ''}
      onClick={() => setActiveTab(value)}
    >
      {children}
    </button>
  );
}

function TabPanels({ children }: { children: React.ReactNode }) {
  return <div className="tab-panels">{children}</div>;
}

function TabPanel({ children, value }: { 
  children: React.ReactNode; 
  value: string; 
}) {
  const { activeTab } = useTabs();
  
  if (activeTab !== value) return null;
  
  return <div className="tab-panel">{children}</div>;
}

// Attach to parent
Tabs.List = TabList;
Tabs.Tab = Tab;
Tabs.Panels = TabPanels;
Tabs.Panel = TabPanel;

// Usage
function App() {
  return (
    <Tabs defaultTab="overview">
      <Tabs.List>
        <Tabs.Tab value="overview">Overview</Tabs.Tab>
        <Tabs.Tab value="settings">Settings</Tabs.Tab>
      </Tabs.List>
      
      <Tabs.Panels>
        <Tabs.Panel value="overview">Overview content</Tabs.Panel>
        <Tabs.Panel value="settings">Settings content</Tabs.Panel>
      </Tabs.Panels>
    </Tabs>
  );
}
```

### Render Props Pattern

```typescript
// Mouse position tracker
interface MousePosition {
  x: number;
  y: number;
}

interface MouseTrackerProps {
  render: (position: MousePosition) => React.ReactNode;
}

function MouseTracker({ render }: MouseTrackerProps) {
  const [position, setPosition] = useState<MousePosition>({ x: 0, y: 0 });
  
  useEffect(() => {
    const handleMouseMove = (e: MouseEvent) => {
      setPosition({ x: e.clientX, y: e.clientY });
    };
    
    window.addEventListener('mousemove', handleMouseMove);
    return () => window.removeEventListener('mousemove', handleMouseMove);
  }, []);
  
  return <>{render(position)}</>;
}

// Usage with render prop
function App() {
  return (
    <MouseTracker
      render={({ x, y }) => (
        <div>
          Mouse position: {x}, {y}
        </div>
      )}
    />
  );
}

// Or as children (common pattern)
interface DataSourceProps<T> {
  url: string;
  children: (data: T, loading: boolean, error: Error | null) => React.ReactNode;
}

function DataSource<T>({ url, children }: DataSourceProps<T>) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);
  
  useEffect(() => {
    fetch(url)
      .then(r => r.json())
      .then(data => { setData(data); setLoading(false); })
      .catch(error => { setError(error); setLoading(false); });
  }, [url]);
  
  return <>{children(data!, loading, error)}</>;
}

// Usage
<DataSource<User[]> url="/api/users">
  {(users, loading, error) => {
    if (loading) return <div>Loading...</div>;
    if (error) return <div>Error!</div>;
    return <UserList users={users} />;
  }}
</DataSource>
```

## Higher-Order Components (HOCs)

### Basic HOC Pattern

```typescript
// HOC for loading state
function withLoading<P extends object>(
  WrappedComponent: React.ComponentType<P>
) {
  return function WithLoadingComponent({ 
    loading, 
    ...props 
  }: P & { loading: boolean }) {
    if (loading) {
      return <div>Loading...</div>;
    }
    
    return <WrappedComponent {...(props as P)} />;
  };
}

// Usage
const UserListWithLoading = withLoading(UserList);

<UserListWithLoading users={users} loading={loading} />

// HOC for error handling
function withErrorBoundary<P extends object>(
  WrappedComponent: React.ComponentType<P>
) {
  return class WithErrorBoundary extends React.Component<
    P,
    { hasError: boolean; error: Error | null }
  > {
    constructor(props: P) {
      super(props);
      this.state = { hasError: false, error: null };
    }
    
    static getDerivedStateFromError(error: Error) {
      return { hasError: true, error };
    }
    
    render() {
      if (this.state.hasError) {
        return <div>Error: {this.state.error?.message}</div>;
      }
      
      return <WrappedComponent {...this.props} />;
    }
  };
}

// Composing HOCs
const EnhancedComponent = withErrorBoundary(
  withLoading(UserList)
);
```

## Controlled vs Uncontrolled Components

### Controlled Components

```typescript
// Controlled input
function ControlledInput() {
  const [value, setValue] = useState('');
  
  return (
    <input
      type="text"
      value={value}
      onChange={(e) => setValue(e.target.value)}
    />
  );
}

// Reusable controlled form
interface FormProps {
  values: Record<string, string>;
  onChange: (field: string, value: string) => void;
  onSubmit: () => void;
}

function Form({ values, onChange, onSubmit }: FormProps) {
  return (
    <form onSubmit={(e) => { e.preventDefault(); onSubmit(); }}>
      {Object.entries(values).map(([field, value]) => (
        <input
          key={field}
          name={field}
          value={value}
          onChange={(e) => onChange(field, e.target.value)}
        />
      ))}
      <button type="submit">Submit</button>
    </form>
  );
}
```

### Uncontrolled Components

```typescript
// Uncontrolled input with ref
function UncontrolledInput() {
  const inputRef = useRef<HTMLInputElement>(null);
  
  const handleSubmit = () => {
    console.log(inputRef.current?.value);
  };
  
  return (
    <>
      <input type="text" ref={inputRef} />
      <button onClick={handleSubmit}>Submit</button>
    </>
  );
}

// File input (always uncontrolled)
function FileInput() {
  const fileInputRef = useRef<HTMLInputElement>(null);
  
  const handleUpload = () => {
    const file = fileInputRef.current?.files?.[0];
    if (file) {
      uploadFile(file);
    }
  };
  
  return (
    <>
      <input type="file" ref={fileInputRef} />
      <button onClick={handleUpload}>Upload</button>
    </>
  );
}
```

## Component Composition Patterns

### Layout Components

```typescript
// Layout component
interface LayoutProps {
  header?: React.ReactNode;
  sidebar?: React.ReactNode;
  footer?: React.ReactNode;
  children: React.ReactNode;
}

function Layout({ header, sidebar, footer, children }: LayoutProps) {
  return (
    <div className="layout">
      {header && <header className="header">{header}</header>}
      
      <div className="main">
        {sidebar && <aside className="sidebar">{sidebar}</aside>}
        <main className="content">{children}</main>
      </div>
      
      {footer && <footer className="footer">{footer}</footer>}
    </div>
  );
}

// Usage
<Layout
  header={<Header />}
  sidebar={<Sidebar />}
  footer={<Footer />}
>
  <Dashboard />
</Layout>
```

### Slot Pattern

```typescript
// Component with slots
interface CardProps {
  header?: React.ReactNode;
  content: React.ReactNode;
  footer?: React.ReactNode;
  actions?: React.ReactNode;
}

function Card({ header, content, footer, actions }: CardProps) {
  return (
    <div className="card">
      {header && <div className="card-header">{header}</div>}
      
      <div className="card-content">{content}</div>
      
      {actions && <div className="card-actions">{actions}</div>}
      
      {footer && <div className="card-footer">{footer}</div>}
    </div>
  );
}

// Usage
<Card
  header={<h2>Title</h2>}
  content={<p>Main content here</p>}
  footer={<small>Footer text</small>}
  actions={
    <>
      <button>Cancel</button>
      <button>Save</button>
    </>
  }
/>
```

## Prop Patterns

### Polymorphic Components

```typescript
type AsProp<E extends React.ElementType> = {
  as?: E;
};

type PropsToOmit<E extends React.ElementType, P> = Omit<
  React.ComponentProps<E>,
  keyof P
>;

type PolymorphicComponentProps<
  E extends React.ElementType,
  P = {}
> = AsProp<E> & P & PropsToOmit<E, P>;

type ButtonBaseProps = {
  variant?: 'primary' | 'secondary';
  size?: 'sm' | 'md' | 'lg';
  children: React.ReactNode;
};

function Button<E extends React.ElementType = 'button'>({
  as,
  variant = 'primary',
  size = 'md',
  children,
  ...props
}: PolymorphicComponentProps<E, ButtonBaseProps>) {
  const Component = as || 'button';
  
  return (
    <Component
      className={`btn btn-${variant} btn-${size}`}
      {...props}
    >
      {children}
    </Component>
  );
}

// Usage
<Button>Click me</Button>
<Button as="a" href="/home">Link button</Button>
```

### Generic Components

```typescript
// Generic list
interface ListProps<T> {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
  keyExtractor: (item: T) => string;
}

function List<T>({ items, renderItem, keyExtractor }: ListProps<T>) {
  return (
    <ul>
      {items.map(item => (
        <li key={keyExtractor(item)}>
          {renderItem(item)}
        </li>
      ))}
    </ul>
  );
}

// Usage
interface User {
  id: string;
  name: string;
}

<List<User>
  items={users}
  keyExtractor={(user) => user.id}
  renderItem={(user) => <span>{user.name}</span>}
/>

// Generic select
interface SelectProps<T> {
  options: T[];
  value: T;
  onChange: (value: T) => void;
  getOptionLabel: (option: T) => string;
  getOptionValue: (option: T) => string;
}

function Select<T>({
  options,
  value,
  onChange,
  getOptionLabel,
  getOptionValue,
}: SelectProps<T>) {
  return (
    <select
      value={getOptionValue(value)}
      onChange={(e) => {
        const selected = options.find(
          opt => getOptionValue(opt) === e.target.value
        );
        if (selected) onChange(selected);
      }}
    >
      {options.map(option => (
        <option key={getOptionValue(option)} value={getOptionValue(option)}>
          {getOptionLabel(option)}
        </option>
      ))}
    </select>
  );
}
```

## State Patterns

### Lift State Up

```typescript
// Shared state in parent
function FilterableProductTable({ products }: { products: Product[] }) {
  const [filterText, setFilterText] = useState('');
  const [inStockOnly, setInStockOnly] = useState(false);
  
  const filteredProducts = products.filter(product => {
    const matchesText = product.name.includes(filterText);
    const matchesStock = !inStockOnly || product.stocked;
    return matchesText && matchesStock;
  });
  
  return (
    <div>
      <SearchBar
        filterText={filterText}
        inStockOnly={inStockOnly}
        onFilterTextChange={setFilterText}
        onInStockOnlyChange={setInStockOnly}
      />
      <ProductTable products={filteredProducts} />
    </div>
  );
}
```

### State Reducer Pattern

```typescript
// Reducer for complex state
type State = {
  count: number;
  step: number;
};

type Action =
  | { type: 'increment' }
  | { type: 'decrement' }
  | { type: 'setStep'; payload: number };

const reducer = (state: State, action: Action): State => {
  switch (action.type) {
    case 'increment':
      return { ...state, count: state.count + state.step };
    case 'decrement':
      return { ...state, count: state.count - state.step };
    case 'setStep':
      return { ...state, step: action.payload };
    default:
      return state;
  }
};

function Counter() {
  const [state, dispatch] = useReducer(reducer, {
    count: 0,
    step: 1,
  });
  
  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>
        +{state.step}
      </button>
      <button onClick={() => dispatch({ type: 'decrement' })}>
        -{state.step}
      </button>
    </div>
  );
}
```

## Best Practices

1. **Keep components small**: One responsibility per component
2. **Use composition over inheritance**: React favors composition
3. **Lift state up**: Share state by moving it to common ancestor
4. **Make components reusable**: Avoid hardcoding values
5. **Use TypeScript**: Define prop interfaces for all components
6. **Memoize expensive operations**: Use React.memo, useMemo, useCallback
7. **Avoid prop drilling**: Use context or composition
8. **Follow naming conventions**: Clear, descriptive names
9. **Document complex props**: Use JSDoc comments
10. **Test components**: Unit tests with @testing-library/react

## Component Testing

```typescript
import { render, screen, fireEvent } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

describe('Button', () => {
  it('renders with text', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByText('Click me')).toBeInTheDocument();
  });
  
  it('calls onClick when clicked', () => {
    const handleClick = jest.fn();
    render(<Button onClick={handleClick}>Click me</Button>);
    
    fireEvent.click(screen.getByText('Click me'));
    
    expect(handleClick).toHaveBeenCalledTimes(1);
  });
  
  it('is disabled when disabled prop is true', () => {
    render(<Button disabled>Click me</Button>);
    
    expect(screen.getByText('Click me')).toBeDisabled();
  });
});
```

## Performance Optimization

### React.memo

```typescript
// Memoize component to prevent unnecessary re-renders
const ExpensiveComponent = React.memo(({ data }: { data: Data }) => {
  // Expensive rendering
  return <div>{/* ... */}</div>;
});

// With custom comparison
const UserCard = React.memo(
  ({ user }: { user: User }) => {
    return <div>{user.name}</div>;
  },
  (prevProps, nextProps) => {
    // Return true if props are equal (skip re-render)
    return prevProps.user.id === nextProps.user.id;
  }
);
```

### Code Splitting

```typescript
// Lazy load component
const HeavyComponent = React.lazy(() => import('./HeavyComponent'));

function App() {
  return (
    <React.Suspense fallback={<div>Loading...</div>}>
      <HeavyComponent />
    </React.Suspense>
  );
}

// Dynamic import based on condition
function App() {
  const [showHeavy, setShowHeavy] = useState(false);
  
  return (
    <div>
      <button onClick={() => setShowHeavy(true)}>Load Heavy</button>
      
      {showHeavy && (
        <React.Suspense fallback={<div>Loading...</div>}>
          <HeavyComponent />
        </React.Suspense>
      )}
    </div>
  );
}
```

---
name: react-hooks
description: Custom and built-in React hooks patterns, best practices, and common use cases for React applications. Use when creating custom hooks or using React hooks effectively.
triggers:
- usehook
- "custom hook"
- usestate
- useeffect
- usecontext
- usereducer
- usecallback
- usememo
- useref
---

# React Hooks Guide

## Built-in Hooks

### useState

```typescript
// Basic usage
function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  );
}

// Lazy initialization
function ExpensiveComponent() {
  const [data, setData] = useState(() => {
    const initial = computeExpensiveValue();
    return initial;
  });
  
  return <div>{data}</div>;
}

// Functional updates
function Counter() {
  const [count, setCount] = useState(0);
  
  const increment = () => {
    setCount(prev => prev + 1); // Use previous state
  };
  
  return <button onClick={increment}>Count: {count}</button>;
}

// Object state
interface FormState {
  email: string;
  password: string;
}

function LoginForm() {
  const [form, setForm] = useState<FormState>({
    email: '',
    password: '',
  });
  
  const handleChange = (field: keyof FormState) => (
    e: React.ChangeEvent<HTMLInputElement>
  ) => {
    setForm(prev => ({ ...prev, [field]: e.target.value }));
  };
  
  return (
    <form>
      <input 
        value={form.email}
        onChange={handleChange('email')}
      />
      <input
        value={form.password}
        onChange={handleChange('password')}
      />
    </form>
  );
}
```

### useEffect

```typescript
// Run on every render
function Component() {
  useEffect(() => {
    console.log('I run on every render');
  });
}

// Run on mount (empty dependency array)
function Component() {
  useEffect(() => {
    console.log('I run only once on mount');
    
    return () => {
      console.log('I run on unmount');
    };
  }, []);
}

// Run when value changes
function UserDashboard({ userId }: { userId: string }) {
  useEffect(() => {
    fetchUserData(userId).then(data => setData(data));
  }, [userId]); // Only re-run if userId changes
}

// Cleanup function
function Chat({ userId }: { userId: string }) {
  useEffect(() => {
    const connection = createChatConnection(userId);
    
    // Cleanup on unmount or userId change
    return () => {
      connection.disconnect();
    };
  }, [userId]);
}

// Multiple effects
function UserProfile({ userId }: { userId: string }) {
  useEffect(() => {
    document.title = `User ${userId}`;
  }, [userId]);
  
  useEffect(() => {
    const handleScroll = () => console.log('Scrolled');
    window.addEventListener('scroll', handleScroll);
    
    return () => window.removeEventListener('scroll', handleScroll);
  }, []);
}
```

### useContext

```typescript
// Create context
interface ThemeContextType {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

// Provider component
export function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');
  
  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };
  
  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// Custom hook to consume
export function useTheme() {
  const context = useContext(ThemeContext);
  if (context === undefined) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
}

// Usage in component
function ThemedButton() {
  const { theme, toggleTheme } = useTheme();
  
  return (
    <button onClick={toggleTheme}>
      Current theme: {theme}
    </button>
  );
}
```

### useReducer

```typescript
// Reducer function
type State = {
  count: number;
  step: number;
};

type Action =
  | { type: 'increment' }
  | { type: 'decrement' }
  | { type: 'setStep'; payload: number }
  | { type: 'reset' };

const initialState: State = {
  count: 0,
  step: 1,
};

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'increment':
      return { ...state, count: state.count + state.step };
    case 'decrement':
      return { ...state, count: state.count - state.step };
    case 'setStep':
      return { ...state, step: action.payload };
    case 'reset':
      return initialState;
    default:
      return state;
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);
  
  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>
        +{state.step}
      </button>
      <button onClick={() => dispatch({ type: 'decrement' })}>
        -{state.step}
      </button>
      <button onClick={() => dispatch({ type: 'setStep', payload: 5 })}>
        Set step to 5
      </button>
      <button onClick={() => dispatch({ type: 'reset' })}>
        Reset
      </button>
    </div>
  );
}
```

### useCallback

```typescript
// Memoize callback to prevent unnecessary re-renders
function ParentComponent() {
  const [count, setCount] = useState(0);
  
  // Without useCallback: New function on every render
  const handleClickBad = () => {
    console.log('Clicked');
  };
  
  // With useCallback: Same function reference
  const handleClickGood = useCallback(() => {
    console.log('Clicked');
  }, []); // Empty deps = never changes
  
  // With dependencies
  const logCount = useCallback(() => {
    console.log(`Count is ${count}`);
  }, [count]); // New function when count changes
  
  return (
    <ChildComponent onClick={handleClickGood} />
  );
}

// Passing parameters to memoized callback
function UserList({ users }: { users: User[] }) {
  const handleDelete = useCallback((userId: string) => {
    deleteUser(userId);
  }, []); // Stable reference
  
  return (
    <ul>
      {users.map(user => (
        <UserListItem 
          key={user.id}
          user={user}
          onDelete={handleDelete}
        />
      ))}
    </ul>
  );
}
```

### useMemo

```typescript
// Expensive computation
function ExpensiveList({ items, filter }: { items: Item[]; filter: string }) {
  // Without useMemo: Recalculated on every render
  const filteredBad = items.filter(item => 
    item.name.includes(filter)
  );
  
  // With useMemo: Only recalculated when items or filter changes
  const filteredGood = useMemo(() => 
    items.filter(item => item.name.includes(filter)),
    [items, filter]
  );
  
  return <ul>{filteredGood.map(...)}</ul>;
}

// Complex object
function DataTable({ data }: { data: DataRow[] }) {
  const columns = useMemo(() => [
    { key: 'name', label: 'Name', sortable: true },
    { key: 'age', label: 'Age', sortable: true },
    { key: 'email', label: 'Email', sortable: false },
  ], []);
  
  return <Table data={data} columns={columns} />;
}
```

### useRef

```typescript
// DOM reference
function TextInputWithFocus() {
  const inputRef = useRef<HTMLInputElement>(null);
  
  useEffect(() => {
    inputRef.current?.focus();
  }, []);
  
  return <input ref={inputRef} type="text" />;
}

// Storing previous value
function PreviousValue({ value }: { value: any }) {
  const prevValue = useRef<any>();
  
  useEffect(() => {
    prevValue.current = value;
  }, [value]);
  
  return (
    <div>
      Current: {value}
      Previous: {prevValue.current}
    </div>
  );
}

// Interval/Timeout
function Timer() {
  const [seconds, setSeconds] = useState(0);
  const intervalRef = useRef<NodeJS.Timeout>();
  
  useEffect(() => {
    intervalRef.current = setInterval(() => {
      setSeconds(s => s + 1);
    }, 1000);
    
    return () => {
      clearInterval(intervalRef.current);
    };
  }, []);
  
  return <div>Seconds: {seconds}</div>;
}

// Storing mutable value (doesn't trigger re-render)
function RenderCount() {
  const renderCount = useRef(0);
  
  renderCount.current += 1;
  
  return <div>Renders: {renderCount.current}</div>;
}
```

## Custom Hooks

### Data Fetching Hook

```typescript
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
  
  const fetchData = useCallback(async () => {
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
  }, [url]);
  
  useEffect(() => {
    fetchData();
  }, [fetchData]);
  
  return { data, loading, error, refetch: fetchData };
}

// Usage
function UserProfile({ userId }: { userId: string }) {
  const { data: user, loading, error } = useFetch<User>(
    `/api/users/${userId}`
  );
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  
  return <div>{user?.name}</div>;
}
```

### Form Hook

```typescript
interface UseFormReturn<T> {
  values: T;
  handleChange: (field: keyof T) => (value: string) => void;
  handleSubmit: (onSubmit: (values: T) => void | Promise<void>) => (e: React.FormEvent) => void;
  reset: () => void;
  errors: Record<keyof T, string>;
  setErrors: (errors: Partial<Record<keyof T, string>>) => void;
}

function useForm<T extends Record<string, any>>(
  initialValues: T,
  validate?: (values: T) => Record<keyof T, string>
): UseFormReturn<T> {
  const [values, setValues] = useState<T>(initialValues);
  const [errors, setErrors] = useState<Record<keyof T, string>>({} as any);
  
  const handleChange = (field: keyof T) => (value: string) => {
    setValues(prev => ({ ...prev, [field]: value }));
    // Clear error when user starts typing
    if (errors[field]) {
      setErrors(prev => ({ ...prev, [field]: '' as any }));
    }
  };
  
  const handleSubmit = (onSubmit: (values: T) => void | Promise<void>) => 
    async (e: React.FormEvent) => {
      e.preventDefault();
      
      // Validate if validation function provided
      if (validate) {
        const validationErrors = validate(values);
        if (Object.keys(validationErrors).length > 0) {
          setErrors(validationErrors);
          return;
        }
      }
      
      await onSubmit(values);
    };
  
  const reset = () => setValues(initialValues);
  
  return { values, handleChange, handleSubmit, reset, errors, setErrors };
}

// Usage
interface LoginFormValues {
  email: string;
  password: string;
}

function LoginForm() {
  const { values, handleChange, handleSubmit, errors } = useForm<LoginFormValues>(
    { email: '', password: '' },
    (values) => {
      const errors: any = {};
      if (!values.email) errors.email = 'Email is required';
      if (!values.password) errors.password = 'Password is required';
      return errors;
    }
  );
  
  const onSubmit = async (values: LoginFormValues) => {
    await login(values);
  };
  
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input
        value={values.email}
        onChange={(e) => handleChange('email')(e.target.value)}
      />
      {errors.email && <span>{errors.email}</span>}
      
      <input
        type="password"
        value={values.password}
        onChange={(e) => handleChange('password')(e.target.value)}
      />
      {errors.password && <span>{errors.password}</span>}
      
      <button type="submit">Submit</button>
    </form>
  );
}
```

### Local Storage Hook

```typescript
function useLocalStorage<T>(key: string, initialValue: T) {
  const [storedValue, setStoredValue] = useState<T>(() => {
    if (typeof window === 'undefined') {
      return initialValue;
    }
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(error);
      return initialValue;
    }
  });
  
  const setValue = useCallback((value: T | ((val: T) => T)) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      if (typeof window !== 'undefined') {
        window.localStorage.setItem(key, JSON.stringify(valueToStore));
      }
    } catch (error) {
      console.error(error);
    }
  }, [key, storedValue]);
  
  return [storedValue, setValue] as const;
}

// Usage
function ThemeToggle() {
  const [theme, setTheme] = useLocalStorage<'light' | 'dark'>('theme', 'light');
  
  return (
    <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
      Current: {theme}
    </button>
  );
}
```

### Debounce Hook

```typescript
function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);
  
  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);
    
    return () => {
      clearTimeout(handler);
    };
  }, [value, delay]);
  
  return debouncedValue;
}

// Usage
function SearchInput() {
  const [searchTerm, setSearchTerm] = useState('');
  const debouncedSearchTerm = useDebounce(searchTerm, 500);
  
  useEffect(() => {
    // API call only happens 500ms after user stops typing
    searchProducts(debouncedSearchTerm);
  }, [debouncedSearchTerm]);
  
  return (
    <input
      value={searchTerm}
      onChange={(e) => setSearchTerm(e.target.value)}
      placeholder="Search..."
    />
  );
}
```

### Toggle Hook

```typescript
function useToggle(initialValue: boolean = false): [boolean, () => void, () => void, () => void] {
  const [value, setValue] = useState(initialValue);
  
  const toggle = useCallback(() => setValue(v => !v), []);
  const setTrue = useCallback(() => setValue(true), []);
  const setFalse = useCallback(() => setValue(false), []);
  
  return [value, toggle, setTrue, setFalse];
}

// Usage
function Modal({ content }: { content: string }) {
  const [isOpen, toggle, setIsOpen, setIsClosed] = useToggle();
  
  return (
    <>
      <button onClick={setIsOpen}>Open Modal</button>
      
      {isOpen && (
        <div className="modal">
          <p>{content}</p>
          <button onClick={setIsClosed}>Close</button>
        </div>
      )}
    </>
  );
}
```

### Array Operations Hook

```typescript
function useArray<T>(initialArray: T[]) {
  const [array, setArray] = useState<T[]>(initialArray);
  
  const push = useCallback((element: T) => {
    setArray(prev => [...prev, element]);
  }, []);
  
  const filter = useCallback((callback: (item: T, index: number) => boolean) => {
    setArray(prev => prev.filter(callback));
  }, []);
  
  const update = useCallback((index: number, newElement: T) => {
    setArray(prev => [
      ...prev.slice(0, index),
      newElement,
      ...prev.slice(index + 1),
    ]);
  }, []);
  
  const remove = useCallback((index: number) => {
    setArray(prev => [
      ...prev.slice(0, index),
      ...prev.slice(index + 1),
    ]);
  }, []);
  
  const clear = useCallback(() => setArray([]), []);
  
  return { array, set: setArray, push, filter, update, remove, clear };
}

// Usage
function TodoList() {
  const { array: todos, push, update, remove } = useArray<Todo>([]);
  
  const addTodo = (text: string) => {
    push({ id: Date.now(), text, completed: false });
  };
  
  const toggleTodo = (id: number) => {
    const index = todos.findIndex(t => t.id === id);
    update(index, { ...todos[index], completed: !todos[index].completed });
  };
  
  const deleteTodo = (id: number) => {
    const index = todos.findIndex(t => t.id === id);
    remove(index);
  };
  
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>
          <input
            type="checkbox"
            checked={todo.completed}
            onChange={() => toggleTodo(todo.id)}
          />
          {todo.text}
          <button onClick={() => deleteTodo(todo.id)}>Delete</button>
        </li>
      ))}
    </ul>
  );
}
```

## Best Practices

1. **Always call hooks at the top level**: Never inside loops, conditions, or nested functions
2. **Only call hooks from React functions**: Not from regular JavaScript functions
3. **Custom hooks should start with "use"**: This is the convention and enables ESLint rules
4. **Use useMemo for expensive computations**: Not everything needs memoization
5. **UseCallback for props passed to memoized components**: Don't overuse
6. **Keep custom hooks focused**: Each hook should do one thing well
7. **Extract complex logic into custom hooks**: This improves testability and reusability
8. **Always specify dependencies**: Don't omit from useEffect, useCallback, useMemo
9. **Use TypeScript with hooks**: Type safety catches many bugs
10. **Test custom hooks**: Use @testing-library/react-hooks or render inside a component

## Common Patterns

### useAsync - Async Operation Manager

```typescript
interface AsyncState<T> {
  data: T | null;
  loading: boolean;
  error: Error | null;
}

function useAsync<T>(asyncFunction: () => Promise<T>) {
  const [state, setState] = useState<AsyncState<T>>({
    data: null,
    loading: true,
    error: null,
  });
  
  useEffect(() => {
    asyncFunction()
      .then(data => setState({ data, loading: false, error: null }))
      .catch(error => setState({ data: null, loading: false, error }));
  }, [asyncFunction]);
  
  return state;
}
```

### useMediaQuery - Responsive Queries

```typescript
function useMediaQuery(query: string): boolean {
  const [matches, setMatches] = useState(() => {
    if (typeof window !== 'undefined') {
      return window.matchMedia(query).matches;
    }
    return false;
  });
  
  useEffect(() => {
    const mediaQuery = window.matchMedia(query);
    const handler = (e: MediaQueryListEvent) => setMatches(e.matches);
    
    mediaQuery.addEventListener('change', handler);
    return () => mediaQuery.removeEventListener('change', handler);
  }, [query]);
  
  return matches;
}

// Usage
function ResponsiveComponent() {
  const isMobile = useMediaQuery('(max-width: 768px)');
  
  return isMobile ? <MobileLayout /> : <DesktopLayout />;
}
```

### usePrevious - Track Previous Value

```typescript
function usePrevious<T>(value: T): T | undefined {
  const ref = useRef<T>();
  
  useEffect(() => {
    ref.current = value;
  }, [value]);
  
  return ref.current;
}

// Usage
function Component({ value }: { value: number }) {
  const prevValue = usePrevious(value);
  
  useEffect(() => {
    if (prevValue !== value) {
      console.log(`Value changed from ${prevValue} to ${value}`);
    }
  }, [value, prevValue]);
  
  return <div>Current: {value}, Previous: {prevValue}</div>;
}
```

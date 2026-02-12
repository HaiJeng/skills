# React + TypeScript + Chakra-UI Development Skills

This directory contains comprehensive skills for building modern React applications with TypeScript and Chakra-UI, following best practices and architectural patterns.

## Available Skills

### 1. React + TypeScript (`react-typescript/`)
**Purpose**: React development with TypeScript best practices and type-safe patterns.

**Triggers**: `react`, `typescript`, `.tsx`, `.ts`, `react component`, `use client`

**What it covers**:
- Component type definitions and props interfaces
- Advanced props patterns (polymorphic components, render props, discriminated unions)
- Type-safe event handlers
- Custom hooks with TypeScript
- Context with TypeScript
- Type guards and narrowing
- Generic components
- Utility types
- Best practices and common patterns

**When to use**: Building React components, hooks, or applications with TypeScript.

---

### 2. Chakra-UI (`chakra-ui/`)
**Purpose**: Chakra-UI component library for building accessible and composable React applications.

**Triggers**: `chakra`, `@chakra-ui`, `useToast`, `useDisclosure`, `<Box>`, `<Flex>`, `<Grid>`

**What it covers**:
- **Component priority order**: Project components first, Chakra primitives last
- Layout primitives (Box, Flex, Grid, Stack)
- Common props patterns (responsive values, style props)
- Theme customization and usage
- Modal with hook pattern
- Form handling
- Data display components
- Loading states
- Responsive design
- Toast notifications
- Color mode

**When to use**: Working with Chakra-UI components, theme customization, or building UI with Chakra-UI patterns.

**Key Principle**: **ALWAYS prioritize project-defined components over Chakra primitives**

---

### 3. React Hooks (`react-hooks/`)
**Purpose**: Custom and built-in React hooks patterns and best practices.

**Triggers**: `usehook`, `custom hook`, `usestate`, `useeffect`, `usecontext`, `usereducer`

**What it covers**:
- Built-in hooks (useState, useEffect, useContext, useReducer, useCallback, useMemo, useRef)
- Custom hooks:
  - useFetch - Data fetching
  - useForm - Form handling with validation
  - useLocalStorage - Persistent storage
  - useDebounce - Debounced values
  - useToggle - Boolean toggle
  - useArray - Array operations
  - useAsync - Async operations
  - useMediaQuery - Responsive queries
  - usePrevious - Track previous values
- Best practices and common patterns

**When to use**: Creating custom hooks or using React hooks effectively.

---

### 4. React Components (`react-components/`)
**Purpose**: React component design patterns and composition strategies.

**Triggers**: `component composition`, `compound components`, `render props`, `higher-order component`

**What it covers**:
- Component patterns:
  - Container/Presentational pattern
  - Compound components
  - Render props
  - Higher-Order Components (HOCs)
  - Controlled vs Uncontrolled components
- Composition patterns:
  - Layout components
  - Slot pattern
- Prop patterns:
  - Polymorphic components
  - Generic components
- State patterns:
  - Lift state up
  - State reducer pattern
- Performance optimization (React.memo, code splitting)
- Component testing

**When to use**: Designing component architecture or building component libraries.

---

### 5. TypeScript + React Patterns (`ts-react-patterns/`)
**Purpose**: TypeScript and React architectural patterns for scalable applications.

**Triggers**: `project structure`, `architecture patterns`, `type-safe react`, `feature-based structure`

**What it covers**:
- **Feature-based architecture**: Organize code by features
- Barrel exports pattern: Clean imports
- **Type-safe API layer**:
  - Axios setup with interceptors
  - Typed API services
  - React Query integration
- **Form patterns**:
  - Type-safe form hook
  - Validation schema
- **Context patterns**: Type-safe context factory
- **Component patterns**:
  - Polymorphic components
  - Generic table component
- **Utility types**: Advanced TypeScript patterns
- **Path aliases configuration**: Clean imports
- Best practices

**When to use**: Setting up project architecture or implementing common patterns.

---

## Quick Start

### Project Setup

1. **Install dependencies**:
```bash
npm install @chakra-ui/react @emotion/react @emotion/styled framer-motion
npm install @tanstack/react-query axios
npm install -D @types/react @types/react-dom
```

2. **Configure TypeScript** (`tsconfig.json`):
```json
{
  "compilerOptions": {
    "strict": true,
    "jsx": "react-jsx",
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

3. **Set up Chakra-UI**:
```typescript
// main.tsx
import { ChakraProvider } from '@chakra-ui/react';
import theme from './lib/chakra-theme';

function App() {
  return (
    <ChakraProvider theme={theme}>
      <App />
    </ChakraProvider>
  );
}
```

### Development Workflow

1. **Start with TypeScript skill** (`react-typescript/`)
   - Define component props with interfaces
   - Use proper typing for hooks and context

2. **Build UI with Chakra-UI skill** (`chakra-ui/`)
   - Use project components first (`@/components/ui`)
   - Use Chakra primitives for layout only
   - Follow responsive design patterns

3. **Add state with hooks** (`react-hooks/`)
   - Use built-in hooks appropriately
   - Create custom hooks for reusable logic
   - Follow best practices

4. **Structure components** (`react-components/`)
   - Use composition over inheritance
   - Apply appropriate patterns (compound, render props, etc.)
   - Optimize performance

5. **Follow architecture patterns** (`ts-react-patterns/`)
   - Use feature-based structure
   - Implement type-safe API layer
   - Use barrel exports

## Integration with Development Workflow

These skills are designed to work together:

1. **Setup**: Use `ts-react-patterns` for project structure
2. **Development**: Use `react-typescript` + `chakra-ui` for building UI
3. **State Management**: Use `react-hooks` for logic
4. **Component Design**: Use `react-components` for patterns
5. **Architecture**: Follow `ts-react-patterns` for best practices

## Component Priority Hierarchy

When building UI, follow this priority order:

1. ✅ **Project-specific components** (`@/components/features/...`)
2. ✅ **Shared UI components** (`@/components/ui/...`)
3. ✅ **Layout components** (`@/components/layout/...`)
4. ⚠️ **Chakra primitives** (only for layout, not for buttons/inputs)

## Example: Building a User Management Feature

```typescript
// 1. Define types (react-typescript)
interface User {
  id: string;
  name: string;
  email: string;
  role: 'admin' | 'user';
}

// 2. Create API service (ts-react-patterns)
export const usersApi = {
  getUsers: (): Promise<User[]> => apiClient.get('/users'),
  createUser: (data: CreateUserRequest): Promise<User> => 
    apiClient.post('/users', data),
};

// 3. Create hooks (react-hooks)
export function useUsers() {
  return useQuery({
    queryKey: ['users'],
    queryFn: usersApi.getUsers,
  });
}

// 4. Build components (react-components + chakra-ui)
function UserList() {
  const { data: users, loading } = useUsers();
  
  return (
    <Box p={4}>
      <Heading size="lg" mb={4}>Users</Heading>
      
      <Grid templateColumns="repeat(3, 1fr)" gap={4}>
        {users?.map(user => (
          <UserCard key={user.id} user={user} />
        ))}
      </Grid>
    </Box>
  );
}

// 5. Use project components
import { Button, Input, Modal } from '@/components/ui';
import { useAuth } from '@/features/auth';

function UserCard({ user }: { user: User }) {
  const { isOpen, onOpen, onClose } = useDisclosure();
  
  return (
    <Box p={4} borderWidth="1px" borderRadius="md">
      <Heading size="md">{user.name}</Heading>
      <Text>{user.email}</Text>
      
      {/* Use project Button, not Chakra Button */}
      <Button onClick={onOpen} variant="primary">
        Edit
      </Button>
      
      <Modal isOpen={isOpen} onClose={onClose}>
        {/* Modal content */}
      </Modal>
    </Box>
  );
}
```

## Key Principles

1. **Type Safety First**: Always use TypeScript with proper types
2. **Project Components First**: Use project components over Chakra primitives
3. **Composition Over Inheritance**: Build complex UIs from simple components
4. **Feature-Based Structure**: Organize code by features, not by type
5. **Reusable Logic**: Extract logic into custom hooks
6. **Performance**: Use memoization and code splitting appropriately
7. **Accessibility**: Chakra-UI is accessible by default, maintain this
8. **Responsive Design**: Use Chakra's responsive props
9. **Clean Imports**: Use barrel exports and path aliases
10. **Testing**: Write tests for components and hooks

## Common Patterns Quick Reference

### Form Handling
```typescript
const { values, handleChange, handleSubmit } = useForm<FormData>(
  initialValues,
  validate
);
```

### API Calls
```typescript
const { data, loading, error } = useFetch<User[]>('/api/users');
```

### Modal Management
```typescript
const { isOpen, onOpen, onClose } = useDisclosure();
```

### Toast Notifications
```typescript
const toast = useToast();
toast({ title: 'Success', status: 'success' });
```

### Responsive Design
```typescript
<Box p={[2, 4, 6]} display={{ base: 'none', md: 'block' }}>
```

## Contributing

To add more React/TypeScript/Chakra skills or improve existing ones:

1. Create a new directory under `/skills/`
2. Add a `SKILL.md` file with proper frontmatter
3. Include practical examples and best practices
4. Test the triggers to ensure they work as expected

## Additional Resources

- [React Documentation](https://react.dev/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/)
- [Chakra-UI Documentation](https://chakra-ui.com/)
- [React Query Documentation](https://tanstack.com/query/latest)
- [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/)

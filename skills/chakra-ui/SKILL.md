---
name: chakra-ui
description: Chakra-UI component library for building accessible and composable React applications. Use when working with Chakra-UI components, theme customization, or building UI with Chakra-UI patterns.
triggers:
- chakra
- "@chakra-ui"
- "useToast"
- "useDisclosure"
- <Box>
- <Flex>
- <Grid>
- chakra-ui
---

# Chakra-UI Development Guide

## Core Principles

### 1. Use Project Components First
**CRITICAL**: Always prioritize using project-defined components over Chakra primitives:

```typescript
// ❌ BAD: Using Chakra primitives directly
import { Button, Input } from '@chakra-ui/react';

function Form() {
  return (
    <Box>
      <Input placeholder="Email" />
      <Button>Submit</Button>
    </Box>
  );
}

// ✅ GOOD: Using project components
import { Button, Input } from '@/components/ui';

function Form() {
  return (
    <form>
      <Input placeholder="Email" />
      <Button variant="primary">Submit</Button>
    </form>
  );
}
```

### 2. Component Priority Order

When building UI, follow this priority:

1. **Project-specific components** (`@/components/features/...`)
   - Components built for specific features
   - Example: `UserProfileCard`, `ProductList`, `DashboardSidebar`

2. **Shared UI components** (`@/components/ui/...`)
   - Generic reusable components
   - Example: `Button`, `Input`, `Modal`, `Select`

3. **Layout components** (`@/components/layout/...`)
   - Structural components
   - Example: `Header`, `Footer`, `Sidebar`, `Container`

4. **Chakra primitives** (only when needed)
   - Use only for quick prototyping or one-off layouts
   - Prefer `Box`, `Flex`, `Grid` for layout
   - Avoid creating custom buttons/inputs with primitives

## Common Chakra Components

### Layout Primitives

```typescript
import { Box, Flex, Grid, Stack, HStack, VStack } from '@chakra-ui/react';

// Box - Generic container
function Card({ children }: { children: React.ReactNode }) {
  return (
    <Box p={4} bg="white" borderRadius="md" boxShadow="md">
      {children}
    </Box>
  );
}

// Flex - Flexbox layout
function Navbar() {
  return (
    <Flex
      as="nav"
      align="center"
      justify="space-between"
      padding={4}
      bg="blue.500"
      color="white"
    >
      <Box fontWeight="bold">Logo</Box>
      <HStack spacing={4}>
        <Box>Home</Box>
        <Box>About</Box>
        <Box>Contact</Box>
      </HStack>
    </Flex>
  );
}

// Grid - CSS Grid layout
function PhotoGallery({ images }: { images: string[] }) {
  return (
    <Grid templateColumns="repeat(3, 1fr)" gap={4}>
      {images.map((img, i) => (
        <Box key={i} bgImg={img} height="200px" bgSize="cover" />
      ))}
    </Grid>
  );
}

// Stack - Spacing container (flex column by default)
function Form() {
  return (
    <Stack spacing={4}>
      <Input placeholder="Name" />
      <Input placeholder="Email" />
      <Button>Submit</Button>
    </Stack>
  );
}

// HStack - Horizontal stack
function Tags({ tags }: { tags: string[] }) {
  return (
    <HStack spacing={2}>
      {tags.map(tag => (
        <Tag key={tag}>{tag}</Tag>
      ))}
    </HStack>
  );
}
```

### Common Props Pattern

```typescript
// Responsive values
<Box 
  p={[2, 4, 6]}        // [mobile, tablet, desktop]
  width={['100%', '50%', '33%']}
  fontSize={['sm', 'md', 'lg']}
>
  Responsive content
</Box>

// Style props
<Box
  margin="auto"           // m="auto"
  padding="4"            // p={4}
  backgroundColor="blue.500"  // bg="blue.500"
  color="white"          // color="white"
  borderRadius="md"       // borderRadius="md"
  _hover={{ bg: 'blue.600' }}  // hover state
  _active={{ bg: 'blue.700' }} // active state
  _focus={{ boxShadow: 'outline' }} // focus state
>
  Interactive box
</Box>

// Conditional props
<Box display={{ base: 'none', md: 'block' }}>
  Hidden on mobile, visible on tablet+
</Box>
```

## Theme Customization

### Using Theme Tokens

```typescript
import { useTheme } from '@chakra-ui/react';

function Component() {
  const theme = useTheme();
  
  // Access theme tokens
  const primaryColor = theme.colors.blue[500];
  const space = theme.space[4];
  const borderRadius = theme.radii.md;
  
  return <Box bg={primaryColor}>Themed</Box>;
}
```

### Extending Theme

```typescript
// src/theme/index.ts
import { extendTheme } from '@chakra-ui/react';

const theme = extendTheme({
  colors: {
    brand: {
      50: '#f0f9ff',
      100: '#e0f2fe',
      500: '#0ea5e9',
      900: '#0c4a6e',
    },
  },
  fonts: {
    heading: 'Inter, sans-serif',
    body: 'Inter, sans-serif',
  },
  styles: {
    global: {
      body: {
        bg: 'gray.50',
        color: 'gray.800',
      },
    },
  },
  components: {
    Button: {
      baseStyle: {
        fontWeight: 'bold',
        borderRadius: 'lg',
      },
      variants: {
        primary: {
          bg: 'brand.500',
          color: 'white',
          _hover: {
            bg: 'brand.600',
          },
        },
      },
    },
  },
});

export default theme;
```

## Patterns with Chakra

### Modal with Hook

```typescript
import { useDisclosure } from '@chakra-ui/react';

function DeleteButton({ itemId, onDelete }: { itemId: string; onDelete: (id: string) => void }) {
  const { isOpen, onOpen, onClose } = useDisclosure();
  const cancelRef = useRef<HTMLButtonElement>(null);

  const handleDelete = () => {
    onDelete(itemId);
    onClose();
  };

  return (
    <>
      <IconButton
        icon={<DeleteIcon />}
        aria-label="Delete"
        onClick={onOpen}
        colorScheme="red"
      />

      <AlertDialog
        isOpen={isOpen}
        leastDestructiveRef={cancelRef}
        onClose={onClose}
      >
        <AlertDialogOverlay>
          <AlertDialogContent>
            <AlertDialogHeader fontSize="lg" fontWeight="bold">
              Delete Item
            </AlertDialogHeader>

            <AlertDialogBody>
              Are you sure? You can't undo this action afterwards.
            </AlertDialogBody>

            <AlertDialogFooter>
              <Button ref={cancelRef} onClick={onClose}>
                Cancel
              </Button>
              <Button colorScheme="red" onClick={handleDelete} ml={3}>
                Delete
              </Button>
            </AlertDialogFooter>
          </AlertDialogContent>
        </AlertDialogOverlay>
      </AlertDialog>
    </>
  );
}
```

### Form Handling

```typescript
import {
  FormControl,
  FormLabel,
  FormErrorMessage,
  FormHelperText,
  Input,
  Textarea,
  Checkbox,
  RadioGroup,
  Radio,
} from '@chakra-ui/react';

interface FormErrors {
  email?: string;
  message?: string;
}

function ContactForm() {
  const [errors, setErrors] = useState<FormErrors>({});
  const { toast } = useToast();

  const validate = (values: FormValues): FormErrors => {
    const errors: FormErrors = {};
    
    if (!values.email) {
      errors.email = 'Email is required';
    } else if (!/\S+@\S+\.\S+/.test(values.email)) {
      errors.email = 'Invalid email address';
    }
    
    if (!values.message) {
      errors.message = 'Message is required';
    }
    
    return errors;
  };

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    const validationErrors = validate(values);
    
    if (Object.keys(validationErrors).length > 0) {
      setErrors(validationErrors);
      return;
    }
    
    try {
      await submitForm(values);
      toast({
        title: 'Form submitted',
        status: 'success',
        duration: 3000,
      });
    } catch (error) {
      toast({
        title: 'Error submitting form',
        status: 'error',
        duration: 3000,
      });
    }
  };

  return (
    <Box as="form" onSubmit={handleSubmit} p={4}>
      <VStack spacing={4}>
        <FormControl isInvalid={!!errors.email}>
          <FormLabel>Email</FormLabel>
          <Input type="email" />
          <FormErrorMessage>{errors.email}</FormErrorMessage>
        </FormControl>

        <FormControl isInvalid={!!errors.message}>
          <FormLabel>Message</FormLabel>
          <Textarea rows={4} />
          <FormErrorMessage>{errors.message}</FormErrorMessage>
        </FormControl>

        <Button type="submit" colorScheme="blue" width="full">
          Submit
        </Button>
      </VStack>
    </Box>
  );
}
```

### Data Display Components

```typescript
import {
  Table,
  Thead,
  Tbody,
  Tr,
  Th,
  Td,
  TableCaption,
  Badge,
} from '@chakra-ui/react';

interface User {
  id: number;
  name: string;
  email: string;
  status: 'active' | 'inactive';
}

function UserTable({ users }: { users: User[] }) {
  const getStatusColor = (status: User['status']) => {
    return status === 'active' ? 'green' : 'gray';
  };

  return (
    <Table variant="simple">
      <TableCaption>User management</TableCaption>
      <Thead>
        <Tr>
          <Th>Name</Th>
          <Th>Email</Th>
          <Th>Status</Th>
          <Th>Actions</Th>
        </Tr>
      </Thead>
      <Tbody>
        {users.map(user => (
          <Tr key={user.id}>
            <Td>{user.name}</Td>
            <Td>{user.email}</Td>
            <Td>
              <Badge colorScheme={getStatusColor(user.status)}>
                {user.status}
              </Badge>
            </Td>
            <Td>
              <HStack spacing={2}>
                <IconButton
                  aria-label="Edit"
                  icon={<EditIcon />}
                  size="sm"
                />
                <IconButton
                  aria-label="Delete"
                  icon={<DeleteIcon />}
                  size="sm"
                  colorScheme="red"
                />
              </HStack>
            </Td>
          </Tr>
        ))}
      </Tbody>
    </Table>
  );
}
```

### Loading States

```typescript
import { Spinner, Center, useDisclosure } from '@chakra-ui/react';

function DataLoader({ children, loading }: { 
  children: React.ReactNode; 
  loading: boolean; 
}) {
  if (loading) {
    return (
      <Center height="200px">
        <Spinner
          thickness="4px"
          speed="0.65s"
          emptyColor="gray.200"
          color="blue.500"
          size="xl"
        />
      </Center>
    );
  }

  return <>{children}</>;
}

// Skeleton loading
function UserCardSkeleton() {
  return (
    <Box p={4} borderWidth="1px" borderRadius="lg">
      <Skeleton height="20px" width="60%" mb={4} />
      <Skeleton height="16px" width="40%" mb={2} />
      <Skeleton height="16px" width="80%" />
    </Box>
  );
}

function UserList({ users, loading }: { users?: User[]; loading: boolean }) {
  if (loading) {
    return (
      <VStack spacing={4}>
        <UserCardSkeleton />
        <UserCardSkeleton />
        <UserCardSkeleton />
      </VStack>
    );
  }

  return (
    <VStack spacing={4}>
      {users?.map(user => <UserCard key={user.id} user={user} />)}
    </VStack>
  );
}
```

## Responsive Design

### Breakpoint-Based Layouts

```typescript
// Mobile-first approach
function ResponsiveLayout() {
  return (
    <Flex direction={{ base: 'column', md: 'row' }} gap={4}>
      <Box flex={1}>Sidebar</Box>
      <Box flex={3}>Content</Box>
    </Flex>
  );
}

// Hide/show based on breakpoint
function MobileOnly({ children }: { children: React.ReactNode }) {
  return <Box display={{ base: 'block', md: 'none' }}>{children}</Box>;
}

function DesktopOnly({ children }: { children: React.ReactNode }) {
  return <Box display={{ base: 'none', md: 'block' }}>{children}</Box>;
}
```

## Best Practices

1. **Always use project components first**: Check `@/components/ui` before using Chakra primitives
2. **Use semantic HTML**: Chakra components pass through props, so use `as` prop when needed
3. **Accessibility first**: Chakra is accessible by default, don't break this
4. **Responsive by default**: Use responsive arrays `p={[2, 4, 6]}`
5. **Theme tokens**: Always use theme tokens, never hardcode colors
6. **Type safety**: Use TypeScript with Chakra for full type support
7. **Composition over inheritance**: Build complex UIs from simple components
8. **Performance**: Use `memo` and `useCallback` where appropriate
9. **Testing**: Test with `@testing-library/react` and Chakra's test utils
10. **Consistent spacing**: Use Chakra's scale (0.25, 0.5, 1, 2, 4, 8, etc.)

## Common Patterns

### Card Component

```typescript
interface CardProps {
  title: string;
  description: string;
  imageUrl?: string;
  actions?: React.ReactNode;
}

function Card({ title, description, imageUrl, actions }: CardProps) {
  return (
    <Box
      p={5}
      borderWidth="1px"
      borderRadius="lg"
      overflow="hidden"
      boxShadow="md"
      _hover={{ boxShadow: 'lg', transform: 'translateY(-2px)' }}
      transition="all 0.2s"
    >
      {imageUrl && (
        <Box height="200px" bgImg={imageUrl} bgSize="cover" mb={4} />
      )}
      <Heading size="md" mb={2}>{title}</Heading>
      <Text color="gray.600" mb={4}>{description}</Text>
      {actions && (
        <Flex gap={2}>
          {actions}
        </Flex>
      )}
    </Box>
  );
}
```

### Navigation Pattern

```typescript
function NavLink({ href, children, icon }: NavLinkProps) {
  return (
    <Link
      as={ReactRouterLink}
      to={href}
      p={2}
      borderRadius="md"
      _hover={{ bg: 'gray.100' }}
      _activeLink={{ bg: 'blue.100', color: 'blue.700' }}
    >
      <HStack spacing={2}>
        {icon}
        <span>{children}</span>
      </HStack>
    </Link>
  );
}
```

## Toast Notifications

```typescript
function SuccessButton() {
  const toast = useToast();

  const handleClick = () => {
    try {
      doSomething();
      toast({
        title: 'Success',
        description: 'Operation completed successfully',
        status: 'success',
        duration: 5000,
        isClosable: true,
        position: 'top-right',
      });
    } catch (error) {
      toast({
        title: 'Error',
        description: error.message,
        status: 'error',
        duration: 5000,
        isClosable: true,
      });
    }
  };

  return <Button onClick={handleClick}>Click me</Button>;
}
```

## Color Mode

```typescript
import { useColorMode, useColorModeValue } from '@chakra-ui/react';

function ThemeToggle() {
  const { colorMode, toggleColorMode } = useColorMode();

  return (
    <Button onClick={toggleColorMode}>
      {colorMode === 'light' ? '🌙' : '☀️'}
    </Button>
  );
}

// Use different values based on color mode
function AdaptiveCard() {
  const bg = useColorModeValue('white', 'gray.800');
  const color = useColorModeValue('gray.800', 'white');

  return (
    <Box bg={bg} color={color} p={4}>
      Adaptive content
    </Box>
  );
}
```

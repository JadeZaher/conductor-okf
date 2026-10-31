# JavaScript Styleguide

Standards for writing clean, modern JavaScript code using ES6+ features.

## General Principles

- Use modern ES6+ features
- Enable strict mode or use modules (implicit strict)
- Use `const` by default, `let` when rebinding needed
- Never use `var`

## Naming Conventions

| Element | Convention | Example |
|---------|------------|---------|
| Variables | camelCase | `userName` |
| Functions | camelCase | `getUserById` |
| Classes | PascalCase | `UserService` |
| React Components | PascalCase | `UserProfile` |
| Constants | SCREAMING_SNAKE | `MAX_RETRIES` |
| Private | Leading underscore | `_internalMethod` |
| Files (modules) | kebab-case | `user-service.js` |
| Files (components) | PascalCase | `UserProfile.jsx` |

## Variables and Constants

### const vs let

```javascript
// Use const by default
const user = { name: 'Alice' };
const items = [1, 2, 3];

// Use let only when rebinding is needed
let count = 0;
count += 1;

// Object properties can still change with const
user.name = 'Bob'; // OK
user = {}; // Error
```

## Functions

### Arrow Functions

```javascript
// Use for callbacks and short functions
const items = list.map((item) => item.id);
const double = (n) => n * 2;

// Use function declarations for hoisting needs
function processData(data) {
  return transform(data);
}
```

### Default Parameters

```javascript
function createUser(name, role = 'user', active = true) {
  return { name, role, active };
}
```

### Rest and Spread

```javascript
// Rest parameters
function sum(...numbers) {
  return numbers.reduce((a, b) => a + b, 0);
}

// Spread for arrays
const combined = [...arr1, ...arr2];

// Spread for objects
const updated = { ...user, name: 'New Name' };
```

## Objects and Arrays

### Destructuring

```javascript
// Object destructuring
const { name, email, role = 'user' } = user;

// Array destructuring
const [first, second, ...rest] = items;

// Parameter destructuring
function greet({ name, title }) {
  return `Hello, ${title} ${name}`;
}
```

### Optional Chaining

```javascript
// Safe property access
const city = user?.address?.city;

// Safe method calls
const result = api?.getData?.();

// With nullish coalescing
const name = user?.name ?? 'Anonymous';
```

### Nullish Coalescing

```javascript
// Use ?? for null/undefined checks (not falsy)
const count = input ?? 0;      // 0 if null/undefined
const name = input || 'default'; // 'default' if falsy
```

## Async Patterns

### async/await

```javascript
// Preferred for readability
async function fetchUser(id) {
  try {
    const response = await api.get(`/users/${id}`);
    return response.data;
  } catch (error) {
    console.error('Failed to fetch user:', error);
    throw error;
  }
}

// Parallel execution
const [users, orders] = await Promise.all([
  fetchUsers(),
  fetchOrders()
]);
```

### Error Handling

```javascript
// Wrap async operations
async function processData(data) {
  try {
    const result = await transform(data);
    return { success: true, data: result };
  } catch (error) {
    return { success: false, error: error.message };
  }
}
```

## Modules

### ES Modules

```javascript
// Named exports (preferred)
export function createUser() { ... }
export class UserService { ... }

// Default export (sparingly)
export default function App() { ... }

// Imports
import { createUser, UserService } from './user-service.js';
import App from './App.jsx';
```

### Import Order

```javascript
// 1. Node built-ins
import path from 'path';

// 2. External packages
import express from 'express';
import lodash from 'lodash';

// 3. Internal modules
import { UserService } from './services/user.js';
import { logger } from './utils/logger.js';

// 4. Relative imports
import { config } from './config.js';
```

## Classes

### Class Syntax

```javascript
class UserService {
  #cache = new Map();

  constructor(repository) {
    this.repository = repository;
  }

  async getUser(id) {
    if (this.#cache.has(id)) {
      return this.#cache.get(id);
    }
    const user = await this.repository.find(id);
    this.#cache.set(id, user);
    return user;
  }

  static create(config) {
    return new UserService(new Repository(config));
  }
}
```

## React-Specific

### Functional Components

```javascript
// Preferred: Functional components with hooks
function UserProfile({ user, onUpdate }) {
  const [isEditing, setIsEditing] = useState(false);

  const handleSave = useCallback((data) => {
    onUpdate(data);
    setIsEditing(false);
  }, [onUpdate]);

  return (
    <div className="user-profile">
      {isEditing ? (
        <EditForm user={user} onSave={handleSave} />
      ) : (
        <ProfileDisplay user={user} onEdit={() => setIsEditing(true)} />
      )}
    </div>
  );
}
```

### Hooks Conventions

```javascript
// Custom hooks start with 'use'
function useUser(id) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchUser(id).then(setUser).finally(() => setLoading(false));
  }, [id]);

  return { user, loading };
}
```

## Testing Patterns

### Jest Structure

```javascript
describe('UserService', () => {
  let service;

  beforeEach(() => {
    service = new UserService(mockRepository);
  });

  describe('getUser', () => {
    it('should return user for valid id', async () => {
      const user = await service.getUser('123');
      expect(user).toEqual(expect.objectContaining({ id: '123' }));
    });

    it('should throw for invalid id', async () => {
      await expect(service.getUser('invalid'))
        .rejects
        .toThrow('User not found');
    });
  });
});
```

## Quick Reference

| Aspect | Recommendation |
|--------|----------------|
| Variables | const default, let when needed |
| var | Never use |
| Async | async/await preferred |
| Modules | ES modules (import/export) |
| Exports | Named preferred |
| Formatting | Prettier |
| Linting | ESLint |
| Semicolons | Consistent (with or without) |

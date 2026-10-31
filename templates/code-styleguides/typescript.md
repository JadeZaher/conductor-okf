# TypeScript Styleguide

Standards for writing clean, type-safe TypeScript code.

## General Principles

- Enable strict mode in `tsconfig.json`
- Explicit types for public APIs, inference for internals
- Use `readonly` where applicable
- Prefer immutability

## Naming Conventions

| Element | Convention | Example |
|---------|------------|---------|
| Classes | PascalCase | `UserService` |
| Interfaces | PascalCase | `UserConfig` |
| Types | PascalCase | `ResponseType` |
| Enums | PascalCase | `HttpStatus` |
| Enum members | PascalCase | `HttpStatus.NotFound` |
| Functions | camelCase | `getUserById` |
| Variables | camelCase | `userName` |
| Constants | SCREAMING_SNAKE | `MAX_RETRIES` |
| Private fields | camelCase with `#` or `_` | `#cache`, `_internal` |

## Type Definitions

### Prefer Interfaces for Objects

```typescript
// Good: Interface for object shapes
interface User {
  id: string;
  name: string;
  email: string;
}

// Good: Type for unions, intersections, utilities
type Status = 'active' | 'inactive' | 'pending';
type UserWithRole = User & { role: Role };
```

### Avoid `any`

```typescript
// Bad
function process(data: any): any { ... }

// Good: Use unknown when type is truly unknown
function process(data: unknown): Result { ... }

// Good: Use generics for flexibility
function process<T>(data: T): ProcessedResult<T> { ... }
```

### Generic Constraints

```typescript
// Good: Constrain generics
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
```

## Functions

### Arrow vs Function Declarations

```typescript
// Use arrow functions for callbacks and short functions
const items = list.map((item) => item.id);

// Use function declarations for hoisting or recursion
function factorial(n: number): number {
  return n <= 1 ? 1 : n * factorial(n - 1);
}
```

### Parameter Destructuring

```typescript
// Good: Destructure with types
function createUser({ name, email }: { name: string; email: string }): User {
  return { id: generateId(), name, email };
}

// Better: Use interface
function createUser({ name, email }: CreateUserInput): User {
  return { id: generateId(), name, email };
}
```

### Async/Await

```typescript
// Good: async/await for readability
async function fetchUser(id: string): Promise<User> {
  const response = await api.get(`/users/${id}`);
  return response.data;
}

// Avoid: Mixing async/await with .then()
```

## Error Handling

### Custom Error Classes

```typescript
class ValidationError extends Error {
  constructor(
    message: string,
    public readonly field: string
  ) {
    super(message);
    this.name = 'ValidationError';
  }
}
```

### Result Types Pattern

```typescript
type Result<T, E = Error> =
  | { success: true; data: T }
  | { success: false; error: E };

function parseConfig(input: string): Result<Config> {
  try {
    return { success: true, data: JSON.parse(input) };
  } catch (e) {
    return { success: false, error: e as Error };
  }
}
```

## Module Organization

### Import Order

```typescript
// 1. Node built-ins
import { readFile } from 'fs/promises';
import path from 'path';

// 2. External packages
import express from 'express';
import { z } from 'zod';

// 3. Internal modules (absolute paths)
import { UserService } from '@/services/user';
import { logger } from '@/utils/logger';

// 4. Relative imports
import { validateInput } from './validation';
import type { Config } from './types';
```

### Named vs Default Exports

```typescript
// Prefer named exports for better refactoring
export function createUser() { ... }
export class UserService { ... }

// Use default exports sparingly (e.g., main component)
export default function App() { ... }
```

### Barrel Files

```typescript
// src/services/index.ts
export { UserService } from './user-service';
export { AuthService } from './auth-service';
export type { ServiceConfig } from './types';
```

## Testing Patterns

### Test Structure

```typescript
describe('UserService', () => {
  let service: UserService;

  beforeEach(() => {
    service = new UserService(mockRepository);
  });

  describe('createUser', () => {
    it('should create user with valid input', async () => {
      const result = await service.createUser(validInput);
      expect(result).toMatchObject({ name: validInput.name });
    });

    it('should throw ValidationError for invalid email', async () => {
      await expect(service.createUser(invalidInput))
        .rejects
        .toThrow(ValidationError);
    });
  });
});
```

### Mock Typing

```typescript
import type { Mock } from 'vitest';

const mockFn: Mock<[string], Promise<User>> = vi.fn();
```

## Quick Reference

| Aspect | Recommendation |
|--------|----------------|
| Strict mode | Always enabled |
| any | Never use |
| Interface vs Type | Interface for objects, Type for unions |
| Async | async/await preferred |
| Exports | Named preferred |
| Tests | Describe/it structure |

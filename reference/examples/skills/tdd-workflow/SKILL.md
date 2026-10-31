---
name: tdd-workflow
description: Test-Driven Development guidance. Use when writing code, implementing features, or fixing bugs in projects that follow TDD methodology. Provides the Red-Green-Refactor cycle structure.
---

# TDD Workflow Skill

This skill provides guidance for Test-Driven Development methodology.

## The Core Cycle

```
┌─────────────────────────────────────────────────┐
│                                                 │
│    ┌───────┐     ┌───────┐     ┌──────────┐   │
│    │  RED  │ ──▶ │ GREEN │ ──▶ │ REFACTOR │   │
│    └───────┘     └───────┘     └──────────┘   │
│        │                             │         │
│        └─────────────────────────────┘         │
│                                                 │
└─────────────────────────────────────────────────┘
```

## Phase 1: RED - Write a Failing Test

### Purpose
Define the expected behavior BEFORE writing implementation.

### Actions
1. Create or open test file
2. Write a test that describes ONE behavior
3. Run the test
4. Verify it FAILS (important!)

### Example (TypeScript/Jest)

```typescript
describe('UserService', () => {
  describe('createUser', () => {
    it('should create a user with valid email', async () => {
      const user = await userService.createUser({
        email: 'test@example.com',
        name: 'Test User'
      });

      expect(user.id).toBeDefined();
      expect(user.email).toBe('test@example.com');
    });

    it('should throw error for invalid email', async () => {
      await expect(
        userService.createUser({ email: 'invalid', name: 'Test' })
      ).rejects.toThrow('Invalid email format');
    });
  });
});
```

### Common Mistakes
- Writing tests that pass immediately (means the test is wrong)
- Testing implementation details instead of behavior
- Writing too many tests before any implementation

## Phase 2: GREEN - Make It Pass

### Purpose
Write the MINIMUM code to make the test pass.

### Actions
1. Implement just enough to pass the failing test
2. No extra features
3. No optimization
4. Run tests to verify PASS

### Example

```typescript
// MINIMUM implementation to pass the tests above
class UserService {
  async createUser(data: { email: string; name: string }) {
    if (!data.email.includes('@')) {
      throw new Error('Invalid email format');
    }
    return {
      id: crypto.randomUUID(),
      email: data.email,
      name: data.name
    };
  }
}
```

### Common Mistakes
- Over-engineering on first pass
- Adding features not covered by tests
- "While I'm here" additions

## Phase 3: REFACTOR - Improve Quality

### Purpose
Clean up the code while keeping tests green.

### Actions
1. Look for improvements:
   - Duplication
   - Poor naming
   - Complex logic
   - Long functions
2. Make ONE change
3. Run tests
4. If green, continue. If red, undo.

### Example

```typescript
// REFACTORED version
const EMAIL_REGEX = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

class UserService {
  async createUser(data: CreateUserInput): Promise<User> {
    this.validateEmail(data.email);
    return this.buildUser(data);
  }

  private validateEmail(email: string): void {
    if (!EMAIL_REGEX.test(email)) {
      throw new InvalidEmailError(email);
    }
  }

  private buildUser(data: CreateUserInput): User {
    return {
      id: crypto.randomUUID(),
      ...data,
      createdAt: new Date()
    };
  }
}
```

### Refactoring Checklist
- [ ] Extract long methods
- [ ] Rename unclear variables/functions
- [ ] Remove duplication (DRY)
- [ ] Simplify conditionals
- [ ] Add type safety

## Test Patterns

### Arrange-Act-Assert (AAA)

```typescript
it('should calculate discount correctly', () => {
  // Arrange
  const cart = new Cart();
  cart.addItem({ price: 100 });

  // Act
  const discount = cart.calculateDiscount();

  // Assert
  expect(discount).toBe(10);
});
```

### Given-When-Then (BDD)

```typescript
describe('given a cart with items over $100', () => {
  describe('when calculating discount', () => {
    it('then should apply 10% discount', () => {
      // ...
    });
  });
});
```

## Coverage Commands

### JavaScript/TypeScript

```bash
# Jest
npx jest --coverage

# Vitest
npx vitest run --coverage
```

### Python

```bash
pytest --cov=src --cov-report=html
```

### Go

```bash
go test -cover ./...
go test -coverprofile=coverage.out ./...
```

## Anti-Patterns

### 1. Test After
Writing code first, tests second defeats the purpose.

### 2. Testing Implementation
```typescript
// BAD: Testing HOW it works
expect(service.internalMethod).toHaveBeenCalled();

// GOOD: Testing WHAT it does
expect(result).toEqual(expectedOutput);
```

### 3. Brittle Tests
```typescript
// BAD: Breaks if order changes
expect(users[0].name).toBe('Alice');

// GOOD: Resilient assertion
expect(users).toContainEqual(expect.objectContaining({ name: 'Alice' }));
```

### 4. No Refactoring
Skipping refactor phase leads to technical debt.

## Quick Reference

| Phase | Question to Answer | Action |
|-------|-------------------|--------|
| RED | What should it do? | Write failing test |
| GREEN | Does it work? | Write minimal code |
| REFACTOR | Is it clean? | Improve structure |

## Integration

This skill works with:
- **conductor-context**: For project-specific coverage targets
- **code-styleguides**: For language-specific test patterns

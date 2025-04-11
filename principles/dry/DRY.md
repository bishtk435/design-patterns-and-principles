# DRY - Don't Repeat Yourself

## What is the DRY Principle?

The DRY (Don't Repeat Yourself) principle is a software development principle formulated by Andy Hunt and Dave Thomas in their book "The Pragmatic Programmer." The principle states:

> Every piece of knowledge must have a single, unambiguous, authoritative representation within a system.

In simpler terms, the DRY principle aims to reduce repetition of information of all kinds, especially useful in multi-tier architectures. When the DRY principle is applied successfully, a modification of any single element of a system does not require a change in other logically unrelated elements.

## Why is DRY Important?

Following the DRY principle leads to code that is:

- **Easier to maintain**: When changes are needed, you only need to make them in one place.
- **Less prone to bugs**: With less code duplication, there are fewer places for bugs to hide.
- **More readable**: Code without repetition is generally easier to understand.
- **More efficient**: Reducing duplication can lead to smaller code bases.

## DRY vs. WET Code

The opposite of DRY code is often referred to as WET code, which can stand for:

- "Write Everything Twice"
- "We Enjoy Typing"
- "Waste Everyone's Time"

WET code contains unnecessary duplication, which makes it harder to maintain and more prone to errors.

## Examples of DRY and WET in TypeScript

### Example 1: Function Duplication

#### WET Code (Violating DRY)

```typescript
// A function to validate an email
function validateEmail(email: string): boolean {
  const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return regex.test(email);
}

// Another function in a different module also validating an email
function isEmailValid(userEmail: string): boolean {
  const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return regex.test(userEmail);
}

// Using the functions
const email1 = "user@example.com";
const email2 = "admin@company.org";

console.log(validateEmail(email1)); // In one module
console.log(isEmailValid(email2));  // In another module
```

#### DRY Code

```typescript
// A single validation utility
export function validateEmail(email: string): boolean {
  const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return regex.test(email);
}

// Using the function across different modules
import { validateEmail } from './validation-utils';

const email1 = "user@example.com";
const email2 = "admin@company.org";

console.log(validateEmail(email1));
console.log(validateEmail(email2));
```

### Example 2: Duplicate Logic in Different Classes

#### WET Code (Violating DRY)

```typescript
class UserService {
  getUsers() {
    // Fetch users from database
    const users = this.fetchFromDatabase('users');
    
    // Log access
    console.log(`Data accessed at ${new Date().toISOString()}`);
    
    // Transform data
    return users.map(user => ({
      ...user,
      fullName: `${user.firstName} ${user.lastName}`
    }));
  }
  
  fetchFromDatabase(table: string) {
    // Database access logic
    console.log(`Fetching from ${table} table`);
    return []; // Simplified for example
  }
}

class ProductService {
  getProducts() {
    // Fetch products from database
    const products = this.fetchFromDatabase('products');
    
    // Log access - same logic as in UserService
    console.log(`Data accessed at ${new Date().toISOString()}`);
    
    return products;
  }
  
  fetchFromDatabase(table: string) {
    // Duplicated database access logic
    console.log(`Fetching from ${table} table`);
    return []; // Simplified for example
  }
}
```

#### DRY Code

```typescript
// A base service class with common functionality
abstract class BaseService {
  protected fetchFromDatabase(table: string) {
    console.log(`Fetching from ${table} table`);
    return []; // Simplified for example
  }
  
  protected logAccess() {
    console.log(`Data accessed at ${new Date().toISOString()}`);
  }
}

class UserService extends BaseService {
  getUsers() {
    // Fetch users from database
    const users = this.fetchFromDatabase('users');
    
    // Log access
    this.logAccess();
    
    // Transform data
    return users.map(user => ({
      ...user,
      fullName: `${user.firstName} ${user.lastName}`
    }));
  }
}

class ProductService extends BaseService {
  getProducts() {
    // Fetch products from database
    const products = this.fetchFromDatabase('products');
    
    // Log access
    this.logAccess();
    
    return products;
  }
}
```

### Example 3: Constants and Configuration

#### WET Code (Violating DRY)

```typescript
// In user authentication module
function validatePassword(password: string): boolean {
  if (password.length < 8) {
    console.log("Password must be at least 8 characters");
    return false;
  }
  // ... rest of validation
  return true;
}

// In user registration module
function registerUser(username: string, password: string): boolean {
  if (password.length < 8) {
    console.log("Password must be at least 8 characters");
    return false;
  }
  // ... rest of registration logic
  return true;
}

// In password reset module
function resetPassword(newPassword: string): boolean {
  if (newPassword.length < 8) {
    console.log("Password must be at least 8 characters");
    return false;
  }
  // ... rest of reset logic
  return true;
}
```

#### DRY Code

```typescript
// A constants file
export const CONFIG = {
  PASSWORD: {
    MIN_LENGTH: 8,
    ERROR_MESSAGE: "Password must be at least 8 characters"
  }
};

// In validation utils
import { CONFIG } from './constants';

export function validatePasswordLength(password: string): boolean {
  if (password.length < CONFIG.PASSWORD.MIN_LENGTH) {
    console.log(CONFIG.PASSWORD.ERROR_MESSAGE);
    return false;
  }
  return true;
}

// In user authentication module
import { validatePasswordLength } from './validation-utils';

function validatePassword(password: string): boolean {
  if (!validatePasswordLength(password)) {
    return false;
  }
  // ... rest of validation
  return true;
}

// In user registration module
import { validatePasswordLength } from './validation-utils';

function registerUser(username: string, password: string): boolean {
  if (!validatePasswordLength(password)) {
    return false;
  }
  // ... rest of registration logic
  return true;
}

// In password reset module
import { validatePasswordLength } from './validation-utils';

function resetPassword(newPassword: string): boolean {
  if (!validatePasswordLength(newPassword)) {
    return false;
  }
  // ... rest of reset logic
  return true;
}
```

## Common Misconceptions About DRY

### 1. DRY is Only About Code Duplication

DRY is about the duplication of knowledge, not just code. Two pieces of code might look different but represent the same knowledge. Conversely, two similar-looking pieces of code might actually represent different knowledge.

### 2. All Duplication Must Be Eliminated

Sometimes a small amount of duplication is acceptable for the sake of clarity or decoupling. Over-applying DRY can lead to overly complex abstractions.

### 3. DRY Always Results in Less Code

While DRY often leads to less code overall, the primary goal is to have knowledge represented in a single place, not necessarily to reduce the line count.

## When to Apply DRY

Apply the DRY principle when:

1. You notice the same logic appearing in multiple places.
2. You find yourself fixing the same bug in multiple places.
3. You're copying and pasting code from one part of your application to another.

## When to Be Cautious with DRY

Be cautious about applying DRY when:

1. The duplicated code actually represents different domain concepts that happen to look similar now but might evolve differently.
2. Creating an abstraction would significantly increase complexity or harm readability.
3. The cost of creating and maintaining the abstraction outweighs the benefits.

## Conclusion

The DRY principle is a powerful tool for creating maintainable software. By ensuring that each piece of knowledge has a single, authoritative representation, you can reduce bugs, improve maintainability, and make your codebase more resilient to change. However, like all principles, it should be applied thoughtfully, with an understanding of its benefits and limitations.

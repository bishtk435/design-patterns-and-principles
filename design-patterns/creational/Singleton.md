# Singleton Pattern

## Intent

The Singleton pattern ensures a class has only one instance and provides a global point of access to it.

## Motivation

Sometimes we need to have exactly one instance of a class in the system. For example:
- A single configuration manager
- A single connection to a database
- A single file manager
- A centralized state manager

Having multiple instances could lead to problems like:
- Inconsistent state
- Incorrect behavior
- Resource overuse
- Reduced performance

## Structure

![Singleton Pattern Structure](https://www.plantuml.com/plantuml/png/SoWkIImgAStDuU8goIp9ILLmJyrBBKfAJYtAJ2fDuN98pKi12WE0sm40)

## Implementation in TypeScript

### Basic Implementation

```typescript
class Singleton {
  private static instance: Singleton;

  // Private constructor prevents direct construction calls with the `new` operator
  private constructor() { }

  // The static method that controls access to the singleton instance
  public static getInstance(): Singleton {
    if (!Singleton.instance) {
      Singleton.instance = new Singleton();
    }
    return Singleton.instance;
  }

  // Some business logic
  public someBusinessLogic() {
    // ...
  }
}

// Usage
function clientCode() {
  const s1 = Singleton.getInstance();
  const s2 = Singleton.getInstance();

  if (s1 === s2) {
    console.log('Singleton works, both variables contain the same instance.');
  } else {
    console.log('Singleton failed, variables contain different instances.');
  }
}

clientCode(); // Output: Singleton works, both variables contain the same instance.
```

### Thread-Safe Implementation (Simulated in TypeScript)

Even though JavaScript/TypeScript is single-threaded in most environments, the concept is important to understand:

```typescript
class ThreadSafeSingleton {
  private static instance: ThreadSafeSingleton;
  private static lockObject = {}; // Simulated lock object

  private constructor() {
    // Initialization code
  }

  public static getInstance(): ThreadSafeSingleton {
    // Double-check locking pattern (simulated)
    if (!ThreadSafeSingleton.instance) {
      // In a real multi-threaded environment, this would acquire a lock
      ThreadSafeSingleton.instance = new ThreadSafeSingleton();
      // And release the lock
    }
    return ThreadSafeSingleton.instance;
  }
}
```

### Lazy Initialization

```typescript
class LazySingleton {
  private static instance: LazySingleton;

  private constructor() {
    console.log('Singleton is initializing');
    // Potentially expensive initialization
  }

  public static getInstance(): LazySingleton {
    if (!LazySingleton.instance) {
      LazySingleton.instance = new LazySingleton();
    }
    return LazySingleton.instance;
  }
}

// The singleton is not created until getInstance is called
console.log('Program started');
// ... other code
const instance = LazySingleton.getInstance(); // Only now is the singleton created
```

### Singleton with Configuration

```typescript
interface DatabaseConfig {
  host: string;
  port: number;
  username: string;
  password: string;
}

class DatabaseConnection {
  private static instance: DatabaseConnection;
  private config: DatabaseConfig;
  private isConnected: boolean = false;

  private constructor(config: DatabaseConfig) {
    this.config = config;
    // Initialize the connection
  }

  public static getInstance(config?: DatabaseConfig): DatabaseConnection {
    if (!DatabaseConnection.instance && config) {
      DatabaseConnection.instance = new DatabaseConnection(config);
    } else if (!DatabaseConnection.instance) {
      throw new Error('Database connection not initialized');
    }
    return DatabaseConnection.instance;
  }

  public connect(): void {
    if (!this.isConnected) {
      console.log(`Connecting to database at ${this.config.host}:${this.config.port}`);
      // Connection logic
      this.isConnected = true;
    } else {
      console.log('Already connected to database');
    }
  }

  public query(sql: string): any {
    if (!this.isConnected) {
      throw new Error('Not connected to database');
    }
    console.log(`Executing query: ${sql}`);
    // Query execution logic
    return { results: [] };
  }

  public disconnect(): void {
    if (this.isConnected) {
      console.log('Disconnecting from database');
      // Disconnection logic
      this.isConnected = false;
    }
  }
}

// Usage
function dbClientCode() {
  // Initialize the singleton with config
  const dbConfig: DatabaseConfig = {
    host: 'localhost',
    port: 5432,
    username: 'admin',
    password: 'secret'
  };
  
  const db1 = DatabaseConnection.getInstance(dbConfig);
  db1.connect();
  
  // Later in the code, we can access the same instance without providing config again
  const db2 = DatabaseConnection.getInstance();
  db2.query('SELECT * FROM users');
  
  // Both variables refer to the same instance
  console.log(db1 === db2); // true
  
  db1.disconnect();
}
```

### Modern JavaScript/TypeScript Module-Based Singleton

In modern JavaScript/TypeScript with modules, you can simply export an instance:

```typescript
// logger.ts
class Logger {
  private logs: string[] = [];

  log(message: string): void {
    const timestamp = new Date().toISOString();
    this.logs.push(`${timestamp}: ${message}`);
    console.log(`${timestamp}: ${message}`);
  }

  getLogs(): string[] {
    return [...this.logs];
  }

  clearLogs(): void {
    this.logs = [];
  }
}

// Export a singleton instance
export default new Logger();
```

Usage in other files:

```typescript
// app.ts
import logger from './logger';

logger.log('Application started');

// another-module.ts
import logger from './logger';

logger.log('Another module initialized');
// Both modules use the same logger instance
```

## Considerations in Modern TypeScript Applications

### Dependency Injection

In modern applications, dependency injection frameworks often provide singleton-like behavior:

```typescript
import { Injectable } from '@angular/core';

@Injectable({
  providedIn: 'root' // This makes the service a singleton in Angular
})
export class ConfigService {
  private config = {
    apiUrl: 'https://api.example.com',
    theme: 'dark'
  };

  getConfig() {
    return { ...this.config };
  }

  setTheme(theme: string) {
    this.config.theme = theme;
  }
}
```

### Singleton in React Context API

```typescript
// AppContext.tsx
import React, { createContext, useContext, useState } from 'react';

interface AppState {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
}

const AppContext = createContext<AppState | undefined>(undefined);

export function AppProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');

  const toggleTheme = () => {
    setTheme(theme === 'light' ? 'dark' : 'light');
  };

  return (
    <AppContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </AppContext.Provider>
  );
}

export function useAppContext() {
  const context = useContext(AppContext);
  if (context === undefined) {
    throw new Error('useAppContext must be used within an AppProvider');
  }
  return context;
}
```

## Pros and Cons

### Pros

1. **Controlled Access**: You can be sure that a class has only one instance.
2. **Global Access Point**: The singleton provides a global access point to that instance.
3. **Lazy Initialization**: The singleton object is initialized only when it's requested for the first time.
4. **Single State**: All clients work with the same state.

### Cons

1. **Violates Single Responsibility Principle**: The pattern solves two problems at once.
2. **Concurrency Issues**: In multithreaded environments, special care must be taken.
3. **Global State**: Singletons introduce global state, making testing and debugging more difficult.
4. **Tight Coupling**: Code that depends on singletons is tightly coupled to them.
5. **Difficult Testing**: Singleton instances cannot be easily mocked for testing.

## Real-World Use Cases

1. **Configuration Management**: Storing and providing access to application settings.
2. **Logging Services**: Central point for logging operations.
3. **Connection Pools**: Managing database or network connections.
4. **Caches**: Centralized caching mechanisms.
5. **State Management**: In front-end frameworks like Redux store.

## Related Patterns

- **Factory Method**: Singletons often use factory methods for instance creation.
- **Facade**: Can be implemented as a Singleton when only one facade object is needed.
- **Flyweight**: The Singleton concept is used in the implementation of shared flyweight objects.

## Example: Thread Pool Singleton

```typescript
class ThreadPool {
  private static instance: ThreadPool;
  private maxThreads: number;
  private busyThreads: number = 0;

  private constructor(maxThreads: number) {
    this.maxThreads = maxThreads;
    console.log(`Thread pool initialized with ${maxThreads} threads`);
  }

  public static getInstance(maxThreads: number = 4): ThreadPool {
    if (!ThreadPool.instance) {
      ThreadPool.instance = new ThreadPool(maxThreads);
    }
    return ThreadPool.instance;
  }

  public executeTask(task: () => void): void {
    if (this.busyThreads < this.maxThreads) {
      this.busyThreads++;
      console.log(`Executing task. Busy threads: ${this.busyThreads}/${this.maxThreads}`);
      
      // Simulate task execution
      setTimeout(() => {
        task();
        this.busyThreads--;
        console.log(`Task completed. Busy threads: ${this.busyThreads}/${this.maxThreads}`);
      }, 1000);
    } else {
      console.log('No threads available. Task queued.');
      // In a real implementation, we would queue the task
      setTimeout(() => this.executeTask(task), 500);
    }
  }
}

// Usage
const pool = ThreadPool.getInstance(3);

for (let i = 1; i <= 5; i++) {
  pool.executeTask(() => console.log(`Task ${i} executed`));
}
```

## Conclusion

The Singleton pattern is one of the simplest but most controversial design patterns. While it solves important problems related to ensuring a single instance and providing global access, it introduces challenges related to testing, coupling, and global state.

In modern TypeScript applications, consider whether the singleton behavior could be better achieved through dependency injection, module patterns, or other mechanisms that offer similar benefits without some of the drawbacks of traditional singletons.

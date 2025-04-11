# Single Responsibility Principle (SRP)

## Definition

> A class should have only one reason to change.

The Single Responsibility Principle, formulated by Robert C. Martin, states that a class should have only one job or responsibility. In other words, a class should have only one reason to change.

## Understanding the Principle

The core idea behind SRP is to keep classes focused, making them easier to understand, maintain, and extend. A class with multiple responsibilities is more likely to need changes as requirements evolve, increasing the risk of introducing bugs or breaking existing functionality.

When a class takes on multiple responsibilities, those responsibilities become coupled. Changes to one responsibility might affect the class's ability to fulfill its other responsibilities.

## Benefits of Following SRP

1. **Reduced Complexity**: Classes are simpler and easier to understand.
2. **Easier Testing**: Classes with a single responsibility are easier to unit test.
3. **Better Organization**: Code is naturally organized around business capabilities.
4. **Increased Reusability**: Focused classes can be reused in different contexts.
5. **Better Maintainability**: Changes to one responsibility won't affect others.

## Examples in TypeScript

### Example 1: Violating SRP

Here's an example of a class that violates the Single Responsibility Principle by having multiple responsibilities:

```typescript
class User {
  private name: string;
  private email: string;
  private database: Database;

  constructor(name: string, email: string) {
    this.name = name;
    this.email = email;
    this.database = new Database();
  }

  // Responsibility 1: User properties management
  getName(): string {
    return this.name;
  }

  setName(name: string): void {
    this.name = name;
  }

  getEmail(): string {
    return this.email;
  }

  setEmail(email: string): void {
    this.email = email;
  }

  // Responsibility 2: Email validation
  validateEmail(email: string): boolean {
    const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return regex.test(email);
  }

  // Responsibility 3: Database operations
  saveToDatabase(): void {
    // Connect to database
    this.database.connect();

    // Save user data
    this.database.save('users', {
      name: this.name,
      email: this.email,
    });

    console.log('User saved to database');
  }

  loadFromDatabase(userId: string): void {
    // Connect to database
    this.database.connect();

    // Load user data
    const userData = this.database.find('users', userId);
    
    this.name = userData.name;
    this.email = userData.email;
    
    console.log('User loaded from database');
  }

  // Responsibility 4: User notification
  sendWelcomeEmail(): void {
    // Code to send welcome email
    console.log(`Sending welcome email to ${this.email}`);
    // ... email sending logic
  }

  notifyPasswordChange(): void {
    // Code to send password change notification
    console.log(`Sending password change notification to ${this.email}`);
    // ... email sending logic
  }
}

// Usage
const user = new User('John Doe', 'john@example.com');
user.saveToDatabase();
user.sendWelcomeEmail();
```

This `User` class has at least four distinct responsibilities:
1. Managing user properties (name, email)
2. Email validation
3. Database operations
4. User notifications

### Example 2: Following SRP

Here's how we can refactor the above class to follow SRP by separating each responsibility into its own class:

```typescript
// Responsibility 1: User properties management
class User {
  constructor(private name: string, private email: string) {}

  getName(): string {
    return this.name;
  }

  setName(name: string): void {
    this.name = name;
  }

  getEmail(): string {
    return this.email;
  }

  setEmail(email: string): void {
    // We can use the EmailValidator here, but the responsibility for validation
    // still belongs to the EmailValidator class
    if (EmailValidator.isValid(email)) {
      this.email = email;
    } else {
      throw new Error('Invalid email');
    }
  }
}

// Responsibility 2: Email validation
class EmailValidator {
  static isValid(email: string): boolean {
    const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return regex.test(email);
  }
}

// Responsibility 3: Database operations
class UserRepository {
  private database: Database;

  constructor() {
    this.database = new Database();
  }

  save(user: User): void {
    this.database.connect();
    this.database.save('users', {
      name: user.getName(),
      email: user.getEmail(),
    });
    console.log('User saved to database');
  }

  findById(userId: string): User {
    this.database.connect();
    const userData = this.database.find('users', userId);
    return new User(userData.name, userData.email);
  }
}

// Responsibility 4: User notification
class UserNotifier {
  sendWelcomeEmail(user: User): void {
    console.log(`Sending welcome email to ${user.getEmail()}`);
    // ... email sending logic
  }

  notifyPasswordChange(user: User): void {
    console.log(`Sending password change notification to ${user.getEmail()}`);
    // ... email sending logic
  }
}

// Usage with SRP
const user = new User('John Doe', 'john@example.com');

const userRepository = new UserRepository();
userRepository.save(user);

const userNotifier = new UserNotifier();
userNotifier.sendWelcomeEmail(user);
```

Now each class has a single responsibility:
- `User` manages user properties
- `EmailValidator` handles email validation
- `UserRepository` manages database operations
- `UserNotifier` handles user notifications

### Example 3: Further SRP Application

Let's look at another example involving a report generator:

```typescript
// Violating SRP
class ReportGenerator {
  generateReport(data: any[]): string {
    // Responsibility 1: Process and analyze data
    const processedData = this.processData(data);
    
    // Responsibility 2: Format the report
    let report = '<html><head><title>Report</title></head><body>';
    report += '<h1>Report</h1>';
    report += '<table>';
    for (const item of processedData) {
      report += `<tr><td>${item.name}</td><td>${item.value}</td></tr>`;
    }
    report += '</table>';
    report += '</body></html>';
    
    // Responsibility 3: Save the report
    this.saveReport(report, 'report.html');
    
    return report;
  }
  
  private processData(data: any[]): any[] {
    // Data processing logic
    return data.map(item => ({
      name: item.name,
      value: item.value * 2 // Some transformation
    }));
  }
  
  private saveReport(report: string, filename: string): void {
    // Save report to file
    console.log(`Saving report to ${filename}`);
    // ... file saving logic
  }
}
```

Following SRP:

```typescript
// Following SRP
class DataProcessor {
  processData(data: any[]): any[] {
    // Data processing logic
    return data.map(item => ({
      name: item.name,
      value: item.value * 2 // Some transformation
    }));
  }
}

class ReportFormatter {
  formatAsHtml(data: any[]): string {
    let report = '<html><head><title>Report</title></head><body>';
    report += '<h1>Report</h1>';
    report += '<table>';
    for (const item of data) {
      report += `<tr><td>${item.name}</td><td>${item.value}</td></tr>`;
    }
    report += '</table>';
    report += '</body></html>';
    
    return report;
  }
  
  formatAsCSV(data: any[]): string {
    let csv = 'name,value\n';
    for (const item of data) {
      csv += `${item.name},${item.value}\n`;
    }
    return csv;
  }
}

class ReportStorage {
  saveToFile(content: string, filename: string): void {
    // Save content to file
    console.log(`Saving report to ${filename}`);
    // ... file saving logic
  }
}

// Coordinating class (uses other classes but with a single responsibility)
class ReportGenerator {
  private dataProcessor: DataProcessor;
  private formatter: ReportFormatter;
  private storage: ReportStorage;
  
  constructor() {
    this.dataProcessor = new DataProcessor();
    this.formatter = new ReportFormatter();
    this.storage = new ReportStorage();
  }
  
  createHtmlReport(data: any[], saveToFile: boolean = false): string {
    const processedData = this.dataProcessor.processData(data);
    const report = this.formatter.formatAsHtml(processedData);
    
    if (saveToFile) {
      this.storage.saveToFile(report, 'report.html');
    }
    
    return report;
  }
  
  createCsvReport(data: any[], saveToFile: boolean = false): string {
    const processedData = this.dataProcessor.processData(data);
    const report = this.formatter.formatAsCSV(processedData);
    
    if (saveToFile) {
      this.storage.saveToFile(report, 'report.csv');
    }
    
    return report;
  }
}
```

## Common Signs of SRP Violations

1. **Large Classes**: Classes with many methods and properties often have multiple responsibilities.
2. **Methods That Change for Different Reasons**: If methods in a class change for different reasons, the class likely has multiple responsibilities.
3. **Methods That Use Only a Subset of the Class's Properties**: This suggests that those methods could be moved to a separate class.
4. **Difficulty Naming the Class**: If you struggle to find a concise name for a class, it might be trying to do too much.

## Practical Tips for Applying SRP

1. **Start with Responsibilities, Not Classes**: Identify responsibilities first, then design classes around them.
2. **Use Composition**: Combine simple, focused classes to create complex behavior.
3. **Watch for "And" in Class Names**: Names like `UserAndEmailValidator` suggest multiple responsibilities.
4. **Be Pragmatic**: Don't split classes to the point where the codebase becomes fragmented and difficult to navigate.
5. **Refactor Gradually**: When faced with a class that violates SRP, refactor it step by step.

## Relationship with Other SOLID Principles

SRP works together with other SOLID principles:

- It helps in achieving **Open/Closed Principle** by making classes more focused and less likely to need changes.
- It aids in **Liskov Substitution Principle** by making inheritance hierarchies simpler and more coherent.
- It supports **Interface Segregation Principle** by encouraging smaller, more focused interfaces.
- It contributes to **Dependency Inversion Principle** by making it easier to create abstractions around focused behaviors.

## Conclusion

The Single Responsibility Principle is a powerful guide for creating clean, maintainable code. By ensuring that each class has exactly one reason to change, you can reduce complexity, improve testability, and make your codebase more adaptable to changing requirements. While it may sometimes lead to more classes, the resulting code is typically more robust, understandable, and easier to maintain.

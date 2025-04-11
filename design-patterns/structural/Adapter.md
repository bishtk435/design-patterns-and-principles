# Adapter Pattern

## Intent

The Adapter pattern converts the interface of a class into another interface that clients expect. It allows classes to work together that couldn't otherwise because of incompatible interfaces.

## Also Known As

- Wrapper

## Motivation

Sometimes, we want to use an existing class, but its interface doesn't match what we need. Instead of changing the existing code (which might not be possible if it's from a third-party library), we can create an adapter that acts as a middle-man between our code and the existing class.

Consider a scenario where:
- You have a modern application that works with JSON data
- You need to integrate with a legacy library that only works with XML
- Instead of rewriting the legacy code, you can create an adapter that converts JSON to XML and vice versa

### Object Adapter Pattern
This uses composition. The adapter implements the target interface and holds an instance of the adaptee.

### Class Adapter Pattern
This uses inheritance. The adapter inherits from both the target interface and the adaptee. (Note: This is not possible in TypeScript/JavaScript due to single inheritance, but can be approximated with mixins.)

## Implementation in TypeScript

### Basic Example

```typescript
// Target interface (what the client expects)
interface Target {
  request(): string;
}

// The existing class with incompatible interface
class Adaptee {
  public specificRequest(): string {
    return 'Specific behavior';
  }
}

// Adapter - makes Adaptee compatible with the Target interface
class Adapter implements Target {
  private adaptee: Adaptee;

  constructor(adaptee: Adaptee) {
    this.adaptee = adaptee;
  }

  public request(): string {
    // Call the specific method on the adaptee and adapt it to the target interface
    const result = this.adaptee.specificRequest();
    return `Adapter: (TRANSLATED) ${result}`;
  }
}

// Client code
function clientCode(target: Target) {
  console.log(target.request());
}

// Using the target interface directly
console.log('Client: I can work with the Target objects:');
const target = {
  request: () => 'Target: The default target\'s behavior.'
};
clientCode(target);
console.log('');

// Using the adaptee through the adapter
console.log('Client: But I can also work with the Adaptee via the Adapter:');
const adaptee = new Adaptee();
const adapter = new Adapter(adaptee);
clientCode(adapter);
```

### Real-World Example: Payment Gateway Adapter

```typescript
// Target interface (what our application expects)
interface PaymentProcessor {
  processPayment(amount: number, currency: string): Promise<PaymentResult>;
}

// PaymentResult interface
interface PaymentResult {
  success: boolean;
  transactionId: string;
  message: string;
}

// Existing third-party payment gateway (the adaptee)
class PayPalGateway {
  // Notice the different interface
  async makePayment(paymentDetails: {
    sum: number;
    currencyCode: string;
  }): Promise<{
    status: string;
    reference: string;
    note: string;
  }> {
    // Simulate API call to PayPal
    console.log(`PayPal: Processing payment of ${paymentDetails.sum} ${paymentDetails.currencyCode}`);
    
    // Simulated response
    return {
      status: 'SUCCESS',
      reference: `PP-${Date.now()}`,
      note: 'Payment processed successfully'
    };
  }
}

// Our adapter that makes PayPalGateway work with our application
class PayPalAdapter implements PaymentProcessor {
  private paypalGateway: PayPalGateway;

  constructor(paypalGateway: PayPalGateway) {
    this.paypalGateway = paypalGateway;
  }

  async processPayment(amount: number, currency: string): Promise<PaymentResult> {
    try {
      // Convert our interface to what PayPal expects
      const paypalResult = await this.paypalGateway.makePayment({
        sum: amount,
        currencyCode: currency
      });
      
      // Convert PayPal's response to our expected format
      return {
        success: paypalResult.status === 'SUCCESS',
        transactionId: paypalResult.reference,
        message: paypalResult.note
      };
    } catch (error) {
      return {
        success: false,
        transactionId: '',
        message: error.message
      };
    }
  }
}

// Another payment gateway we might use (the adaptee)
class StripeGateway {
  // Again, a different interface
  async createCharge(options: {
    amount: number;
    currency: string;
  }): Promise<{
    id: string;
    successful: boolean;
    errorMessage?: string;
  }> {
    // Simulate API call to Stripe
    console.log(`Stripe: Creating charge of ${options.amount} ${options.currency}`);
    
    // Simulated response
    return {
      id: `STRIPE-${Date.now()}`,
      successful: true
    };
  }
}

// Adapter for Stripe
class StripeAdapter implements PaymentProcessor {
  private stripeGateway: StripeGateway;

  constructor(stripeGateway: StripeGateway) {
    this.stripeGateway = stripeGateway;
  }

  async processPayment(amount: number, currency: string): Promise<PaymentResult> {
    try {
      // Convert our interface to what Stripe expects
      const stripeResult = await this.stripeGateway.createCharge({
        amount: amount,
        currency: currency
      });
      
      // Convert Stripe's response to our expected format
      return {
        success: stripeResult.successful,
        transactionId: stripeResult.id,
        message: stripeResult.errorMessage || 'Payment processed successfully'
      };
    } catch (error) {
      return {
        success: false,
        transactionId: '',
        message: error.message
      };
    }
  }
}

// Client code that works with the PaymentProcessor interface
async function processOrder(processor: PaymentProcessor, orderId: string, amount: number) {
  console.log(`Processing order ${orderId} for $${amount}`);
  
  const result = await processor.processPayment(amount, 'USD');
  
  if (result.success) {
    console.log(`Order ${orderId} completed successfully!`);
    console.log(`Transaction ID: ${result.transactionId}`);
  } else {
    console.log(`Order ${orderId} failed: ${result.message}`);
  }
}

// Usage
async function run() {
  // Process with PayPal
  const paypalGateway = new PayPalGateway();
  const paypalAdapter = new PayPalAdapter(paypalGateway);
  await processOrder(paypalAdapter, 'ORD-PP-001', 99.99);
  
  console.log('\n---\n');
  
  // Process with Stripe
  const stripeGateway = new StripeGateway();
  const stripeAdapter = new StripeAdapter(stripeGateway);
  await processOrder(stripeAdapter, 'ORD-ST-001', 149.99);
}

run();
```

### Two-Way Adapter Example

Sometimes you might need an adapter that can convert both ways:

```typescript
// Two different date formats
interface IsoDateFormatter {
  formatToIso(date: Date): string;
  parseFromIso(isoString: string): Date;
}

interface LocalDateFormatter {
  formatToLocal(date: Date): string;
  parseFromLocal(localString: string): Date;
}

// Implementation of the ISO formatter
class IsoFormatter implements IsoDateFormatter {
  formatToIso(date: Date): string {
    return date.toISOString();
  }
  
  parseFromIso(isoString: string): Date {
    return new Date(isoString);
  }
}

// Implementation of the local formatter
class USLocalFormatter implements LocalDateFormatter {
  formatToLocal(date: Date): string {
    return date.toLocaleDateString('en-US');
  }
  
  parseFromLocal(localString: string): Date {
    const parts = localString.split('/');
    // US date format: MM/DD/YYYY
    return new Date(
      parseInt(parts[2]), 
      parseInt(parts[0]) - 1, 
      parseInt(parts[1])
    );
  }
}

// Two-way adapter that works with both interfaces
class DateFormatAdapter implements IsoDateFormatter, LocalDateFormatter {
  private isoFormatter: IsoDateFormatter;
  private localFormatter: LocalDateFormatter;
  
  constructor(isoFormatter: IsoDateFormatter, localFormatter: LocalDateFormatter) {
    this.isoFormatter = isoFormatter;
    this.localFormatter = localFormatter;
  }
  
  // IsoDateFormatter methods
  formatToIso(date: Date): string {
    return this.isoFormatter.formatToIso(date);
  }
  
  parseFromIso(isoString: string): Date {
    return this.isoFormatter.parseFromIso(isoString);
  }
  
  // LocalDateFormatter methods
  formatToLocal(date: Date): string {
    return this.localFormatter.formatToLocal(date);
  }
  
  parseFromLocal(localString: string): Date {
    return this.localFormatter.parseFromLocal(localString);
  }
  
  // Additional conversion methods
  convertIsoToLocal(isoString: string): string {
    const date = this.parseFromIso(isoString);
    return this.formatToLocal(date);
  }
  
  convertLocalToIso(localString: string): string {
    const date = this.parseFromLocal(localString);
    return this.formatToIso(date);
  }
}

// Usage
const isoFormatter = new IsoFormatter();
const usFormatter = new USLocalFormatter();
const adapter = new DateFormatAdapter(isoFormatter, usFormatter);

const today = new Date();
console.log('ISO format:', adapter.formatToIso(today));
console.log('US local format:', adapter.formatToLocal(today));

const isoString = '2023-04-15T12:00:00.000Z';
console.log(`Converting ISO to local: ${adapter.convertIsoToLocal(isoString)}`);

const usString = '04/15/2023';
console.log(`Converting US local to ISO: ${adapter.convertLocalToIso(usString)}`);
```

### Class Adapter Simulation with Mixins

While TypeScript doesn't support multiple inheritance directly, we can simulate a class adapter using mixins:

```typescript
// Target interface
interface Target {
  request(): string;
}

// Adaptee class
class Adaptee {
  public specificRequest(): string {
    return 'Specific behavior';
  }
}

// Mixin function to add Target functionality to Adaptee
function withTarget<T extends new (...args: any[]) => Adaptee>(Base: T) {
  return class extends Base implements Target {
    request(): string {
      return `ClassAdapter: (TRANSLATED) ${this.specificRequest()}`;
    }
  };
}

// Create the class adapter using the mixin
const ClassAdapter = withTarget(Adaptee);

// Usage
function clientCode(target: Target) {
  console.log(target.request());
}

console.log('Client: Using the ClassAdapter:');
const instance = new ClassAdapter();
clientCode(instance);
```

## Object Adapter vs. Class Adapter

### Object Adapter
- Uses composition
- Can adapt classes that are not related by inheritance
- More flexible but might involve more code
- Can adapt multiple adaptees

### Class Adapter
- Uses inheritance
- Can override adaptee's behavior
- Less code, but requires language support for multiple inheritance
- Limited to adapting a single adaptee

## Pros and Cons

### Pros
1. **Single Responsibility Principle**: You can separate the interface or data conversion code from the primary business logic.
2. **Open/Closed Principle**: You can introduce new adapters without changing the existing code.
3. **Integration**: Makes it possible to use existing classes even if their interfaces don't match what you need.
4. **Reusability**: Allows for reusing existing classes rather than duplicating functionality.

### Cons
1. **Complexity**: The code becomes more complex by introducing additional interfaces and classes.
2. **Performance Overhead**: Extra calls might introduce a slight performance overhead.
3. **Adapter Proliferation**: Could result in a proliferation of adapters if not managed carefully.

## Real-World Applications

1. **API Integration**: Adapting between different API formats (REST, SOAP, GraphQL)
2. **Data Conversion**: Converting between data formats (JSON, XML, CSV)
3. **Legacy System Integration**: Making old systems work with new code
4. **Third-Party Libraries**: Adapting third-party libraries to fit your application's interfaces
5. **Cross-Platform Compatibility**: Making code work across different platforms

## Related Patterns

- **Bridge**: Both patterns have similar structures, but they address different problems. Adapter makes unrelated classes work together, while Bridge separates abstraction from implementation.
- **Decorator**: Decorator changes the behavior of a component without changing its interface, while Adapter changes the interface without changing the behavior.
- **Proxy**: Proxy provides the same interface as its service, while Adapter provides a different interface.
- **Facade**: Facade defines a new interface to simplify access to a subsystem, while Adapter adapts an existing interface.

## Variations

### Service Adapter

In modern TypeScript applications, adapters are often used for service abstraction:

```typescript
// Different logging services
interface Logger {
  log(message: string, level: 'info' | 'warn' | 'error'): void;
}

// ConsoleLogger directly matches our interface
class ConsoleLogger implements Logger {
  log(message: string, level: 'info' | 'warn' | 'error'): void {
    console[level](message);
  }
}

// Third-party logger with a different interface
class ThirdPartyLogger {
  debug(msg: string): void {
    console.log(`[DEBUG] ${msg}`);
  }
  
  information(msg: string): void {
    console.log(`[INFO] ${msg}`);
  }
  
  warning(msg: string): void {
    console.log(`[WARNING] ${msg}`);
  }
  
  error(msg: string): void {
    console.log(`[ERROR] ${msg}`);
  }
}

// Adapter for the third-party logger
class ThirdPartyLoggerAdapter implements Logger {
  private adaptee: ThirdPartyLogger;
  
  constructor(adaptee: ThirdPartyLogger) {
    this.adaptee = adaptee;
  }
  
  log(message: string, level: 'info' | 'warn' | 'error'): void {
    switch (level) {
      case 'info':
        this.adaptee.information(message);
        break;
      case 'warn':
        this.adaptee.warning(message);
        break;
      case 'error':
        this.adaptee.error(message);
        break;
    }
  }
}

// Client code
function logActivity(logger: Logger, activity: string) {
  logger.log(`User performed: ${activity}`, 'info');
}

// Usage with direct implementation
const consoleLogger = new ConsoleLogger();
logActivity(consoleLogger, 'login');

// Usage with adapter
const thirdPartyLogger = new ThirdPartyLogger();
const adapter = new ThirdPartyLoggerAdapter(thirdPartyLogger);
logActivity(adapter, 'checkout');
```

## Conclusion

The Adapter pattern is a powerful structural design pattern that allows objects with incompatible interfaces to collaborate. It's especially useful in systems that need to integrate with external components or legacy code without modifying the existing code.

In TypeScript applications, adapters are commonly used to:
- Create typesafe wrappers around external APIs
- Convert between different data formats
- Integrate third-party libraries in a way that fits your application architecture
- Isolate changes to external services behind a consistent interface

By understanding and applying the Adapter pattern, you can write more maintainable and flexible code that effectively integrates disparate components.

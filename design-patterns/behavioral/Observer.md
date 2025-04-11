# Observer Pattern

## Intent

The Observer pattern defines a one-to-many dependency between objects so that when one object (the subject) changes state, all its dependents (observers) are notified and updated automatically.

## Also Known As

- Publish-Subscribe
- Event-Listener
- Dependents

## Motivation

In many applications, objects need to be notified when certain things happen:
- UI components need to know when data changes
- Services need to respond to system events
- Business logic needs to react to changes in application state

The Observer pattern solves this problem by establishing a subscription mechanism where "observer" objects can subscribe to "subject" objects. The subject notifies all registered observers when its state changes.

## Structure

![Observer Pattern Structure](https://www.plantuml.com/plantuml/png/TP11Yzim38Nl-HN3tZrWQ1Pe7YLBGYcwIDXaDeKjzs8nfXbLaGTt7SLaKurlldyixgdP1rLODJ99grgDC2VDBa9Qne1A9h3Qh7CrPcJmWKhEWm_1nZf6MXssbXaKfbP_Qiv4kGKlsXJnm6OJcuQCBY5-A33_h6u0N8hW2QDQzedBMdjG0K2XCxCOPsaQ4KGx1gqFfgYQCelL2kwQm4qDWAvWC0WKpNdLnxrKtLVJo2b4lVvWqn3PbkXMUMTUgAWbJ6fTjKqPB9_3iJVdAaLgY8D_hGhMy0wFmIVMsTpdV0C_)

## Implementation in TypeScript

### Basic Implementation

```typescript
// Observer interface
interface Observer {
  update(subject: Subject): void;
}

// Subject interface
interface Subject {
  attach(observer: Observer): void;
  detach(observer: Observer): void;
  notify(): void;
}

// Concrete Subject
class ConcreteSubject implements Subject {
  private observers: Observer[] = [];
  private state: number = 0;

  public attach(observer: Observer): void {
    const isExist = this.observers.includes(observer);
    if (isExist) {
      return console.log('Observer has been attached already.');
    }
    
    console.log('Observer attached.');
    this.observers.push(observer);
  }

  public detach(observer: Observer): void {
    const observerIndex = this.observers.indexOf(observer);
    
    if (observerIndex === -1) {
      return console.log('Nonexistent observer.');
    }
    
    this.observers.splice(observerIndex, 1);
    console.log('Observer detached.');
  }

  public notify(): void {
    console.log('Notifying observers...');
    for (const observer of this.observers) {
      observer.update(this);
    }
  }

  public getState(): number {
    return this.state;
  }

  public setState(state: number): void {
    console.log(`Setting state to: ${state}`);
    this.state = state;
    this.notify();
  }
}

// Concrete Observer A
class ConcreteObserverA implements Observer {
  public update(subject: Subject): void {
    if (subject instanceof ConcreteSubject && subject.getState() < 3) {
      console.log('ConcreteObserverA: Reacted to the event.');
    }
  }
}

// Concrete Observer B
class ConcreteObserverB implements Observer {
  public update(subject: Subject): void {
    if (subject instanceof ConcreteSubject && (subject.getState() === 0 || subject.getState() >= 2)) {
      console.log('ConcreteObserverB: Reacted to the event.');
    }
  }
}

// Client code
const subject = new ConcreteSubject();

const observerA = new ConcreteObserverA();
const observerB = new ConcreteObserverB();

subject.attach(observerA);
subject.attach(observerB);

subject.setState(1);
// Output:
// Setting state to: 1
// Notifying observers...
// ConcreteObserverA: Reacted to the event.

subject.setState(2);
// Output:
// Setting state to: 2
// Notifying observers...
// ConcreteObserverA: Reacted to the event.
// ConcreteObserverB: Reacted to the event.

subject.detach(observerA);

subject.setState(3);
// Output:
// Setting state to: 3
// Notifying observers...
// ConcreteObserverB: Reacted to the event.
```

### Real-World Example: News Subscription Service

```typescript
// Observer interface for subscribers
interface Subscriber {
  id: string;
  update(publisher: Publisher, data: any): void;
}

// Subject interface for publishers
interface Publisher {
  subscribe(subscriber: Subscriber): void;
  unsubscribe(subscriber: Subscriber): void;
  notify(data: any): void;
}

// Concrete publisher: News Agency
class NewsAgency implements Publisher {
  private subscribers: Map<string, Subscriber> = new Map();
  private name: string;
  
  constructor(name: string) {
    this.name = name;
  }
  
  public subscribe(subscriber: Subscriber): void {
    this.subscribers.set(subscriber.id, subscriber);
    console.log(`${subscriber.id} subscribed to ${this.name}`);
  }
  
  public unsubscribe(subscriber: Subscriber): void {
    this.subscribers.delete(subscriber.id);
    console.log(`${subscriber.id} unsubscribed from ${this.name}`);
  }
  
  public notify(data: any): void {
    console.log(`${this.name} is notifying subscribers about: ${data.title}`);
    for (const subscriber of this.subscribers.values()) {
      subscriber.update(this, data);
    }
  }
  
  public publishNews(title: string, content: string): void {
    const newsData = {
      title,
      content,
      timestamp: new Date()
    };
    
    console.log(`${this.name} published: ${title}`);
    this.notify(newsData);
  }
}

// Concrete subscriber: NewsReader
class NewsReader implements Subscriber {
  public id: string;
  private preferences: string[];
  
  constructor(id: string, preferences: string[] = []) {
    this.id = id;
    this.preferences = preferences;
  }
  
  public update(publisher: Publisher, data: any): void {
    // Only interested in news that matches preferences if any are set
    if (this.preferences.length === 0 || 
        this.preferences.some(preference => 
          data.title.toLowerCase().includes(preference) || 
          data.content.toLowerCase().includes(preference)
        )) {
      console.log(`${this.id} received news: ${data.title}`);
      // In a real app, we might display the news to the user
    } else {
      console.log(`${this.id} ignored news: ${data.title} (not matching preferences)`);
    }
  }
}

// Usage
const bbcNews = new NewsAgency('BBC News');
const cnnNews = new NewsAgency('CNN');

const reader1 = new NewsReader('John', ['politics', 'technology']);
const reader2 = new NewsReader('Alice', ['sports', 'health']);
const reader3 = new NewsReader('Bob'); // No preferences, interested in all news

bbcNews.subscribe(reader1);
bbcNews.subscribe(reader3);

cnnNews.subscribe(reader2);
cnnNews.subscribe(reader3);

bbcNews.publishNews(
  'New Technology Regulations', 
  'Governments worldwide are implementing new technology regulations...'
);
// John and Bob will receive this news

cnnNews.publishNews(
  'Sports Championship Results', 
  'The championship concluded with unexpected results...'
);
// Alice and Bob will receive this news

bbcNews.unsubscribe(reader1);

bbcNews.publishNews(
  'Political Developments', 
  'New political developments are shaping the global landscape...'
);
// Only Bob will receive this news now
```

### Event-Driven Implementation

```typescript
// A simple event emitter implementation
class EventEmitter {
  private events: Map<string, Array<(data: any) => void>> = new Map();
  
  public on(event: string, listener: (data: any) => void): void {
    if (!this.events.has(event)) {
      this.events.set(event, []);
    }
    
    const listeners = this.events.get(event)!;
    listeners.push(listener);
  }
  
  public off(event: string, listener: (data: any) => void): void {
    if (!this.events.has(event)) {
      return;
    }
    
    const listeners = this.events.get(event)!;
    const index = listeners.indexOf(listener);
    
    if (index !== -1) {
      listeners.splice(index, 1);
    }
    
    if (listeners.length === 0) {
      this.events.delete(event);
    }
  }
  
  public emit(event: string, data?: any): void {
    if (!this.events.has(event)) {
      return;
    }
    
    const listeners = this.events.get(event)!;
    listeners.forEach(listener => listener(data));
  }
}

// Example usage
class ShoppingCart extends EventEmitter {
  private items: string[] = [];
  
  public addItem(item: string): void {
    this.items.push(item);
    this.emit('itemAdded', item);
    this.emit('cartUpdated', this.items);
  }
  
  public removeItem(index: number): void {
    if (index >= 0 && index < this.items.length) {
      const removedItem = this.items.splice(index, 1)[0];
      this.emit('itemRemoved', removedItem);
      this.emit('cartUpdated', this.items);
    }
  }
  
  public getItems(): string[] {
    return [...this.items];
  }
}

// Client code
const cart = new ShoppingCart();

// Subscribe to events
const itemAddedListener = (item: string) => {
  console.log(`Item added: ${item}`);
};

const itemRemovedListener = (item: string) => {
  console.log(`Item removed: ${item}`);
};

const cartUpdatedListener = (items: string[]) => {
  console.log(`Cart now has ${items.length} items: ${items.join(', ')}`);
};

cart.on('itemAdded', itemAddedListener);
cart.on('itemRemoved', itemRemovedListener);
cart.on('cartUpdated', cartUpdatedListener);

// Add items to cart
cart.addItem('Apple');
// Output:
// Item added: Apple
// Cart now has 1 items: Apple

cart.addItem('Banana');
// Output:
// Item added: Banana
// Cart now has 2 items: Apple, Banana

// Remove an item
cart.removeItem(0);
// Output:
// Item removed: Apple
// Cart now has 1 items: Banana

// Unsubscribe from an event
cart.off('cartUpdated', cartUpdatedListener);

cart.addItem('Orange');
// Output:
// Item added: Orange
// (cartUpdatedListener is not called)
```

## Modern TypeScript Implementations

### Using TypeScript Generics

```typescript
// Generic Observer interface
interface Observer<T> {
  update(data: T): void;
}

// Generic Subject interface
interface Subject<T> {
  attach(observer: Observer<T>): void;
  detach(observer: Observer<T>): void;
  notify(data: T): void;
}

// Concrete subject with strong typing
class WeatherStation implements Subject<WeatherData> {
  private observers: Observer<WeatherData>[] = [];
  private currentWeather: WeatherData = {
    temperature: 0,
    humidity: 0,
    pressure: 0
  };
  
  public attach(observer: Observer<WeatherData>): void {
    const isExist = this.observers.includes(observer);
    if (isExist) {
      return;
    }
    
    this.observers.push(observer);
    console.log('Weather observer attached');
  }
  
  public detach(observer: Observer<WeatherData>): void {
    const observerIndex = this.observers.indexOf(observer);
    
    if (observerIndex === -1) {
      return;
    }
    
    this.observers.splice(observerIndex, 1);
    console.log('Weather observer detached');
  }
  
  public notify(data: WeatherData): void {
    for (const observer of this.observers) {
      observer.update(data);
    }
  }
  
  public setMeasurements(temperature: number, humidity: number, pressure: number): void {
    this.currentWeather = { temperature, humidity, pressure };
    console.log(`Weather updated: ${temperature}°C, ${humidity}% humidity, ${pressure} pressure`);
    this.notify(this.currentWeather);
  }
  
  public getCurrentWeather(): WeatherData {
    return { ...this.currentWeather };
  }
}

// Interface for the data passed to observers
interface WeatherData {
  temperature: number;
  humidity: number;
  pressure: number;
}

// Concrete observer with type safety
class TemperatureDisplay implements Observer<WeatherData> {
  private id: string;
  
  constructor(id: string) {
    this.id = id;
  }
  
  public update(data: WeatherData): void {
    console.log(`${this.id} Display: Temperature is ${data.temperature}°C`);
  }
}

class WeatherStatistics implements Observer<WeatherData> {
  private temperatureSum: number = 0;
  private readingCount: number = 0;
  
  public update(data: WeatherData): void {
    this.temperatureSum += data.temperature;
    this.readingCount++;
    
    console.log(`WeatherStats: Avg temperature is ${this.temperatureSum / this.readingCount}°C after ${this.readingCount} readings`);
  }
}

// Usage
const weatherStation = new WeatherStation();

const phoneDisplay = new TemperatureDisplay('Phone');
const tabletDisplay = new TemperatureDisplay('Tablet');
const weatherStats = new WeatherStatistics();

weatherStation.attach(phoneDisplay);
weatherStation.attach(tabletDisplay);
weatherStation.attach(weatherStats);

// Simulate weather changes
weatherStation.setMeasurements(25.2, 65, 1013);
weatherStation.setMeasurements(26.5, 70, 1014);

// Detach an observer
weatherStation.detach(tabletDisplay);

weatherStation.setMeasurements(23.9, 90, 1010);
```

### Using Reactive Programming (RxJS)

```typescript
import { Subject } from 'rxjs';

interface StockData {
  symbol: string;
  price: number;
  timestamp: Date;
}

class StockMarket {
  // RxJS Subject acts as both an Observable and an Observer
  private stockUpdates = new Subject<StockData>();
  
  // Expose the Observable part of the Subject
  public getStockUpdates() {
    return this.stockUpdates.asObservable();
  }
  
  // Method to update stock prices
  public updateStock(symbol: string, price: number) {
    const stockData: StockData = {
      symbol,
      price,
      timestamp: new Date()
    };
    
    console.log(`Stock update: ${symbol} at $${price}`);
    
    // Publish the update to all subscribers
    this.stockUpdates.next(stockData);
  }
}

// Clients/Observers
const stockMarket = new StockMarket();

// Subscribe to all stock updates
const allStocksSubscription = stockMarket.getStockUpdates()
  .subscribe(data => {
    console.log(`All stocks observer: ${data.symbol} is now $${data.price}`);
  });

// Subscribe only to specific stocks (AAPL)
const appleStockSubscription = stockMarket.getStockUpdates()
  .pipe(
    filter(data => data.symbol === 'AAPL')
  )
  .subscribe(data => {
    console.log(`Apple stock observer: Price is now $${data.price}`);
  });

// Simulate stock price updates
stockMarket.updateStock('AAPL', 150.25);
stockMarket.updateStock('GOOGL', 2750.80);
stockMarket.updateStock('AAPL', 152.35);

// Unsubscribe (equivalent to detaching observers)
setTimeout(() => {
  console.log('Unsubscribing from all stocks');
  allStocksSubscription.unsubscribe();
  
  stockMarket.updateStock('AAPL', 153.50);
  // Only appleStockSubscription will receive this update
}, 5000);
```

## Pros and Cons

### Pros

1. **Open/Closed Principle**: You can introduce new subscriber classes without changing the publisher's code.
2. **Loose Coupling**: Publishers don't need to know anything about subscribers.
3. **Dynamic Relationships**: You can establish relationships between objects at runtime.
4. **1:N Communication**: Efficiently communicate state changes to multiple objects at once.

### Cons

1. **Unexpected Updates**: Subscribers are notified in an unpredictable order.
2. **Memory Leaks**: If observers don't unsubscribe properly, they might prevent garbage collection.
3. **Complexity**: In complex systems, it can be hard to track the flow of updates.
4. **Performance Issues**: With many observers, notification can become a performance bottleneck.

## Real-World Applications

1. **User Interface Development**: Updating UI components when data changes.
2. **Event Handling Systems**: Browser events (click, keypress) use the observer pattern.
3. **Messaging Systems**: Publish-subscribe messaging architectures.
4. **Model-View-Controller (MVC)**: Models notify views of data changes.
5. **Real-time Systems**: Monitoring and reacting to events in real-time.

## Related Patterns

- **Mediator**: Instead of observers and subjects communicating directly, they can use a mediator.
- **Singleton**: The subject is often a singleton since only one instance is needed.
- **Command**: Commands can be passed as observers to the subject.

## Variations

### Push vs. Pull Model

- **Push Model**: The subject sends detailed information to observers (what we've seen so far).
- **Pull Model**: The subject only sends a notification, and observers request the data they need.

### Asynchronous Notification

In modern JavaScript/TypeScript applications, asynchronous notification is common:

```typescript
class AsyncSubject {
  private observers: ((data: any) => void)[] = [];
  
  public subscribe(callback: (data: any) => void): void {
    this.observers.push(callback);
  }
  
  public unsubscribe(callback: (data: any) => void): void {
    this.observers = this.observers.filter(observer => observer !== callback);
  }
  
  public async notify(data: any): Promise<void> {
    const notifyPromises = this.observers.map(observer => 
      // Execute each observer asynchronously
      Promise.resolve().then(() => observer(data))
    );
    
    // Wait for all notifications to complete
    await Promise.all(notifyPromises);
  }
}
```

## Conclusion

The Observer pattern is one of the most widely used patterns in software development, especially in event-driven systems and user interfaces. By establishing a clear communication mechanism between subjects and observers, it enables loosely coupled designs where objects can interact without having deep knowledge of each other.

In modern TypeScript applications, while you might use direct implementations of the pattern, you'll often leverage existing implementations in frameworks and libraries like RxJS, Angular's EventEmitter, React's state management, or Node.js's EventEmitter.

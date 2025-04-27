# Behavioral Design Patterns: An In-Depth Guide

Behavioral design patterns are concerned with algorithms and the assignment of responsibilities between objects. They describe not just patterns of objects or classes but also the patterns of communication between them. Let's explore these patterns in order of their general relevance and practical application.

# Strategy Pattern

### Intent
Define a family of algorithms, encapsulate each one, and make them interchangeable. Strategy lets the algorithm vary independently from clients that use it.

### Problem Solved
- When you need to use different variants of an algorithm within an object
- When you want to avoid exposing complex algorithm-specific data structures
- When you need to isolate the algorithm from the client that uses it

### Relevance
The Strategy pattern is one of the most widely used behavioral patterns because it:
- Enables runtime algorithm selection
- Provides a clean way to vary part of an algorithm
- Promotes composition over inheritance
- Implements the Open/Closed Principle

### Structure
- **Context**: Maintains a reference to a Strategy object and may define an interface for Strategy to access its data
- **Strategy**: Declares an interface common to all supported algorithms
- **ConcreteStrategy**: Implements the Strategy interface, providing specific algorithm implementations

### Implementation in Python

```python
from abc import ABC, abstractmethod

# Strategy interface
class SortStrategy(ABC):
    @abstractmethod
    def sort(self, dataset):
        pass

# Concrete Strategies
class QuickSortStrategy(SortStrategy):
    def sort(self, dataset):
        print("Sorting using Quick Sort")
        # Implementation of quick sort algorithm
        return sorted(dataset)  # Simplified for this example

class MergeSortStrategy(SortStrategy):
    def sort(self, dataset):
        print("Sorting using Merge Sort")
        # Implementation of merge sort algorithm
        return sorted(dataset)  # Simplified for this example

class BubbleSortStrategy(SortStrategy):
    def sort(self, dataset):
        print("Sorting using Bubble Sort")
        # Implementation of bubble sort algorithm
        return sorted(dataset)  # Simplified for this example

# Context
class Sorter:
    def __init__(self, strategy=None):
        self._strategy = strategy
        
    def set_strategy(self, strategy):
        self._strategy = strategy
        
    def sort(self, dataset):
        if self._strategy is None:
            raise ValueError("Sorting strategy not set")
        return self._strategy.sort(dataset)

# Client code
if __name__ == "__main__":
    # Dataset to sort
    data = [5, 2, 8, 1, 9, 3]
    
    # Create context with initial strategy
    sorter = Sorter(QuickSortStrategy())
    print(f"Original data: {data}")
    
    # Sort using the initial strategy
    sorted_data = sorter.sort(data)
    print(f"Sorted data: {sorted_data}")
    
    # Change the strategy and sort again
    sorter.set_strategy(MergeSortStrategy())
    sorted_data = sorter.sort(data)
    print(f"Sorted data: {sorted_data}")
    
    # For small datasets, use bubble sort
    small_data = [3, 1, 2]
    sorter.set_strategy(BubbleSortStrategy())
    sorted_small_data = sorter.sort(small_data)
    print(f"Sorted small data: {sorted_small_data}")
```

### Real-World Example: Payment Processing

```python
from abc import ABC, abstractmethod

# Strategy interface
class PaymentStrategy(ABC):
    @abstractmethod
    def pay(self, amount):
        pass

# Concrete Strategies
class CreditCardPayment(PaymentStrategy):
    def __init__(self, card_number, expiry_date, cvv):
        self.card_number = card_number
        self.expiry_date = expiry_date
        self.cvv = cvv
        
    def pay(self, amount):
        # Logic to process credit card payment
        print(f"Paying ${amount} using Credit Card {self.card_number[-4:]}")
        return True

class PayPalPayment(PaymentStrategy):
    def __init__(self, email, password):
        self.email = email
        self.password = password
        
    def pay(self, amount):
        # Logic to process PayPal payment
        print(f"Paying ${amount} using PayPal account {self.email}")
        return True

class BankTransferPayment(PaymentStrategy):
    def __init__(self, account_number, routing_number):
        self.account_number = account_number
        self.routing_number = routing_number
        
    def pay(self, amount):
        # Logic to process bank transfer
        print(f"Paying ${amount} using Bank Transfer from account {self.account_number[-4:]}")
        return True

# Context
class ShoppingCart:
    def __init__(self):
        self.items = []
        self.payment_strategy = None
    
    def add_item(self, item, price):
        self.items.append({"item": item, "price": price})
    
    def calculate_total(self):
        return sum(item["price"] for item in self.items)
    
    def set_payment_strategy(self, payment_strategy):
        self.payment_strategy = payment_strategy
    
    def checkout(self):
        total = self.calculate_total()
        if self.payment_strategy is None:
            raise ValueError("Payment strategy not set")
        
        print(f"Checking out {len(self.items)} items:")
        for item in self.items:
            print(f"- {item['item']}: ${item['price']}")
        print(f"Total: ${total}")
        
        return self.payment_strategy.pay(total)

# Client code
if __name__ == "__main__":
    # Creating a shopping cart
    cart = ShoppingCart()
    cart.add_item("Laptop", 1200)
    cart.add_item("Mouse", 50)
    cart.add_item("Keyboard", 80)
    
    # Using credit card for payment
    credit_card = CreditCardPayment("1234567890123456", "12/25", "123")
    cart.set_payment_strategy(credit_card)
    cart.checkout()
    
    print("\nCreating a new cart...")
    # Creating another cart
    cart2 = ShoppingCart()
    cart2.add_item("Headphones", 150)
    
    # Using PayPal for payment
    paypal = PayPalPayment("user@example.com", "password")
    cart2.set_payment_strategy(paypal)
    cart2.checkout()
```

### When to Use
- When you have a class that needs to use different variants of an algorithm
- When you want to avoid exposing complex algorithm-specific data structures
- When you need to isolate the algorithm from the client that uses it
- When an algorithm has data that clients shouldn't know about

### Considerations
1. **Runtime Flexibility**: The Strategy pattern provides flexibility to change algorithms at runtime
2. **Reduced Conditional Statements**: Eliminates conditional statements in the context class
3. **Increased Number of Objects**: Creates additional objects for each strategy
4. **Client Awareness**: Clients need to be aware of different strategies to choose between them

# Observer Pattern

### Intent
Define a one-to-many dependency between objects so that when one object changes state, all its dependents are notified and updated automatically.

### Problem Solved
- When a change to one object requires changing others, and you don't know how many objects need to change
- When an object should be able to notify other objects without making assumptions about what those objects are
- When you need to maintain consistency between related objects without making them tightly coupled

### Relevance
The Observer pattern is foundational to event-driven programming and is used extensively in:
- User interfaces (event handling)
- Distributed systems (event notifications)
- News feeds and subscription services
- Implementing the Publish-Subscribe architectural pattern

### Structure
- **Subject**: Knows its observers and provides an interface for attaching and detaching observers
- **Observer**: Defines an updating interface for objects that should be notified of changes
- **ConcreteSubject**: Stores state of interest to ConcreteObserver objects and sends notifications
- **ConcreteObserver**: Maintains a reference to a ConcreteSubject object and implements the Observer updating interface

### Implementation in Python

```python
from abc import ABC, abstractmethod
from typing import List

# Observer interface
class Observer(ABC):
    @abstractmethod
    def update(self, subject):
        pass

# Subject interface
class Subject(ABC):
    @abstractmethod
    def attach(self, observer):
        pass
    
    @abstractmethod
    def detach(self, observer):
        pass
    
    @abstractmethod
    def notify(self):
        pass

# Concrete Subject
class WeatherStation(Subject):
    def __init__(self):
        self._observers = []
        self._temperature = 0
        self._humidity = 0
        self._pressure = 0
    
    def attach(self, observer):
        if observer not in self._observers:
            self._observers.append(observer)
    
    def detach(self, observer):
        self._observers.remove(observer)
    
    def notify(self):
        for observer in self._observers:
            observer.update(self)
    
    def set_measurements(self, temperature, humidity, pressure):
        self._temperature = temperature
        self._humidity = humidity
        self._pressure = pressure
        self.notify()  # Notify all observers when measurements change
    
    @property
    def temperature(self):
        return self._temperature
    
    @property
    def humidity(self):
        return self._humidity
    
    @property
    def pressure(self):
        return self._pressure

# Concrete Observers
class TemperatureDisplay(Observer):
    def update(self, subject):
        print(f"Temperature Display: {subject.temperature}°C")

class HumidityDisplay(Observer):
    def update(self, subject):
        print(f"Humidity Display: {subject.humidity}%")

class WeatherStatsDisplay(Observer):
    def __init__(self):
        self.temperature_sum = 0
        self.humidity_sum = 0
        self.pressure_sum = 0
        self.num_readings = 0
    
    def update(self, subject):
        self.temperature_sum += subject.temperature
        self.humidity_sum += subject.humidity
        self.pressure_sum += subject.pressure
        self.num_readings += 1
        
        avg_temp = self.temperature_sum / self.num_readings
        avg_humidity = self.humidity_sum / self.num_readings
        avg_pressure = self.pressure_sum / self.num_readings
        
        print(f"Weather Stats Display: Avg temperature: {avg_temp:.1f}°C, "
              f"Avg humidity: {avg_humidity:.1f}%, "
              f"Avg pressure: {avg_pressure:.1f} hPa")

# Client code
if __name__ == "__main__":
    # Create the subject
    weather_station = WeatherStation()
    
    # Create observers
    temp_display = TemperatureDisplay()
    humidity_display = HumidityDisplay()
    stats_display = WeatherStatsDisplay()
    
    # Register observers
    weather_station.attach(temp_display)
    weather_station.attach(humidity_display)
    weather_station.attach(stats_display)
    
    # Simulate weather changes
    print("Weather measurement update 1:")
    weather_station.set_measurements(25, 65, 1013)
    
    print("\nWeather measurement update 2:")
    weather_station.set_measurements(26, 70, 1014)
    
    # Remove the humidity display
    print("\nDetaching humidity display...")
    weather_station.detach(humidity_display)
    
    print("\nWeather measurement update 3:")
    weather_station.set_measurements(24, 80, 1010)
```

### Real-World Example: Stock Market Notification System

```python
from abc import ABC, abstractmethod
from typing import List, Dict
import time

# Observer interface
class StockObserver(ABC):
    @abstractmethod
    def update(self, stock_name: str, price: float):
        pass

# Subject interface
class StockSubject(ABC):
    @abstractmethod
    def register_observer(self, observer: StockObserver):
        pass
    
    @abstractmethod
    def remove_observer(self, observer: StockObserver):
        pass
    
    @abstractmethod
    def notify_observers(self, stock_name: str, price: float):
        pass

# Concrete Subject
class StockMarket(StockSubject):
    def __init__(self):
        self._observers: List[StockObserver] = []
        self._stock_prices: Dict[str, float] = {}
    
    def register_observer(self, observer: StockObserver):
        if observer not in self._observers:
            self._observers.append(observer)
    
    def remove_observer(self, observer: StockObserver):
        if observer in self._observers:
            self._observers.remove(observer)
    
    def notify_observers(self, stock_name: str, price: float):
        for observer in self._observers:
            observer.update(stock_name, price)
    
    def set_stock_price(self, stock_name: str, price: float):
        # Check if price has changed
        old_price = self._stock_prices.get(stock_name)
        price_changed = old_price is None or abs(old_price - price) > 0.01
        
        # Update price
        self._stock_prices[stock_name] = price
        
        # Notify observers if price has changed
        if price_changed:
            self.notify_observers(stock_name, price)
    
    def get_stock_price(self, stock_name: str) -> float:
        return self._stock_prices.get(stock_name, 0.0)

# Concrete Observer: StockDisplay
class StockDisplay(StockObserver):
    def __init__(self, name: str):
        self.name = name
    
    def update(self, stock_name: str, price: float):
        print(f"{self.name} - Stock Update: {stock_name} is now ${price:.2f}")

# Concrete Observer: InvestmentPortfolio
class InvestmentPortfolio(StockObserver):
    def __init__(self, name: str):
        self.name = name
        self.holdings: Dict[str, int] = {}  # Stock name -> number of shares
    
    def buy_stock(self, stock_name: str, shares: int):
        self.holdings[stock_name] = self.holdings.get(stock_name, 0) + shares
        print(f"{self.name} - Bought {shares} shares of {stock_name}")
    
    def update(self, stock_name: str, price: float):
        if stock_name in self.holdings:
            shares = self.holdings[stock_name]
            value = shares * price
            print(f"{self.name} - Portfolio Update: {stock_name} ({shares} shares) is now worth ${value:.2f}")

# Concrete Observer: StockAnalyzer
class StockAnalyzer(StockObserver):
    def __init__(self, threshold: float):
        self.price_history: Dict[str, List[float]] = {}
        self.threshold = threshold
    
    def update(self, stock_name: str, price: float):
        # Initialize history if not exists
        if stock_name not in self.price_history:
            self.price_history[stock_name] = []
        
        history = self.price_history[stock_name]
        history.append(price)
        
        # Keep only the last 5 prices
        if len(history) > 5:
            history.pop(0)
        
        # If we have at least 2 prices, check for significant changes
        if len(history) >= 2:
            prev_price = history[-2]
            change_percent = ((price - prev_price) / prev_price) * 100
            
            if abs(change_percent) >= self.threshold:
                direction = "up" if change_percent > 0 else "down"
                print(f"Analyst Alert: {stock_name} has moved {direction} by {abs(change_percent):.2f}%")

# Client code
if __name__ == "__main__":
    # Create the stock market
    market = StockMarket()
    
    # Create observers
    display1 = StockDisplay("Main Display")
    portfolio1 = InvestmentPortfolio("John's Portfolio")
    portfolio2 = InvestmentPortfolio("Sarah's Portfolio")
    analyzer = StockAnalyzer(5.0)  # Alert on 5% changes
    
    # Register observers
    market.register_observer(display1)
    market.register_observer(portfolio1)
    market.register_observer(portfolio2)
    market.register_observer(analyzer)
    
    # Set up portfolios
    portfolio1.buy_stock("AAPL", 10)
    portfolio1.buy_stock("MSFT", 5)
    portfolio2.buy_stock("GOOGL", 3)
    portfolio2.buy_stock("AAPL", 2)
    
    # Simulate stock price changes
    print("\n--- Day 1 Morning ---")
    market.set_stock_price("AAPL", 150.0)
    market.set_stock_price("MSFT", 250.0)
    market.set_stock_price("GOOGL", 2800.0)
    
    print("\n--- Day 1 Afternoon ---")
    market.set_stock_price("AAPL", 152.5)  # +1.67%
    market.set_stock_price("MSFT", 255.0)  # +2.00%
    market.set_stock_price("GOOGL", 2750.0)  # -1.79%
    
    # Sarah is no longer interested in updates
    print("\n--- Sarah unsubscribes ---")
    market.remove_observer(portfolio2)
    
    print("\n--- Day 2 Morning ---")
    market.set_stock_price("AAPL", 160.0)  # +4.92% (close to threshold)
    market.set_stock_price("MSFT", 245.0)  # -3.92%
    market.set_stock_price("GOOGL", 2900.0)  # +5.45% (exceeds threshold)
```

### When to Use
- When an abstraction has two aspects, one dependent on the other
- When changing one object requires changing others, and you don't know how many objects need to change
- When an object should be able to notify other objects without making assumptions about who these objects are
- When you need a loosely coupled system with respect to notifications

### Considerations
1. **Unexpected Updates**: Observers may be notified in an unexpected order
2. **Dangling References**: Need to manage subscriptions properly to avoid memory leaks
3. **Performance**: With many observers, notification can be costly
4. **Change Management**: Careful implementation needed to avoid infinite loops of updates

# Command Pattern

### Intent
Encapsulate a request as an object, thereby allowing for parameterized clients, queuing of requests, and supporting undoable operations.

### Problem Solved
- When you need to decouple the object that invokes an operation from the one that performs it
- When you want to parameterize objects with operations
- When you need to support undo/redo functionality
- When you want to implement callbacks in event-driven systems

### Relevance
The Command pattern is highly practical for:
- GUI systems (like menu items, buttons)
- Transaction management systems
- Task schedulers and job queues
- Macro recorders

### Structure
- **Command**: Declares an interface for executing an operation
- **ConcreteCommand**: Defines a binding between a Receiver object and an action
- **Client**: Creates a ConcreteCommand and sets its receiver
- **Invoker**: Asks the command to carry out the request
- **Receiver**: Knows how to perform the operations

### Implementation in Python

```python
from abc import ABC, abstractmethod
from typing import List

# Command interface
class Command(ABC):
    @abstractmethod
    def execute(self):
        pass
    
    @abstractmethod
    def undo(self):
        pass

# Receiver
class Light:
    def __init__(self, location):
        self.location = location
        self.is_on = False
    
    def turn_on(self):
        self.is_on = True
        print(f"{self.location} light is now ON")
    
    def turn_off(self):
        self.is_on = False
        print(f"{self.location} light is now OFF")

# Concrete Commands
class LightOnCommand(Command):
    def __init__(self, light):
        self.light = light
    
    def execute(self):
        self.light.turn_on()
    
    def undo(self):
        self.light.turn_off()

class LightOffCommand(Command):
    def __init__(self, light):
        self.light = light
    
    def execute(self):
        self.light.turn_off()
    
    def undo(self):
        self.light.turn_on()

# Invoker
class RemoteControl:
    def __init__(self):
        self.command = None
        self.command_history = []
    
    def set_command(self, command):
        self.command = command
    
    def press_button(self):
        if self.command:
            self.command.execute()
            self.command_history.append(self.command)
    
    def press_undo_button(self):
        if self.command_history:
            last_command = self.command_history.pop()
            last_command.undo()

# Client code
if __name__ == "__main__":
    # Create receivers
    living_room_light = Light("Living Room")
    kitchen_light = Light("Kitchen")
    
    # Create commands
    living_room_light_on = LightOnCommand(living_room_light)
    living_room_light_off = LightOffCommand(living_room_light)
    kitchen_light_on = LightOnCommand(kitchen_light)
    kitchen_light_off = LightOffCommand(kitchen_light)
    
    # Create invoker
    remote = RemoteControl()
    
    # Execute commands
    remote.set_command(living_room_light_on)
    remote.press_button()
    
    remote.set_command(kitchen_light_on)
    remote.press_button()
    
    remote.set_command(living_room_light_off)
    remote.press_button()
    
    # Undo the last command (living room light off)
    print("\nUndoing last command...")
    remote.press_undo_button()  # Should turn living room light back on
```

### Real-World Example: Text Editor with Undo/Redo

```python
from abc import ABC, abstractmethod
from typing import List, Optional

# Command interface
class EditorCommand(ABC):
    @abstractmethod
    def execute(self):
        pass
    
    @abstractmethod
    def undo(self):
        pass

# Receiver
class TextEditor:
    def __init__(self):
        self.text = ""
        self.selection_start = 0
        self.selection_end = 0
        self.clipboard = ""
    
    def get_text(self):
        return self.text
    
    def write(self, text):
        # Replace any selected text or insert at cursor
        if self.selection_start != self.selection_end:
            before = self.text[:self.selection_start]
            after = self.text[self.selection_end:]
            self.text = before + text + after
            self.selection_start += len(text)
            self.selection_end = self.selection_start
        else:
            before = self.text[:self.selection_start]
            after = self.text[self.selection_start:]
            self.text = before + text + after
            self.selection_start += len(text)
            self.selection_end = self.selection_start
    
    def delete(self):
        # Delete selected text
        if self.selection_start != self.selection_end:
            before = self.text[:self.selection_start]
            after = self.text[self.selection_end:]
            deleted_text = self.text[self.selection_start:self.selection_end]
            self.text = before + after
            self.selection_end = self.selection_start
            return deleted_text
        return ""
    
    def select(self, start, end):
        # Set selection range
        self.selection_start = max(0, min(start, len(self.text)))
        self.selection_end = max(0, min(end, len(self.text)))
    
    def get_selection(self):
        return self.text[self.selection_start:self.selection_end]
    
    def cut(self):
        # Cut selected text to clipboard
        selected_text = self.get_selection()
        if selected_text:
            self.clipboard = selected_text
            self.delete()
            return selected_text
        return ""
    
    def copy(self):
        # Copy selected text to clipboard
        selected_text = self.get_selection()
        if selected_text:
            self.clipboard = selected_text
        return selected_text
    
    def paste(self):
        # Paste clipboard content
        if self.clipboard:
            self.write(self.clipboard)

# Concrete Commands
class WriteCommand(EditorCommand):
    def __init__(self, editor, text):
        self.editor = editor
        self.text = text
        self.old_text = ""
        self.old_selection_start = 0
        self.old_selection_end = 0
    
    def execute(self):
        # Save state for undo
        self.old_text = self.editor.get_text()
        self.old_selection_start = self.editor.selection_start
        self.old_selection_end = self.editor.selection_end
        
        # Execute write operation
        self.editor.write(self.text)
    
    def undo(self):
        # Restore previous state
        self.editor.text = self.old_text
        self.editor.selection_start = self.old_selection_start
        self.editor.selection_end = self.old_selection_end

class DeleteCommand(EditorCommand):
    def __init__(self, editor):
        self.editor = editor
        self.deleted_text = ""
        self.old_selection_start = 0
        self.old_selection_end = 0
    
    def execute(self):
        # Save state for undo
        self.old_selection_start = self.editor.selection_start
        self.old_selection_end = self.editor.selection_end
        
        # Execute delete operation
        self.deleted_text = self.editor.delete()
    
    def undo(self):
        # Restore selection
        self.editor.selection_start = self.old_selection_start
        self.editor.selection_end = self.old_selection_start
        
        # Insert the deleted text back
        if self.deleted_text:
            self.editor.write(self.deleted_text)
            self.editor.selection_start = self.old_selection_start
            self.editor.selection_end = self.old_selection_end

class CutCommand(EditorCommand):
    def __init__(self, editor):
        self.editor = editor
        self.cut_text = ""
        self.old_clipboard = ""
        self.old_selection_start = 0
        self.old_selection_end = 0
        self.old_text = ""
    
    def execute(self):
        # Save state for undo
        self.old_clipboard = self.editor.clipboard
        self.old_selection_start = self.editor.selection_start
        self.old_selection_end = self.editor.selection_end
        self.old_text = self.editor.get_text()
        
        # Execute cut operation
        self.cut_text = self.editor.cut()
    
    def undo(self):
        # Restore previous state
        self.editor.text = self.old_text
        self.editor.clipboard = self.old_clipboard
        self.editor.selection_start = self.old_selection_start
        self.editor.selection_end = self.old_selection_end

class PasteCommand(EditorCommand):
    def __init__(self, editor):
        self.editor = editor
        self.old_text = ""
        self.old_selection_start = 0
        self.old_selection_end = 0
    
    def execute(self):
        # Save state for undo
        self.old_text = self.editor.get_text()
        self.old_selection_start = self.editor.selection_start
        self.old_selection_end = self.editor.selection_end
        
        # Execute paste operation
        self.editor.paste()
    
    def undo(self):
        # Restore previous state
        self.editor.text = self.old_text
        self.editor.selection_start = self.old_selection_start
        self.editor.selection_end = self.old_selection_end

# Invoker
class EditorInvoker:
    def __init__(self):
        self.undo_stack = []
        self.redo_stack = []
    
    def execute_command(self, command):
        command.execute()
        self.undo_stack.append(command)
        # Clear redo stack when new command is executed
        self.redo_stack.clear()
    
    def undo(self):
        if self.undo_stack:
            command = self.undo_stack.pop()
            command.undo()
            self.redo_stack.append(command)
            return True
        return False
    
    def redo(self):
        if self.redo_stack:
            command = self.redo_stack.pop()
            command.execute()
            self.undo_stack.append(command)
            return True
        return False

# Client code
if __name__ == "__main__":
    # Create the receiver
    editor = TextEditor()
    
    # Create the invoker
    invoker = EditorInvoker()
    
    # Simulate user actions
    print("Initial state:", editor.get_text())
    
    # Write some text
    write_cmd = WriteCommand(editor, "Hello, world!")
    invoker.execute_command(write_cmd)
    print("After writing:", editor.get_text())
    
    # Select part of the text
    editor.select(7, 12)
    print(f"Selected text: '{editor.get_selection()}'")
    
    # Replace the selected text
    replace_cmd = WriteCommand(editor, "Python")
    invoker.execute_command(replace_cmd)
    print("After replace:", editor.get_text())
    
    # Undo the replace
    invoker.undo()
    print("After undo:", editor.get_text())
    
    # Redo the replace
    invoker.redo()
    print("After redo:", editor.get_text())
    
    # Select all text
    editor.select(0, len(editor.get_text()))
    
    # Cut the text
    cut_cmd = CutCommand(editor)
    invoker.execute_command(cut_cmd)
    print("After cut:", editor.get_text())
    print("Clipboard:", editor.clipboard)
    
    # Write new text
    write_cmd2 = WriteCommand(editor, "New text")
    invoker.execute_command(write_cmd2)
    print("After writing new text:", editor.get_text())
    
    # Paste the cut text
    editor.select(0, 0)  # Move cursor to beginning
    paste_cmd = PasteCommand(editor)
    invoker.execute_command(paste_cmd)
    print("After paste:", editor.get_text())
    
    # Undo multiple times
    invoker.undo()
    print("After undo paste:", editor.get_text())
    invoker.undo()
    print("After undo paste:", editor.get_text())

```

# Chain of Responsibility Pattern

The Chain of Responsibility pattern passes requests along a chain of handlers, with each handler deciding either to process the request or pass it to the next handler in the chain.

#### Core Concepts:
- **Handler**: Interface declaring a method for handling requests and setting successor
- **ConcreteHandler**: Handles requests it's responsible for; passes others to successor
- **Client**: Initiates requests to the first handler in the chain

#### When to Use:
- When multiple objects might handle a request, and the handler isn't known a priori
- When you want to issue a request to one of several objects without specifying the receiver explicitly
- When the set of objects that can handle a request should be specified dynamically

#### Python Implementation:

```python
from abc import ABC, abstractmethod
from typing import Optional, Dict, Any


class Handler(ABC):
    """Base handler interface"""
    
    @abstractmethod
    def set_next(self, handler: 'Handler') -> 'Handler':
        pass
    
    @abstractmethod
    def handle(self, request: Dict[str, Any]) -> Optional[str]:
        pass


class AbstractHandler(Handler):
    """Base implementation of handler that follows the chain of responsibility pattern"""
    
    _next_handler: Optional[Handler] = None
    
    def set_next(self, handler: Handler) -> Handler:
        self._next_handler = handler
        # Return a handler to allow chaining
        return handler
    
    @abstractmethod
    def handle(self, request: Dict[str, Any]) -> Optional[str]:
        if self._next_handler:
            return self._next_handler.handle(request)
        return None


class AuthenticationHandler(AbstractHandler):
    """Concrete handler for authentication checks"""
    
    def handle(self, request: Dict[str, Any]) -> Optional[str]:
        if not request.get("authenticated"):
            return "Authentication failed: User not authenticated"
        
        print("Authentication successful")
        return super().handle(request)


class AuthorizationHandler(AbstractHandler):
    """Concrete handler for authorization checks"""
    
    def handle(self, request: Dict[str, Any]) -> Optional[str]:
        if request.get("authenticated") and not request.get("authorized"):
            return "Authorization failed: User not authorized"
        
        print("Authorization successful")
        return super().handle(request)


class ValidationHandler(AbstractHandler):
    """Concrete handler for input validation"""
    
    def handle(self, request: Dict[str, Any]) -> Optional[str]:
        data = request.get("data", {})
        
        # Validate required fields
        required_fields = ["username", "action"]
        for field in required_fields:
            if field not in data:
                return f"Validation failed: Missing required field '{field}'"
        
        print("Validation successful")
        return super().handle(request)


class BusinessLogicHandler(AbstractHandler):
    """Concrete handler for processing the business logic"""
    
    def handle(self, request: Dict[str, Any]) -> Optional[str]:
        print(f"Processing business logic for action: {request['data']['action']}")
        return "Request processed successfully"


# Client code
def client_code(handler: Handler) -> None:
    """The client code that uses the chain of handlers"""
    
    # Test case 1: Not authenticated
    request1 = {
        "authenticated": False,
        "authorized": False,
        "data": {"username": "john", "action": "view"}
    }
    
    # Test case 2: Authenticated but not authorized
    request2 = {
        "authenticated": True,
        "authorized": False,
        "data": {"username": "john", "action": "view"}
    }
    
    # Test case 3: Missing required field
    request3 = {
        "authenticated": True,
        "authorized": True,
        "data": {"action": "view"}  # Missing username
    }
    
    # Test case 4: Valid request
    request4 = {
        "authenticated": True,
        "authorized": True,
        "data": {"username": "john", "action": "view"}
    }
    
    requests = [request1, request2, request3, request4]
    
    for i, request in enumerate(requests, 1):
        print(f"\nProcessing request {i}:")
        result = handler.handle(request)
        print(f"Result: {result}\n" + "-" * 30)


def main():
    # Create handlers
    authentication = AuthenticationHandler()
    authorization = AuthorizationHandler()
    validation = ValidationHandler()
    business_logic = BusinessLogicHandler()
    
    # Build the chain
    authentication.set_next(authorization).set_next(validation).set_next(business_logic)
    
    # Process requests
    client_code(authentication)


if __name__ == "__main__":
    main()
```

#### Benefits:
- Reduces coupling between sender and receivers
- Adds flexibility in assigning responsibilities to objects
- Allows you to add or remove responsibilities dynamically
- Follows the Single Responsibility Principle by separating concerns
- Allows handlers to be arranged in different ways

#### Real-world Applications:
- HTTP request processing in web frameworks
- Event handling in UI frameworks
- Exception handling in software
- Logging frameworks with different severity levels
- Authentication and authorization middleware

# State Pattern

The State pattern allows an object to alter its behavior when its internal state changes, appearing to change its class.

#### Core Concepts:
- **Context**: Maintains an instance of ConcreteState that defines the current state
- **State**: Interface defining state-specific behavior
- **ConcreteState**: Implements behavior associated with a particular state

#### When to Use:
- When an object's behavior depends on its state, and it must change behavior at runtime
- When operations have large, multipart conditional statements based on the object's state
- When state transitions are explicit and involve specific rules

#### Python Implementation:

```python
from abc import ABC, abstractmethod
from typing import List


class State(ABC):
    """Abstract base class for all states"""
    
    @abstractmethod
    def insert_coin(self, vending_machine: 'VendingMachine') -> None:
        pass
    
    @abstractmethod
    def eject_coin(self, vending_machine: 'VendingMachine') -> None:
        pass
    
    @abstractmethod
    def select_item(self, vending_machine: 'VendingMachine', item_code: str) -> None:
        pass
    
    @abstractmethod
    def dispense_item(self, vending_machine: 'VendingMachine') -> None:
        pass
    
    @abstractmethod
    def refill(self, vending_machine: 'VendingMachine', items: List[str]) -> None:
        pass


class NoCoinState(State):
    """Concrete state when the machine has no coins inserted"""
    
    def insert_coin(self, vending_machine: 'VendingMachine') -> None:
        print("Coin accepted")
        vending_machine.set_state(vending_machine.has_coin_state)
    
    def eject_coin(self, vending_machine: 'VendingMachine') -> None:
        print("No coin to return")
    
    def select_item(self, vending_machine: 'VendingMachine', item_code: str) -> None:
        print("Please insert a coin first")
    
    def dispense_item(self, vending_machine: 'VendingMachine') -> None:
        print("Please insert a coin first")
    
    def refill(self, vending_machine: 'VendingMachine', items: List[str]) -> None:
        vending_machine.add_items(items)
        print(f"Machine refilled with {len(items)} items")


class HasCoinState(State):
    """Concrete state when the machine has coins inserted"""
    
    def insert_coin(self, vending_machine: 'VendingMachine') -> None:
        print("Coin already inserted, returning extra coin")
    
    def eject_coin(self, vending_machine: 'VendingMachine') -> None:
        print("Coin returned")
        vending_machine.set_state(vending_machine.no_coin_state)
    
    def select_item(self, vending_machine: 'VendingMachine', item_code: str) -> None:
        if vending_machine.has_item(item_code):
            print(f"Item {item_code} selected")
            vending_machine.set_selected_item(item_code)
            vending_machine.set_state(vending_machine.item_selected_state)
        else:
            print(f"Item {item_code} not available")
    
    def dispense_item(self, vending_machine: 'VendingMachine') -> None:
        print("Please select an item first")
    
    def refill(self, vending_machine: 'VendingMachine', items: List[str]) -> None:
        print("Cannot refill while coin is inserted")


class ItemSelectedState(State):
    """Concrete state when an item has been selected"""
    
    def insert_coin(self, vending_machine: 'VendingMachine') -> None:
        print("Coin already inserted, returning extra coin")
    
    def eject_coin(self, vending_machine: 'VendingMachine') -> None:
        print("Coin returned, cancelling selection")
        vending_machine.set_selected_item(None)
        vending_machine.set_state(vending_machine.no_coin_state)
    
    def select_item(self, vending_machine: 'VendingMachine', item_code: str) -> None:
        if vending_machine.has_item(item_code):
            print(f"Changed selection to item {item_code}")
            vending_machine.set_selected_item(item_code)
        else:
            print(f"Item {item_code} not available")
    
    def dispense_item(self, vending_machine: 'VendingMachine') -> None:
        selected_item = vending_machine.get_selected_item()
        if selected_item:
            vending_machine.remove_item(selected_item)
            print(f"Dispensing item {selected_item}")
            vending_machine.set_selected_item(None)
            
            # Check if we have more items
            if vending_machine.is_empty():
                vending_machine.set_state(vending_machine.sold_out_state)
            else:
                vending_machine.set_state(vending_machine.no_coin_state)
    
    def refill(self, vending_machine: 'VendingMachine', items: List[str]) -> None:
        print("Cannot refill while item is selected")


class SoldOutState(State):
    """Concrete state when the machine is out of items"""
    
    def insert_coin(self, vending_machine: 'VendingMachine') -> None:
        print("Machine is sold out, returning coin")
    
    def eject_coin(self, vending_machine: 'VendingMachine') -> None:
        print("No coin inserted")
    
    def select_item(self, vending_machine: 'VendingMachine', item_code: str) -> None:
        print("Machine is sold out")
    
    def dispense_item(self, vending_machine: 'VendingMachine') -> None:
        print("Machine is sold out")
    
    def refill(self, vending_machine: 'VendingMachine', items: List[str]) -> None:
        vending_machine.add_items(items)
        print(f"Machine refilled with {len(items)} items")
        vending_machine.set_state(vending_machine.no_coin_state)


class VendingMachine:
    """Context class that maintains the current state"""
    
    def __init__(self):
        # Initialize all possible states
        self.no_coin_state = NoCoinState()
        self.has_coin_state = HasCoinState()
        self.item_selected_state = ItemSelectedState()
        self.sold_out_state = SoldOutState()
        
        # Set initial state
        self._state = self.sold_out_state
        
        # Initialize inventory
        self._inventory = {}
        self._selected_item = None
    
    def set_state(self, state: State) -> None:
        """Change the current state"""
        self._state = state
    
    def insert_coin(self) -> None:
        """Forward the insert_coin request to the current state"""
        self._state.insert_coin(self)
    
    def eject_coin(self) -> None:
        """Forward the eject_coin request to the current state"""
        self._state.eject_coin(self)
    
    def select_item(self, item_code: str) -> None:
        """Forward the select_item request to the current state"""
        self._state.select_item(self, item_code)
    
    def dispense_item(self) -> None:
        """Forward the dispense_item request to the current state"""
        self._state.dispense_item(self)
    
    def refill(self, items: List[str]) -> None:
        """Forward the refill request to the current state"""
        self._state.refill(self, items)
    
    def add_items(self, items: List[str]) -> None:
        """Helper method to add items to inventory"""
        for item in items:
            if item in self._inventory:
                self._inventory[item] += 1
            else:
                self._inventory[item] = 1
    
    def remove_item(self, item_code: str) -> None:
        """Helper method to remove an item from inventory"""
        if item_code in self._inventory and self._inventory[item_code] > 0:
            self._inventory[item_code] -= 1
            if self._inventory[item_code] == 0:
                del self._inventory[item_code]
    
    def has_item(self, item_code: str) -> bool:
        """Helper method to check if an item is available"""
        return item_code in self._inventory and self._inventory[item_code] > 0
    
    def is_empty(self) -> bool:
        """Helper method to check if the machine is empty"""
        return not self._inventory
    
    def set_selected_item(self, item_code: str) -> None:
        """Helper method to set the selected item"""
        self._selected_item = item_code
    
    def get_selected_item(self) -> str:
        """Helper method to get the selected item"""
        return self._selected_item
    
    def display_inventory(self) -> None:
        """Display the current inventory"""
        if not self._inventory:
            print("Inventory: Empty")
        else:
            print("Inventory:")
            for item, count in self._inventory.items():
                print(f"  {item}: {count}")


# Client code
def main():
    # Create a vending machine
    vending_machine = VendingMachine()
    
    # Display initial state
    print("=== Initial State ===")
    vending_machine.display_inventory()
    
    # Refill the machine
    print("\n=== Refilling Machine ===")
    vending_machine.refill(["A1", "A2", "B1", "B1", "C1"])
    vending_machine.display_inventory()
    
    # Test the machine
    print("\n=== Testing Machine ===")
    vending_machine.select_item("A1")  # Should fail (no coin)
    vending_machine.insert_coin()
    vending_machine.select_item("X1")  # Should fail (not available)
    vending_machine.select_item("A1")
    vending_machine.dispense_item()
    vending_machine.display_inventory()
    
    print("\n=== Another Transaction ===")
    vending_machine.insert_coin()
    vending_machine.select_item("B1")
    vending_machine.eject_coin()  # Cancel transaction
    vending_machine.display_inventory()
    
    print("\n=== Last Items ===")
    vending_machine.insert_coin()
    vending_machine.select_item("B1")
    vending_machine.dispense_item()
    vending_machine.insert_coin()
    vending_machine.select_item("A2")
    vending_machine.dispense_item()
    vending_machine.insert_coin()
    vending_machine.select_item("C1")
    vending_machine.dispense_item()
    vending_machine.display_inventory()
    
    # Try to use the empty machine
    print("\n=== Empty Machine ===")
    vending_machine.insert_coin()
    vending_machine.select_item("A1")
    
    # Refill and try again
    print("\n=== Refill and Try Again ===")
    vending_machine.refill(["D1", "D2"])
    vending_machine.display_inventory()
    vending_machine.insert_coin()
    vending_machine.select_item("D1")
    vending_machine.dispense_item()
    vending_machine.display_inventory()


if __name__ == "__main__":
    main()
```

#### Benefits:
- Eliminates conditional statements by encapsulating state-specific behavior
- Makes state transitions explicit and controlled
- Localizes state-specific behavior in separate classes
- Makes adding new states easier without modifying existing states
- Improves code organization and readability

#### Real-world Applications:
- Order processing systems (new, paid, shipped, delivered states)
- Document workflow systems
- Game character behavior based on status (normal, powered-up, injured)
- UI controls that change appearance based on state
- Connection management in networking

# Template Method Pattern

The Template Method pattern defines the skeleton of an algorithm in a method, deferring some steps to subclasses. It lets subclasses redefine certain steps without changing the algorithm's structure.

#### Core Concepts:
- **AbstractClass**: Defines template methods and abstract operations
- **ConcreteClass**: Implements the abstract operations

#### When to Use:
- When you want to implement the invariant parts of an algorithm once and leave it up to subclasses to implement the variant parts
- When common behavior among subclasses should be factored and localized to avoid code duplication
- To control subclasses extensions by letting them override only specific steps

#### Python Implementation:

```python
from abc import ABC, abstractmethod
import time


class DataProcessor(ABC):
    """Abstract class that defines the template method pattern"""
    
    def process_data(self, data):
        """Template method defining the algorithm structure"""
        start_time = time.time()
        
        # Step 1: Validate the data
        self.validate_data(data)
        
        # Step 2: Preprocess the data
        processed_data = self.preprocess_data(data)
        
        # Step 3: Transform the data
        transformed_data = self.transform_data(processed_data)
        
        # Step 4: Analyze the data
        results = self.analyze_data(transformed_data)
        
        # Step 5: Format the results
        formatted_results = self.format_results(results)
        
        # Hook method - optional override
        self.additional_formatting(formatted_results)
        
        end_time = time.time()
        print(f"Processing completed in {end_time - start_time:.2f} seconds")
        
        return formatted_results
    
    def validate_data(self, data):
        """Default implementation for validating data"""
        if not data:
            raise ValueError("Data cannot be empty")
        print("Base validation complete")
    
    @abstractmethod
    def preprocess_data(self, data):
        """Abstract method to be implemented by subclasses"""
        pass
    
    @abstractmethod
    def transform_data(self, data):
        """Abstract method to be implemented by subclasses"""
        pass
    
    @abstractmethod
    def analyze_data(self, data):
        """Abstract method to be implemented by subclasses"""
        pass
    
    @abstractmethod
    def format_results(self, results):
        """Abstract method to be implemented by subclasses"""
        pass
    
    def additional_formatting(self, formatted_results):
        """Hook method - optional override by subclasses"""
        # Default implementation does nothing
        pass


class NumericDataProcessor(DataProcessor):
    """Concrete class implementing the template method for numeric data"""
    
    def validate_data(self, data):
        """Override validation to ensure numeric data"""
        super().validate_data(data)
        
        if not all(isinstance(item, (int, float)) for item in data):
            raise ValueError("All items must be numeric")
        
        print("Numeric validation complete")
    
    def preprocess_data(self, data):
        """Preprocess numeric data"""
        print("Preprocessing numeric data...")
        # Remove outliers (simplified example)
        mean = sum(data) / len(data)
        std_dev = (sum((x - mean) ** 2 for x in data) / len(data)) ** 0.5
        threshold = 2 * std_dev
        
        filtered_data = [x for x in data if abs(x - mean) <= threshold]
        print(f"Removed {len(data) - len(filtered_data)} outliers")
        
        return filtered_data
    
    def transform_data(self, data):
        """Transform numeric data"""
        print("Transforming numeric data...")
        # Normalize data to [0, 1] range
        if not data:
            return []
        
        min_val = min(data)
        max_val = max(data)
        range_val = max_val - min_val if max_val != min_val else 1
        
        normalized = [(x - min_val) / range_val for x in data]
        print("Data normalized")
        
        return normalized
    
    def analyze_data(self, data):
        """Analyze numeric data"""
        print("Analyzing numeric data...")
        if not data:
            return {"mean": 0, "median": 0, "std_dev": 0, "min": 0, "max": 0}
        
        # Calculate basic statistics
        mean = sum(data) / len(data)
        sorted_data = sorted(data)
        mid = len(sorted_data) // 2
        median = sorted_data[mid] if len(sorted_data) % 2 else (sorted_data[mid-1] + sorted_data[mid]) / 2
        std_dev = (sum((x - mean) ** 2 for x in data) / len(data)) ** 0.5
        
        results = {
            "mean": mean,
            "median": median,
            "std_dev": std_dev,
            "min": min(data),
            "max": max(data)
        }
        
        print("Analysis complete")
        return results
    
    def format_results(self, results):
        """Format numeric results"""
        print("Formatting numeric results...")
        formatted = {}
        
        for key, value in results.items():
            formatted[key] = round(value, 4)
        
        return formatted
    
    def additional_formatting(self, formatted_results):
        """Optional hook method implementation"""
        print("Adding additional formatting...")
        formatted_results["range"] = formatted_results["max"] - formatted_results["min"]


class TextDataProcessor(DataProcessor):
    """Concrete class implementing the template method for text data"""
    
    def validate_data(self, data):
        """Override validation to ensure text data"""
        super().validate_data(data)
        
        if not all(isinstance(item, str) for item in data):
            raise ValueError("All items must be strings")
        
        print("Text validation complete")
    
    def preprocess_data(self, data):
        """Preprocess text data"""
        print("Preprocessing text data...")
        # Convert to lowercase and remove leading/trailing whitespace
        cleaned = [text.lower().strip() for text in data]
        
        # Remove empty strings
        cleaned = [text for text in cleaned if text]
        
        print(f"Cleaned {len(data) - len(cleaned)} text entries")
        return cleaned
    
    def transform_data(self, data):
        """Transform text data"""
        print("Transforming text data...")
        # Count word frequencies
        word_freq = {}
        
        for text in data:
            words = text.split()
            for word in words:
                # Remove punctuation (simplified)
                word = word.strip(".,!?;:()'\"")
                if word:
                    word_freq[word] = word_freq.get(word, 0) + 1
        
        print(f"Found {len(word_freq)} unique words")
        return word_freq
    
    def analyze_data(self, word_freq):
        """Analyze text data"""
        print("Analyzing text data...")
        if not word_freq:
            return {"total_words": 0, "unique_words": 0, "top_words": []}
        
        # Sort words by frequency
        sorted_words = sorted(word_freq.items(), key=lambda x: x[1], reverse=True)
        top_words = sorted_words[:10]
        
        results = {
            "total_words": sum(word_freq.values()),
            "unique_words": len(word_freq),
            "top_words": top_words
        }
        
        print("Analysis complete")
        return results
    
    def format_results(self, results):
        """Format text results"""
        print("Formatting text results...")
        formatted = {
            "total_words": results["total_words"],
            "unique_words": results["unique_words"],
            "top_words": [{"word": word, "count": count} for word, count in results["top_words"]]
        }
        
        return formatted


# Client code
def main():
    # Test with numeric data
    print("=== Processing Numeric Data ===")
    numeric_processor = NumericDataProcessor()
    numeric_data = [12, 5, 7, 15, 3, 9, 100, 2, 8, 10]
    try:
        numeric_results = numeric_processor.process_data(numeric_data)
        print("\nResults:")
        for key, value in numeric_results.items():
            print(f"{key}: {value}")
    except ValueError as e:
        print(f"Error: {e}")
    
    print("\n" + "=" * 50 + "\n")
    
    # Test with text data
    print("=== Processing Text Data ===")
    text_processor = TextDataProcessor()
    text_data = [
        "The quick brown fox jumps over the lazy dog",
        "A quick movement of the enemy will jeopardize five gunboats",
        "How vexingly quick daft zebras jump!",
        "Pack my box with five dozen liquor jugs",
        "The five boxing wizards jump quickly",
        ""  # Empty string to test preprocessing
    ]
    try:
        text_results = text_processor.process_data(text_data)
        print("\nResults:")
        print(f"Total words: {text_results['total_words']}")
        print(f"Unique words: {text_results['unique_words']}")
        print("Top words:")
        for word_info in text_results['top_words']:
            print(f"  {word_info['word']}: {word_info['count']}")
    except ValueError as e:
        print(f"Error: {e}")


if __name__ == "__main__":
    main()
```

#### Benefits:
- Reuses common code in superclass
- Allows subclasses to override only specific parts of a large algorithm
- Enforces a standardized structure while allowing variability
- Follows the Hollywood Principle: "Don't call us, we'll call you"
- Isolates and centralizes the skeleton algorithm

#### Real-world Applications:
- Data processing pipelines
- Document generation frameworks
- Build automation tools
- Testing frameworks
- Web application frameworks with lifecycle hooks

## Moderately Used Behavioral Patterns

# Iterator Pattern

The Iterator pattern provides a way to access elements of an aggregate object sequentially without exposing its underlying representation.

#### Core Concepts:
- **Iterator**: Interface defining methods for traversing a collection
- **ConcreteIterator**: Implements Iterator interface for specific collection
- **Aggregate**: Interface defining a method for creating an Iterator
- **ConcreteAggregate**: Implements Aggregate interface and returns an Iterator

#### When to Use:
- When you need to access a collection's contents without exposing its internal structure
- When you want to provide a uniform interface for traversing different collection types
- When you need to support multiple traversal methods for a collection

#### Python Implementation:

```python
from abc import ABC, abstractmethod
from typing import Any, List, Optional, Dict


class Iterator(ABC):
    """Abstract iterator interface"""
    
    @abstractmethod
    def has_next(self) -> bool:
        """Return True if iteration has more elements"""
        pass
    
    @abstractmethod
    def next(self) -> Any:
        """Return the next element in the iteration"""
        pass


class Aggregate(ABC):
    """Abstract collection interface"""
    
    @abstractmethod
    def create_iterator(self) -> Iterator:
        """Create an iterator for the collection"""
        pass


class Book:
    """Simple book class for demonstration"""
    
    def __init__(self, title: str, author: str, genre: str):
        self.title = title
        self.author = author
        self.genre = genre
    
    def __str__(self) -> str:
        return f"{self.title} by {self.author} ({self.genre})"


class BookIterator(Iterator):
    """Concrete iterator for traversing books"""
    
    def __init__(self, books: List[Book]):
        self._books = books
        self._position = 0
    
    def has_next(self) -> bool:
        return self._position < len(self._books)
    
    def next(self) -> Book:
        if not self.has_next():
            raise StopIteration("No more books")
        
        book = self._books[self._position]
        self._position += 1
        return book


class GenreIterator(Iterator):
    """Concrete iterator for traversing books of a specific genre"""
    
    def __init__(self, books: List[Book], genre: str):
        self._books = books
        self._genre = genre
        self._position = 0
    
    def has_next(self) -> bool:
        while self._position < len(self._books):
            if self._books[self._position].genre == self._genre:
                return True
            self._position += 1
        return False
    
    def next(self) -> Book:
        if not self.has_next():
            raise StopIteration(f"No more books in genre: {self._genre}")
        
        book = self._books[self._position]
        self._position += 1
        return book


class AuthorIterator(Iterator):
    """Concrete iterator for traversing books by a specific author"""
    
    def __init__(self, books: List[Book], author: str):
        self._books = books
        self._author = author
        self._position = 0
    
    def has_next(self) -> bool:
        while self._position < len(self._books):
            if self._books[self._position].author == self._author:
                return True
            self._position += 1
        return False
    
    def next(self) -> Book:
        if not self.has_next():
            raise StopIteration(f"No more books by author: {self._author}")
        
        book = self._books[self._position]
        self._position += 1
        return book


class BookCollection(Aggregate):
    """Concrete collection that stores books"""
    
    def __init__(self):
        self._books: List[Book] = []
    
    def add_book(self, book: Book) -> None:
        self._books.append(book)
    
    def create_iterator(self) -> Iterator:
        """Create a default iterator"""
        return BookIterator(self._books)
    
    def create_genre_iterator(self, genre: str) -> Iterator:
        """Create a genre-specific iterator"""
        return GenreIterator(self._books, genre)
    
    def create_author_iterator(self, author: str) -> Iterator:
        """Create an author-specific iterator"""
        return AuthorIterator(self._books, author)


# Python's built-in iteration support
class IterableBookCollection(BookCollection):
    """Book collection that supports Python's iteration protocol"""
    
    def __iter__(self):
        """Return a Python iterator"""
        self._position = 0
        return self
    
    def __next__(self) -> Book:
        """Get the next book for Python iteration"""
        if self._position >= len(self._books):
            raise StopIteration
        
        book = self._books[self._position]
        self._position += 1
        return book
    
    def filter_by_genre(self, genre: str):
        """Return a generator for books of a specific genre"""
        for book in self._books:
            if book.genre == genre:
                yield book
    
    def filter_by_author(self, author: str):
        """Return a generator for books by a specific author"""
        for book in self._books:
            if book.author == author:
                yield book


# Client code
def main():
    # Create a book collection
    collection = BookCollection()
    
    # Add books
    collection.add_book(Book("The Great Gatsby", "F. Scott Fitzgerald", "Fiction"))
    collection.add_book(Book("To Kill a Mockingbird", "Harper Lee", "Fiction"))
    collection.add_book(Book("1984", "George Orwell", "Science Fiction"))
    collection.add_book(Book("Animal Farm", "George Orwell", "Fiction"))
    collection.add_book(Book("The Hobbit", "J.R.R. Tolkien", "Fantasy"))
    collection.add_book(Book("The Lord of the Rings", "J.R.R. Tolkien", "Fantasy"))
    collection.add_book(Book("Brave New World", "Aldous Huxley", "Science Fiction"))
    
    # Use the default iterator
    print("=== All Books ===")
    iterator = collection.create_iterator()
    while iterator.has_next():
        book = iterator.next()
        print(book)
    
    # Use the genre iterator
    print("\n=== Fantasy Books ===")
    genre_iterator = collection.create_genre_iterator("Fantasy")
    while genre_iterator.has_next():
        book = genre_iterator.next()
        print(book)
    
    # Use the author iterator
    print("\n=== Books by George Orwell ===")
    author_iterator = collection.create_author_iterator("George Orwell")
    while author_iterator.has_next():
        book = author_iterator.next()
        print(book)
    
    # Using Python's built-in iteration support
    print("\n=== Using Python's Built-in Iteration ===")
    python_collection = IterableBookCollection()
    
    # Add the same books
    python_collection.add_book(Book("The Great Gatsby", "F. Scott Fitzgerald", "Fiction"))
    python_collection.add_book(Book("To Kill a Mockingbird", "Harper Lee", "Fiction"))
    python_collection.add_book(Book("1984", "George Orwell", "Science Fiction"))
    python_collection.add_book(Book("Animal Farm", "George Orwell", "Fiction"))
    python_collection.add_book(Book("The Hobbit", "J.R.R. Tolkien", "Fantasy"))
    python_collection.add_book(Book("The Lord of the Rings", "J.R.R. Tolkien", "Fantasy"))
    python_collection.add_book(Book("Brave New World", "Aldous Huxley", "Science Fiction"))
    
    # Iterate using Python's for loop
    print("=== All Books (Python iteration) ===")
    for book in python_collection:
        print(book)
    
    # Use genre filter
    print("\n=== Science Fiction Books (Python filter) ===")
    for book in python_collection.filter_by_genre("Science Fiction"):
        print(book)
    
    # Use author filter
    print("\n=== Books by J.R.R. Tolkien (Python filter) ===")
    for book in python_collection.filter_by_author("J.R.R. Tolkien"):
        print(book)


if __name__ == "__main__":
    main()
```

#### Benefits:
- Simplifies the client interface for traversing collections
- Supports different traversal algorithms without changing collection interfaces
- Enables parallel traversal of the same collection
- Isolates the traversal algorithm from the collection implementation
- Makes collections easier to test and maintain

#### Real-world Applications:
- Database result set iteration
- Directory/file traversal in file systems
- Navigation in user interfaces
- Processing items in collections without exposing internal structure
- Tree traversal algorithms

# Mediator Pattern

The Mediator pattern defines an object that encapsulates how a set of objects interact. It promotes loose coupling by keeping objects from referring to each other explicitly.

#### Core Concepts:
- **Mediator**: Interface defining communication between Colleague objects
- **ConcreteMediator**: Implements Mediator interface, coordinating Colleague objects
- **Colleague**: Class that communicates with other Colleagues through the Mediator
- **ConcreteColleague**: Implements Colleague interface

#### When to Use:
- When a set of objects communicate in well-defined but complex ways
- When reusing an object is difficult because it references many other objects
- When behavior distributed among several classes should be customizable without subclassing

#### Python Implementation:

```python
from abc import ABC, abstractmethod
from typing import Dict, List, Any, Optional


class Colleague(ABC):
    """Abstract base class for all colleagues"""
    
    def __init__(self, mediator: 'Mediator', id: str):
        self._mediator = mediator
        self._id = id
        mediator.add_colleague(self)
    
    def get_id(self) -> str:
        """Get the unique identifier for this colleague"""
        return self._id
    
    @abstractmethod
    def send(self, message: str, to_id: Optional[str] = None) -> None:
        """Send a message to other colleagues via the mediator"""
        pass
    
    @abstractmethod
    def receive(self, message: str, from_id: str) -> None:
        """Receive a message from the mediator"""
        pass


class Mediator(ABC):
    """Abstract mediator interface"""
    
    @abstractmethod
    def add_colleague(self, colleague: Colleague) -> None:
        """Register a colleague with this mediator"""
        pass
    
    @abstractmethod
    def send_message(self, message: str, from_id: str, to_id: Optional[str] = None) -> None:
        """Send a message between colleagues"""
        pass


class ChatMediator(Mediator):
    """Concrete mediator implementation for chat system"""
    
    def __init__(self):
        self._colleagues: Dict[str, Colleague] = {}
        self._history: List[Dict[str, Any]] = []
    
    def add_colleague(self, colleague: Colleague) -> None:
        """Register a colleague with this mediator"""
        colleague_id = colleague.get_id()
        if colleague_id in self._colleagues:
            raise ValueError(f"Colleague with ID {colleague_id} already exists")
        
        self._colleagues[colleague_id] = colleague
        print(f"Mediator: Colleague {colleague_id} registered")
    
    def remove_colleague(self, colleague_id: str) -> None:
        """Remove a colleague from this mediator"""
        if colleague_id in self._colleagues:
            del self._colleagues[colleague_id]
            print(f"Mediator: Colleague {colleague_id} removed")
    
    def send_message(self, message: str, from_id: str, to_id: Optional[str] = None) -> None:
        """Send a message between colleagues"""
        if from_id not in self._colleagues:
            raise ValueError(f"Unknown sender: {from_id}")
        
        # Record message in history
        self._history.append({
            "from": from_id,
            "to": to_id,
            "message": message
        })
        
        if to_id is None:
            # Broadcast to all colleagues except sender
            print(f"Mediator: Broadcasting message from {from_id}")
            for colleague_id, colleague in self._colleagues.items():
                if colleague_id != from_id:
                    colleague.receive(message, from_id)
        else:
            # Direct message to specific colleague
            if to_id not in self._colleagues:
                raise ValueError(f"Unknown recipient: {to_id}")
            
            print(f"Mediator: Sending message from {from_id} to {to_id}")
            self._colleagues[to_id].receive(message, from_id)
    
    def display_history(self) -> None:
        """Display the message history"""
        print("\n=== Message History ===")
        for i, entry in enumerate(self._history, 1):
            if entry["to"] is None:
                print(f"{i}. {entry['from']} (broadcast): {entry['message']}")
            else:
                print(f"{i}. {entry['from']} to {entry['to']}: {entry['message']}")


class User(Colleague):
    """Concrete colleague representing a chat user"""
    
    def __init__(self, mediator: Mediator, id: str, name: str):
        super().__init__(mediator, id)
        self._name = name
        self._messages: List[Dict[str, Any]] = []
    
    def get_name(self) -> str:
        """Get the user's display name"""
        return self._name
    
    def send(self, message: str, to_id: Optional[str] = None) -> None:
        """Send a message to other users via the mediator"""
        print(f"{self._name} sending message: {message}")
        if to_id:
            print(f"(privately to {to_id})")
        
        self._mediator.send_message(message, self._id, to_id)
    
    def receive(self, message: str, from_id: str) -> None:
        """Receive a message from the mediator"""
        self._messages.append({
            "from": from_id,
            "message": message
        })
        print(f"{self._name} received message from {from_id}: {message}")
    
    def display_messages(self) -> None:
        """Display all messages received by this user"""
        print(f"\n=== {self._name}'s Messages ===")
        if not self._messages:
            print("No messages")
            return
        
        for i, message in enumerate(self._messages, 1):
            print(f"{i}. From {message['from']}: {message['message']}")


class ChatRoom:
    """Higher-level facade for chat system"""
    
    def __init__(self, name: str):
        self._name = name
        self._mediator = ChatMediator()
        self._users: Dict[str, User] = {}
        print(f"Chat room '{name}' created")
    
    def add_user(self, user_id: str, name: str) -> User:
        """Add a user to the chat room"""
        if user_id in self._users:
            raise ValueError(f"User ID {user_id} already exists")
        
        user = User(self._mediator, user_id, name)
        self._users[user_id] = user
        print(f"User {name} (ID: {user_id}) joined {self._name}")
        return user
    
    def remove_user(self, user_id: str) -> None:
        """Remove a user from the chat room"""
        if user_id in self._users:
            user = self._users[user_id]
            del self._users[user_id]
            self._mediator.remove_colleague(user_id)
            print(f"User {user.get_name()} (ID: {user_id}) left {self._name}")
    
    def display_users(self) -> None:
        """Display all users in the chat room"""
        print(f"\n=== Users in {self._name} ===")
        if not self._users:
            print("No users")
            return
        
        for user_id, user in self._users.items():
            print(f"ID: {user_id}, Name: {user.get_name()}")
    
    def display_history(self) -> None:
        """Display the message history"""
        self._mediator.display_history()


# Client code
def main():
    # Create a chat room
    chat_room = ChatRoom("Python Developers")
    
    # Add users
    alice = chat_room.add_user("alice123", "Alice")
    bob = chat_room.add_user("bob456", "Bob")
    charlie = chat_room.add_user("charlie789", "Charlie")
    dave = chat_room.add_user("dave101", "Dave")
    
    # Display users
    chat_room.display_users()
    
    # Send messages
    print("\n=== Sending Messages ===")
    alice.send("Hello everyone!", None)  # Broadcast
    bob.send("Hi Alice!", "alice123")    # Direct message
    charlie.send("Is everyone ready for the meeting?", None)  # Broadcast
    dave.send("I'll be 5 minutes late", "charlie789")  # Direct message
    alice.send("No problem, Dave", "dave101")  # Direct message
    
    # Display individual message lists
    alice.display_messages()
    bob.display_messages()
    charlie.display_messages()
    dave.display_messages()
    
    # Display chat history
    chat_room.display_history()
    
    # Remove a user
    print("\n=== User Leaving ===")
    chat_room.remove_user("bob456")
    
    # More messages
    print("\n=== More Messages ===")
    alice.send("Where did Bob go?", None)  # Broadcast
    charlie.send("He had to leave", "alice123")  # Direct message
    
    # Final history
    chat_room.display_history()


if __name__ == "__main__":
    main()
```

#### Benefits:
- Reduces the number of connections between objects
- Centralizes control logic in a single component
- Simplifies object protocols by replacing many-to-many with one-to-many
- Abstracts how objects cooperate, making the system more independent
- Makes it easier to modify and extend the system

#### Real-world Applications:
- Chat applications
- Air traffic control systems
- Event management systems
- GUI component communication
- Coordination between services in distributed systems

# Memento Pattern

The Memento pattern captures and externalizes an object's internal state without violating encapsulation, so the object can be restored to this state later.

#### Core Concepts:
- **Originator**: Creates a memento containing a snapshot of its current state
- **Memento**: Stores the internal state of the Originator
- **Caretaker**: Keeps track of the mementos but never modifies them

#### When to Use:
- When you need to create snapshots of an object's state to restore it later
- When direct access to an object's fields, getters, or setters violates encapsulation
- When you need to implement undo/redo functionality

#### Python Implementation:

```python
from abc import ABC, abstractmethod
from datetime import datetime
from typing import Dict, List, Any, Optional
import copy
import json


class Memento:
    """Memento class that stores the state of an object"""
    
    def __init__(self, state: Dict[str, Any], timestamp: Optional[datetime] = None):
        self._state = copy.deepcopy(state)  # Create a deep copy to prevent modifications
        self._timestamp = timestamp or datetime.now()
    
    def get_state(self) -> Dict[str, Any]:
        """Get the stored state"""
        return copy.deepcopy(self._state)  # Return a copy to maintain immutability
    
    def get_timestamp(self) -> datetime:
        """Get the timestamp when the memento was created"""
        return self._timestamp
    
    def get_metadata(self) -> str:
        """Get metadata about this memento for display purposes"""
        return f"Snapshot taken on {self._timestamp.strftime('%Y-%m-%d %H:%M:%S')}"


class TextDocument:
    """Originator class that creates and restores from mementos"""
    
    def __init__(self, title: str = "", content: str = ""):
        self._title = title
        self._content = content
        self._format_settings = {
            "font": "Arial",
            "font_size": 12,
            "line_spacing": 1.15,
            "margins": {"top": 1.0, "right": 1.0, "bottom": 1.0, "left": 1.0}
        }
    
    def set_title(self, title: str) -> None:
        """Set the document title"""
        self._title = title
    
    def get_title(self) -> str:
        """Get the document title"""
        return self._title
    
    def set_content(self, content: str) -> None:
        """Set the document content"""
        self._content = content
    
    def get_content(self) -> str:
        """Get the document content"""
        return self._content
    
    def update_format_setting(self, setting_name: str, value: Any) -> None:
        """Update a format setting"""
        if setting_name in self._format_settings:
            self._format_settings[setting_name] = value
        elif isinstance(self._format_settings.get("margins"), dict) and setting_name in self._format_settings["margins"]:
            self._format_settings["margins"][setting_name] = value
        else:
            raise ValueError(f"Unknown format setting: {setting_name}")
    
    def get_format_settings(self) -> Dict[str, Any]:
        """Get the format settings"""
        return copy.deepcopy(self._format_settings)
    
    def create_memento(self) -> Memento:
        """Create a memento containing the current state"""
        state = {
            "title": self._title,
            "content": self._content,
            "format_settings": copy.deepcopy(self._format_settings)
        }
        return Memento(state)
    
    def restore_from_memento(self, memento: Memento) -> None:
        """Restore state from a memento"""
        state = memento.get_state()
        self._title = state["title"]
        self._content = state["content"]
        self._format_settings = state["format_settings"]
    
    def __str__(self) -> str:
        """String representation of the document"""
        return f"Document: {self._title}\n" + \
               f"Format: {self._format_settings['font']}, {self._format_settings['font_size']}pt, " + \
               f"{self._format_settings['line_spacing']} line spacing\n" + \
               f"Content: {self._content[:50]}{'...' if len(self._content) > 50 else ''}"


class DocumentHistory:
    """Caretaker class that manages document mementos"""
    
    def __init__(self, document: TextDocument):
        self._document = document
        self._history: List[Memento] = []
        self._current_index = -1
    
    def save(self) -> None:
        """Save the current state of the document"""
        # Remove any future history if we're not at the latest state
        if self._current_index < len(self._history) - 1:
            self._history = self._history[:self._current_index + 1]
        
        # Save the current state
        memento = self._document.create_memento()
        self._history.append(memento)
        self._current_index = len(self._history) - 1
        print(f"Document saved. {memento.get_metadata()}")
    
    def undo(self) -> bool:
        """Restore the previous state if available"""
        if self._current_index <= 0:
            print("Cannot undo: No previous state available")
            return False
        
        self._current_index -= 1
        self._document.restore_from_memento(self._history[self._current_index])
        print(f"Undo successful. {self._history[self._current_index].get_metadata()}")
        return True
    
    def redo(self) -> bool:
        """Restore the next state if available"""
        if self._current_index >= len(self._history) - 1:
            print("Cannot redo: No future state available")
            return False
        
        self._current_index += 1
        self._document.restore_from_memento(self._history[self._current_index])
        print(f"Redo successful. {self._history[self._current_index].get_metadata()}")
        return True
    
    def get_history_size(self) -> int:
        """Get the size of the history"""
        return len(self._history)
    
    def display_history(self) -> None:
        """Display the history of changes"""
        print("\n=== Document History ===")
        if not self._history:
            print("No history available")
            return
        
        for i, memento in enumerate(self._history):
            state = memento.get_state()
            prefix = "→ " if i == self._current_index else "  "
            print(f"{prefix}{i + 1}. {memento.get_metadata()}")
            print(f"   Title: {state['title']}")
            content_preview = state['content'][:30].replace('\n', ' ')
            if len(state['content']) > 30:
                content_preview += "..."
            print(f"   Content: {content_preview}")
    
    def get_memento_at(self, index: int) -> Optional[Memento]:
        """Get a specific memento by index"""
        if 0 <= index < len(self._history):
            return self._history[index]
        return None
    
    def restore_to(self, index: int) -> bool:
        """Restore to a specific point in history"""
        if 0 <= index < len(self._history):
            self._current_index = index
            self._document.restore_from_memento(self._history[index])
            print(f"Restored to: {self._history[index].get_metadata()}")
            return True
        
        print("Invalid history index")
        return False
    
    def export_history(self, filename: str) -> bool:
        """Export history to a file"""
        try:
            export_data = []
            for memento in self._history:
                memento_data = {
                    "timestamp": memento.get_timestamp().isoformat(),
                    "state": memento.get_state()
                }
                export_data.append(memento_data)
            
            with open(filename, 'w') as f:
                json.dump(export_data, f, indent=2)
            
            print(f"History exported to {filename}")
            return True
        except Exception as e:
            print(f"Failed to export history: {e}")
            return False


# Client code
def main():
    # Create a text document
    document = TextDocument("My Document", "This is the initial content.")
    history = DocumentHistory(document)
    
    # Save the initial state
    history.save()
    
    # Make some changes and save after each
    document.set_content("This is the initial content. Added more text.")
    history.save()
    
    document.set_title("My Important Document")
    document.update_format_setting("font", "Times New Roman")
    history.save()
    
    document.set_content("Completely rewritten content with new information.")
    document.update_format_setting("font_size", 14)
    history.save()
    
    document.update_format_setting("margins", {"top": 1.5, "right": 1.0, "bottom": 1.5, "left": 1.0})
    history.save()
    
    # Display the current document
    print("\n=== Current Document ===")
    print(document)
    
    # Display history
    history.display_history()
    
    # Test undo and redo
    print("\n=== Testing Undo/Redo ===")
    history.undo()
    print(document)
    
    history.undo()
    print(document)
    
    history.redo()
    print(document)
    
    # Restore to a specific version
    print("\n=== Restore to Specific Version ===")
    history.restore_to(0)  # Restore to the first version
    print(document)
    
    # Make new changes after restoring
    document.set_content("New content after restore.")
    document.update_format_setting("font", "Calibri")
    history.save()
    
    # Display final history
    history.display_history()
    
    # Export history to a file
    history.export_history("document_history.json")


if __name__ == "__main__":
    main()
```

#### Benefits:
- Preserves encapsulation by not exposing the object's internal structure
- Simplifies the originator's code by letting the caretaker maintain the history
- Provides a clean way to implement undo/redo functionality
- Allows storing the state externally without violating encapsulation
- Improves code organization by separating state management from business logic

#### Real-world Applications:
- Text editors with undo/redo capability
- Design tools and graphic editors
- Form state management in web applications
- Transactional systems with rollback capability
- Game state saving

# Visitor Pattern

The Visitor pattern represents an operation to be performed on elements of an object structure. It lets you define a new operation without changing the classes of the elements on which it operates.

#### Core Concepts:
- **Visitor**: Interface declaring visit methods for each element type
- **ConcreteVisitor**: Implements operations defined by the Visitor interface
- **Element**: Interface defining an accept method that takes a visitor
- **ConcreteElement**: Implements the Element interface
- **ObjectStructure**: Collection that can enumerate its elements

#### When to Use:
- When you need to perform operations on objects of different types in a hierarchy
- When an object structure contains many classes with different interfaces
- When you need to add new operations frequently without changing class structures
- When operations need to accumulate state while traversing a structure

#### Python Implementation:
```python
from abc import ABC, abstractmethod
from typing import List, Dict, Any


class Visitor(ABC):
    """Abstract visitor interface"""
    
    @abstractmethod
    def visit_circle(self, circle: 'Circle') -> None:
        pass
    
    @abstractmethod
    def visit_rectangle(self, rectangle: 'Rectangle') -> None:
        pass
    
    @abstractmethod
    def visit_triangle(self, triangle: 'Triangle') -> None:
        pass
    
    @abstractmethod
    def visit_composite(self, composite: 'CompositeShape') -> None:
        pass


class Shape(ABC):
    """Abstract element interface"""
    
    @abstractmethod
    def accept(self, visitor: Visitor) -> None:
        """Accept a visitor to perform an operation"""
        pass


class Circle(Shape):
    """Concrete element representing a circle"""
    
    def __init__(self, x: float, y: float, radius: float):
        self._x = x
        self._y = y
        self._radius = radius
    
    def accept(self, visitor: Visitor) -> None:
        """Accept the visitor"""
        visitor.visit_circle(self)
    
    def get_x(self) -> float:
        return self._x
    
    def get_y(self) -> float:
        return self._y
    
    def get_radius(self) -> float:
        return self._radius


class Rectangle(Shape):
    """Concrete element representing a rectangle"""
    
    def __init__(self, x: float, y: float, width: float, height: float):
        self._x = x
        self._y = y
        self._width = width
        self._height = height
    
    def accept(self, visitor: Visitor) -> None:
        """Accept the visitor"""
        visitor.visit_rectangle(self)
    
    def get_x(self) -> float:
        return self._x
    
    def get_y(self) -> float:
        return self._y
    
    def get_width(self) -> float:
        return self._width
    
    def get_height(self) -> float:
        return self._height


class Triangle(Shape):
    """Concrete element representing a triangle"""
    
    def __init__(self, x1: float, y1: float, x2: float, y2: float, x3: float, y3: float):
        self._x1, self._y1 = x1, y1
        self._x2, self._y2 = x2, y2
        self._x3, self._y3 = x3, y3
    
    def accept(self, visitor: Visitor) -> None:
        """Accept the visitor"""
        visitor.visit_triangle(self)
    
    def get_points(self) -> List[tuple]:
        return [(self._x1, self._y1), (self._x2, self._y2), (self._x3, self._y3)]


class CompositeShape(Shape):
    """Composite element that contains multiple shapes"""
    
    def __init__(self, name: str):
        self._name = name
        self._shapes: List[Shape] = []
    
    def add_shape(self, shape: Shape) -> None:
        """Add a shape to the composite"""
        self._shapes.append(shape)
    
    def get_shapes(self) -> List[Shape]:
        """Get all shapes in the composite"""
        return self._shapes
    
    def get_name(self) -> str:
        """Get the name of the composite"""
        return self._name
    
    def accept(self, visitor: Visitor) -> None:
        """Accept the visitor and pass it to all child shapes"""
        visitor.visit_composite(self)
        
        # Each child also accepts the visitor
        for shape in self._shapes:
            shape.accept(visitor)


class XMLExportVisitor(Visitor):
    """Concrete visitor that exports shapes to XML"""
    
    def __init__(self):
        self._xml_output = ["<document>"]
        self._indent = 2
    
    def visit_circle(self, circle: Circle) -> None:
        """Visit a circle and create XML for it"""
        self._xml_output.append(" " * self._indent + f"<circle>")
        self._indent += 2
        self._xml_output.append(" " * self._indent + f"<center x='{circle.get_x()}' y='{circle.get_y()}' />")
        self._xml_output.append(" " * self._indent + f"<radius>{circle.get_radius()}</radius>")
        self._indent -= 2
        self._xml_output.append(" " * self._indent + "</circle>")
    
    def visit_rectangle(self, rectangle: Rectangle) -> None:
        """Visit a rectangle and create XML for it"""
        self._xml_output.append(" " * self._indent + "<rectangle>")
        self._indent += 2
        self._xml_output.append(" " * self._indent + 
                                f"<position x='{rectangle.get_x()}' y='{rectangle.get_y()}' />")
        self._xml_output.append(" " * self._indent + 
                                f"<dimensions width='{rectangle.get_width()}' height='{rectangle.get_height()}' />")
        self._indent -= 2
        self._xml_output.append(" " * self._indent + "</rectangle>")
    
    def visit_triangle(self, triangle: Triangle) -> None:
        """Visit a triangle and create XML for it"""
        points = triangle.get_points()
        self._xml_output.append(" " * self._indent + "<triangle>")
        self._indent += 2
        self._xml_output.append(" " * self._indent + "<points>")
        for i, (x, y) in enumerate(points, 1):
            self._xml_output.append(" " * (self._indent + 2) + f"<point{i} x='{x}' y='{y}' />")
        self._xml_output.append(" " * self._indent + "</points>")
        self._indent -= 2
        self._xml_output.append(" " * self._indent + "</triangle>")
    
    def visit_composite(self, composite: CompositeShape) -> None:
        """Visit a composite and create XML for it"""
        self._xml_output.append(" " * self._indent + f"<group name='{composite.get_name()}'>")
        self._indent += 2
        # Child shapes will be visited separately by their accept() methods
    
    def get_xml(self) -> str:
        """Get the completed XML document"""
        return "\n".join(self._xml_output + ["</document>"])


class JSONExportVisitor(Visitor):
    """Concrete visitor that exports shapes to JSON"""
    
    def __init__(self):
        self._current_object: Dict[str, Any] = {"shapes": []}
        self._objects_stack: List[Dict[str, Any]] = [self._current_object]
    
    def visit_circle(self, circle: Circle) -> None:
        """Visit a circle and create JSON for it"""
        circle_json = {
            "type": "circle",
            "center": {"x": circle.get_x(), "y": circle.get_y()},
            "radius": circle.get_radius()
        }
        self._current_object["shapes"].append(circle_json)
    
    def visit_rectangle(self, rectangle: Rectangle) -> None:
        """Visit a rectangle and create JSON for it"""
        rectangle_json = {
            "type": "rectangle",
            "position": {"x": rectangle.get_x(), "y": rectangle.get_y()},
            "dimensions": {"width": rectangle.get_width(), "height": rectangle.get_height()}
        }
        self._current_object["shapes"].append(rectangle_json)
    
    def visit_triangle(self, triangle: Triangle) -> None:
        """Visit a triangle and create JSON for it"""
        points = triangle.get_points()
        triangle_json = {
            "type": "triangle",
            "points": [
                {"x": points[0][0], "y": points[0][1]},
                {"x": points[1][0], "y": points[1][1]},
                {"x": points[2][0], "y": points[2][1]}
            ]
        }
        self._current_object["shapes"].append(triangle_json)
    
    def visit_composite(self, composite: CompositeShape) -> None:
        """Visit a composite and create JSON for it"""
        composite_json = {
            "type": "group",
            "name": composite.get_name(),
            "shapes": []
        }
        self._current_object["shapes"].append(composite_json)
        
        # Set current object to the new composite before visiting children
        self._objects_stack.append(self._current_object)
        self._current_object = composite_json
    
    def get_json(self) -> Dict[str, Any]:
        """Get the completed JSON document"""
        return self._objects_stack[0]


class AreaCalculationVisitor(Visitor):
    """Concrete visitor that calculates the area of shapes"""
    
    def __init__(self):
        self._total_area = 0
    
    def visit_circle(self, circle: Circle) -> None:
        """Calculate and add the area of a circle"""
        import math
        area = math.pi * circle.get_radius() ** 2
        print(f"Circle area: {area:.2f}")
        self._total_area += area
    
    def visit_rectangle(self, rectangle: Rectangle) -> None:
        """Calculate and add the area of a rectangle"""
        area = rectangle.get_width() * rectangle.get_height()
        print(f"Rectangle area: {area:.2f}")
        self._total_area += area
    
    def visit_triangle(self, triangle: Triangle) -> None:
        """Calculate and add the area of a triangle"""
        # Using the Shoelace formula (Gauss's area formula)
        points = triangle.get_points()
        x1, y1 = points[0]
        x2, y2 = points[1]
        x3, y3 = points[2]
        
        area = 0.5 * abs((x1 * (y2 - y3) + x2 * (y3 - y1) + x3 * (y1 - y2)))
        print(f"Triangle area: {area:.2f}")
        self._total_area += area
    
    def visit_composite(self, composite: CompositeShape) -> None:
        """Visit a composite shape (area calculation happens at leaf nodes)"""
        print(f"Calculating area for composite: {composite.get_name()}")
    
    def get_total_area(self) -> float:
        """Get the total calculated area"""
        return self._total_area


# Client code
def main():
    # Create shapes
    circle1 = Circle(5, 5, 10)
    circle2 = Circle(20, 20, 3)
    rectangle = Rectangle(10, 10, 20, 15)
    triangle = Triangle(0, 0, 10, 0, 5, 10)
    
    # Create composite shapes
    drawing = CompositeShape("Main Drawing")
    drawing.add_shape(circle1)
    drawing.add_shape(rectangle)
    
    group = CompositeShape("Nested Group")
    group.add_shape(circle2)
    group.add_shape(triangle)
    
    drawing.add_shape(group)
    
    # Use XML export visitor
    print("=== XML Export ===")
    xml_visitor = XMLExportVisitor()
    drawing.accept(xml_visitor)
    xml_output = xml_visitor.get_xml()
    print(xml_output)
    
    # Use JSON export visitor
    print("\n=== JSON Export ===")
    json_visitor = JSONExportVisitor()
    drawing.accept(json_visitor)
    import json
    json_output = json.dumps(json_visitor.get_json(), indent=2)
    print(json_output)
    
    # Use area calculation visitor
    print("\n=== Area Calculation ===")
    area_visitor = AreaCalculationVisitor()
    drawing.accept(area_visitor)
    print(f"Total area: {area_visitor.get_total_area():.2f}")


if __name__ == "__main__":
    main()
```

```python
from abc import ABC, abstractmethod
from typing import List, Dict, Any


class Visitor(ABC):
    """Abstract visitor interface"""
    
    @abstractmethod
    def visit_circle(self, circle: 'Circle') -> None:
        pass
    
    @abstractmethod
    def visit_rectangle(self, rectangle: 'Rectangle') -> None:
        pass
    
    @abstractmethod
    def visit_triangle(self, triangle: 'Triangle') -> None:
        pass
    
    @abstractmethod
    def visit_composite(self, composite: 'CompositeShape') -> None:
        pass


class Shape(ABC):
    """Abstract element interface"""
    
    @abstractmethod
    def accept(self, visitor: Visitor) -> None:
        """Accept a visitor to perform an operation"""
        pass


class Circle(Shape):
    """Concrete element representing a circle"""
    
    def __init__(self, x: float, y: float, radius: float):
        self._x = x
        self._y = y
        self._radius = radius
    
    def accept(self, visitor: Visitor) -> None:
        """Accept the visitor"""
        visitor.visit_circle(self)
    
    def get_x(self) -> float:
        return self._x
    
    def get_y(self) -> float:
        return self._y
    
    def get_radius(self) -> float:
        return self._radius


class Rectangle(Shape):
    """Concrete element representing a rectangle"""
    
    def __init__(self, x: float, y: float, width: float, height: float):
        self._x = x
        self._y = y
        self._width = width
        self._height = height
    
    def accept(self, visitor: Visitor) -> None:
        """Accept the visitor"""
        visitor.visit_rectangle(self)
    
    def get_x(self) -> float:
        return self._x
    
    def get_y(self) -> float:
        return self._y
    
    def get_width(self) -> float:
        return self._width
    
    def get_height(self) -> float:
        return self._height


class Triangle(Shape):
    """Concrete element representing a triangle"""
    
    def __init__(self, x1: float, y1: float, x2: float, y2: float, x3: float, y3: float):
        self._x1, self._y1 = x1, y1
        self._x2, self._y2 = x2, y2
        self._x3, self._y3 = x3, y3
    
    def accept(self, visitor: Visitor) -> None:
        """Accept the visitor"""
        visitor.visit_triangle(self)
    
    def get_points(self) -> List[tuple]:
        return [(self._x1, self._y1), (self._x2, self._y2), (self._x3, self._y3)]


class CompositeShape(Shape):
    """Composite element that contains multiple shapes"""
    
    def __init__(self, name: str):
        self._name = name
        self._shapes: List[Shape] = []
    
    def add_shape(self, shape: Shape) -> None:
        """Add a shape to the composite"""
        self._shapes.append(shape)
    
    def get_shapes(self) -> List[Shape]:
        """Get all shapes in the composite"""
        return self._shapes
    
    def get_name(self) -> str:
        """Get the name of the composite"""
        return self._name
    
    def accept(self, visitor: Visitor) -> None:
        """Accept the visitor and pass it to all child shapes"""
        visitor.visit_composite(self)
        
        # Each child also accepts the visitor
        for shape in self._shapes:
            shape.accept(visitor)


class XMLExportVisitor(Visitor):
    """Concrete visitor that exports shapes to XML"""
    
    def __init__(self):
        self._xml_output = ["<document>"]
        self._indent = 2
    
    def visit_circle(self, circle: Circle) -> None:
        """Visit a circle and create XML for it"""
        self._xml_output.append(" " * self._indent + f"<circle>")
        self._indent += 2
        self._xml_output.append(" " * self._indent + f"<center x='{circle.get_x()}' y='{circle.get_y()}' />")
        self._xml_output.append(" " * self._indent + f"<radius>{circle.get_radius()}</radius>")
        self._indent -= 2
        self._xml_output.append(" " * self._indent + "</circle>")
    
    def visit_rectangle(self, rectangle: Rectangle) -> None:
        """Visit a rectangle and create XML for it"""
        self._xml_output.append(" " * self._indent + "<rectangle>")
        self._indent += 2
        self._xml_output.append(" " * self._indent + 
                                f"<position x='{rectangle.get_x()}' y='{rectangle.get_y()}' />")
        self._xml_output.append(" " * self._indent + 
                                f"<dimensions width='{rectangle.get_width()}' height='{rectangle.get_height()}' />")
        self._indent -= 2
        self._xml_output.append(" " * self._indent + "</rectangle>")
    
    def visit_triangle(self, triangle: Triangle) -> None:
        """Visit a triangle and create XML for it"""
        points = triangle.get_points()
        self._xml_output.append(" " * self._indent + "<triangle>")
        self._indent += 2
        self._xml_output.append(" " * self._indent + "<points>")
        for i, (x, y) in enumerate(points, 1):
            self._xml_output.append(" " * (self._indent + 2) + f"<point{i} x='{x}' y='{y}' />")
        self._xml_output.append(" " * self._indent + "</points>")
        self._indent -= 2
        self._xml_output.append(" " * self._indent + "</triangle>")
    
    def visit_composite(self, composite: CompositeShape) -> None:
        """Visit a composite and create XML for it"""
        self._xml_output.append(" " * self._indent + f"<group name='{composite.get_name()}'>")
        self._indent += 2
        # Child shapes will be visited separately by their accept() methods
    
    def get_xml(self) -> str:
        """Get the completed XML document"""
        return "\n".join(self._xml_output + ["</document>"])


class JSONExportVisitor(Visitor):
    """Concrete visitor that exports shapes to JSON"""
    
    def __init__(self):
        self._current_object: Dict[str, Any] = {"shapes": []}
        self._objects_stack: List[Dict[str, Any]] = [self._current_object]
    
    def visit_circle(self, circle: Circle) -> None:
        """Visit a circle and create JSON for it"""
        circle_json = {
            "type": "circle",
            "center": {"x": circle.get_x(), "y": circle.get_y()},
            "radius": circle.get_radius()
        }
        self._current_object["shapes"].append(circle_json)
    
    def visit_rectangle(self, rectangle: Rectangle) -> None:
        """Visit a rectangle and create JSON for it"""
        rectangle_json = {
            "type": "rectangle",
            "position": {"x": rectangle.get_x(), "y": rectangle.get_y()},
            "dimensions": {"width": rectangle.get_width(), "height": rectangle.get_height()}
        }
        self._current_object["shapes"].append(rectangle_json)
    
    def visit_triangle(self, triangle: Triangle) -> None:
        """Visit a triangle and create JSON for it"""
        points = triangle.get_points()
        triangle_json = {
            "type": "triangle",
            "points": [
                {"x": points[0][0], "y": points[0][1]},
                {"x": points[1][0], "y": points[1][1]},
                {"x": points[2][0], "y": points[2][1]}
            ]
        }
        self._current_object["shapes"].append(triangle_json)
    
    def visit_composite(self, composite: CompositeShape) -> None:
        """Visit a composite and create JSON for it"""
        composite_json = {
            "type": "group",
            "name": composite.get_name(),
            "shapes": []
        }
        self._current_object["shapes"].append(composite_json)
        
        # Set current object to the new composite before visiting children
        self._objects_stack.append(self._current_object)
        self._current_object = composite_json
    
    def get_json(self) -> Dict[str, Any]:
        """Get the completed JSON document"""
        return self._objects_stack[0]


class AreaCalculationVisitor(Visitor):
    """Concrete visitor that calculates the area of shapes"""
    
    def __init__(self):
        self._total_area = 0
    
    def visit_circle(self, circle: Circle) -> None:
        """Calculate and add the area of a circle"""
        import math
        area = math.pi * circle.get_radius() ** 2
        print(f"Circle area: {area:.2f}")
        self._total_area += area
    
    def visit_rectangle(self, rectangle: Rectangle) -> None:
        """Calculate and add the area of a rectangle"""
        area = rectangle.get_width() * rectangle.get_height()
        print(f"Rectangle area: {area:.2f}")
        self._total_area += area
    
    def visit_triangle(self, triangle: Triangle) -> None:
        """Calculate and add the area of a triangle"""
        # Using the Shoelace formula (Gauss's area formula)
        points = triangle.get_points()
        x1, y1 = points[0]
        x2, y2 = points[1]
        x3, y3 = points[2]
        
        area = 0.5 * abs((x1 * (y2 - y3) + x2 * (y3 - y1) + x3 * (y1 - y2)))
        print(f"Triangle area: {area:.2f}")
        self._total_area += area
    
    def visit_composite(self, composite: CompositeShape) -> None:
        """Visit a composite shape (area calculation happens at leaf nodes)"""
        print(f"Calculating area for composite: {composite.get_name()}")
    
    def get_total_area(self) -> float:
        """Get the total calculated area"""
        return self._total_area


# Client code
def main():
    # Create shapes
    circle1 = Circle(5, 5, 10)
    circle2 = Circle(20, 20, 3)
    rectangle = Rectangle(10, 10, 20, 15)
    triangle = Triangle(0, 0, 10, 0, 5, 10)
    
    # Create composite shapes
    drawing = CompositeShape("Main Drawing")
    drawing.add_shape(circle1)
    drawing.add_shape(rectangle)
    
    group = CompositeShape("Nested Group")
    group.add_shape(circle2)
    group.add_shape(triangle)
    
    drawing.add_shape(group)
    
    # Use XML export visitor
    print("=== XML Export ===")
    xml_visitor = XMLExportVisitor()
    drawing.accept(xml_visitor)
    xml_output = xml_visitor.get_xml()
    print(xml_output)
    
    # Use JSON export visitor
    print("\n=== JSON Export ===")
    json_visitor = JSONExportVisitor()
    drawing.accept(json_visitor)
    import json
    json_output = json.dumps(json_visitor.get_json(), indent=2)
    print(json_output)
    
    # Use area calculation visitor
    print("\n=== Area Calculation ===")
    area_visitor = AreaCalculationVisitor()
    drawing.accept(area_visitor)
    print(f"Total area: {area_visitor.get_total_area():.2f}")


if __name__ == "__main__":
    main()

```

Let me continue with the detailed explanation of the remaining patterns.

#### Benefits of the Visitor Pattern:
- Separates operations from object structures
- Makes adding new operations easy without modifying existing classes
- Centralizes related operations in a single visitor class
- Allows visitors to accumulate state when traversing structures
- Enables double dispatch to select the proper operation based on both object types

#### Real-world Applications:
- Document export in different formats (HTML, PDF, etc.)
- Compiler and interpreter design
- UI frameworks for rendering different components
- Structured data processing
- Static code analysis tools

## Less Frequently Used But Still Valuable Patterns

### 9. Template Method Pattern (continued)

#### Additional Applications:
- Data validation frameworks
- Import/export processes
- Report generation systems
- Design systems with "hooks" for extensibility
- API client implementations

### 10. Chain of Responsibility (continued) 

#### Additional Applications:
- Request processing in web frameworks
- Command processing pipelines
- Text parsing and processing
- Document approval workflows
- Error handling hierarchies

## Comparative Analysis and Selection Guide

When choosing between behavioral patterns, consider these factors:

### Communication Style
- **Command**: Communication through encapsulated requests
- **Mediator**: Centralized communication
- **Chain of Responsibility**: Sequential processing
- **Observer**: Publish-subscribe model (not covered as requested)

### State Management
- **State**: Object behavior changes based on internal state
- **Memento**: Captures and restores object state
- **Template Method**: Fixed algorithm structure with customizable steps

### Traversal and Processing
- **Iterator**: Sequential access to collection elements
- **Visitor**: Separate algorithms from object structures

## Decision Tree for Pattern Selection

1. **Do you need to encapsulate method calls?**
   - Yes → Consider **Command**

2. **Does your system have multiple objects communicating in complex ways?**
   - Yes → Consider **Mediator**

3. **Do you need to pass requests through a series of handlers?**
   - Yes → Consider **Chain of Responsibility**

4. **Does your object's behavior change based on internal state?**
   - Yes → Consider **State**

5. **Do you need to save and restore object state?**
   - Yes → Consider **Memento**

6. **Do you need to traverse a collection without exposing its structure?**
   - Yes → Consider **Iterator**

7. **Do you need to define operations on a class hierarchy without modifying the classes?**
   - Yes → Consider **Visitor**

8. **Do you have an algorithm with fixed structure but variable steps?**
   - Yes → Consider **Template Method**

# Practical Implementation Tips

### 1. Pattern Combinations

Behavioral patterns often work well together:
- **Command** + **Memento** for undoable operations
- **State** + **Template Method** for state-specific algorithms
- **Iterator** + **Visitor** for complex traversal operations
- **Chain of Responsibility** + **Command** for flexible request handling

### 2. Performance Considerations

- **Memento** can consume significant memory when storing large states
- **Visitor** requires traversing the entire object structure
- **Chain of Responsibility** might have performance impacts with long chains
- **Mediator** can become a bottleneck if handling too many communications

### 3. Testability

Behavioral patterns generally improve testability:
- **Command** allows testing execution logic separately
- **State** enables testing each state in isolation
- **Template Method** lets you test steps independently
- **Visitor** separates algorithms from data structures for easier testing

## Conclusion

Behavioral design patterns provide powerful solutions for managing object interactions and responsibilities. By understanding these patterns, you can create more flexible, maintainable, and extensible systems.

The patterns covered here represent the most important behavioral patterns in modern software development. Choose the right pattern (or combination of patterns) based on your specific requirements, keeping in mind factors like communication style, state management needs, and traversal requirements.

Remember that patterns are guidelines, not strict rules. Feel free to adapt them to your specific context while preserving their core principles.

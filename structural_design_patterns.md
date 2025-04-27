# Structural Design Patterns

Structural design patterns focus on how classes and objects are composed to form larger structures. They help ensure that when parts of a system change, the entire system doesn't need to change along with them. These patterns use inheritance and composition to assemble objects and classes into larger structures while keeping these structures flexible and efficient.

## Core Structural Design Patterns

Let me walk you through the key structural design patterns, explaining each with Python examples to illustrate how they work in practice.

# 1. Adapter Pattern

The Adapter pattern allows incompatible interfaces to work together. It acts as a bridge between two incompatible interfaces by wrapping an instance of one class into an adapter class that presents the interface expected by clients.

#### When to use:
- When you need to use an existing class with a different interface
- When you want to create a reusable class that cooperates with classes that don't necessarily have compatible interfaces

#### Python Example:

```python
# The interface the client expects to use
class Target:
    def request(self):
        return "Target: The default target's behavior."

# The interface that needs adapting
class Adaptee:
    def specific_request(self):
        return "Adaptee: .tseuqer cificeps eht"

# The Adapter makes the Adaptee's interface compatible with the Target's
class Adapter(Target):
    def __init__(self, adaptee):
        self.adaptee = adaptee
        
    def request(self):
        # Here we're translating one interface to another
        return f"Adapter: (TRANSLATED) {self.adaptee.specific_request()[::-1]}"

# Client code
def client_code(target):
    print(target.request())

# Using the adapter
adaptee = Adaptee()
print("Client: I can work with the Adaptee directly:")
print(f"Adaptee: {adaptee.specific_request()}")

print("\nClient: But I need an Adapter to work with it:")
adapter = Adapter(adaptee)
client_code(adapter)
```

# 2. Bridge Pattern

The Bridge pattern decouples an abstraction from its implementation so that the two can vary independently. It involves an interface acting as a bridge between the abstract class and implementor classes.

#### When to use:
- When you want to avoid a permanent binding between an abstraction and its implementation
- When both the abstractions and their implementations should be extensible through subclasses
- When changes in the implementation shouldn't impact the client code

#### Python Example:

```python
# Implementor interface
class DrawingAPI:
    def draw_circle(self, x, y, radius):
        pass

# Concrete Implementor A
class DrawingAPI1(DrawingAPI):
    def draw_circle(self, x, y, radius):
        return f"API1.circle at {x}:{y} radius {radius}"

# Concrete Implementor B
class DrawingAPI2(DrawingAPI):
    def draw_circle(self, x, y, radius):
        return f"API2.circle at {x}:{y} radius {radius}"

# Abstraction
class Shape:
    def __init__(self, drawing_api):
        self.drawing_api = drawing_api
        
    def draw(self):
        pass
    
    def resize_by_percentage(self, percent):
        pass

# Refined Abstraction
class CircleShape(Shape):
    def __init__(self, x, y, radius, drawing_api):
        super().__init__(drawing_api)
        self.x = x
        self.y = y
        self.radius = radius
        
    def draw(self):
        return self.drawing_api.draw_circle(self.x, self.y, self.radius)
    
    def resize_by_percentage(self, percent):
        self.radius *= percent

# Client code
shapes = [
    CircleShape(1, 2, 3, DrawingAPI1()),
    CircleShape(5, 7, 11, DrawingAPI2())
]

for shape in shapes:
    print(shape.draw())
    
# Resize the first shape and draw again
shapes[0].resize_by_percentage(2.5)
print(shapes[0].draw())
```

# 3. Composite Pattern

The Composite pattern allows you to compose objects into tree structures to represent part-whole hierarchies. It lets clients treat individual objects and compositions of objects uniformly.

#### When to use:
- When you want to represent part-whole hierarchies of objects
- When you want clients to be able to ignore the difference between compositions of objects and individual objects

#### Python Example:

```python
from abc import ABC, abstractmethod

# Component - defines the interface for objects in the composition
class Component(ABC):
    @abstractmethod
    def operation(self):
        pass
        
    def add(self, component):
        pass
        
    def remove(self, component):
        pass
        
    def is_composite(self):
        return False

# Leaf - represents end objects of a composition with no children
class Leaf(Component):
    def __init__(self, name):
        self.name = name
    
    def operation(self):
        return f"Leaf {self.name}"

# Composite - represents a component that may have children
class Composite(Component):
    def __init__(self, name):
        self.name = name
        self.children = []
    
    def add(self, component):
        self.children.append(component)
        return self
    
    def remove(self, component):
        self.children.remove(component)
        return self
    
    def is_composite(self):
        return True
    
    def operation(self):
        results = [child.operation() for child in self.children]
        return f"Branch {self.name} ({'+'.join(results)})"

# Client code
def client_code(component):
    print(f"RESULT: {component.operation()}")

# Using individual leaf components
leaf1 = Leaf("A")
print("Client: Working with a leaf:")
client_code(leaf1)

# Using a composite structure
tree = Composite("Root")
branch1 = Composite("Branch1")
branch2 = Composite("Branch2")

leaf2 = Leaf("B")
leaf3 = Leaf("C")
leaf4 = Leaf("D")

branch1.add(leaf2).add(leaf3)
branch2.add(leaf4)
tree.add(branch1).add(branch2)

print("\nClient: Now working with a complex tree:")
client_code(tree)
```

# 4. Decorator Pattern

The Decorator pattern attaches additional responsibilities to an object dynamically. It provides a flexible alternative to subclassing for extending functionality.

#### When to use:
- When you need to add responsibilities to objects dynamically without affecting other objects
- When extension by subclassing is impractical
- When you want to add behavior that can be withdrawn later

#### Python Example:

```python
from abc import ABC, abstractmethod

# The Component interface - defines operations that can be altered by decorators
class Component(ABC):
    @abstractmethod
    def operation(self):
        pass

# Concrete Component - defines a concrete object to which additional responsibilities can be attached
class ConcreteComponent(Component):
    def operation(self):
        return "ConcreteComponent"

# Decorator - maintains a reference to a Component object and defines an interface that conforms to Component's
class Decorator(Component):
    def __init__(self, component):
        self._component = component
        
    @abstractmethod
    def operation(self):
        pass

# Concrete Decorators - add responsibilities to the component
class ConcreteDecoratorA(Decorator):
    def operation(self):
        return f"ConcreteDecoratorA({self._component.operation()})"

class ConcreteDecoratorB(Decorator):
    def operation(self):
        return f"ConcreteDecoratorB({self._component.operation()})"

# Client code
def client_code(component):
    print(f"RESULT: {component.operation()}")

# Using the basic component
simple = ConcreteComponent()
print("Client: Using the basic component:")
client_code(simple)

# Using decorated components
decorator1 = ConcreteDecoratorA(simple)
decorator2 = ConcreteDecoratorB(decorator1)
print("\nClient: Using the decorated component:")
client_code(decorator2)
```

# 5. Facade Pattern

The Facade pattern provides a unified interface to a set of interfaces in a subsystem. It defines a higher-level interface that makes the subsystem easier to use.

#### When to use:
- When you want to provide a simple interface to a complex subsystem
- When you want to layer your subsystems
- When you want to decouple your client from a complex subsystem

#### Python Example:

```python
# Complex subsystem classes
class CPU:
    def freeze(self):
        return "Freezing processor."
        
    def jump(self, position):
        return f"Jumping to position: {position}"
        
    def execute(self):
        return "Executing commands."

class Memory:
    def load(self, position, data):
        return f"Loading data {data} at position {position}"

class HardDrive:
    def read(self, sector, size):
        return f"Reading {size} bytes from sector {sector}"

# Facade
class ComputerFacade:
    def __init__(self):
        self.cpu = CPU()
        self.memory = Memory()
        self.hard_drive = HardDrive()
        
    def start(self):
        result = []
        result.append(self.cpu.freeze())
        result.append(self.memory.load(0, "boot data"))
        result.append(self.cpu.jump(10))
        result.append(self.cpu.execute())
        return "\n".join(result)

# Client code
computer = ComputerFacade()
print(computer.start())
```

# 6. Flyweight Pattern

The Flyweight pattern minimizes memory usage by sharing as much data as possible with similar objects. It's used to support large numbers of fine-grained objects efficiently.

#### When to use:
- When an application uses a large number of objects
- When storage costs are high because of the quantity of objects
- When most object state can be made extrinsic
- When many groups of objects may be replaced by relatively few shared objects once extrinsic state is removed

#### Python Example:

```python
# Flyweight class - holds intrinsic state
class Character:
    def __init__(self, symbol):
        self.symbol = symbol  # Intrinsic state

    def display(self, size, position_x, position_y):  # Extrinsic state passed as parameters
        return f"Character {self.symbol} of size {size} at position ({position_x}, {position_y})"

# Flyweight Factory - manages flyweight objects
class CharacterFactory:
    _characters = {}
    
    @classmethod
    def get_character(cls, symbol):
        # Return an existing flyweight or create a new one if it doesn't exist
        return cls._characters.setdefault(symbol, Character(symbol))

# Client code
factory = CharacterFactory()

# Using the same character 'A' (shared flyweight)
char_a1 = factory.get_character('A')
char_a2 = factory.get_character('A')

# Different extrinsic state
print(char_a1.display(12, 10, 20))
print(char_a2.display(14, 30, 40))

# Different character 'B' (different flyweight)
char_b = factory.get_character('B')
print(char_b.display(16, 50, 60))

# Verify that we're reusing objects
print(f"Are char_a1 and char_a2 the same object? {char_a1 is char_a2}")
print(f"Are char_a1 and char_b the same object? {char_a1 is char_b}")
```

# 7. Proxy Pattern

The Proxy pattern provides a surrogate or placeholder for another object to control access to it or to add functionality.

#### When to use:
- When you need a more versatile or sophisticated reference to an object than a simple pointer
- Remote proxy: representing an object located remotely
- Virtual proxy: creating expensive objects on demand
- Protection proxy: controlling access to the original object
- Smart reference: performing additional actions when an object is accessed

#### Python Example:

```python
from abc import ABC, abstractmethod

# Subject interface - defines common interface for RealSubject and Proxy
class Subject(ABC):
    @abstractmethod
    def request(self):
        pass

# RealSubject - defines the real object that the proxy represents
class RealSubject(Subject):
    def request(self):
        return "RealSubject: Handling request."

# Proxy - maintains a reference that lets the proxy access the real subject
class Proxy(Subject):
    def __init__(self, real_subject):
        self._real_subject = real_subject
        
    def request(self):
        # Additional behavior before forwarding to the real subject
        if self.check_access():
            # Log the request time
            result = self._real_subject.request()
            self.log_access()
            return result
        else:
            return "Proxy: Access denied."
            
    def check_access(self):
        # Some real checks would go here, this is just a simplified example
        print("Proxy: Checking access prior to firing a real request.")
        return True
        
    def log_access(self):
        print("Proxy: Logging the time of request.")

# Client code
def client_code(subject):
    return subject.request()

# Using the proxy
real_subject = RealSubject()
proxy = Proxy(real_subject)

print(client_code(proxy))
```

## Real-World Applications

Now, let's explore how these patterns are used in real-world software development:

### Adapter Pattern
- Database adapters that enable different database systems to be accessed through a common interface
- Third-party library integration
- Legacy system integration
- In Python's `io` module, which adapts file-like objects

### Bridge Pattern
- GUI frameworks like Qt that separate window systems from application code
- Device drivers that abstract hardware interfaces
- Persistence frameworks that separate database operations from business logic

### Composite Pattern
- File systems where files and directories share common operations
- GUI frameworks for nested components
- Menu systems where items and submenus are treated uniformly

### Decorator Pattern
- Java I/O classes (InputStream, BufferedInputStream, etc.)
- Python's `@decorators` syntax
- Middleware in web frameworks like Django

### Facade Pattern
- Libraries like jQuery that simplify DOM manipulation
- Service-oriented architectures
- API gateways in microservices

### Flyweight Pattern
- Text editors and word processors for character representation
- Game development for shared textures and models
- Factory pattern implementations

### Proxy Pattern
- ORM frameworks like SQLAlchemy
- Remote procedure calls
- Lazy loading of resources
- Caching systems

## Choosing the Right Structural Pattern

To select the appropriate structural pattern, consider the following questions:

1. **Adapter**: Do you need to make existing classes work with others without modifying their source code?
2. **Bridge**: Do you need to separate an abstraction from its implementation?
3. **Composite**: Do you need to work with tree-like object structures?
4. **Decorator**: Do you need to add responsibilities to objects dynamically?
5. **Facade**: Do you need to provide a simplified interface to a complex subsystem?
6. **Flyweight**: Do you need to support a large number of fine-grained objects efficiently?
7. **Proxy**: Do you need to control access to an object, or add functionality when accessing an object?

## Comparing Structural Patterns

| Pattern | Intent | Primary Focus |
|---------|--------|---------------|
| Adapter | Makes incompatible interfaces work together | Interface conversion |
| Bridge | Separates abstraction from implementation | Separation of concerns |
| Composite | Composes objects into tree structures | Object aggregation |
| Decorator | Adds responsibilities to objects dynamically | Extending functionality |
| Facade | Provides a simplified interface to a complex subsystem | Interface simplification |
| Flyweight | Minimizes memory use by sharing objects | Resource optimization |
| Proxy | Controls access to objects | Object access control |

## Implementation Considerations

When implementing structural patterns, keep these considerations in mind:

1. **Interface vs. Abstract Class**: Depending on the language, you may need to choose between interfaces and abstract classes. In Python, you can use abstract base classes from the `abc` module.

2. **Composition Over Inheritance**: Structural patterns like Decorator and Composite favor composition over inheritance, which leads to more flexible designs.

3. **Performance Implications**: Some patterns like Flyweight optimize for memory usage, while others like Proxy might introduce performance overhead.

4. **Complexity Trade-offs**: Patterns like Facade reduce complexity for clients but may add complexity to the system's implementation.

5. **Balancing Flexibility and Complexity**: More flexibility often means more complexity. Assess whether the additional complexity is justified for your specific case.

## Advanced Pattern Combinations

Structural patterns can be combined to solve complex design problems:

1. **Decorator + Composite**: Create a flexible tree structure where each node can have additional responsibilities.

2. **Proxy + Flyweight**: Control access to shared flyweight objects.

3. **Adapter + Bridge**: Adapt multiple implementations through a unified abstraction.

4. **Facade + Adapter**: Simplify access to a system while adapting its interface.

## Conclusion

Structural design patterns provide solutions to common object composition problems. By understanding these patterns, you can create more flexible, reusable, and maintainable software. The key to mastering these patterns is recognizing the contexts in which they apply and understanding their benefits and trade-offs.

Remember that patterns are not rigid rules but guidelines. Feel free to adapt them to your specific context, and don't force a pattern where it doesn't fit naturally. The best design is one that solves your problem while remaining maintainable and understandable.



# Advanced Structural Design Pattern Examples

Let's explore more advanced and real-world examples of the most commonly used structural design patterns, focusing on practical applications that you might encounter in professional software development.

## 1. Decorator Pattern (Advanced Examples)

The Decorator pattern is one of the most widely used structural patterns, especially in frameworks and libraries.

### Authentication Middleware Example

This example shows how decorators can chain authentication and logging functionality around HTTP requests:

```python
from functools import wraps
from datetime import datetime
import json

class Request:
    def __init__(self, headers=None, body=None):
        self.headers = headers or {}
        self.body = body or {}
        self.user = None

class Response:
    def __init__(self, status_code=200, body=None):
        self.status_code = status_code
        self.body = body or {}

# Base component - HTTP Handler
def base_handler(request):
    # Process the request and return a response
    return Response(200, {"message": "Success"})

# Decorator for authentication
def auth_decorator(handler_func):
    @wraps(handler_func)
    def wrapper(request, *args, **kwargs):
        # Check for authentication token
        auth_token = request.headers.get('Authorization')
        if not auth_token or not auth_token.startswith('Bearer '):
            return Response(401, {"error": "Unauthorized - Missing or invalid token"})
        
        # In a real implementation, validate the token and retrieve user
        # For demonstration, we're just simulating a user lookup
        token_value = auth_token.split(' ')[1]
        if token_value == "valid_token":
            request.user = {"id": 123, "name": "John Doe", "role": "admin"}
        else:
            return Response(401, {"error": "Unauthorized - Invalid token"})
            
        # Call the original handler
        return handler_func(request, *args, **kwargs)
    return wrapper

# Decorator for logging
def logging_decorator(handler_func):
    @wraps(handler_func)
    def wrapper(request, *args, **kwargs):
        start_time = datetime.now()
        print(f"[{start_time}] Request received: {request.headers}")
        
        try:
            response = handler_func(request, *args, **kwargs)
            end_time = datetime.now()
            duration = (end_time - start_time).total_seconds() * 1000
            print(f"[{end_time}] Response sent: status={response.status_code}, duration={duration:.2f}ms")
            return response
        except Exception as e:
            end_time = datetime.now()
            duration = (end_time - start_time).total_seconds() * 1000
            print(f"[{end_time}] Error occurred: {str(e)}, duration={duration:.2f}ms")
            raise
    return wrapper

# Decorator for response formatting
def json_response_decorator(handler_func):
    @wraps(handler_func)
    def wrapper(request, *args, **kwargs):
        response = handler_func(request, *args, **kwargs)
        # Add proper content type header
        response.headers = getattr(response, 'headers', {})
        response.headers['Content-Type'] = 'application/json'
        return response
    return wrapper

# Apply decorators to create a decorated handler
@json_response_decorator
@logging_decorator
@auth_decorator
def api_handler(request):
    # This handler assumes authentication is completed
    user = request.user
    return Response(200, {
        "message": f"Hello, {user['name']}!",
        "user_id": user['id'],
        "role": user['role']
    })

# Test the decorated handler
test_request = Request(
    headers={"Authorization": "Bearer valid_token"},
    body={"action": "get_data"}
)

response = api_handler(test_request)
print(f"Response status: {response.status_code}")
print(f"Response body: {response.body}")
```

### File System Decorator

This example shows how to use decorators to add encryption, compression, and validation to file operations:

```python
import os
import zlib
import hashlib
from abc import ABC, abstractmethod
from cryptography.fernet import Fernet

# Generate a key for encryption
def generate_key():
    return Fernet.generate_key()

# Abstract component
class FileHandler(ABC):
    @abstractmethod
    def read(self, filename):
        pass
    
    @abstractmethod
    def write(self, filename, data):
        pass

# Concrete component
class BasicFileHandler(FileHandler):
    def read(self, filename):
        with open(filename, 'rb') as file:
            return file.read()
    
    def write(self, filename, data):
        with open(filename, 'wb') as file:
            file.write(data)
        return len(data)

# Base decorator
class FileHandlerDecorator(FileHandler):
    def __init__(self, file_handler):
        self._file_handler = file_handler
    
    def read(self, filename):
        return self._file_handler.read(filename)
    
    def write(self, filename, data):
        return self._file_handler.write(filename, data)

# Encryption decorator
class EncryptionDecorator(FileHandlerDecorator):
    def __init__(self, file_handler, key=None):
        super().__init__(file_handler)
        self.key = key or generate_key()
        self.cipher = Fernet(self.key)
    
    def read(self, filename):
        encrypted_data = super().read(filename)
        try:
            decrypted_data = self.cipher.decrypt(encrypted_data)
            return decrypted_data
        except Exception as e:
            raise ValueError(f"Decryption failed: {e}")
    
    def write(self, filename, data):
        encrypted_data = self.cipher.encrypt(data)
        return super().write(filename, encrypted_data)

# Compression decorator
class CompressionDecorator(FileHandlerDecorator):
    def __init__(self, file_handler, compression_level=9):
        super().__init__(file_handler)
        self.compression_level = compression_level
    
    def read(self, filename):
        compressed_data = super().read(filename)
        try:
            decompressed_data = zlib.decompress(compressed_data)
            return decompressed_data
        except Exception as e:
            raise ValueError(f"Decompression failed: {e}")
    
    def write(self, filename, data):
        compressed_data = zlib.compress(data, self.compression_level)
        return super().write(filename, compressed_data)

# Checksum validation decorator
class ChecksumDecorator(FileHandlerDecorator):
    def read(self, filename):
        data = super().read(filename)
        # Extract the checksum from the first 32 bytes
        stored_checksum = data[:32]
        actual_data = data[32:]
        
        # Calculate checksum of the data
        calculated_checksum = hashlib.md5(actual_data).digest()
        
        # Verify the checksum
        if stored_checksum != calculated_checksum:
            raise ValueError("Checksum validation failed: data may be corrupted")
        
        return actual_data
    
    def write(self, filename, data):
        # Calculate checksum
        checksum = hashlib.md5(data).digest()
        
        # Prepend checksum to the data
        data_with_checksum = checksum + data
        
        return super().write(filename, data_with_checksum)

# Usage example
if __name__ == "__main__":
    # Create a test file
    test_data = b"This is a test of the file handler system with decorators."
    
    # Create basic file handler
    basic_handler = BasicFileHandler()
    
    # Create a decorated file handler with multiple decorators
    encrypted_compressed_handler = EncryptionDecorator(
        CompressionDecorator(
            ChecksumDecorator(
                basic_handler
            )
        )
    )
    
    # Write and read using the decorated handler
    filename = "test_file.dat"
    
    print(f"Original data size: {len(test_data)} bytes")
    size_written = encrypted_compressed_handler.write(filename, test_data)
    print(f"Size written to file: {size_written} bytes")
    
    # Read the data back
    read_data = encrypted_compressed_handler.read(filename)
    print(f"Read data: {read_data.decode('utf-8')}")
    print(f"Data integrity maintained: {read_data == test_data}")
    
    # Clean up
    os.remove(filename)
```

## 2. Adapter Pattern (Advanced Examples)

The Adapter pattern is extremely useful when integrating different systems or APIs.

### Payment Gateway Integration Example

This example shows how to use adapters to provide a unified interface for multiple payment providers:

```python
from abc import ABC, abstractmethod
import uuid
import requests
from dataclasses import dataclass

@dataclass
class PaymentDetails:
    amount: float
    currency: str
    card_number: str
    expiry_date: str
    cvv: str
    holder_name: str

@dataclass
class TransactionResult:
    transaction_id: str
    success: bool
    error_message: str = None
    provider_reference: str = None

# Target interface that our code will use
class PaymentProcessor(ABC):
    @abstractmethod
    def process_payment(self, payment_details: PaymentDetails) -> TransactionResult:
        pass
    
    @abstractmethod
    def refund_payment(self, transaction_id: str) -> bool:
        pass
    
    @abstractmethod
    def get_transaction_status(self, transaction_id: str) -> str:
        pass

# First payment provider with incompatible interface
class StripeAPI:
    def create_charge(self, token, amount_cents, currency, description=None):
        # In a real implementation, this would call the Stripe API
        print(f"Stripe: Charging {amount_cents/100} {currency}")
        # Simulate successful payment
        return {
            "id": f"ch_{uuid.uuid4().hex}",
            "status": "succeeded",
            "amount": amount_cents,
            "currency": currency
        }
    
    def create_token(self, card_details):
        # In a real implementation, this would call the Stripe API to tokenize card details
        return {"id": f"tok_{uuid.uuid4().hex}"}
    
    def refund_charge(self, charge_id):
        # In a real implementation, this would call the Stripe API
        print(f"Stripe: Refunding charge {charge_id}")
        return {"id": f"re_{uuid.uuid4().hex}", "status": "succeeded"}
    
    def get_charge(self, charge_id):
        # In a real implementation, this would call the Stripe API
        return {"id": charge_id, "status": "succeeded"}

# Second payment provider with a different interface
class PayPalAPI:
    def make_payment(self, payer_info, payment_amount, currency_code):
        # In a real implementation, this would call the PayPal API
        print(f"PayPal: Processing payment of {payment_amount} {currency_code}")
        # Simulate successful payment
        return {
            "payment_id": f"PAY-{uuid.uuid4().hex}",
            "state": "approved",
            "transactions": [{"amount": {"total": payment_amount, "currency": currency_code}}]
        }
    
    def issue_refund(self, payment_id, amount=None):
        # In a real implementation, this would call the PayPal API
        print(f"PayPal: Refunding payment {payment_id}")
        return {"refund_id": f"REF-{uuid.uuid4().hex}", "state": "completed"}
    
    def get_payment_details(self, payment_id):
        # In a real implementation, this would call the PayPal API
        return {"payment_id": payment_id, "state": "approved"}

# Adapter for Stripe
class StripeAdapter(PaymentProcessor):
    def __init__(self):
        self.stripe = StripeAPI()
        self._transaction_map = {}  # Map our transaction IDs to Stripe charge IDs
    
    def process_payment(self, payment_details: PaymentDetails) -> TransactionResult:
        try:
            # Create a token with card details
            token = self.stripe.create_token({
                "number": payment_details.card_number,
                "exp_month": payment_details.expiry_date.split('/')[0],
                "exp_year": payment_details.expiry_date.split('/')[1],
                "cvc": payment_details.cvv,
                "name": payment_details.holder_name
            })
            
            # Create a charge with the token
            amount_cents = int(payment_details.amount * 100)  # Convert to cents
            charge = self.stripe.create_charge(
                token["id"], 
                amount_cents, 
                payment_details.currency
            )
            
            # Generate our transaction ID and map it to the Stripe charge ID
            transaction_id = f"txn_{uuid.uuid4().hex}"
            self._transaction_map[transaction_id] = charge["id"]
            
            return TransactionResult(
                transaction_id=transaction_id,
                success=charge["status"] == "succeeded",
                provider_reference=charge["id"]
            )
        except Exception as e:
            return TransactionResult(
                transaction_id=f"failed_{uuid.uuid4().hex}",
                success=False,
                error_message=str(e)
            )
    
    def refund_payment(self, transaction_id: str) -> bool:
        if transaction_id not in self._transaction_map:
            return False
        
        stripe_charge_id = self._transaction_map[transaction_id]
        try:
            refund = self.stripe.refund_charge(stripe_charge_id)
            return refund["status"] == "succeeded"
        except Exception:
            return False
    
    def get_transaction_status(self, transaction_id: str) -> str:
        if transaction_id not in self._transaction_map:
            return "unknown"
        
        stripe_charge_id = self._transaction_map[transaction_id]
        try:
            charge = self.stripe.get_charge(stripe_charge_id)
            status_map = {
                "succeeded": "completed",
                "pending": "pending",
                "failed": "failed"
            }
            return status_map.get(charge["status"], "unknown")
        except Exception:
            return "error"

# Adapter for PayPal
class PayPalAdapter(PaymentProcessor):
    def __init__(self):
        self.paypal = PayPalAPI()
        self._transaction_map = {}  # Map our transaction IDs to PayPal payment IDs
    
    def process_payment(self, payment_details: PaymentDetails) -> TransactionResult:
        try:
            # Create payer info from card details
            payer_info = {
                "credit_card": {
                    "number": payment_details.card_number,
                    "expire_month": payment_details.expiry_date.split('/')[0],
                    "expire_year": payment_details.expiry_date.split('/')[1],
                    "cvv2": payment_details.cvv,
                    "first_name": payment_details.holder_name.split()[0],
                    "last_name": payment_details.holder_name.split()[-1] if len(payment_details.holder_name.split()) > 1 else ""
                }
            }
            
            # Make the payment
            payment = self.paypal.make_payment(
                payer_info,
                payment_details.amount,
                payment_details.currency
            )
            
            # Generate our transaction ID and map it to the PayPal payment ID
            transaction_id = f"txn_{uuid.uuid4().hex}"
            self._transaction_map[transaction_id] = payment["payment_id"]
            
            return TransactionResult(
                transaction_id=transaction_id,
                success=payment["state"] == "approved",
                provider_reference=payment["payment_id"]
            )
        except Exception as e:
            return TransactionResult(
                transaction_id=f"failed_{uuid.uuid4().hex}",
                success=False,
                error_message=str(e)
            )
    
    def refund_payment(self, transaction_id: str) -> bool:
        if transaction_id not in self._transaction_map:
            return False
        
        paypal_payment_id = self._transaction_map[transaction_id]
        try:
            refund = self.paypal.issue_refund(paypal_payment_id)
            return refund["state"] == "completed"
        except Exception:
            return False
    
    def get_transaction_status(self, transaction_id: str) -> str:
        if transaction_id not in self._transaction_map:
            return "unknown"
        
        paypal_payment_id = self._transaction_map[transaction_id]
        try:
            payment = self.paypal.get_payment_details(paypal_payment_id)
            status_map = {
                "approved": "completed",
                "created": "pending",
                "failed": "failed"
            }
            return status_map.get(payment["state"], "unknown")
        except Exception:
            return "error"

# Payment Service that uses the common interface
class PaymentService:
    def __init__(self, payment_processor: PaymentProcessor):
        self.payment_processor = payment_processor
    
    def charge_customer(self, payment_details: PaymentDetails):
        # Process payment using the adapter
        result = self.payment_processor.process_payment(payment_details)
        
        if result.success:
            print(f"Payment successful! Transaction ID: {result.transaction_id}")
            print(f"Provider Reference: {result.provider_reference}")
        else:
            print(f"Payment failed: {result.error_message}")
        
        return result
    
    def refund_transaction(self, transaction_id: str):
        success = self.payment_processor.refund_payment(transaction_id)
        if success:
            print(f"Refund for transaction {transaction_id} was successful!")
        else:
            print(f"Refund for transaction {transaction_id} failed!")
        return success
    
    def check_transaction_status(self, transaction_id: str):
        status = self.payment_processor.get_transaction_status(transaction_id)
        print(f"Transaction {transaction_id} status: {status}")
        return status

# Usage example
if __name__ == "__main__":
    # Test with Stripe adapter
    stripe_service = PaymentService(StripeAdapter())
    
    payment_details = PaymentDetails(
        amount=99.99,
        currency="USD",
        card_number="4242424242424242",
        expiry_date="12/25",
        cvv="123",
        holder_name="John Doe"
    )
    
    print("Testing Stripe payment:")
    stripe_result = stripe_service.charge_customer(payment_details)
    
    if stripe_result.success:
        # Check status
        stripe_service.check_transaction_status(stripe_result.transaction_id)
        
        # Process refund
        stripe_service.refund_transaction(stripe_result.transaction_id)
    
    print("\n" + "-"*50 + "\n")
    
    # Test with PayPal adapter
    paypal_service = PaymentService(PayPalAdapter())
    
    print("Testing PayPal payment:")
    paypal_result = paypal_service.charge_customer(payment_details)
    
    if paypal_result.success:
        # Check status
        paypal_service.check_transaction_status(paypal_result.transaction_id)
        
        # Process refund
        paypal_service.refund_transaction(paypal_result.transaction_id)
```

## 3. Facade Pattern (Advanced Examples)

The Facade pattern is highly practical for simplifying complex subsystems.

### Machine Learning Model Pipeline Facade

This example shows how a facade can simplify a machine learning workflow:

```python
import numpy as np
import pickle
import os
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.compose import ColumnTransformer
from sklearn.pipeline import Pipeline
from sklearn.impute import SimpleImputer
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score, precision_score, recall_score, f1_score

# Complex subsystem components
class DataLoader:
    def load_data(self, file_path):
        """Load data from a file."""
        print(f"Loading data from {file_path}")
        # In a real implementation, this would read from a file
        # Here we'll create some synthetic data for demonstration
        
        # Create synthetic data for a classification problem
        np.random.seed(42)
        n_samples = 1000
        
        # Features: age, income, education_level, and has_credit_card
        age = np.random.normal(35, 10, n_samples).reshape(-1, 1)
        income = np.random.normal(50000, 15000, n_samples).reshape(-1, 1)
        education_level = np.random.choice(['high_school', 'bachelor', 'master', 'phd'], n_samples).reshape(-1, 1)
        has_credit_card = np.random.choice([0, 1], n_samples, p=[0.3, 0.7]).reshape(-1, 1)
        
        # Target: loan_approved (0 or 1)
        # Higher income, education, and having a credit card increase chances of approval
        education_score = np.zeros(n_samples)
        education_score[education_level.flatten() == 'high_school'] = 1
        education_score[education_level.flatten() == 'bachelor'] = 2
        education_score[education_level.flatten() == 'master'] = 3
        education_score[education_level.flatten() == 'phd'] = 4
        
        # Probability of approval based on features
        prob_approve = 0.1 + 0.4 * (age.flatten() > 25) + 0.3 * (income.flatten() > 40000) / 100000 + \
                       0.2 * education_score / 4 + 0.2 * has_credit_card.flatten()
        prob_approve = np.clip(prob_approve, 0, 0.9)  # Cap at 90%
        
        loan_approved = np.random.binomial(1, prob_approve).reshape(-1, 1)
        
        # Combine features and target
        X = np.hstack([age, income, education_level, has_credit_card])
        y = loan_approved
        
        # Create a structured view similar to a pandas DataFrame
        feature_names = ['age', 'income', 'education_level', 'has_credit_card']
        
        # Create a dictionary with column names as keys
        data = {feature_names[i]: X[:, i] for i in range(len(feature_names))}
        data['loan_approved'] = y.flatten()
        
        return data, feature_names

class DataPreprocessor:
    def __init__(self):
        self.preprocessor = None
    
    def fit_transform(self, data, feature_names):
        """Preprocess the data for model training."""
        print("Preprocessing data")
        
        # Identify numerical and categorical columns
        numerical_features = ['age', 'income']
        categorical_features = ['education_level']
        binary_features = ['has_credit_card']
        
        # Create preprocessing steps for different column types
        numerical_transformer = Pipeline(steps=[
            ('imputer', SimpleImputer(strategy='median')),
            ('scaler', StandardScaler())
        ])
        
        categorical_transformer = Pipeline(steps=[
            ('imputer', SimpleImputer(strategy='constant', fill_value='missing')),
            ('onehot', OneHotEncoder(handle_unknown='ignore'))
        ])
        
        # No preprocessing needed for binary features
        
        # Combine preprocessing steps
        self.preprocessor = ColumnTransformer(
            transformers=[
                ('num', numerical_transformer, numerical_features),
                ('cat', categorical_transformer, categorical_features),
                ('bin', 'passthrough', binary_features)
            ])
        
        # Extract features and target
        X = {key: value for key, value in data.items() if key in feature_names}
        X_array = np.column_stack([X[feature] for feature in feature_names])
        
        # Fit and transform the data
        X_transformed = self.preprocessor.fit_transform(X_array)
        
        return X_transformed
    
    def transform(self, data, feature_names):
        """Apply preprocessing to new data."""
        if self.preprocessor is None:
            raise ValueError("Preprocessor has not been fitted yet.")
        
        # Extract features
        X = {key: data[key] for key in feature_names}
        X_array = np.column_stack([X[feature] for feature in feature_names])
        
        # Transform the data
        X_transformed = self.preprocessor.transform(X_array)
        
        return X_transformed
    
    def save(self, file_path):
        """Save the preprocessor to a file."""
        with open(file_path, 'wb') as f:
            pickle.dump(self.preprocessor, f)
    
    def load(self, file_path):
        """Load the preprocessor from a file."""
        with open(file_path, 'rb') as f:
            self.preprocessor = pickle.load(f)

class ModelTrainer:
    def __init__(self):
        self.model = None
    
    def train(self, X, y, **model_params):
        """Train a machine learning model."""
        print("Training model")
        
        # Default parameters for RandomForest
        params = {
            'n_estimators': 100,
            'random_state': 42
        }
        
        # Update with any provided parameters
        params.update(model_params)
        
        # Create and train the model
        self.model = RandomForestClassifier(**params)
        self.model.fit(X, y)
        
        return self.model
    
    def save(self, file_path):
        """Save the model to a file."""
        with open(file_path, 'wb') as f:
            pickle.dump(self.model, f)
    
    def load(self, file_path):
        """Load the model from a file."""
        with open(file_path, 'rb') as f:
            self.model = pickle.load(f)
        return self.model

class ModelEvaluator:
    def evaluate(self, model, X, y):
        """Evaluate the model performance."""
        print("Evaluating model")
        
        # Make predictions
        y_pred = model.predict(X)
        
        # Calculate metrics
        accuracy = accuracy_score(y, y_pred)
        precision = precision_score(y, y_pred)
        recall = recall_score(y, y_pred)
        f1 = f1_score(y, y_pred)
        
        # Return metrics as a dictionary
        metrics = {
            'accuracy': accuracy,
            'precision': precision,
            'recall': recall,
            'f1_score': f1
        }
        
        return metrics

class ModelPredictor:
    def predict(self, model, X):
        """Make predictions using the trained model."""
        print("Making predictions")
        
        # Generate predictions
        y_pred = model.predict(X)
        
        # Get prediction probabilities
        y_prob = model.predict_proba(X)[:, 1]
        
        return y_pred, y_prob

# Facade for the ML pipeline
class MLPipelineFacade:
    def __init__(self, model_dir="models"):
        self.data_loader = DataLoader()
        self.preprocessor = DataPreprocessor()
        self.model_trainer = ModelTrainer()
        self.model_evaluator = ModelEvaluator()
        self.model_predictor = ModelPredictor()
        self.model_dir = model_dir
        self.feature_names = None
        
        # Create model directory if it doesn't exist
        os.makedirs(model_dir, exist_ok=True)
    
    def train_model(self, data_path, test_size=0.2, random_state=42, **model_params):
        """Train a new model using the provided data."""
        # Load data
        data, self.feature_names = self.data_loader.load_data(data_path)
        
        # Extract features and target
        X = {key: value for key, value in data.items() if key != 'loan_approved'}
        y = data['loan_approved']
        
        # Split data
        X_train_dict, X_test_dict = {}, {}
        for feature in self.feature_names:
            X_train_dict[feature], X_test_dict[feature] = train_test_split(
                X[feature], test_size=test_size, random_state=random_state
            )
        y_train, y_test = train_test_split(y, test_size=test_size, random_state=random_state)
        
        # Preprocess data
        X_train_transformed = self.preprocessor.fit_transform(X_train_dict, self.feature_names)
        X_test_transformed = self.preprocessor.transform(X_test_dict, self.feature_names)
        
        # Train model
        model = self.model_trainer.train(X_train_transformed, y_train, **model_params)
        
        # Evaluate model
        metrics = self.model_evaluator.evaluate(model, X_test_transformed, y_test)
        
        # Save model and preprocessor
        self.preprocessor.save(os.path.join(self.model_dir, 'preprocessor.pkl'))
        self.model_trainer.save(os.path.join(self.model_dir, 'model.pkl'))
        
        # Save feature names
        with open(os.path.join(self.model_dir, 'feature_names.pkl'), 'wb') as f:
            pickle.dump(self.feature_names, f)
        
        return metrics
    
    def load_model(self):
        """Load a trained model and preprocessor."""
        # Load feature names
        with open(os.path.join(self.model_dir, 'feature_names.pkl'), 'rb') as f:
            self.feature_names = pickle.load(f)
        
        # Load preprocessor and model
        self.preprocessor.load(os.path.join(self.model_dir, 'preprocessor.pkl'))
        model = self.model_trainer.load(os.path.join(self.model_dir, 'model.pkl'))
        
        return model
    
    def predict(self, data):
        """Make predictions for new data."""
        # Load model if not already loaded
        if self.model_trainer.model is None:
            model = self.load_model()
        else:
            model = self.model_trainer.model
        
        # Preprocess data
        X_transformed = self.preprocessor.transform(data, self.feature_names)
        
        # Make predictions
        predictions, probabilities = self.model_predictor.predict(model, X_transformed)
        
        return {
            'predictions': predictions,
            'probabilities': probabilities
        }
    
    def batch_predict(self, data_path):
        """Make predictions for a batch of data."""
        # Load data
        data, _ = self.data_loader.load_data(data_path)
        
        # Make predictions
        prediction_results = self.predict(data)
        
        return prediction_results
# Client code
if __name__ == "__main__":
    # Creating the facade
    ml_pipeline = MLPipelineFacade(model_dir="loan_model")
    
    # Train a new model
    print("Training a new loan approval prediction model:")
    metrics = ml_pipeline.train_model(
        "loan_data.csv",  # This file doesn't exist but our loader creates synthetic data
        n_estimators=200,
        max_depth=10
    )
    
    print("\nModel performance metrics:")
    for metric, value in metrics.items():
        print(f"{metric}: {value:.4f}")
    
    # Make a prediction for a new loan application
    new_application = {
        'age': np.array([32]),
        'income': np.array([75000]),
        'education_level': np.array(['master']),
        'has_credit_card': np.array([1])
    }
    
    print("\nPredicting loan approval for a new application:")
    prediction_result = ml_pipeline.predict(new_application)
    
    approval = "Approved" if prediction_result['predictions'][0] == 1 else "Denied"
    confidence = prediction_result['probabilities'][0]
    
    print(f"Loan decision: {approval}")
    print(f"Confidence: {confidence:.2f}")
```

### Enterprise Integration Facade

This example demonstrates a facade that integrates various enterprise systems:

```python
import json
import datetime
import uuid
import logging
from typing import Dict, List, Any, Optional

# Configure logging
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(name)s - %(levelname)s - %(message)s')
logger = logging.getLogger("EnterpriseIntegration")

# Complex subsystems
class CRMSystem:
    def __init__(self):
        self.customers = {}
        logger.info("CRM System initialized")
    
    def get_customer(self, customer_id):
        logger.info(f"CRM: Retrieving customer {customer_id}")
        if customer_id not in self.customers:
            # Create a fake customer for demonstration
            self.customers[customer_id] = {
                "id": customer_id,
                "name": f"Customer {customer_id}",
                "email": f"customer{customer_id}@example.com",
                "phone": f"555-{customer_id}-1234",
                "created_at": datetime.datetime.now().isoformat(),
                "status": "active"
            }
        return self.customers[customer_id]
    
    def update_customer(self, customer_id, data):
        logger.info(f"CRM: Updating customer {customer_id}")
        if customer_id not in self.customers:
            self.customers[customer_id] = {"id": customer_id}
        self.customers[customer_id].update(data)
        return self.customers[customer_id]
    
    def get_customer_orders(self, customer_id):
        logger.info(f"CRM: Retrieving orders for customer {customer_id}")
        # For demonstration, return some fake orders
        return [
            {
                "id": f"ORD-{uuid.uuid4().hex[:8]}",
                "customer_id": customer_id,
                "date": (datetime.datetime.now() - datetime.timedelta(days=i*10)).isoformat(),
                "amount": round(100 * (i + 1) * 1.5, 2),
                "status": "completed"
            }
            for i in range(3)
        ]

class InventorySystem:
    def __init__(self):
        self.products = {}
        logger.info("Inventory System initialized")
        
        # Create some sample products
        sample_products = [
            {"id": "P1001", "name": "Laptop", "price": 1200.00, "stock": 45},
            {"id": "P1002", "name": "Smartphone", "price": 800.00, "stock": 120},
            {"id": "P1003", "name": "Tablet", "price": 450.00, "stock": 75},
            {"id": "P1004", "name": "Monitor", "price": 350.00, "stock": 60},
            {"id": "P1005", "name": "Keyboard", "price": 100.00, "stock": 200}
        ]
        
        for product in sample_products:
            self.products[product["id"]] = product
    
    def get_product(self, product_id):
        logger.info(f"Inventory: Retrieving product {product_id}")
        return self.products.get(product_id, {"error": "Product not found"})
    
    def update_stock(self, product_id, quantity_change):
        logger.info(f"Inventory: Updating stock for product {product_id} by {quantity_change}")
        if product_id in self.products:
            self.products[product_id]["stock"] += quantity_change
            return {
                "product_id": product_id, 
                "new_stock": self.products[product_id]["stock"]
            }
        return {"error": "Product not found"}
    
    def get_low_stock_products(self, threshold=20):
        logger.info(f"Inventory: Retrieving products with stock below {threshold}")
        return [product for product in self.products.values() if product["stock"] < threshold]

class OrderSystem:
    def __init__(self):
        self.orders = {}
        logger.info("Order System initialized")
    
    def create_order(self, customer_id, products):
        logger.info(f"Orders: Creating new order for customer {customer_id}")
        order_id = f"ORD-{uuid.uuid4().hex[:8]}"
        order_total = sum(product["price"] * product["quantity"] for product in products)
        
        order = {
            "id": order_id,
            "customer_id": customer_id,
            "products": products,
            "total": order_total,
            "status": "pending",
            "created_at": datetime.datetime.now().isoformat()
        }
        
        self.orders[order_id] = order
        return order
    
    def get_order(self, order_id):
        logger.info(f"Orders: Retrieving order {order_id}")
        return self.orders.get(order_id, {"error": "Order not found"})
    
    def update_order_status(self, order_id, status):
        logger.info(f"Orders: Updating status for order {order_id} to {status}")
        if order_id in self.orders:
            self.orders[order_id]["status"] = status
            self.orders[order_id]["updated_at"] = datetime.datetime.now().isoformat()
            return self.orders[order_id]
        return {"error": "Order not found"}

class PaymentSystem:
    def __init__(self):
        self.transactions = {}
        logger.info("Payment System initialized")
    
    def process_payment(self, order_id, amount, payment_method):
        logger.info(f"Payments: Processing payment of {amount} for order {order_id}")
        transaction_id = f"PMT-{uuid.uuid4().hex[:8]}"
        
        # Simulate payment processing
        success = amount > 0 and payment_method in ["credit_card", "paypal", "bank_transfer"]
        
        transaction = {
            "id": transaction_id,
            "order_id": order_id,
            "amount": amount,
            "payment_method": payment_method,
            "status": "completed" if success else "failed",
            "timestamp": datetime.datetime.now().isoformat()
        }
        
        self.transactions[transaction_id] = transaction
        return transaction
    
    def refund_payment(self, transaction_id, amount=None):
        logger.info(f"Payments: Processing refund for transaction {transaction_id}")
        if transaction_id not in self.transactions:
            return {"error": "Transaction not found"}
        
        original = self.transactions[transaction_id]
        refund_amount = amount if amount is not None else original["amount"]
        
        refund_id = f"REF-{uuid.uuid4().hex[:8]}"
        refund = {
            "id": refund_id,
            "original_transaction_id": transaction_id,
            "amount": refund_amount,
            "status": "completed",
            "timestamp": datetime.datetime.now().isoformat()
        }
        
        self.transactions[refund_id] = refund
        return refund

class ShippingSystem:
    def __init__(self):
        self.shipments = {}
        logger.info("Shipping System initialized")
    
    def create_shipment(self, order_id, address, shipping_method="standard"):
        logger.info(f"Shipping: Creating shipment for order {order_id}")
        shipment_id = f"SHP-{uuid.uuid4().hex[:8]}"
        
        # Calculate estimated delivery date based on shipping method
        days_to_add = {"express": 2, "standard": 5, "economy": 10}.get(shipping_method, 5)
        estimated_delivery = (datetime.datetime.now() + datetime.timedelta(days=days_to_add)).isoformat()
        
        shipment = {
            "id": shipment_id,
            "order_id": order_id,
            "address": address,
            "shipping_method": shipping_method,
            "status": "pending",
            "tracking_number": f"TRK{uuid.uuid4().hex[:12].upper()}",
            "created_at": datetime.datetime.now().isoformat(),
            "estimated_delivery": estimated_delivery
        }
        
        self.shipments[shipment_id] = shipment
        return shipment
    
    def update_shipment_status(self, shipment_id, status):
        logger.info(f"Shipping: Updating status for shipment {shipment_id} to {status}")
        if shipment_id in self.shipments:
            self.shipments[shipment_id]["status"] = status
            self.shipments[shipment_id]["updated_at"] = datetime.datetime.now().isoformat()
            return self.shipments[shipment_id]
        return {"error": "Shipment not found"}
    
    def get_shipment(self, shipment_id=None, order_id=None):
        if shipment_id:
            logger.info(f"Shipping: Retrieving shipment {shipment_id}")
            return self.shipments.get(shipment_id, {"error": "Shipment not found"})
        elif order_id:
            logger.info(f"Shipping: Retrieving shipments for order {order_id}")
            return [s for s in self.shipments.values() if s["order_id"] == order_id]
        return {"error": "Either shipment_id or order_id must be provided"}

# Enterprise Integration Facade
class EnterpriseSystemFacade:
    def __init__(self):
        self.crm = CRMSystem()
        self.inventory = InventorySystem()
        self.orders = OrderSystem()
        self.payments = PaymentSystem()
        self.shipping = ShippingSystem()
        logger.info("Enterprise System Facade initialized")
    
    def get_customer_profile(self, customer_id):
        """Get complete customer profile with order history."""
        customer = self.crm.get_customer(customer_id)
        orders = self.crm.get_customer_orders(customer_id)
        
        return {
            "customer": customer,
            "orders": orders
        }
    
    def place_order(self, customer_id, products, payment_method, shipping_address, shipping_method="standard"):
        """Place a new order, process payment, adjust inventory, and create shipment."""
        # Verify product availability
        for product in products:
            product_info = self.inventory.get_product(product["id"])
            if "error" in product_info:
                return {"error": f"Product {product['id']} not found"}
            
            if product_info["stock"] < product["quantity"]:
                return {"error": f"Insufficient stock for product {product['id']}"}
            
            # Add product price to the product item
            product["price"] = product_info["price"]
        
        # Create order
        order = self.orders.create_order(customer_id, products)
        
        # Process payment
        payment = self.payments.process_payment(order["id"], order["total"], payment_method)
        
        if payment["status"] == "failed":
            self.orders.update_order_status(order["id"], "payment_failed")
            return {
                "error": "Payment processing failed",
                "order": order,
                "payment": payment
            }
        
        # Update order status
        self.orders.update_order_status(order["id"], "paid")
        
        # Update inventory
        for product in products:
            self.inventory.update_stock(product["id"], -product["quantity"])
        
        # Create shipment
        shipment = self.shipping.create_shipment(order["id"], shipping_address, shipping_method)
        
        # Update customer in CRM
        self.crm.update_customer(customer_id, {
            "last_order_date": datetime.datetime.now().isoformat(),
            "lifetime_value": order["total"]  # In a real system, this would be added to existing value
        })
        
        return {
            "success": True,
            "order": order,
            "payment": payment,
            "shipment": shipment
        }
    
    def cancel_order(self, order_id):
        """Cancel an order, refund payment, return products to inventory."""
        # Get order details
        order = self.orders.get_order(order_id)
        if "error" in order:
            return {"error": f"Order {order_id} not found"}
        
        # Check if order can be canceled
        if order["status"] not in ["pending", "paid"]:
            return {"error": f"Order {order_id} cannot be canceled (status: {order['status']})"}
        
        # Update order status
        self.orders.update_order_status(order_id, "canceled")
        
        # Find and cancel shipments
        shipments = self.shipping.get_shipment(order_id=order_id)
        if isinstance(shipments, list):
            for shipment in shipments:
                if shipment["status"] in ["pending", "processing"]:
                    self.shipping.update_shipment_status(shipment["id"], "canceled")
        
        # Find and refund payments
        for transaction in self.payments.transactions.values():
            if transaction.get("order_id") == order_id and transaction["status"] == "completed":
                self.payments.refund_payment(transaction["id"])
        
        # Return products to inventory
        for product in order["products"]:
            self.inventory.update_stock(product["id"], product["quantity"])
        
        return {
            "success": True,
            "message": f"Order {order_id} has been canceled"
        }
    
    def check_order_status(self, order_id):
        """Get complete status of an order including payment and shipping."""
        # Get order details
        order = self.orders.get_order(order_id)
        if "error" in order:
            return {"error": f"Order {order_id} not found"}
        
        # Find associated shipments
        shipments = self.shipping.get_shipment(order_id=order_id)
        if not isinstance(shipments, list):
            shipments = []
        
        # Find associated payments
        payments = [t for t in self.payments.transactions.values() if t.get("order_id") == order_id]
        
        return {
            "order": order,
            "shipments": shipments,
            "payments": payments
        }
        
    def inventory_report(self, low_stock_threshold=None):
        """Generate inventory report, optionally highlighting low stock items."""
        all_products = list(self.inventory.products.values())
        
        if low_stock_threshold is not None:
            low_stock = self.inventory.get_low_stock_products(low_stock_threshold)
            return {
                "all_products": all_products,
                "low_stock_products": low_stock,
                "total_products": len(all_products),
                "low_stock_count": len(low_stock)
            }
        
        return {
            "all_products": all_products,
            "total_products": len(all_products)
        }

# Client code
if __name__ == "__main__":
    # Create the enterprise system facade
    enterprise = EnterpriseSystemFacade()
    
    # Example usage: Place an order
    print("\n1. Placing a new order:")
    order_result = enterprise.place_order(
        customer_id="C12345",
        products=[
            {"id": "P1001", "quantity": 1},  # Laptop
            {"id": "P1003", "quantity": 2}   # Tablet
        ],
        payment_method="credit_card",
        shipping_address={
            "street": "123 Main St",
            "city": "Anytown",
            "state": "CA",
            "zip": "12345",
            "country": "USA"
        },
        shipping_method="express"
    )
    print(json.dumps(order_result, indent=2))
    
    # Save the order ID for later
    order_id = order_result["order"]["id"]
    
    # Check order status
    print("\n2. Checking order status:")
    status_result = enterprise.check_order_status(order_id)
    print(json.dumps(status_result, indent=2))
    
    # Generate inventory report
    print("\n3. Generating inventory report:")
    inventory_report = enterprise.inventory_report(low_stock_threshold=50)
    print(f"Total products: {inventory_report['total_products']}")
    print(f"Low stock products: {inventory_report['low_stock_count']}")
    for product in inventory_report['low_stock_products']:
        print(f"  - {product['name']}: {product['stock']} remaining")
    
    # Cancel the order
    print("\n4. Canceling the order:")
    cancel_result = enterprise.cancel_order(order_id)
    print(json.dumps(cancel_result, indent=2))
    
    # Check updated order status
    print("\n5. Checking updated order status:")
    updated_status = enterprise.check_order_status(order_id)
    print(json.dumps(updated_status, indent=2))
```

## 4. Composite Pattern (Advanced Examples)

The Composite pattern is particularly useful for tree-structured hierarchies.

### UI Component System

This example demonstrates a composite pattern for building UI component trees:

```python
from abc import ABC, abstractmethod
from typing import List, Optional, Dict, Any

class UIComponent(ABC):
    """Abstract base class for all UI components."""
    
    def __init__(self, id: str):
        self.id = id
        self.parent: Optional[UIComponent] = None
        self.styles: Dict[str, str] = {}
    
    @abstractmethod
    def render(self) -> str:
        """Render this component and return HTML."""
        pass
    
    def add_style(self, property_name: str, value: str) -> 'UIComponent':
        """Add a CSS style to this component."""
        self.styles[property_name] = value
        return self  # Return self for method chaining
    
    def get_style_string(self) -> str:
        """Convert styles dict to CSS style string."""
        if not self.styles:
            return ""
        return " ".join([f"{k}: {v};" for k, v in self.styles.items()])

class UIContainer(UIComponent):
    """Container component that can have children."""
    
    def __init__(self, id: str, tag: str = "div"):
        super().__init__(id)
        self.children: List[UIComponent] = []
        self.tag = tag
    
    def add(self, component: UIComponent) -> 'UIContainer':
        """Add a child component to this container."""
        component.parent = self
        self.children.append(component)
        return self  # Return self for method chaining
    
    def remove(self, component: UIComponent) -> bool:
        """Remove a child component from this container."""
        if component in self.children:
            self.children.remove(component)
            component.parent = None
            return True
        return False
    
    def render(self) -> str:
        """Render this container with all its children."""
        style_attr = f' style="{self.get_style_string()}"' if self.styles else ''
        result = [f'<{self.tag} id="{self.id}"{style_attr}>']
        
        for child in self.children:
            child_html = child.render()
            # Indent child HTML
            indented = '\n'.join(['  ' + line for line in child_html.split('\n')])
            result.append(indented)
        
        result.append(f'</{self.tag}>')
        return '\n'.join(result)

class UILeaf(UIComponent):
    """Leaf component that cannot have children."""
    
    def __init__(self, id: str, tag: str, content: str = ""):
        super().__init__(id)
        self.tag = tag
        self.content = content
        self.attributes: Dict[str, str] = {}
    
    def set_attribute(self, name: str, value: str) -> 'UILeaf':
        """Set an HTML attribute on this component."""
        self.attributes[name] = value
        return self  # Return self for method chaining
    
    def set_content(self, content: str) -> 'UILeaf':
        """Set the content of this component."""
        self.content = content
        return self  # Return self for method chaining
    
    def render(self) -> str:
        """Render this leaf component."""
        attrs = [f'id="{self.id}"']
        if self.styles:
            attrs.append(f'style="{self.get_style_string()}"')
        
        for name, value in self.attributes.items():
            attrs.append(f'{name}="{value}"')
        
        attrs_str = ' '.join(attrs)
        
        if self.content or self.tag in ["div", "p", "span", "h1", "h2", "h3", "h4", "h5", "h6", "button", "li", "td", "th"]:
            return f'<{self.tag} {attrs_str}>{self.content}</{self.tag}>'
        else:
            return f'<{self.tag} {attrs_str} />'

# UI Component Factory for convenience
class UIComponentFactory:
    @staticmethod
    def create_container(id: str, tag: str = "div") -> UIContainer:
        return UIContainer(id, tag)
    
    @staticmethod
    def create_text(id: str, text: str, tag: str = "p") -> UILeaf:
        return UILeaf(id, tag, text)
    
    @staticmethod
    def create_image(id: str, src: str, alt: str = "") -> UILeaf:
        return UILeaf(id, "img").set_attribute("src", src).set_attribute("alt", alt)
    
    @staticmethod
    def create_button(id: str, text: str) -> UILeaf:
        return UILeaf(id, "button", text)
    
    @staticmethod
    def create_link(id: str, href: str, text: str) -> UILeaf:
        return UILeaf(id, "a", text).set_attribute("href", href)
    
    @staticmethod
    def create_input(id: str, type_: str = "text", placeholder: str = "") -> UILeaf:
        return UILeaf(id, "input").set_attribute("type", type_).set_attribute("placeholder", placeholder)

# Client code
if __name__ == "__main__":
    # Create a UI factory
    ui = UIComponentFactory()
    
    # Create a complex UI structure
    page = ui.create_container("page", "div")
    
    # Add a header
    header = ui.create_container("header", "header")
    header.add_style("background-color", "#f0f0f0")
    header.add_style("padding", "1rem")
    
    logo = ui.create_image("logo", "logo.png", "Company Logo")
    logo.add_style("height", "50px")
    
    nav = ui.create_container("nav", "nav")
    nav.add_style("display", "flex")
    nav.add_style("gap", "1rem")
    
    nav.add(ui.create_link("home-link", "/", "Home"))
    nav.add(ui.create_link("about-link", "/about", "About"))
    nav.add(ui.create_link("contact-link", "/contact", "Contact"))
    
    header.add(logo)
    header.add(nav)
    
    # Add main content
    main = ui.create_container("main-content", "main")
    main.add_style("padding", "2rem")
    
    article = ui.create_container("article", "article")
    
    article.add(ui.create_text("heading", "Welcome to our Website", "h1"))
    
    paragraph = ui.create_text("intro", "This is a demonstration of the Composite Pattern for UI component trees.", "p")
    paragraph.add_style("color", "#333")
    paragraph.add_style("line-height", "1.6")
    
    article.add(paragraph)
    
    # Add a form section
    form = ui.create_container("contact-form", "form")
    
    form_title = ui.create_text("form-title", "Contact Us", "h2")
    
    name_input = ui.create_input("name", "text", "Your Name")
    name_input.add_style("margin-bottom", "1rem")
    name_input.add_style("padding", "0.5rem")
    name_input.add_style("width", "100%")
    
    email_input = ui.create_input("email", "email", "Your Email")
    email_input.add_style("margin-bottom", "1rem")
    email_input.add_style("padding", "0.5rem")
    email_input.add_style("width", "100%")
    
    message_container = ui.create_container("message-container", "div")
    message_container.add_style("margin-bottom", "1rem")
    
    message_label = ui.create_text("message-label", "Your Message:", "label")
    message_label.set_attribute("for", "message")
    
    message_input = UILeaf("message", "textarea")
    message_input.add_style("width", "100%")
    message_input.add_style("padding", "0.5rem")
    message_input.add_style("height", "150px")
    
    message_container.add(message_label)
    message_container.add(message_input)
    
    submit_button = ui.create_button("submit-btn", "Send Message")
    submit_button.add_style("background-color", "#4CAF50")
    submit_button.add_style("color", "white")
    submit_button.add_style("padding", "0.75rem 1.5rem")
    submit_button.add_style("border", "none")
    submit_button.add_style("cursor", "pointer")
    
    form.add(form_title)
    form.add(name_input)
    form.add(email_input)
    form.add(message_container)
    form.add(submit_button)
    
    article.add(form)
    main.add(article)
    
    # Add a footer
    footer = ui.create_container("footer", "footer")
    footer.add_style("background-color", "#333")
    footer.add_style("color", "white")
    footer.add_style("padding", "1rem")
    footer.add_style("text-align", "center")
    
    footer.add(ui.create_text("copyright", "© 2025 Company Name. All rights reserved.", "p"))
    
    # Add all sections to the page
    page.add(header)
    page.add(main)
    page.add(footer)
    
    # Render the page
    html = page.render()
    print(html)
```

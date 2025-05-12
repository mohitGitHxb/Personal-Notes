# Python Design Patterns: Practical Combined Examples

## Introduction

Design patterns are proven solutions to recurring software design problems. While understanding individual patterns is valuable, knowing how to combine them effectively is what separates good developers from great ones. This guide demonstrates practical examples where multiple design patterns work together to create elegant solutions in Python.

## 1. Factory Method + Observer Pattern

This example combines the Factory Method (creational pattern) with the Observer pattern (behavioral pattern) to create a flexible notification system.

```python
from abc import ABC, abstractmethod
from typing import List, Dict, Any


# Observer Pattern Components
class Observer(ABC):
    @abstractmethod
    def update(self, data: Dict[str, Any]) -> None:
        pass


class Subject:
    def __init__(self):
        self._observers: List[Observer] = []

    def attach(self, observer: Observer) -> None:
        if observer not in self._observers:
            self._observers.append(observer)

    def detach(self, observer: Observer) -> None:
        self._observers.remove(observer)

    def notify(self, data: Dict[str, Any]) -> None:
        for observer in self._observers:
            observer.update(data)


# Factory Method Pattern Components
class NotificationChannel(ABC):
    @abstractmethod
    def send(self, message: str, recipient: str) -> None:
        pass


class EmailNotification(NotificationChannel):
    def send(self, message: str, recipient: str) -> None:
        print(f"Sending Email to {recipient}: {message}")


class SMSNotification(NotificationChannel):
    def send(self, message: str, recipient: str) -> None:
        print(f"Sending SMS to {recipient}: {message}")


class PushNotification(NotificationChannel):
    def send(self, message: str, recipient: str) -> None:
        print(f"Sending Push Notification to {recipient}: {message}")


class NotificationFactory:
    @staticmethod
    def create_notification(notification_type: str) -> NotificationChannel:
        if notification_type == "email":
            return EmailNotification()
        elif notification_type == "sms":
            return SMSNotification()
        elif notification_type == "push":
            return PushNotification()
        else:
            raise ValueError(f"Unknown notification type: {notification_type}")


# Combining the patterns
class NotificationService(Subject):
    def __init__(self):
        super().__init__()
        self.factory = NotificationFactory()

    def send_notification(self, notification_type: str, message: str, recipient: str) -> None:
        notification_channel = self.factory.create_notification(notification_type)
        notification_channel.send(message, recipient)
        
        # Notify all observers about this notification
        self.notify({
            "type": notification_type,
            "message": message,
            "recipient": recipient,
            "status": "sent"
        })


class NotificationLogger(Observer):
    def update(self, data: Dict[str, Any]) -> None:
        print(f"LOG: {data['type']} notification sent to {data['recipient']}")


class AnalyticsTracker(Observer):
    def update(self, data: Dict[str, Any]) -> None:
        print(f"ANALYTICS: Tracked {data['type']} notification")


# Usage example
if __name__ == "__main__":
    notification_service = NotificationService()
    
    # Register observers
    logger = NotificationLogger()
    analytics = AnalyticsTracker()
    
    notification_service.attach(logger)
    notification_service.attach(analytics)
    
    # Send notifications
    notification_service.send_notification("email", "Welcome to our platform!", "user@example.com")
    print("-" * 50)
    notification_service.send_notification("sms", "Your verification code: 123456", "+1234567890")
```

### Benefits of this Combination

1. **Flexibility in notification channels**: The Factory Method allows for easy addition of new notification types.
2. **Loose coupling**: The system that sends notifications doesn't need to know about systems that track or log them.
3. **Separation of concerns**: Each component has a single responsibility.
4. **Extensibility**: You can add new observers without modifying the notification service.

## 2. Singleton + Strategy + Decorator

This example combines three patterns to create a configurable logging system:
- Singleton (creational): Ensures a single logger instance throughout the application
- Strategy (behavioral): Allows switching between different logging strategies
- Decorator (structural): Adds extra functionality to the logger

```python
from abc import ABC, abstractmethod
from datetime import datetime
from typing import Callable, Any, Dict


# Singleton Pattern
class LoggerSingleton:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super(LoggerSingleton, cls).__new__(cls)
            cls._instance.initialize()
        return cls._instance
    
    def initialize(self):
        self._strategy = None
        self._decorators = []
    
    def set_strategy(self, strategy):
        self._strategy = strategy
    
    def add_decorator(self, decorator):
        self._decorators.append(decorator)
    
    def remove_decorator(self, decorator):
        if decorator in self._decorators:
            self._decorators.remove(decorator)
    
    def log(self, message: str) -> None:
        if self._strategy is None:
            raise ValueError("Logging strategy not set")
        
        # Apply all decorators
        decorated_message = message
        for decorator in self._decorators:
            decorated_message = decorator(decorated_message)
        
        # Use the strategy to log the decorated message
        self._strategy.log(decorated_message)


# Strategy Pattern
class LoggingStrategy(ABC):
    @abstractmethod
    def log(self, message: str) -> None:
        pass


class ConsoleLoggingStrategy(LoggingStrategy):
    def log(self, message: str) -> None:
        print(f"[CONSOLE] {message}")


class FileLoggingStrategy(LoggingStrategy):
    def __init__(self, filename: str):
        self.filename = filename
    
    def log(self, message: str) -> None:
        with open(self.filename, 'a') as f:
            f.write(f"{message}\n")
        print(f"[FILE] Logged to {self.filename}")


class NetworkLoggingStrategy(LoggingStrategy):
    def __init__(self, endpoint: str):
        self.endpoint = endpoint
    
    def log(self, message: str) -> None:
        # In a real implementation, this would send the log to a network endpoint
        print(f"[NETWORK] Sending log to {self.endpoint}: {message}")


# Decorator Pattern (implemented as functions for simplicity)
def timestamp_decorator(message: str) -> str:
    return f"[{datetime.now().isoformat()}] {message}"


def level_decorator(level: str) -> Callable[[str], str]:
    def decorator(message: str) -> str:
        return f"[{level.upper()}] {message}"
    return decorator


def context_decorator(context: Dict[str, Any]) -> Callable[[str], str]:
    def decorator(message: str) -> str:
        context_str = " ".join(f"{k}={v}" for k, v in context.items())
        return f"{message} [Context: {context_str}]"
    return decorator


# Usage example
if __name__ == "__main__":
    # Get the singleton instance
    logger = LoggerSingleton()
    
    # Configure the logger with a strategy
    console_strategy = ConsoleLoggingStrategy()
    file_strategy = FileLoggingStrategy("app.log")
    
    logger.set_strategy(console_strategy)
    
    # Add decorators
    logger.add_decorator(timestamp_decorator)
    logger.add_decorator(level_decorator("info"))
    
    # Log a message
    logger.log("Application started")
    
    # Change strategy and add context
    logger.set_strategy(file_strategy)
    logger.add_decorator(context_decorator({"user": "admin", "session": "12345"}))
    
    logger.log("User logged in")
    
    # Switch back to console with fewer decorators
    logger.set_strategy(console_strategy)
    logger.remove_decorator(level_decorator("info"))
    
    logger.log("Processing complete")
```

### Benefits of this Combination

1. **Global access**: The Singleton ensures consistent logging across the application.
2. **Flexibility**: The Strategy pattern allows changing how logs are stored/transmitted.
3. **Enhanced functionality**: Decorators add cross-cutting concerns like timestamps and context.
4. **Runtime configuration**: The system can be reconfigured at runtime without modifying existing code.

## 3. Builder + Template Method + Adapter

This example demonstrates how to combine:
- Builder (creational): For step-by-step construction of complex objects
- Template Method (behavioral): For defining a skeleton algorithm with customizable steps
- Adapter (structural): For making incompatible interfaces work together

The example implements a document processing system that can build, process, and export documents in various formats.

```python
from abc import ABC, abstractmethod
from typing import List, Dict, Any


# Template Method Pattern
class DocumentProcessor(ABC):
    def process(self, document: Dict[str, Any]) -> Dict[str, Any]:
        """Template method defining the document processing algorithm"""
        document = self.validate(document)
        document = self.normalize(document)
        document = self.transform(document)
        document = self.enrich(document)
        return document
    
    @abstractmethod
    def validate(self, document: Dict[str, Any]) -> Dict[str, Any]:
        pass
    
    @abstractmethod
    def normalize(self, document: Dict[str, Any]) -> Dict[str, Any]:
        pass
    
    @abstractmethod
    def transform(self, document: Dict[str, Any]) -> Dict[str, Any]:
        pass
    
    @abstractmethod
    def enrich(self, document: Dict[str, Any]) -> Dict[str, Any]:
        pass


class ReportProcessor(DocumentProcessor):
    def validate(self, document: Dict[str, Any]) -> Dict[str, Any]:
        print("Validating report structure...")
        if "title" not in document:
            document["title"] = "Untitled Report"
        return document
    
    def normalize(self, document: Dict[str, Any]) -> Dict[str, Any]:
        print("Normalizing report data...")
        # Normalize date formats, capitalization, etc.
        return document
    
    def transform(self, document: Dict[str, Any]) -> Dict[str, Any]:
        print("Transforming report content...")
        # Apply transformations specific to reports
        return document
    
    def enrich(self, document: Dict[str, Any]) -> Dict[str, Any]:
        print("Enriching report with metadata...")
        document["metadata"] = {
            "processed_by": "ReportProcessor",
            "version": "1.0"
        }
        return document


# Builder Pattern
class DocumentBuilder:
    def __init__(self):
        self.reset()
    
    def reset(self):
        self._document = {}
    
    def set_title(self, title: str):
        self._document["title"] = title
        return self
    
    def set_content(self, content: str):
        self._document["content"] = content
        return self
    
    def add_section(self, section_title: str, section_content: str):
        if "sections" not in self._document:
            self._document["sections"] = []
        
        self._document["sections"].append({
            "title": section_title,
            "content": section_content
        })
        return self
    
    def set_author(self, author: str):
        self._document["author"] = author
        return self
    
    def set_date(self, date: str):
        self._document["date"] = date
        return self
    
    def build(self):
        document = self._document.copy()
        self.reset()
        return document


# Adapter Pattern - Third-party exporters
class ThirdPartyPDFExporter:
    """A third-party library with an incompatible interface"""
    def generate_pdf(self, title: str, content_parts: List[Dict[str, str]]) -> str:
        print(f"Generating PDF: {title}")
        for part in content_parts:
            print(f"  - Added section: {part['title']}")
        return f"pdf_file_{title.lower().replace(' ', '_')}.pdf"


class ThirdPartyHTMLExporter:
    """Another third-party library with an incompatible interface"""
    def create_html_document(self, data: Dict[str, Any]) -> str:
        print(f"Creating HTML document: {data.get('title', 'Untitled')}")
        return f"<html><head><title>{data.get('title', 'Untitled')}</title></head><body>...</body></html>"


# Adapter interface
class DocumentExporter(ABC):
    @abstractmethod
    def export(self, document: Dict[str, Any]) -> str:
        pass


# Concrete adapters
class PDFExporterAdapter(DocumentExporter):
    def __init__(self):
        self.pdf_exporter = ThirdPartyPDFExporter()
    
    def export(self, document: Dict[str, Any]) -> str:
        title = document.get("title", "Untitled")
        
        # Prepare content parts in the format expected by ThirdPartyPDFExporter
        content_parts = []
        if "content" in document:
            content_parts.append({"title": "Main Content", "content": document["content"]})
        
        if "sections" in document:
            content_parts.extend(document["sections"])
        
        return self.pdf_exporter.generate_pdf(title, content_parts)


class HTMLExporterAdapter(DocumentExporter):
    def __init__(self):
        self.html_exporter = ThirdPartyHTMLExporter()
    
    def export(self, document: Dict[str, Any]) -> str:
        # The adapter translates our document format to what the HTML exporter expects
        return self.html_exporter.create_html_document(document)


# Bringing it all together - Director class
class DocumentDirector:
    def __init__(self, builder: DocumentBuilder, processor: DocumentProcessor):
        self.builder = builder
        self.processor = processor
        self.exporters = {}
    
    def register_exporter(self, format_name: str, exporter: DocumentExporter):
        self.exporters[format_name] = exporter
    
    def create_report(self, title: str, content: str, author: str, date: str, sections: List[Dict[str, str]]):
        # Use the builder to create the document
        self.builder.set_title(title).set_content(content).set_author(author).set_date(date)
        
        for section in sections:
            self.builder.add_section(section["title"], section["content"])
        
        document = self.builder.build()
        
        # Process the document using the template method
        processed_document = self.processor.process(document)
        return processed_document
    
    def export_document(self, document: Dict[str, Any], format_name: str) -> str:
        if format_name not in self.exporters:
            raise ValueError(f"Unsupported export format: {format_name}")
        
        return self.exporters[format_name].export(document)


# Usage example
if __name__ == "__main__":
    # Create builder and processor
    builder = DocumentBuilder()
    processor = ReportProcessor()
    
    # Create director with builder and processor
    director = DocumentDirector(builder, processor)
    
    # Register exporters
    director.register_exporter("pdf", PDFExporterAdapter())
    director.register_exporter("html", HTMLExporterAdapter())
    
    # Create and process a document
    sections = [
        {"title": "Introduction", "content": "This is the introduction."},
        {"title": "Methodology", "content": "This is the methodology section."},
        {"title": "Results", "content": "These are the results."},
        {"title": "Conclusion", "content": "This is the conclusion."}
    ]
    
    document = director.create_report(
        title="Annual Sales Report",
        content="Executive summary of annual sales performance.",
        author="Finance Department",
        date="2025-05-12",
        sections=sections
    )
    
    print("\nDocument created and processed:")
    print(f"Title: {document['title']}")
    print(f"Author: {document['author']}")
    print(f"Sections: {len(document['sections'])}")
    print(f"Metadata: {document['metadata']}")
    
    # Export the document
    print("\nExporting document:")
    pdf_file = director.export_document(document, "pdf")
    html_file = director.export_document(document, "html")
    
    print(f"\nExported files: {pdf_file}, and HTML content")
```

### Benefits of this Combination

1. **Separation of concerns**: Each pattern handles a specific aspect of document management.
2. **Flexibility**: New document types, processing steps, and export formats can be added without changing existing code.
3. **Reusability**: The Builder and Template Method components can be reused in different contexts.
4. **Clean integration**: The Adapter pattern allows seamless integration with third-party libraries.

## 4. Command + Composite + Memento

This example showcases a text editor that combines:
- Command (behavioral): For implementing undo/redo functionality
- Composite (structural): For managing complex document structures
- Memento (behavioral): For saving and restoring document states

```python
from abc import ABC, abstractmethod
from typing import List, Dict, Any, Optional
import copy
import time


# Memento Pattern
class DocumentMemento:
    def __init__(self, state: Dict[str, Any]):
        self._state = copy.deepcopy(state)
        self._timestamp = time.time()
    
    @property
    def state(self) -> Dict[str, Any]:
        return copy.deepcopy(self._state)
    
    @property
    def timestamp(self) -> float:
        return self._timestamp


# Composite Pattern
class DocumentElement(ABC):
    def __init__(self, element_id: str):
        self.element_id = element_id
    
    @abstractmethod
    def render(self, indent: int = 0) -> str:
        pass
    
    @abstractmethod
    def get_plain_text(self) -> str:
        pass
    
    @abstractmethod
    def clone(self) -> 'DocumentElement':
        pass
    
    @abstractmethod
    def to_dict(self) -> Dict[str, Any]:
        pass
    
    @classmethod
    @abstractmethod
    def from_dict(cls, data: Dict[str, Any]) -> 'DocumentElement':
        pass


class TextElement(DocumentElement):
    def __init__(self, element_id: str, content: str, bold: bool = False, italic: bool = False):
        super().__init__(element_id)
        self.content = content
        self.bold = bold
        self.italic = italic
    
    def render(self, indent: int = 0) -> str:
        text = self.content
        if self.bold:
            text = f"**{text}**"
        if self.italic:
            text = f"_{text}_"
        return " " * indent + text
    
    def get_plain_text(self) -> str:
        return self.content
    
    def clone(self) -> 'TextElement':
        return TextElement(self.element_id, self.content, self.bold, self.italic)
    
    def to_dict(self) -> Dict[str, Any]:
        return {
            "type": "text",
            "element_id": self.element_id,
            "content": self.content,
            "bold": self.bold,
            "italic": self.italic
        }
    
    @classmethod
    def from_dict(cls, data: Dict[str, Any]) -> 'TextElement':
        return cls(
            data["element_id"],
            data["content"],
            data.get("bold", False),
            data.get("italic", False)
        )


class ContainerElement(DocumentElement):
    def __init__(self, element_id: str, element_type: str, children: List[DocumentElement] = None):
        super().__init__(element_id)
        self.element_type = element_type  # paragraph, section, etc.
        self.children = children or []
    
    def add(self, child: DocumentElement) -> None:
        self.children.append(child)
    
    def remove(self, child_id: str) -> bool:
        for i, child in enumerate(self.children):
            if child.element_id == child_id:
                self.children.pop(i)
                return True
        return False
    
    def render(self, indent: int = 0) -> str:
        result = f"{' ' * indent}<{self.element_type}>\n"
        for child in self.children:
            result += child.render(indent + 2) + "\n"
        result += f"{' ' * indent}</{self.element_type}>"
        return result
    
    def get_plain_text(self) -> str:
        return " ".join(child.get_plain_text() for child in self.children)
    
    def clone(self) -> 'ContainerElement':
        container = ContainerElement(self.element_id, self.element_type)
        for child in self.children:
            container.add(child.clone())
        return container
    
    def to_dict(self) -> Dict[str, Any]:
        return {
            "type": "container",
            "element_id": self.element_id,
            "element_type": self.element_type,
            "children": [child.to_dict() for child in self.children]
        }
    
    @classmethod
    def from_dict(cls, data: Dict[str, Any]) -> 'ContainerElement':
        container = cls(data["element_id"], data["element_type"])
        for child_data in data["children"]:
            if child_data["type"] == "text":
                container.add(TextElement.from_dict(child_data))
            elif child_data["type"] == "container":
                container.add(ContainerElement.from_dict(child_data))
        return container


class Document:
    def __init__(self, title: str):
        self.title = title
        self.root = ContainerElement("root", "document")
        self._version = 0
    
    def add_element(self, parent_id: str, element: DocumentElement) -> bool:
        if parent_id == "root":
            self.root.add(element)
            return True
        
        # Recursive helper function to find parent
        def find_and_add(container: ContainerElement) -> bool:
            if container.element_id == parent_id:
                container.add(element)
                return True
            
            for child in container.children:
                if isinstance(child, ContainerElement):
                    if find_and_add(child):
                        return True
            return False
        
        return find_and_add(self.root)
    
    def remove_element(self, element_id: str) -> bool:
        # Try to remove directly from root
        if self.root.remove(element_id):
            return True
        
        # Recursive helper function
        def find_and_remove(container: ContainerElement) -> bool:
            if container.remove(element_id):
                return True
            
            for child in container.children:
                if isinstance(child, ContainerElement):
                    if find_and_remove(child):
                        return True
            return False
        
        return find_and_remove(self.root)
    
    def find_element(self, element_id: str) -> Optional[DocumentElement]:
        if self.root.element_id == element_id:
            return self.root
        
        # Recursive helper function
        def find(container: ContainerElement) -> Optional[DocumentElement]:
            for child in container.children:
                if child.element_id == element_id:
                    return child
                
                if isinstance(child, ContainerElement):
                    result = find(child)
                    if result:
                        return result
            return None
        
        return find(self.root)
    
    def render(self) -> str:
        return f"# {self.title}\n\n{self.root.render()}"
    
    def create_memento(self) -> DocumentMemento:
        state = {
            "title": self.title,
            "root": self.root.to_dict(),
            "version": self._version
        }
        self._version += 1
        return DocumentMemento(state)
    
    def restore_from_memento(self, memento: DocumentMemento) -> None:
        state = memento.state
        self.title = state["title"]
        self.root = ContainerElement.from_dict(state["root"])
        self._version = state["version"]


# Command Pattern
class Command(ABC):
    @abstractmethod
    def execute(self) -> None:
        pass
    
    @abstractmethod
    def undo(self) -> None:
        pass


class AddElementCommand(Command):
    def __init__(self, document: Document, parent_id: str, element: DocumentElement):
        self.document = document
        self.parent_id = parent_id
        self.element = element
        self.memento = None
    
    def execute(self) -> None:
        self.memento = self.document.create_memento()
        self.document.add_element(self.parent_id, self.element)
    
    def undo(self) -> None:
        if self.memento:
            self.document.restore_from_memento(self.memento)


class RemoveElementCommand(Command):
    def __init__(self, document: Document, element_id: str):
        self.document = document
        self.element_id = element_id
        self.memento = None
        
    def execute(self) -> None:
        self.memento = self.document.create_memento()
        self.document.remove_element(self.element_id)
    
    def undo(self) -> None:
        if self.memento:
            self.document.restore_from_memento(self.memento)


class EditTextCommand(Command):
    def __init__(self, document: Document, element_id: str, new_content: str):
        self.document = document
        self.element_id = element_id
        self.new_content = new_content
        self.memento = None
    
    def execute(self) -> None:
        self.memento = self.document.create_memento()
        element = self.document.find_element(self.element_id)
        if element and isinstance(element, TextElement):
            element.content = self.new_content
    
    def undo(self) -> None:
        if self.memento:
            self.document.restore_from_memento(self.memento)


class CommandManager:
    def __init__(self):
        self._history: List[Command] = []
        self._redo_stack: List[Command] = []
    
    def execute(self, command: Command) -> None:
        command.execute()
        self._history.append(command)
        self._redo_stack.clear()  # Clear redo stack when a new command is executed
    
    def undo(self) -> bool:
        if not self._history:
            return False
        
        command = self._history.pop()
        command.undo()
        self._redo_stack.append(command)
        return True
    
    def redo(self) -> bool:
        if not self._redo_stack:
            return False
        
        command = self._redo_stack.pop()
        command.execute()
        self._history.append(command)
        return True
    
    def get_history_size(self) -> int:
        return len(self._history)
    
    def get_redo_size(self) -> int:
        return len(self._redo_stack)


# TextEditor - Bringing it all together
class TextEditor:
    def __init__(self, title: str = "Untitled Document"):
        self.document = Document(title)
        self.command_manager = CommandManager()
        self.snapshots: List[DocumentMemento] = []
    
    def create_paragraph(self, parent_id: str, element_id: str) -> None:
        paragraph = ContainerElement(element_id, "paragraph")
        command = AddElementCommand(self.document, parent_id, paragraph)
        self.command_manager.execute(command)
    
    def create_text(self, parent_id: str, element_id: str, content: str, bold: bool = False, italic: bool = False) -> None:
        text = TextElement(element_id, content, bold, italic)
        command = AddElementCommand(self.document, parent_id, text)
        self.command_manager.execute(command)
    
    def remove_element(self, element_id: str) -> None:
        command = RemoveElementCommand(self.document, element_id)
        self.command_manager.execute(command)
    
    def edit_text(self, element_id: str, new_content: str) -> None:
        command = EditTextCommand(self.document, element_id, new_content)
        self.command_manager.execute(command)
    
    def undo(self) -> bool:
        return self.command_manager.undo()
    
    def redo(self) -> bool:
        return self.command_manager.redo()
    
    def save_snapshot(self) -> int:
        snapshot = self.document.create_memento()
        self.snapshots.append(snapshot)
        return len(self.snapshots) - 1
    
    def restore_snapshot(self, index: int) -> bool:
        if 0 <= index < len(self.snapshots):
            self.document.restore_from_memento(self.snapshots[index])
            return True
        return False
    
    def display_document(self) -> str:
        return self.document.render()


# Usage example
if __name__ == "__main__":
    editor = TextEditor("My Document")
    
    # Create document structure
    editor.create_paragraph("root", "p1")
    editor.create_text("p1", "t1", "This is the first paragraph.")
    
    editor.create_paragraph("root", "p2")
    editor.create_text("p2", "t2", "This is another paragraph with ")
    editor.create_text("p2", "t3", "bold", True)
    editor.create_text("p2", "t4", " and ")
    editor.create_text("p2", "t5", "italic", False, True)
    editor.create_text("p2", "t6", " text.")
    
    # Save a snapshot
    first_version = editor.save_snapshot()
    print("Initial document:")
    print(editor.display_document())
    print("\n" + "-" * 40 + "\n")
    
    # Make changes
    editor.edit_text("t1", "This is the updated first paragraph.")
    editor.create_paragraph("root", "p3")
    editor.create_text("p3", "t7", "This is a third paragraph added later.")
    
    print("After edits:")
    print(editor.display_document())
    print("\n" + "-" * 40 + "\n")
    
    # Undo the last change
    editor.undo()
    
    print("After undo:")
    print(editor.display_document())
    print("\n" + "-" * 40 + "\n")
    
    # Redo the change
    editor.redo()
    
    print("After redo:")
    print(editor.display_document())
    print("\n" + "-" * 40 + "\n")
    
    # Restore to the first snapshot
    editor.restore_snapshot(first_version)
    
    print("After restoring to first snapshot:")
    print(editor.display_document())
```

## 5. Proxy + Flyweight + Decorator

This example combines three structural patterns to optimize resource usage in a document rendering system:
- Proxy: For lazy loading of heavy resources
- Flyweight: For sharing common elements
- Decorator: For adding formatting options

```python
from abc import ABC, abstractmethod
from typing import Dict, List, Tuple, Set, Optional
import time
import hashlib


# Base interface
class Graphic(ABC):
    @abstractmethod
    def draw(self, position: Tuple[int, int]) -> None:
        pass
    
    @abstractmethod
    def get_dimensions(self) -> Tuple[int, int]:
        pass


# Flyweight Pattern - Shared image data
class ImageFlyweight:
    def __init__(self, image_data: bytes):
        self.image_data = image_data
        self.width = len(image_data) // 100  # Simplified dimension calculation
        self.height = len(image_data) // 100
        
        # Simulate expensive processing of image data
        time.sleep(0.01)
        
        # Calculate hash for identity
        self.hash = hashlib.md5(image_data).hexdigest()
    
    def render(self, position: Tuple[int, int]) -> None:
        print(f"Rendering image with hash {self.hash[:6]} at position {position}")


# Flyweight Factory
class ImageFlyweightFactory:
    _flyweights: Dict[str, ImageFlyweight] = {}
    
    @classmethod
    def get_flyweight(cls, image_data: bytes) -> ImageFlyweight:
        # Use MD5 hash of image data as key
        key = hashlib.md5(image_data).hexdigest()
        
        if key not in cls._flyweights:
            print(f"Creating new flyweight for image with hash {key[:6]}")
            cls._flyweights[key] = ImageFlyweight(image_data)
        else:
            print(f"Reusing flyweight for image with hash {key[:6]}")
        
        return cls._flyweights[key]
    
    @classmethod
    def get_flyweight_count(cls) -> int:
        return len(cls._flyweights)


# Proxy Pattern - Lazy loading proxy
class LazyLoadingImageProxy(Graphic):
    def __init__(self, image_path: str):
        self.image_path = image_path
        self._image_flyweight: Optional[ImageFlyweight] = None
        self._is_loaded = False
        
        # These are estimates until the image is loaded
        self.estimated_width = 100
        self.estimated_height = 100
    
    def load_if_needed(self) -> None:
        if not self._is_loaded:
            print(f"Loading image from {self.image_path}")
            # Simulate loading image data from disk
            time.sleep(0.05)
            
            # In a real application, we would load actual data from the path
            # Here we just create some dummy data based on the path
            image_data = self.image_path.encode() * 100
            
            self._image_flyweight = ImageFlyweightFactory.get_flyweight(image_data)
            self._is_loaded = True
    
    def draw(self, position: Tuple[int, int]) -> None:
        self.load_if_needed()
        self._image_flyweight.render(position)
    
    def get_dimensions(self) -> Tuple[int, int]:
        if self._is_loaded:
            return (self._image_flyweight.width, self._image_flyweight.height)
        return (self.estimated_width, self.estimated_height)


# Decorator Pattern - Add effects to graphics
class GraphicDecorator(Graphic):
    def __init__(self, decorated: Graphic):
        self.decorated = decorated
    
    def draw(self, position: Tuple[int, int]) -> None:
        self.decorated.draw(position)
    
    def get_dimensions(self) -> Tuple[int, int]:
        return self.decorated.get_dimensions()


class BorderDecorator(GraphicDecorator):
    def __init__(self, decorated: Graphic, border_width: int = 1):
        super().__init__(decorated)
        self.border_width = border_width
    
    def draw(self, position: Tuple[int, int]) -> None:
        # Draw the original graphic
        super().draw(position)
        
        # Draw border effect
        width, height = self.decorated.get_dimensions()
        print(f"Drawing border with width {self.border_width} around graphic at {position} with dimensions {width}x{height}")
    
    def get_dimensions(self) -> Tuple[int, int]:
        width, height = super().get_dimensions()
        return (width + 2 * self.border_width, height + 2 * self.border_width)


class ShadowDecorator(GraphicDecorator):
    def __init__(self, decorated: Graphic, shadow_offset: Tuple[int, int] = (5, 5)):
        super().__init__(decorated)
        self.shadow_offset = shadow_offset
    
    def draw(self, position: Tuple[int, int]) -> None:
        # Draw shadow first
        shadow_position = (position[0] + self.shadow_offset[0], position[1] + self.shadow_offset[1])
        print(f"Drawing shadow at {shadow_position}")
        
        # Draw the original graphic
        super().draw(position)
    
    def get_dimensions(self) -> Tuple[int, int]:
        width, height = super().get_dimensions()
        return (width + abs(self.shadow_offset[0]), height + abs(self.shadow_offset[1]))


class ScaleDecorator(GraphicDecorator):
    def __init__(self, decorated: Graphic, scale_factor: float = 1.0):
        super().__init__(decorated)
        self.scale_factor = scale_factor
    
    def draw(self, position: Tuple[int, int]) -> None:
        print(f"Scaling graphic by factor {self.scale_factor}")
        super().draw(position)
    
    def get_dimensions(self) -> Tuple[int, int]:
        width, height = super().get_dimensions()
        return (int(width * self.scale_factor), int(height * self.scale_factor))


# Document class using the patterns
class Document:
    def __init__(self, title: str):
        self.title = title
        self.graphics: List[Tuple[Graphic, Tuple[int, int]]] = []
    
    def add_graphic(self, graphic: Graphic, position: Tuple[int, int]) -> None:
        self.graphics.append((graphic, position))
    
    def render(self) -> None:
        print(f"Rendering document: {self.title}")
        print("-" * 50)
        
        for graphic, position in self.graphics:
            graphic.draw(position)
        
        print("-" * 50)
        print("Document rendering complete")


# Usage example
if __name__ == "__main__":
    # Create a document
    doc = Document("Image Gallery")
    
    # Create some image proxies (using the same path multiple times to test flyweight)
    image1 = LazyLoadingImageProxy("image1.jpg")
    image2 = LazyLoadingImageProxy("image2.jpg")
    image3 = LazyLoadingImageProxy("image1.jpg")  # Same as image1
    
    # Apply decorators
    decorated_image1 = BorderDecorator(image1, 2)
    decorated_image2 = ShadowDecorator(ScaleDecorator(image2, 1.5), (10, 10))
    decorated_image3 = BorderDecorator(ShadowDecorator(image3), 3)
    
    # Add to document (images won't load yet due to lazy loading)
    doc.add_graphic(decorated_image1, (10, 10))
    doc.add_graphic(decorated_image2, (200, 30))
    doc.add_graphic(decorated_image3, (50, 200))
    
    print(f"Document created with {len(doc.graphics)} graphics")
    print(f"Image flyweights in memory: {ImageFlyweightFactory.get_flyweight_count()}")
    
    # Render the document (this will trigger loading)
    print("\nStarting document rendering...")
    doc.render()
    
    # Check how many unique image flyweights were created
    print(f"\nImage flyweights in memory after rendering: {ImageFlyweightFactory.get_flyweight_count()}")
```

## 6. State + Chain of Responsibility + Observer

This example combines three behavioral patterns to create a workflow processing system:
- State: For managing workflow states and transitions
- Chain of Responsibility: For processing workflow tasks
- Observer: For notifying listeners about state changes

```python
from abc import ABC, abstractmethod
from enum import Enum, auto
from typing import List, Dict, Any, Optional, Set, Callable


# Observer Pattern
class WorkflowObserver(ABC):
    @abstractmethod
    def update(self, workflow_id: str, old_state: str, new_state: str, workflow_data: Dict[str, Any]) -> None:
        pass


class LoggingObserver(WorkflowObserver):
    def update(self, workflow_id: str, old_state: str, new_state: str, workflow_data: Dict[str, Any]) -> None:
        print(f"LOG: Workflow {workflow_id} transitioned from {old_state} to {new_state}")


class NotificationObserver(WorkflowObserver):
    def __init__(self):
        self.notifications: List[str] = []
    
    def update(self, workflow_id: str, old_state: str, new_state: str, workflow_data: Dict[str, Any]) -> None:
        message = f"Workflow {workflow_id} is now in state {new_state}"
        self.notifications.append(message)
        print(f"NOTIFICATION: {message}")


class AuditObserver(WorkflowObserver):
    def __init__(self):
        self.audit_trail: List[Dict[str, Any]] = []
    
    def update(self, workflow_id: str, old_state: str, new_state: str, workflow_data: Dict[str, Any]) -> None:
        entry = {
            "workflow_id": workflow_id,
            "from_state": old_state,
            "to_state": new_state,
            "timestamp": time.time(),
            "data_snapshot": workflow_data.copy()
        }
        self.audit_trail.append(entry)
        print(f"AUDIT: Recorded state change for workflow {workflow_id}")


# State Pattern
class WorkflowState(ABC):
    def __init__(self, name: str):
        self.name = name
    
    @abstractmethod
    def process(self, workflow: 'Workflow') -> None:
        pass
    
    @abstractmethod
    def get_allowed_transitions(self) -> Set[str]:
        pass
    
    def __str__(self) -> str:
        return self.name


class DraftState(WorkflowState):
    def __init__(self):
        super().__init__("DRAFT")
    
    def process(self, workflow: 'Workflow') -> None:
        print(f"Processing workflow {workflow.workflow_id} in DRAFT state")
        # Implement draft-specific logic
        if workflow.data.get("auto_submit", False):
            workflow.transition_to("SUBMITTED")
    
    def get_allowed_transitions(self) -> Set[str]:
        return {"SUBMITTED", "CANCELED"}


class SubmittedState(WorkflowState):
    def __init__(self):
        super().__init__("SUBMITTED")
    
    def process(self, workflow: 'Workflow') -> None:
        print(f"Processing workflow {workflow.workflow_id} in SUBMITTED state")
        # Implement submitted-specific logic
        if workflow.data.get("requires_review", True):
            workflow.transition_to("IN_REVIEW")
        else:
            workflow.transition_to("APPROVED")
    
    def get_allowed_transitions(self) -> Set[str]:
        return {"IN_REVIEW", "APPROVED", "REJECTED"}


class InReviewState(WorkflowState):
    def __init__(self):
        super().__init__("IN_REVIEW")
    
    def process(self, workflow: 'Workflow') -> None:
        print(f"Processing workflow {workflow.workflow_id} in IN_REVIEW state")
        # In a real system, this would wait for human input
        # For this example, we'll use a value in the data
        if workflow.data.get("review_approved", False):
            workflow.transition_to("APPROVED")
        elif workflow.data.get("review_rejected", False):
            workflow.transition_to("REJECTED")
    
    def get_allowed_transitions(self) -> Set[str]:
        return {"APPROVED", "REJECTED"}


class ApprovedState(WorkflowState):
    def __init__(self):
        super().__init__("APPROVED")
    
    def process(self, workflow: 'Workflow') -> None:
        print(f"Processing workflow {workflow.workflow_id} in APPROVED state")
        workflow.transition_to("COMPLETED")
    
    def get_allowed_transitions(self) -> Set[str]:
        return {"COMPLETED"}


class RejectedState(WorkflowState):
    def __init__(self):
        super().__init__("REJECTED")
    
    def process(self, workflow: 'Workflow') -> None:
        print(f"Processing workflow {workflow.workflow_id} in REJECTED state")
        # Rejected workflows can be resubmitted or remain rejected
        if workflow.data.get("resubmit", False):
            workflow.transition_to("DRAFT")
    
    def get_allowed_transitions(self) -> Set[str]:
        return {"DRAFT", "CANCELED"}


class CompletedState(WorkflowState):
    def __init__(self):
        super().__init__("COMPLETED")
    
    def process(self, workflow: 'Workflow') -> None:
        print(f"Processing workflow {workflow.workflow_id} in COMPLETED state")
        # Terminal state - no further processing
    
    def get_allowed_transitions(self) -> Set[str]:
        return set()  # No transitions allowed from completed state


class CanceledState(WorkflowState):
    def __init__(self):
        super().__init__("CANCELED")
    
    def process(self, workflow: 'Workflow') -> None:
        print(f"Processing workflow {workflow.workflow_id} in CANCELED state")
        # Terminal state - no further processing
    
    def get_allowed_transitions(self) -> Set[str]:
        return {"DRAFT"}  # Can only restart from draft


# Chain of Responsibility Pattern
class WorkflowHandler(ABC):
    def __init__(self):
        self._next_handler: Optional[WorkflowHandler] = None
    
    def set_next(self, handler: 'WorkflowHandler') -> 'WorkflowHandler':
        self._next_handler = handler
        return handler  # Return handler to allow chaining
    
    def handle(self, workflow: 'Workflow') -> bool:
        if self._can_handle(workflow):
            return self._handle(workflow)
        elif self._next_handler:
            return self._next_handler.handle(workflow)
        return False  # No handler could process the workflow
    
    @abstractmethod
    def _can_handle(self, workflow: 'Workflow') -> bool:
        pass
    
    @abstractmethod
    def _handle(self, workflow: 'Workflow') -> bool:
        pass


class ValidationHandler(WorkflowHandler):
    def _can_handle(self, workflow: 'Workflow') -> bool:
        # Always validate
        return True
    
    def _handle(self, workflow: 'Workflow') -> bool:
        print(f"Validating workflow {workflow.workflow_id}")
        
        if not workflow.data.get("title"):
            workflow.add_error("title", "Title is required")
        
        if not workflow.data.get("description"):
            workflow.add_error("description", "Description is required")
        
        # Continue to next handler regardless of validation result
        if self._next_handler:
            return self._next_handler.handle(workflow)
        return True


class StateProcessingHandler(WorkflowHandler):
    def _can_handle(self, workflow: 'Workflow') -> bool:
        # Only process if there are no validation errors
        return len(workflow.errors) == 0
    
    def _handle(self, workflow: 'Workflow') -> bool:
        print(f"Processing state for workflow {workflow.workflow_id}")
        workflow.process_current_state()
        
        if self._next_handler:
            return self._next_handler.handle(workflow)
        return True


class NotificationHandler(WorkflowHandler):
    def _can_handle(self, workflow: 'Workflow') -> bool:
        # Always handle notifications
        return True
    
    def _handle(self, workflow: 'Workflow') -> bool:
        if len(workflow.errors) > 0:
            print(f"Sending error notifications for workflow {workflow.workflow_id}")
        else:
            print(f"Sending state update notifications for workflow {workflow.workflow_id}")
        
        if self._next_handler:
            return self._next_handler.handle(workflow)
        return True


# Main Workflow class integrating all patterns
class Workflow:
    _states: Dict[str, WorkflowState] = {
        "DRAFT": DraftState(),
        "SUBMITTED": SubmittedState(),
        "IN_REVIEW": InReviewState(),
        "APPROVED": ApprovedState(),
        "REJECTED": RejectedState(),
        "COMPLETED": CompletedState(),
        "CANCELED": CanceledState()
    }
    
    def __init__(self, workflow_id: str, title: str = "", description: str = ""):
        self.workflow_id = workflow_id
        self.data: Dict[str, Any] = {
            "title": title,
            "description": description
        }
        self._current_state = self._states["DRAFT"]
        self.errors: Dict[str, str] = {}
        self._observers: List[WorkflowObserver] = []
    
    def attach_observer(self, observer: WorkflowObserver) -> None:
        if observer not in self._observers:
            self._observers.append(observer)
    
    def detach_observer(self, observer: WorkflowObserver) -> None:
        if observer in self._observers:
            self._observers.remove(observer)
    
    def notify_observers(self, old_state: str, new_state: str) -> None:
        for observer in self._observers:
            observer.update(self.workflow_id, old_state, new_state, self.data)
    
    def transition_to(self, state_name: str) -> bool:
        if state_name not in self._states:
            print(f"Error: Unknown state {state_name}")
            return False
        
        if state_name not in self._current_state.get_allowed_transitions():
            print(f"Error: Cannot transition from {self._current_state} to {state_name}")
            return False
        
        old_state = str(self._current_state)
        self._current_state = self._states[state_name]
        
        print(f"Workflow {self.workflow_id} transitioned from {old_state} to {self._current_state}")
        self.notify_observers(old_state, str(self._current_state))
        return True
    
    def process_current_state(self) -> None:
        self._current_state.process(self)
    
    def get_current_state(self) -> str:
        return str(self._current_state)
    
    def add_error(self, field: str, message: str) -> None:
        self.errors[field] = message
    
    def clear_errors(self) -> None:
        self.errors.clear()
    
    def has_errors(self) -> bool:
        return len(self.errors) > 0


# Workflow processing system
class WorkflowProcessor:
    def __init__(self):
        # Set up the chain of responsibility
        validation_handler = ValidationHandler()
        state_handler = StateProcessingHandler()
        notification_handler = NotificationHandler()
        
        validation_handler.set_next(state_handler).set_next(notification_handler)
        self.handler = validation_handler
    
    def process(self, workflow: Workflow) -> None:
        workflow.clear_errors()
        self.handler.handle(workflow)
        
        if workflow.has_errors():
            print(f"Workflow {workflow.workflow_id} has errors:")
            for field, error in workflow.errors.items():
                print(f"  - {field}: {error}")


# Usage example
if __name__ == "__main__":
    import time
    
    # Create observers
    logger = LoggingObserver()
    notifier = NotificationObserver()
    auditor = AuditObserver()
    
    # Create workflow processor
    processor = WorkflowProcessor()
    
    # Create a new workflow
    workflow = Workflow("WF-001", "Sample Workflow", "This is a test workflow")
    
    # Attach observers
    workflow.attach_observer(logger)
    workflow.attach_observer(notifier)
    workflow.attach_observer(auditor)
    
    # Process workflow in different states
    print("\n=== Initial processing ===")
    processor.process(workflow)
    
    print("\n=== Setting auto_submit and processing again ===")
    workflow.data["auto_submit"] = True
    processor.process(workflow)
    
    print("\n=== Current state should now be SUBMITTED ===")
    print(f"Current state: {workflow.get_current_state()}")
    
    print("\n=== Setting requires_review and processing again ===")
    workflow.data["requires_review"] = True
    processor.process(workflow)
    
    print("\n=== Current state should now be IN_REVIEW ===")
    print(f"Current state: {workflow.get_current_state()}")
    
    print("\n=== Approving the review ===")
    workflow.data["review_approved"] = True
    processor.process(workflow)
    
    print("\n=== Current state should now be APPROVED ===")
    print(f"Current state: {workflow.get_current_state()}")
    
    print("\n=== Processing one more time to complete the workflow ===")
    processor.process(workflow)
    
    print("\n=== Current state should now be COMPLETED ===")
    print(f"Current state: {workflow.get_current_state()}")
    
    print("\n=== Audit trail ===")
    for i, entry in enumerate(auditor.audit_trail):
        print(f"{i+1}. {entry['from_state']} -> {entry['to_state']}")
```

## 7. Visitor + Bridge + Mediator

This example combines:
- Visitor (behavioral): For operations on complex object structures without modifying them
- Bridge (structural): For separating abstraction from implementation
- Mediator (behavioral): For coordinating interactions between objects

The example implements a data processing system for generating reports.

```python
from abc import ABC, abstractmethod
from typing import List, Dict, Any, Set, Optional
import json
import csv
import io


# Bridge Pattern - Implementation
class DataSourceImplementation(ABC):
    @abstractmethod
    def open(self) -> None:
        pass
    
    @abstractmethod
    def read_data(self) -> List[Dict[str, Any]]:
        pass
    
    @abstractmethod
    def write_data(self, data: List[Dict[str, Any]]) -> None:
        pass
    
    @abstractmethod
    def close(self) -> None:
        pass


class JSONDataSourceImpl(DataSourceImplementation):
    def __init__(self, filename: str):
        self.filename = filename
        self.file = None
    
    def open(self) -> None:
        print(f"Opening JSON file: {self.filename}")
        # In a real system, we would open the file
        # self.file = open(self.filename, 'r')
        self.data = [
            {"id": 1, "name": "Product A", "price": 99.99, "in_stock": True},
            {"id": 2, "name": "Product B", "price": 149.99, "in_stock": False},
            {"id": 3, "name": "Product C", "price": 49.99, "in_stock": True}
        ]
    
    def read_data(self) -> List[Dict[str, Any]]:
        print(f"Reading data from JSON file: {self.filename}")
        # In a real system, we would read from the file
        # return json.load(self.file)
        return self.data
    
    def write_data(self, data: List[Dict[str, Any]]) -> None:
        print(f"Writing data to JSON file: {self.filename}")
        # In a real system, we would write to the file
        # with open(self.filename, 'w') as f:
        #     json.dump(data, f, indent=2)
        self.data = data
    
    def close(self) -> None:
        print(f"Closing JSON file: {self.filename}")
        # if self.file:
        #     self.file.close()


class CSVDataSourceImpl(DataSourceImplementation):
    def __init__(self, filename: str):
        self.filename = filename
        self.file = None
        self.fieldnames = None
    
    def open(self) -> None:
        print(f"Opening CSV file: {self.filename}")
        # In a real system, we would open the file
        # self.file = open(self.filename, 'r')
        self.data = [
            {"id": 1, "name": "Product A", "price": 99.99, "in_stock": True},
            {"id": 2, "name": "Product B", "price": 149.99, "in_stock": False},
            {"id": 3, "name": "Product C", "price": 49.99, "in_stock": True}
        ]
        self.fieldnames = ["id", "name", "price", "in_stock"]
    
    def read_data(self) -> List[Dict[str, Any]]:
        print(f"Reading data from CSV file: {self.filename}")
        # In a real system, we would read from the file
        # reader = csv.DictReader(self.file)
        # return list(reader)
        return self.data
    
    def write_data(self, data: List[Dict[str, Any]]) -> None:
        print(f"Writing data to CSV file: {self.filename}")
        # In a real system, we would write to the file
        # with open(self.filename, 'w', newline='') as f:
        #     if data:
        #         writer = csv.DictWriter(f, fieldnames=data[0].keys())
        #         writer.writeheader()
        #         writer.writerows(data)
        self.data = data
    
    def close(self) -> None:
        print(f"Closing CSV file: {self.filename}")
        # if self.file:
        #    self.file.close()


class DatabaseDataSourceImpl(DataSourceImplementation):
    def __init__(self, connection_string: str, table_name: str):
        self.connection_string = connection_string
        self.table_name = table_name
        self.connection = None
    
    def open(self) -> None:
        print(f"Opening database connection to {self.connection_string}, table: {self.table_name}")
        # In a real system, we would open a database connection
        # self.connection = some_db_library.connect(self.connection_string)
        self.data = [
            {"id": 1, "name": "Product A", "price": 99.99, "in_stock": True},
            {"id": 2, "name": "Product B", "price": 149.99, "in_stock": False},
            {"id": 3, "name": "Product C", "price": 49.99, "in_stock": True}
        ]
    
    def read_data(self) -> List[Dict[str, Any]]:
        print(f"Reading data from database table: {self.table_name}")
        # In a real system, we would query the database
        # cursor = self.connection.cursor()
        # cursor.execute(f"SELECT * FROM {self.table_name}")
        # return cursor.fetchall()
        return self.data
    
    def write_data(self, data: List[Dict[str, Any]]) -> None:
        print(f"Writing data to database table: {self.table_name}")
        # In a real system, we would write to the database
        # cursor = self.connection.cursor()
        # for row in data:
        #     # Execute insert or update
        #     pass
        # self.connection.commit()
        self.data = data
    
    def close(self) -> None:
        print(f"Closing database connection")
        # if self.connection:
        #     self.connection.close()


# Bridge Pattern - Abstraction
class DataSource(ABC):
    def __init__(self, implementation: DataSourceImplementation):
        self.implementation = implementation
    
    def load(self) -> List[Dict[str, Any]]:
        self.implementation.open()
        data = self.implementation.read_data()
        self.implementation.close()
        return data
    
    def save(self, data: List[Dict[str, Any]]) -> None:
        self.implementation.open()
        self.implementation.write_data(data)
        self.implementation.close()
    
    @abstractmethod
    def get_data(self) -> List[Dict[str, Any]]:
        pass


class ProductDataSource(DataSource):
    def get_data(self) -> List[Dict[str, Any]]:
        print("Retrieving product data")
        raw_data = self.load()
        
        # Process the data (e.g., filter, transform)
        processed_data = []
        for item in raw_data:
            # Convert to consistent format
            processed_data.append({
                "id": item.get("id", 0),
                "name": item.get("name", ""),
                "price": float(item.get("price", 0)),
                "in_stock": bool(item.get("in_stock", False))
            })
        
        return processed_data


class OrderDataSource(DataSource):
    def get_data(self) -> List[Dict[str, Any]]:
        print("Retrieving order data")
        raw_data = self.load()
        
        # Process the data (e.g., filter, transform)
        processed_data = []
        for item in raw_data:
            # Convert to consistent format
            processed_data.append({
                "order_id": item.get("order_id", item.get("id", 0)),
                "customer": item.get("customer", ""),
                "total": float(item.get("total", item.get("price", 0))),
                "status": item.get("status", "unknown")
            })
        
        return processed_data


# Visitor Pattern

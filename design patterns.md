# Advanced Design Patterns in Python: Architectural Blueprints & Best Practices

> 💡 **Tip:** This document is designed for both quick reference and deep learning. Use the Table of Contents below to jump to specific patterns or interview preparation sections.

-----

## Table of Contents

1. [What Are Design Patterns?](#1-what-are-design-patterns)
1. [Design Pattern vs. Algorithm](#2-design-pattern-vs-algorithm)
1. [History: The Gang of Four (GoF)](#3-history-the-gang-of-four-gof)
1. [Five Core Benefits](#4-five-core-benefits)
1. [Criticisms & Misconceptions](#5-criticisms--misconceptions)
1. [The Three GoF Categories](#6-the-three-gof-categories)
1. [The Complexity Rule](#7-the-complexity-rule)
1. [Creational Patterns](#8-creational-patterns)
- [Singleton Pattern](#81-singleton-pattern)
- [Factory Method Pattern](#82-factory-method-pattern)
- [Abstract Factory Pattern](#83-abstract-factory-pattern)
- [Builder Pattern](#84-builder-pattern)
- [Prototype Pattern](#85-prototype-pattern)
1. [Structural Patterns](#9-structural-patterns)
- [Adapter Pattern](#91-adapter-pattern)
- [Bridge Pattern](#92-bridge-pattern)
- [Composite Pattern](#93-composite-pattern)
- [Decorator Pattern](#94-decorator-pattern)
- [Facade Pattern](#95-facade-pattern)
- [Flyweight Pattern](#96-flyweight-pattern)
- [Proxy Pattern](#97-proxy-pattern)
1. [Behavioral Patterns](#10-behavioral-patterns)
- [Observer Pattern](#101-observer-pattern)
- [Strategy Pattern](#102-strategy-pattern)
- [State Pattern](#103-state-pattern)
- [Chain of Responsibility](#104-chain-of-responsibility)
- [Command Pattern](#105-command-pattern)
- [Iterator Pattern](#106-iterator-pattern)
- [Mediator Pattern](#107-mediator-pattern)
- [Memento Pattern](#108-memento-pattern)
- [Template Method Pattern](#109-template-method-pattern)
- [Visitor Pattern](#1010-visitor-pattern)
1. [Advanced Pythonic Implementations](#11-advanced-pythonic-implementations)
1. [Master Revision Table](#12-master-revision-table)
1. [Real-World Industry Usage](#13-real-world-industry-usage)
1. [Senior Interview Toolkit](#14-senior-interview-toolkit)

-----

## 1. What Are Design Patterns?

**Design Patterns** are standard, reusable solutions for commonly occurring problems in software design.

Think of them like:

> 🏗️ **Blueprints — not ready-made buildings.**

You don’t copy a design pattern verbatim. You understand it, internalize the intent, and then implement it according to your specific problem context. Design patterns help you design better *architecture*, not just write code.

> ✅ A Design Pattern is **not** syntax. It is **not** a library. It is **not** a framework. It is a **thinking approach** for designing scalable, maintainable software.

-----

## 2. Design Pattern vs. Algorithm

This distinction is **critically important** and a frequent interview topic.

|Dimension         |**Design Pattern**                         |**Algorithm**                      |
|------------------|-------------------------------------------|-----------------------------------|
|**Solves**        |Design / architecture problems             |Computational / logic problems     |
|**Level**         |High-level concept                         |Step-by-step logic                 |
|**Implementation**|Flexible — adapt to context                |Fixed — concrete execution         |
|**Nature**        |Reusable strategy                          |Reusable procedure                 |
|**Example**       |Strategy Pattern → choose sorting algorithm|Quick Sort → specific sorting logic|


> 💡 A Strategy Pattern helps you *select* which sorting algorithm to use at runtime. The Quick Sort algorithm is the specific *implementation* you plug in. They operate at entirely different levels of abstraction.

-----

## 3. History: The Gang of Four (GoF)

Design patterns were popularized by the landmark book:

📘 ***Design Patterns: Elements of Reusable Object-Oriented Software***
*by Erich Gamma, Richard Helm, Ralph Johnson, and John Vlissides — collectively known as the **Gang of Four (GoF)***

Published in 1994, this book introduced **23 canonical design patterns** divided into three categories:

1. **Creational** — how objects are created
1. **Structural** — how objects are composed
1. **Behavioral** — how objects communicate

This three-category classification remains the industry standard to this day.

-----

## 4. Five Core Benefits

### 1. Reusability

Solutions already tested by the software community save you time and prevent reinventing the wheel.
*Example: The Factory pattern means you never redesign centralized object-creation logic.*

### 2. Maintainability

Pattern-based code is modular, organized, easier to debug, and easier to modify. When a future developer reads your code, they immediately understand the intent.

### 3. Scalability

Systems built on patterns can grow without breaking.
*Example: Observer pattern — add new listeners without touching any core logic.*

### 4. Consistency

Patterns give developers a shared vocabulary:

- *“Use a Singleton here”*
- *“Make this a Strategy Pattern”*

Team communication becomes faster and more precise because everyone speaks the same architectural language.

### 5. Efficiency

Avoid reinventing solutions. Proven patterns encode decades of collective engineering wisdom.

-----

## 5. Criticisms & Misconceptions

### ❌ Myth 1 — Design Patterns Fix Bad Code

**Reality:** Patterns improve *good* design. They cannot rescue poor architecture. A pattern applied to a fundamentally flawed system will only make the flaws more elaborate.

### ❌ Myth 2 — Patterns Are Rigid Templates

**Reality:** Patterns are **guidelines, not rules**. Python especially allows functional style, duck typing, decorators, and highly dynamic behavior. The same GoF pattern often looks very different in Python versus Java. The *intent* is preserved; the *form* adapts.

### Academic Criticisms (Deep Concepts)

1. **Patterns exist because languages are weak** — In languages with higher-order functions, some patterns become trivial. For example, higher-order functions can replace the Strategy pattern entirely in Python.
1. **Patterns may cause code duplication** — A poorly applied pattern leads to boilerplate-heavy, hard-to-maintain code.
1. **Patterns sometimes rename old ideas** — MVC (Model-View-Controller), for instance, existed before GoF formally catalogued it.

> 🔥 Despite these criticisms, patterns remain **powerful thinking tools**. They help organize systems, scale architecture, and communicate design decisions across teams and generations of engineers.

-----

## 6. The Three GoF Categories

### 📦 1. Creational Patterns — *Object Creation*

**Focus:** *How* objects are created.
**Patterns:** Singleton, Factory Method, Abstract Factory, Builder, Prototype.
**Goal:** Flexible object creation + reduced coupling between client code and concrete classes.

### 🧱 2. Structural Patterns — *Object Composition*

**Focus:** *How* objects and classes are combined into larger structures.
**Patterns:** Adapter, Bridge, Composite, Decorator, Facade, Flyweight, Proxy.
**Goal:** Build large, complex structures while keeping them flexible and maintainable.

### 🤝 3. Behavioral Patterns — *Object Communication*

**Focus:** *How* objects interact and distribute responsibility.
**Patterns:** Chain of Responsibility, Command, Interpreter, Iterator, Mediator, Memento, Observer, State, Strategy, Template Method, Visitor.
**Goal:** Clean responsibility distribution and decoupled communication between objects.

-----

## 7. The Complexity Rule

> 🔥 **Pattern Complexity = Problem Complexity**

Pattern use is an art, not a checklist. Breaking this rule is one of the most common mistakes senior developers identify in code reviews.

**❌ Bad Example:**
Using a Factory pattern to create just 2 simple, static objects → this is **over-engineering**.

**✅ Good Example:**
Multiple object types where the correct type is decided at runtime → Factory makes architectural sense.

The guiding principle: **use a pattern only when the problem demands it.** Otherwise, the code becomes harder to read, debug, and maintain — the exact opposite of the pattern’s intent.

-----

## 8. Creational Patterns

-----

### 8.1 Singleton Pattern

#### Plain English Concept

Singleton ensures that **only ONE instance of a class exists** in the entire application, and provides a single global access point to that instance.

#### Real-World Analogy

> 🏛️ A country has only ONE Election Commission (or Prime Minister). Everyone accesses the same authority. No duplicate authorities exist. This is Singleton.

#### The Problem It Solves

When multiple objects are created for the same shared resource, the consequences are severe:

- Too many database connections → server crash
- Memory waste
- Data inconsistency between modules
- Synchronization issues in multithreaded environments

**Real scenarios requiring Singleton:** Database connection pool, Logger system, Configuration manager, Cache manager, Thread pool manager, Hardware device controller.

#### Production-Level Python Implementation

**Classic `__new__` Override:**

```python
class Singleton:
    """
    Classic Singleton using __new__ override.
    Controls object creation at the memory-allocation level.
    """
    _instance = None  # Class-level variable: stores the single reference

    def __new__(cls) -> "Singleton":
        # __new__ controls CREATION; __init__ controls INITIALIZATION
        # We override __new__ because we must intercept before the object exists
        if cls._instance is None:
            print("Creating instance for the first time")
            cls._instance = super().__new__(cls)
        return cls._instance  # Always return the same object


# Dry Run
s1 = Singleton()  # Output: "Creating instance for the first time"
s2 = Singleton()  # No output — returns existing instance
print(s1 is s2)   # True — both variables point to the SAME object
```

**Singleton with Initialization Guard:**

```python
class Singleton:
    """
    Prevents __init__ from re-running on subsequent accesses.
    Without this guard, __init__ fires every time the object is 'created'.
    """
    _instance = None
    _initialized: bool = False

    def __new__(cls) -> "Singleton":
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

    def __init__(self) -> None:
        if not self._initialized:
            print("Initializing for the first time")
            # Perform expensive setup here (DB connect, config load, etc.)
            self._initialized = True
```

**Thread-Safe Singleton (Interview Critical):**

```python
from threading import Lock


class Singleton:
    """
    Thread-safe Singleton using a Lock.
    Without this, two threads might simultaneously pass the 'is None' check
    and each create a separate instance — breaking the Singleton guarantee.
    """
    _instance = None
    _lock: Lock = Lock()  # Class-level lock shared by all threads

    def __new__(cls) -> "Singleton":
        with cls._lock:  # Only one thread enters this block at a time
            if cls._instance is None:
                cls._instance = super().__new__(cls)
        return cls._instance
```

**Pythonic Singleton via Decorator (Most Elegant):**

```python
from typing import Any, Callable, Dict, Type


def singleton(cls: Type) -> Callable:
    """
    A decorator that transforms any class into a Singleton.
    Pythonic, reusable, and requires zero modification to the target class.
    """
    instances: Dict[Type, Any] = {}

    def wrapper(*args: Any, **kwargs: Any) -> Any:
        if cls not in instances:
            instances[cls] = cls(*args, **kwargs)
        return instances[cls]

    return wrapper


@singleton
class Database:
    def __init__(self, url: str = "localhost") -> None:
        self.url = url
        print(f"Database connected to {self.url}")


db1 = Database()
db2 = Database()
print(db1 is db2)  # True
```

**The Most Pythonic: Module-Level Singleton:**

```python
# config.py — This entire module IS a Singleton in Python
# Python's import system loads a module exactly ONCE and caches it.
# Every subsequent import reuses the same module object.

DB_URL: str = "postgresql://localhost:5432/mydb"
MAX_CONNECTIONS: int = 10
DEBUG: bool = False

# Usage anywhere in the application:
# import config
# config.DB_URL  ← always the same object
```

> 🔥 **Senior Python developers prefer the Module Singleton** for most use cases. It’s idiomatic, thread-safe by Python’s GIL, and requires no extra machinery.

#### Pros & Cons

|✅ Advantages                                    |❌ Disadvantages                                           |
|------------------------------------------------|----------------------------------------------------------|
|Memory efficient — prevents resource duplication|Hard to unit test (global dependency is difficult to mock)|
|Controlled global access                        |Hidden coupling between modules                           |
|Consistent shared state                         |Can become an anti-pattern if overused                    |
|Lazy initialization possible                    |Breaks modularity in large systems                        |
|Centralized logic                               |Concurrency complexity without locking                    |

#### When NOT to Use

- When you actually need multiple instances with independent states
- When object has no shared responsibility
- When testing isolation is critical
- When dependency injection is preferred

> ⚠️ **Senior Warning:** Singleton overuse = **Hidden Global Variable Problem**. The pattern creates global mutable state that makes systems hard to reason about. Always question whether you truly need a Singleton or whether dependency injection is the cleaner solution.

-----

### 8.2 Factory Method Pattern

#### Plain English Concept

Factory Method moves object creation logic into a dedicated method (the “factory”), so the client code never directly instantiates concrete classes. The client says *“give me this type of object”* and the factory decides how to create it.

#### Real-World Analogy

> 🍽️ A restaurant kitchen. The customer doesn’t cook the food — they give the order to the kitchen. The kitchen (factory) prepares the correct dish. The customer (client) only interacts with the finished product.

#### The Problem It Solves

Without a factory, creation logic proliferates:

```python
# ❌ Without Factory — tight coupling, logic duplicated everywhere
if payment_type == "card":
    payment = CardPayment()
elif payment_type == "upi":
    payment = UPIPayment()
elif payment_type == "netbank":
    payment = NetBankingPayment()
# This same if-else block gets copied to: checkout, refund, subscription...
```

This creates tight coupling to concrete classes, difficult scalability, and hardcoded dependencies everywhere.

#### Production-Level Python Implementation

**Step 1 — Product Interface:**

```python
from abc import ABC, abstractmethod


class Animal(ABC):
    """Abstract product — defines the interface all concrete products must follow."""
    @abstractmethod
    def speak(self) -> str:
        pass
```

**Step 2 — Concrete Products:**

```python
class Dog(Animal):
    def speak(self) -> str:
        return "Bark"


class Cat(Animal):
    def speak(self) -> str:
        return "Meow"
```

**Step 3 — Simple Factory:**

```python
class AnimalFactory:
    """
    Centralizes object creation. Client code depends only on the interface,
    never on concrete classes — follows the Dependency Inversion Principle (SOLID).
    """
    @staticmethod
    def create_animal(animal_type: str) -> Animal:
        animals = {
            "dog": Dog,
            "cat": Cat,
        }
        if animal_type not in animals:
            raise ValueError(f"Unknown animal type: {animal_type}")
        return animals[animal_type]()  # Dictionary dispatch — more Pythonic than if-else


animal = AnimalFactory.create_animal("dog")
print(animal.speak())  # Bark
```

**Step 4 — True Factory Method (Polymorphic, GoF-Compliant):**

```python
class Creator(ABC):
    """
    Abstract Creator — declares the factory method.
    The 'operation' method uses the factory method without knowing
    which concrete product will be returned.
    """
    @abstractmethod
    def factory_method(self) -> Animal:
        pass

    def operation(self) -> str:
        """Business logic that works with ANY product via the interface."""
        product = self.factory_method()
        return product.speak()


class DogCreator(Creator):
    def factory_method(self) -> Animal:
        return Dog()


class CatCreator(Creator):
    def factory_method(self) -> Animal:
        return Cat()


# Usage — fully polymorphic, no condition logic, easily extendable
creator: Creator = DogCreator()
print(creator.operation())  # Bark
```

#### Real Backend Example

```python
# Flask API supporting multiple databases — Factory in action
class DBFactory:
    @staticmethod
    def get_db(db_type: str):
        databases = {
            "mongo":    MongoDatabase,
            "postgres": PostgresDatabase,
            "redis":    RedisDatabase,
        }
        return databases[db_type]()

# Tomorrow you add Cassandra and DynamoDB.
# Client code NEVER changes. Only the factory expands.
db = DBFactory.get_db("mongo")
```

#### Pros & Cons

|✅ Advantages                                                 |❌ Disadvantages                     |
|-------------------------------------------------------------|------------------------------------|
|Loose coupling — client depends on interface only            |More classes required               |
|Single Responsibility Principle                              |Increased abstraction               |
|Scalable architecture — add new types without changing client|Slight complexity overhead          |
|Easy to test with mock objects                               |Steeper learning curve for beginners|
|Open/Closed Principle compliance                             |May be overkill for trivial creation|
|Runtime flexibility                                          |                                    |

#### Key Comparison

|                  |Simple Factory               |Factory Method           |
|------------------|-----------------------------|-------------------------|
|**Mechanism**     |if-else / dictionary dispatch|Polymorphism / subclasses|
|**Logic location**|Centralized                  |Distributed              |
|**Scalability**   |Less scalable                |Highly scalable          |
|**GoF pattern**   |Not a true GoF pattern       |True GoF pattern         |

-----

### 8.3 Abstract Factory Pattern

#### Plain English Concept

A factory that creates **multiple related objects (a family of products)** without specifying their concrete classes. Where Factory Method creates one product, Abstract Factory creates a *group* of compatible products.

#### Real-World Analogy

> 🪑 A furniture store. You choose either a Modern Set or Victorian Set — never mix Modern Sofa with Victorian Table. The Abstract Factory ensures the entire product family is consistent and compatible.

#### The Problem It Solves

Without Abstract Factory, inconsistency creeps in:

```python
# ❌ Without Abstract Factory — accidental mixing of incompatible families
button = DarkButton()
textbox = LightTextBox()  # Wrong mix — UI becomes visually inconsistent
```

#### Production-Level Python Implementation

```python
from abc import ABC, abstractmethod


# ── Abstract Products ──────────────────────────────────────────────────────
class Button(ABC):
    @abstractmethod
    def render(self) -> str:
        pass


class Checkbox(ABC):
    @abstractmethod
    def render(self) -> str:
        pass


# ── Concrete Products: Light Theme ────────────────────────────────────────
class LightButton(Button):
    def render(self) -> str:
        return "[ Light Button ]"


class LightCheckbox(Checkbox):
    def render(self) -> str:
        return "☐ Light Checkbox"


# ── Concrete Products: Dark Theme ─────────────────────────────────────────
class DarkButton(Button):
    def render(self) -> str:
        return "[ Dark Button ]"


class DarkCheckbox(Checkbox):
    def render(self) -> str:
        return "■ Dark Checkbox"


# ── Abstract Factory ──────────────────────────────────────────────────────
class UIFactory(ABC):
    """
    Declares creation methods for each distinct product type.
    Concrete factories implement this interface and produce compatible families.
    """
    @abstractmethod
    def create_button(self) -> Button:
        pass

    @abstractmethod
    def create_checkbox(self) -> Checkbox:
        pass


# ── Concrete Factories ────────────────────────────────────────────────────
class LightThemeFactory(UIFactory):
    def create_button(self) -> Button:
        return LightButton()

    def create_checkbox(self) -> Checkbox:
        return LightCheckbox()


class DarkThemeFactory(UIFactory):
    def create_button(self) -> Button:
        return DarkButton()

    def create_checkbox(self) -> Checkbox:
        return DarkCheckbox()


# ── Client Code ───────────────────────────────────────────────────────────
def render_ui(factory: UIFactory) -> None:
    """
    Client depends only on the abstract factory interface.
    It never knows which concrete family it's working with.
    This is Dependency Injection-friendly design.
    """
    button = factory.create_button()
    checkbox = factory.create_checkbox()
    print(button.render())
    print(checkbox.render())


# Switch entire theme by changing ONE line:
render_ui(DarkThemeFactory())
# Output:
# [ Dark Button ]
# ■ Dark Checkbox
```

#### Real Backend Example

```python
# ORM-style: switching entire database backend
class PostgresFactory(DBFactory):
    def create_connection(self): return PostgresConnection()
    def create_query_builder(self): return PostgresQueryBuilder()
    def create_transaction(self): return PostgresTransaction()

class MongoFactory(DBFactory):
    def create_connection(self): return MongoConnection()
    def create_query_builder(self): return MongoQueryBuilder()
    def create_transaction(self): return MongoTransaction()

# Switch database → just change factory. Entire backend adapts.
# This is how ORMs like SQLAlchemy work internally with dialects.
```

#### Pros & Cons

|✅ Advantages                                             |❌ Disadvantages                                              |
|---------------------------------------------------------|-------------------------------------------------------------|
|Strong consistency — enforces compatible product families|Large number of classes                                      |
|Loose coupling                                           |Hard to understand initially                                 |
|Easy to swap entire environments (dev/prod/test)         |Complex architecture                                         |
|High abstraction level                                   |Adding new products requires modifying all factory interfaces|
|Open/Closed Principle                                    |                                                             |

#### Factory vs Abstract Factory

|              |Factory Method         |Abstract Factory                   |
|--------------|-----------------------|-----------------------------------|
|**Creates**   |One product            |Family of related products         |
|**Complexity**|Simpler                |More complex                       |
|**Mechanism** |Polymorphism           |Multiple factory interfaces        |
|**Best for**  |Small-to-medium systems|Large, multi-platform architectures|

-----

### 8.4 Builder Pattern

#### Plain English Concept

Builder constructs a **complex object step-by-step**, separating the construction process from the final representation. It solves the “telescoping constructor” problem — constructors with so many parameters they become unreadable.

#### Real-World Analogy

> 🍔 Ordering a custom burger. You don’t pass all toppings in one confusing call. You add them one by one: `add_patty()`, `add_cheese()`, `add_sauce()`. The builder assembles the final product only when you call `build()`.

#### The Problem It Solves

```python
# ❌ Telescoping constructor anti-pattern
user = User("Siddhi", 25, "Mumbai", "+91-99999", "s@s.com", "123456789012")
# Which argument is which? Order-dependent, error-prone, unreadable.

# ✅ Builder approach
user = (UserBuilder()
    .set_name("Siddhi")
    .set_age(25)
    .set_address("Mumbai")
    .set_phone("+91-99999")
    .build())
```

#### Production-Level Python Implementation

```python
from __future__ import annotations
from dataclasses import dataclass, field
from typing import List, Optional


@dataclass
class Computer:
    """The complex product being constructed."""
    cpu: str = ""
    ram: str = ""
    storage: str = ""
    gpu: Optional[str] = None
    extras: List[str] = field(default_factory=list)

    def __str__(self) -> str:
        parts = [f"CPU: {self.cpu}", f"RAM: {self.ram}", f"Storage: {self.storage}"]
        if self.gpu:
            parts.append(f"GPU: {self.gpu}")
        if self.extras:
            parts.append(f"Extras: {', '.join(self.extras)}")
        return " | ".join(parts)


class ComputerBuilder:
    """
    Fluent Builder — each method returns 'self' to enable method chaining.
    This is idiomatic Python and produces highly readable construction code.
    """
    def __init__(self) -> None:
        self._computer = Computer()

    def set_cpu(self, cpu: str) -> ComputerBuilder:
        self._computer.cpu = cpu
        return self

    def set_ram(self, ram: str) -> ComputerBuilder:
        self._computer.ram = ram
        return self

    def set_storage(self, storage: str) -> ComputerBuilder:
        self._computer.storage = storage
        return self

    def set_gpu(self, gpu: str) -> ComputerBuilder:
        self._computer.gpu = gpu
        return self

    def add_extra(self, extra: str) -> ComputerBuilder:
        self._computer.extras.append(extra)
        return self

    def build(self) -> Computer:
        """Validates and returns the final assembled product."""
        if not self._computer.cpu:
            raise ValueError("CPU is required")
        return self._computer


# Fluent interface — clean, readable, order-independent
gaming_pc = (ComputerBuilder()
    .set_cpu("Intel i9-13900K")
    .set_ram("64GB DDR5")
    .set_storage("2TB NVMe SSD")
    .set_gpu("RTX 4090")
    .add_extra("Liquid Cooling")
    .add_extra("RGB Lighting")
    .build())

print(gaming_pc)
```

#### Real-World Usage

- SQLAlchemy query builders
- Python `requests` library-style config building
- Pydantic model construction
- HTTP request builders
- Test fixture factories

#### Pros & Cons

|✅ Advantages                                   |❌ Disadvantages                  |
|-----------------------------------------------|---------------------------------|
|Readable, self-documenting construction        |Slightly more verbose code       |
|Flexible — optional fields handled cleanly     |Overkill for simple objects      |
|Same builder process, different representations|Requires a separate Builder class|
|Fluent interface improves developer experience |Learning curve                   |

#### Builder vs Abstract Factory

|              |Builder                         |Abstract Factory        |
|--------------|--------------------------------|------------------------|
|**Creates**   |ONE complex object, step by step|Multiple related objects|
|**Focus**     |Construction process            |Family compatibility    |
|**Interface** |Fluent (chained methods)        |Factory switching       |
|**End result**|Single assembled product        |Compatible object set   |

-----

### 8.5 Prototype Pattern

#### Plain English Concept

Create a new object by **cloning an existing object** instead of building from scratch. When object construction is expensive, cloning is dramatically faster.

#### Real-World Analogy

> 📄 A photocopier. Instead of rewriting a document from scratch, you make a copy and modify only what’s different.

#### The Problem It Solves

Game character with many attributes (weapons, skills, skins, stats) — instead of rebuilding the entire object graph, clone a template and customize the clone.

#### Production-Level Python Implementation

```python
import copy
from typing import List


class GameCharacter:
    """
    Prototype — any object that supports cloning is a prototype.
    Python's copy module provides both shallow and deep copy.
    """
    def __init__(self, name: str, level: int, skills: List[str]) -> None:
        self.name = name
        self.level = level
        self.skills = skills  # Mutable list — requires deep copy

    def clone(self) -> "GameCharacter":
        """
        Deep copy creates independent copies of all nested objects.
        Use shallow copy only if all fields are immutable (strings, ints).
        """
        return copy.deepcopy(self)

    def __repr__(self) -> str:
        return f"GameCharacter(name={self.name}, level={self.level}, skills={self.skills})"


# Base template
template_warrior = GameCharacter("Warrior", 1, ["Slash", "Block"])

# Clone and customize — fast, no expensive re-initialization
warrior_clone = template_warrior.clone()
warrior_clone.name = "Elite Warrior"
warrior_clone.level = 50
warrior_clone.skills.append("Berserker Rage")

print(template_warrior)  # skills: ['Slash', 'Block'] — unchanged
print(warrior_clone)     # skills: ['Slash', 'Block', 'Berserker Rage']
```

#### When to Use

- Caching object templates (ML pipeline configurations)
- Game engines (enemy/character templates)
- UI template cloning
- Test fixtures with pre-built state

#### Pros & Cons

|✅ Advantages                                             |❌ Disadvantages                                     |
|---------------------------------------------------------|----------------------------------------------------|
|Fast object creation — avoids expensive re-initialization|Deep copy complexity with nested/circular references|
|Reduces subclass explosion                               |Hard to debug copy-related bugs                     |
|Flexible template system                                 |Requires careful attention to deep vs shallow copy  |

-----

## 9. Structural Patterns

-----

### 9.1 Adapter Pattern

#### Plain English Concept

Converts one interface into another interface that clients expect. Acts as a **translator** between incompatible interfaces.

#### Real-World Analogy

> 🔌 A power adapter. A US plug doesn’t fit a UK socket directly. The adapter converts the interface — no modification to either device.

#### The Problem It Solves

```python
# Old payment API in production — cannot be modified
class OldPaymentAPI:
    def pay_now(self, amount: float) -> str:
        return f"Paid {amount} via old system"

# New system expects a different method name
# We need: obj.process_payment(amount)
```

#### Production-Level Python Implementation

```python
class OldPaymentAPI:
    """Legacy system — cannot be modified (third-party or production-frozen)."""
    def pay_now(self, amount: float) -> str:
        return f"Paid ${amount:.2f} via OldAPI"


class NewPaymentInterface:
    """The interface our new system expects all payment providers to implement."""
    def process_payment(self, amount: float) -> str:
        raise NotImplementedError


class PaymentAdapter(NewPaymentInterface):
    """
    Adapter — wraps OldPaymentAPI and exposes NewPaymentInterface.
    Client code uses the new interface; adapter delegates to the old one.
    """
    def __init__(self, old_api: OldPaymentAPI) -> None:
        self._old_api = old_api  # Composition over inheritance

    def process_payment(self, amount: float) -> str:
        # Translate the new call to the old method
        return self._old_api.pay_now(amount)


# Client code only knows NewPaymentInterface — no legacy knowledge required
adapter = PaymentAdapter(OldPaymentAPI())
print(adapter.process_payment(99.99))  # Paid $99.99 via OldAPI
```

#### Real-World Usage

- Third-party payment gateway integration
- Legacy system migration without rewrite
- Different API contracts between microservices
- Database driver abstraction

#### Pros & Cons

|✅ Advantages                              |❌ Disadvantages                                  |
|------------------------------------------|-------------------------------------------------|
|Reuse legacy code without modification    |Adds an indirection layer                        |
|Smooth integration of incompatible systems|Can hide underlying complexity                   |
|Follows Open/Closed Principle             |Proliferation of adapter classes in large systems|

-----

### 9.2 Bridge Pattern

#### Plain English Concept

Separates **abstraction from implementation** so that both can vary independently. Prevents the combinatorial class explosion that occurs when you have multiple dimensions of variation.

#### Real-World Analogy

> 📺 A universal remote control. The same remote (abstraction) can control a TV, a Radio, or an AC (implementations). You can swap either side independently without rewriting the other.

#### The Problem It Solves

```python
# ❌ Without Bridge — class explosion
# BasicTVRemote, SmartTVRemote, BasicRadioRemote, SmartRadioRemote
# 2 remotes × 3 devices = 6 classes. Add one device → 2 more classes.

# ✅ With Bridge: 2 remote classes + 3 device classes = 5 total, scales linearly
```

#### Production-Level Python Implementation

```python
from abc import ABC, abstractmethod


# ── Implementation Hierarchy ──────────────────────────────────────────────
class Device(ABC):
    """Implementation interface — concrete devices live here."""
    @abstractmethod
    def turn_on(self) -> str:
        pass

    @abstractmethod
    def turn_off(self) -> str:
        pass


class TV(Device):
    def turn_on(self) -> str:
        return "TV: powering on"

    def turn_off(self) -> str:
        return "TV: shutting down"


class Radio(Device):
    def turn_on(self) -> str:
        return "Radio: broadcasting"

    def turn_off(self) -> str:
        return "Radio: going silent"


# ── Abstraction Hierarchy ─────────────────────────────────────────────────
class Remote(ABC):
    """
    Abstraction — holds a REFERENCE to the implementation.
    This composition link IS the bridge.
    """
    def __init__(self, device: Device) -> None:
        self._device = device  # Bridge: composition, not inheritance

    @abstractmethod
    def power(self) -> str:
        pass


class BasicRemote(Remote):
    def power(self) -> str:
        return self._device.turn_on()


class SmartRemote(Remote):
    def power(self) -> str:
        result = self._device.turn_on()
        return f"Smart Mode: {result} + connecting to WiFi"


# Mix and match freely — no class explosion
tv_remote = BasicRemote(TV())
smart_radio = SmartRemote(Radio())
print(tv_remote.power())    # TV: powering on
print(smart_radio.power())  # Smart Mode: Radio: broadcasting + connecting to WiFi
```

#### Pros & Cons

|✅ Advantages                                       |❌ Disadvantages                       |
|---------------------------------------------------|--------------------------------------|
|Prevents class explosion                           |More abstraction layers to understand |
|Abstraction and implementation evolve independently|Hard to grasp initially               |
|High flexibility via composition                   |Can be overkill for simple hierarchies|

-----

### 9.3 Composite Pattern

#### Plain English Concept

Allows you to **treat individual objects and groups of objects uniformly** using a tree structure. A leaf and a branch respond to the same interface.

#### Real-World Analogy

> 📁 A file system. A File and a Folder both support `show()`, `delete()`, `get_size()`. A Folder simply delegates to its children. You call the same operations on both without caring which type it is.

#### Production-Level Python Implementation

```python
from abc import ABC, abstractmethod
from typing import List, Optional


class FileSystemComponent(ABC):
    """Component — the common interface for both leaves and composites."""
    def __init__(self, name: str) -> None:
        self.name = name

    @abstractmethod
    def show(self, indent: int = 0) -> None:
        pass

    @abstractmethod
    def get_size(self) -> int:
        pass


class File(FileSystemComponent):
    """Leaf — no children. Performs operations directly."""
    def __init__(self, name: str, size: int) -> None:
        super().__init__(name)
        self._size = size

    def show(self, indent: int = 0) -> None:
        print(" " * indent + f"📄 {self.name} ({self._size}KB)")

    def get_size(self) -> int:
        return self._size


class Folder(FileSystemComponent):
    """
    Composite — contains children (can be Files or other Folders).
    Delegates operations to all children recursively.
    """
    def __init__(self, name: str) -> None:
        super().__init__(name)
        self._children: List[FileSystemComponent] = []

    def add(self, component: FileSystemComponent) -> None:
        self._children.append(component)

    def show(self, indent: int = 0) -> None:
        print(" " * indent + f"📁 {self.name}/")
        for child in self._children:
            child.show(indent + 4)  # Recursive traversal

    def get_size(self) -> int:
        return sum(child.get_size() for child in self._children)


# Build a tree
root = Folder("root")
docs = Folder("documents")
docs.add(File("resume.pdf", 150))
docs.add(File("cover_letter.pdf", 80))
root.add(docs)
root.add(File("readme.txt", 5))

root.show()
print(f"Total size: {root.get_size()}KB")
```

#### Pros & Cons

|✅ Advantages                                         |❌ Disadvantages                 |
|-----------------------------------------------------|--------------------------------|
|Uniform treatment of individual and composite objects|Can make design overly general  |
|Recursive tree structures become natural             |Hard to restrict component types|
|Easy to add new component types                      |                                |

-----

### 9.4 Decorator Pattern

#### Plain English Concept

Adds behavior to an object **dynamically at runtime** without modifying its class. Wrappers stack on top of each other like layers.

#### Real-World Analogy

> 🧥 Wearing clothes. You wear a T-shirt → Jacket → Scarf. Each layer adds a feature (warmth, style). You can add or remove layers without modifying the underlying person.

#### The Problem It Solves

Without Decorator, you face subclass explosion:
`Coffee`, `CoffeeWithMilk`, `CoffeeWithSugar`, `CoffeeWithMilkAndSugar`… → combinatorial explosion.

#### Production-Level Python Implementation

```python
from abc import ABC, abstractmethod


class Coffee(ABC):
    """Component — the base interface."""
    @abstractmethod
    def cost(self) -> float:
        pass

    @abstractmethod
    def description(self) -> str:
        pass


class SimpleCoffee(Coffee):
    """Concrete component — base object being decorated."""
    def cost(self) -> float:
        return 5.0

    def description(self) -> str:
        return "Simple Coffee"


class CoffeeDecorator(Coffee, ABC):
    """
    Abstract Decorator — wraps a Coffee instance.
    Maintains the same interface as Coffee (key to transparent wrapping).
    """
    def __init__(self, coffee: Coffee) -> None:
        self._coffee = coffee  # Wrapped component

    def cost(self) -> float:
        return self._coffee.cost()

    def description(self) -> str:
        return self._coffee.description()


class MilkDecorator(CoffeeDecorator):
    """Concrete Decorator — adds milk."""
    def cost(self) -> float:
        return self._coffee.cost() + 2.0

    def description(self) -> str:
        return self._coffee.description() + " + Milk"


class SugarDecorator(CoffeeDecorator):
    """Concrete Decorator — adds sugar."""
    def cost(self) -> float:
        return self._coffee.cost() + 1.0

    def description(self) -> str:
        return self._coffee.description() + " + Sugar"


# Stacking decorators at runtime
coffee: Coffee = SimpleCoffee()
coffee = MilkDecorator(coffee)    # Wrap with milk
coffee = SugarDecorator(coffee)   # Wrap again with sugar

print(coffee.description())  # Simple Coffee + Milk + Sugar
print(f"${coffee.cost():.2f}")  # $8.00
```

#### Python’s Native Decorator Syntax

```python
import functools
from typing import Callable


def rate_limiter(func: Callable) -> Callable:
    """Pythonic decorator — adds rate limiting to any function dynamically."""
    call_count = 0

    @functools.wraps(func)  # Preserves docstrings and function metadata
    def wrapper(*args, **kwargs):
        nonlocal call_count
        call_count += 1
        print(f"Call #{call_count} to {func.__name__}")
        return func(*args, **kwargs)

    return wrapper


@rate_limiter
def fetch_data(url: str) -> str:
    return f"Data from {url}"


# Used extensively in: Flask routes, authentication, caching, rate limiting
```

#### Pros & Cons

|✅ Advantages                                             |❌ Disadvantages                       |
|---------------------------------------------------------|--------------------------------------|
|Runtime flexibility — behavior added without code changes|Many small wrapper objects            |
|Avoids subclass explosion                                |Debugging call stack becomes confusing|
|Follows Open/Closed Principle                            |Order of decoration matters           |
|Clean feature layering                                   |                                      |

-----

### 9.5 Facade Pattern

#### Plain English Concept

Provides a **simplified interface to a complex subsystem**. Hides internal complexity behind a clean, unified API.

#### Real-World Analogy

> 💻 Starting a computer. You press one button. Behind the scenes: BIOS checks, CPU initializes, RAM is loaded, disk drivers start. The power button is the Facade.

#### Production-Level Python Implementation

```python
class CPU:
    def start(self) -> str:
        return "CPU: initialized"


class Memory:
    def load(self) -> str:
        return "Memory: loaded OS into RAM"


class DiskDriver:
    def initialize(self) -> str:
        return "Disk: drivers loaded"


class NetworkCard:
    def connect(self) -> str:
        return "Network: connected"


class ComputerFacade:
    """
    Facade — client calls one simple method.
    All subsystem complexity is hidden and coordinated internally.
    """
    def __init__(self) -> None:
        self._cpu = CPU()
        self._memory = Memory()
        self._disk = DiskDriver()
        self._network = NetworkCard()

    def start(self) -> None:
        """Single entry point hides all subsystem orchestration."""
        print(self._cpu.start())
        print(self._memory.load())
        print(self._disk.initialize())
        print(self._network.connect())
        print("System ready.")


# Client code — beautifully simple
computer = ComputerFacade()
computer.start()
```

#### Real-World Usage

- SDK entry points (e.g., `boto3.client("s3")`)
- Django ORM (`Model.objects.filter(...)`)
- Flask app initialization (`create_app()`)

-----

### 9.6 Flyweight Pattern

#### Plain English Concept

Reduces memory usage by **sharing common object data** across many objects. Splits object state into *intrinsic* (shared) and *extrinsic* (unique per instance).

#### Real-World Analogy

> 📚 A library. Many students share the same book copy (shared intrinsic data — title, content). Each student has their own bookmark position (extrinsic data — their personal state).

#### The Problem It Solves

A game with 10,000 trees. Each tree stores texture, color, height, and position. But most trees share the same texture and color. Storing these per-tree wastes massive memory.

#### Production-Level Python Implementation

```python
from typing import Dict


class TreeType:
    """
    Flyweight — stores INTRINSIC (shared) state.
    This object is shared across all trees with the same type.
    """
    def __init__(self, color: str, texture: str) -> None:
        self.color = color    # Shared — same for thousands of trees
        self.texture = texture  # Shared — loaded once

    def render(self, x: int, y: int) -> None:
        print(f"Rendering {self.color} tree at ({x}, {y}) with {self.texture} texture")


class TreeFactory:
    """Factory manages the pool of shared Flyweight objects."""
    _types: Dict[str, TreeType] = {}

    @classmethod
    def get_type(cls, color: str, texture: str) -> TreeType:
        key = f"{color}_{texture}"
        if key not in cls._types:
            print(f"Creating new TreeType: {key}")  # Created only ONCE
            cls._types[key] = TreeType(color, texture)
        return cls._types[key]


class Tree:
    """
    Context — stores EXTRINSIC (unique) state per instance.
    References the shared Flyweight instead of copying it.
    """
    def __init__(self, x: int, y: int, tree_type: TreeType) -> None:
        self.x = x           # Unique per tree
        self.y = y           # Unique per tree
        self.type = tree_type  # SHARED — pointer to flyweight, not a copy

    def render(self) -> None:
        self.type.render(self.x, self.y)


# 10,000 trees — only 2 TreeType objects created in memory
trees = [
    Tree(x * 5, y * 5, TreeFactory.get_type("Green", "Oak"))
    for x in range(100) for y in range(100)
]

# Verify sharing
t1_type = trees[0].type
t2_type = trees[999].type
print(t1_type is t2_type)  # True — same object in memory
```

#### Real-World Usage

- Game engines (particles, vegetation, characters)
- Text editors (character rendering objects)
- Browser DOM optimization
- Icon rendering systems
- Map tile systems

#### Pros & Cons

|✅ Advantages                                |❌ Disadvantages                                     |
|--------------------------------------------|----------------------------------------------------|
|Massive memory savings                      |Complex design — requires strict separation of state|
|Performance improvement through object reuse|Hard to debug                                       |
|Enables massive-scale object counts         |Requires immutability discipline for shared state   |

-----

### 9.7 Proxy Pattern

#### Plain English Concept

Provides a **substitute object that controls access** to the real object. The proxy sits between the client and the real object, intercepting calls to add behavior.

#### Real-World Analogy

> 🛡️ A security guard at an office building. You don’t directly meet the CEO — the guard controls access, verifies credentials, and may block or log the request.

#### The Problem It Solves

Real objects may be:

- **Heavy to create** → use Virtual Proxy (lazy loading)
- **Remote** → use Remote Proxy (network call)
- **Sensitive** → use Protection Proxy (access control)
- **Expensive to call repeatedly** → use Caching Proxy

#### Production-Level Python Implementation

```python
from abc import ABC, abstractmethod
from typing import Optional


class ImageInterface(ABC):
    @abstractmethod
    def display(self) -> None:
        pass


class RealImage(ImageInterface):
    """
    Heavy real object — expensive to load from disk.
    Should only be created when absolutely necessary.
    """
    def __init__(self, filepath: str) -> None:
        self._filepath = filepath
        self._load_from_disk()  # Expensive operation

    def _load_from_disk(self) -> None:
        print(f"Loading image from disk: {self._filepath}")

    def display(self) -> None:
        print(f"Displaying: {self._filepath}")


class ProxyImage(ImageInterface):
    """
    Proxy — same interface as RealImage.
    Implements lazy loading: real image created only on first display().
    Subsequent calls reuse the already-loaded image.
    """
    def __init__(self, filepath: str) -> None:
        self._filepath = filepath
        self._real_image: Optional[RealImage] = None  # Not loaded yet

    def display(self) -> None:
        if self._real_image is None:
            # Lazy initialization — create real object only when needed
            self._real_image = RealImage(self._filepath)
        print("Proxy: access granted")
        self._real_image.display()


# Usage
img = ProxyImage("photo.jpg")   # No disk load yet
img.display()                   # Loads from disk on first call
img.display()                   # Uses cached real image
```

#### Real-World Usage

- **API Gateway** — Proxy that adds auth, rate limiting, logging
- **Lazy database loading** (SQLAlchemy’s lazy relationship loading)
- **Virtual image loading** in browsers
- **Network proxies** (caching, load balancing)
- **Mocking in tests** — proxy replaces real external service

#### Pros & Cons

|✅ Advantages                                        |❌ Disadvantages               |
|----------------------------------------------------|------------------------------|
|Lazy loading — defer expensive creation             |Extra indirection layer       |
|Security layer — control access to sensitive objects|Can increase response time    |
|Transparent logging and caching                     |Adds implementation complexity|
|Follows Open/Closed Principle                       |                              |

-----

## 10. Behavioral Patterns

-----

### 10.1 Observer Pattern

#### Plain English Concept

Defines a **one-to-many subscription relationship** between objects. When one object (Subject) changes state, all subscribed dependents (Observers) are notified automatically.

#### Real-World Analogy

> 📧 Email newsletter subscription. You subscribe once. The company sends updates to all subscribers whenever new content is published. You can subscribe or unsubscribe at any time.

#### The Problem It Solves

Without Observer, modules must constantly poll for changes — wasteful and tightly coupled. Observer enables true **event-driven architecture**.

#### Production-Level Python Implementation

```python
from abc import ABC, abstractmethod
from typing import List


class Observer(ABC):
    """Observer interface — all subscribers must implement update()."""
    @abstractmethod
    def update(self, data: str) -> None:
        pass


class Subject:
    """
    Subject (Publisher) — maintains a list of observers.
    Notifies all observers when state changes.
    """
    def __init__(self) -> None:
        self._observers: List[Observer] = []
        self._state: str = ""

    def subscribe(self, observer: Observer) -> None:
        self._observers.append(observer)

    def unsubscribe(self, observer: Observer) -> None:
        self._observers.remove(observer)  # Important: prevent memory leaks

    def notify(self) -> None:
        for observer in self._observers:
            observer.update(self._state)

    def set_state(self, new_state: str) -> None:
        """Triggers notification automatically on state change."""
        self._state = new_state
        self.notify()  # Push model — subject pushes data to observers


class UserPortfolio(Observer):
    def update(self, data: str) -> None:
        print(f"Portfolio updated: {data}")


class NewsAlert(Observer):
    def update(self, data: str) -> None:
        print(f"ALERT — Breaking news: {data}")


class RiskEngine(Observer):
    def update(self, data: str) -> None:
        print(f"Risk engine recalculating for: {data}")


# Usage
stock_market = Subject()
stock_market.subscribe(UserPortfolio())
stock_market.subscribe(NewsAlert())
stock_market.subscribe(RiskEngine())

stock_market.set_state("AAPL dropped 5%")
# All three observers notified simultaneously
```

#### Real-World Usage

- Django signals (`post_save`, `pre_delete`)
- Event buses in microservices (Kafka, RabbitMQ conceptually)
- YouTube / newsletter subscription systems
- Live sports score feeds
- GUI event systems (button clicks)
- Pub/Sub messaging patterns

#### Pros & Cons

|✅ Advantages                                     |❌ Disadvantages                                      |
|-------------------------------------------------|-----------------------------------------------------|
|Real-time, event-driven updates                  |Debugging is difficult — notification order matters  |
|Decoupled communication                          |Too many notifications can cascade performance issues|
|Scalable — add observers without touching Subject|Memory leaks if observers are not unsubscribed       |
|Open/Closed Principle                            |                                                     |

-----

### 10.2 Strategy Pattern

#### Plain English Concept

Defines a family of algorithms, encapsulates each one, and makes them **interchangeable at runtime** without changing the client.

#### Real-World Analogy

> 🗺️ Navigation app. Same app, different routing strategies: Car Route, Walking Route, Bicycle Route, Public Transit. You switch strategies without rebuilding the navigation system.

#### Production-Level Python Implementation

```python
from abc import ABC, abstractmethod
from typing import List


class SortStrategy(ABC):
    """Strategy interface — all algorithms must implement sort()."""
    @abstractmethod
    def sort(self, data: List[int]) -> List[int]:
        pass


class BubbleSortStrategy(SortStrategy):
    def sort(self, data: List[int]) -> List[int]:
        arr = data.copy()
        n = len(arr)
        for i in range(n):
            for j in range(0, n - i - 1):
                if arr[j] > arr[j + 1]:
                    arr[j], arr[j + 1] = arr[j + 1], arr[j]
        return arr


class QuickSortStrategy(SortStrategy):
    def sort(self, data: List[int]) -> List[int]:
        if len(data) <= 1:
            return data
        pivot = data[len(data) // 2]
        left = [x for x in data if x < pivot]
        middle = [x for x in data if x == pivot]
        right = [x for x in data if x > pivot]
        return self.sort(left) + middle + self.sort(right)


class Sorter:
    """
    Context — holds a reference to a strategy.
    Can switch strategies at runtime without any internal changes.
    """
    def __init__(self, strategy: SortStrategy) -> None:
        self._strategy = strategy

    def set_strategy(self, strategy: SortStrategy) -> None:
        """Runtime strategy swap — no code changes needed."""
        self._strategy = strategy

    def sort(self, data: List[int]) -> List[int]:
        return self._strategy.sort(data)


sorter = Sorter(QuickSortStrategy())
print(sorter.sort([3, 1, 4, 1, 5]))  # [1, 1, 3, 4, 5]

sorter.set_strategy(BubbleSortStrategy())  # Swap at runtime
print(sorter.sort([3, 1, 4, 1, 5]))  # Same result, different algorithm
```

#### Pythonic Strategy with First-Class Functions

```python
from typing import Callable, List

# In Python, functions are first-class objects.
# A simple dictionary of functions can replace the entire Strategy class hierarchy.

strategies: dict[str, Callable[[List[int]], List[int]]] = {
    "bubble": lambda d: sorted(d),   # Simplified for demonstration
    "quick":  lambda d: sorted(d, reverse=False),
    "merge":  lambda d: sorted(d),
}

data = [5, 2, 8, 1, 9]
selected_strategy = "quick"
result = strategies[selected_strategy](data)
print(result)  # [1, 2, 5, 8, 9]
```

#### Pros & Cons

|✅ Advantages                                                   |❌ Disadvantages                              |
|---------------------------------------------------------------|---------------------------------------------|
|Open/Closed Principle — add algorithms without changing context|Many strategy classes in complex systems     |
|Replace logic dynamically at runtime                           |Clients must be aware of different strategies|
|Cleaner code — eliminates large if-else chains                 |Overhead for simple cases                    |

-----

### 10.3 State Pattern

#### Plain English Concept

Allows an object to **alter its behavior when its internal state changes**. The object appears to change its class.

#### Real-World Analogy

> 🏧 ATM machine. When it has no card inserted, it only accepts cards. When a card is inserted, it asks for PIN. When authenticated, it allows transactions. The ATM’s behavior is entirely determined by its current state.

#### Production-Level Python Implementation

```python
from abc import ABC, abstractmethod


class OrderState(ABC):
    """State interface — each concrete state knows what behavior is valid."""
    @abstractmethod
    def process(self, order: "Order") -> None:
        pass

    @abstractmethod
    def cancel(self, order: "Order") -> None:
        pass


class CreatedState(OrderState):
    def process(self, order: "Order") -> None:
        print("Payment received. Moving to Paid.")
        order.change_state(PaidState())

    def cancel(self, order: "Order") -> None:
        print("Order cancelled before payment.")


class PaidState(OrderState):
    def process(self, order: "Order") -> None:
        print("Order dispatched. Moving to Shipped.")
        order.change_state(ShippedState())

    def cancel(self, order: "Order") -> None:
        print("Order cancelled — refund initiated.")


class ShippedState(OrderState):
    def process(self, order: "Order") -> None:
        print("Order delivered. Moving to Delivered.")
        order.change_state(DeliveredState())

    def cancel(self, order: "Order") -> None:
        print("Cannot cancel — order already shipped.")


class DeliveredState(OrderState):
    def process(self, order: "Order") -> None:
        print("Order complete.")

    def cancel(self, order: "Order") -> None:
        print("Cannot cancel — order already delivered.")


class Order:
    """
    Context — delegates all behavior to the current state object.
    State transitions happen inside state classes — no giant if-else.
    """
    def __init__(self) -> None:
        self._state: OrderState = CreatedState()

    def change_state(self, state: OrderState) -> None:
        self._state = state

    def process(self) -> None:
        self._state.process(self)

    def cancel(self) -> None:
        self._state.cancel(self)


order = Order()
order.process()   # Payment received. Moving to Paid.
order.process()   # Order dispatched. Moving to Shipped.
order.cancel()    # Cannot cancel — order already shipped.
```

#### Pros & Cons

|✅ Advantages                            |❌ Disadvantages           |
|----------------------------------------|--------------------------|
|Eliminates large if-else / switch chains|Many state classes        |
|Clean, explicit state transitions       |Slight complexity overhead|
|Each state is independently maintainable|                          |

-----

### 10.4 Chain of Responsibility

**Concept:** Pass a request along a chain of handlers. Each handler decides to process the request or pass it to the next handler in the chain.

**Real Usage:** Django/Flask middleware, logging level hierarchies (DEBUG → INFO → WARNING → ERROR), HTTP request pipelines.

```python
from abc import ABC, abstractmethod
from typing import Optional


class Handler(ABC):
    def __init__(self) -> None:
        self._next: Optional["Handler"] = None

    def set_next(self, handler: "Handler") -> "Handler":
        self._next = handler
        return handler  # Enables chaining: h1.set_next(h2).set_next(h3)

    @abstractmethod
    def handle(self, level: int, message: str) -> None:
        pass


class DebugHandler(Handler):
    def handle(self, level: int, message: str) -> None:
        if level == 1:
            print(f"DEBUG: {message}")
        elif self._next:
            self._next.handle(level, message)


class InfoHandler(Handler):
    def handle(self, level: int, message: str) -> None:
        if level == 2:
            print(f"INFO: {message}")
        elif self._next:
            self._next.handle(level, message)


class ErrorHandler(Handler):
    def handle(self, level: int, message: str) -> None:
        if level >= 3:
            print(f"ERROR: {message}")
        elif self._next:
            self._next.handle(level, message)


# Build chain
debug = DebugHandler()
info = InfoHandler()
error = ErrorHandler()
debug.set_next(info).set_next(error)

debug.handle(1, "Variable x = 5")    # DEBUG
debug.handle(2, "Server started")    # INFO
debug.handle(3, "Disk full!")        # ERROR
```

> 📝 **Revision:** Chain = Pipeline processing.

-----

### 10.5 Command Pattern

**Concept:** Encapsulates a request as a standalone object. This enables parameterization, queuing, logging, and undo/redo operations.

**Real Usage:** Undo/Redo in editors, job queues (Celery tasks), GUI button actions, transaction logging.

```python
from abc import ABC, abstractmethod
from typing import List


class Command(ABC):
    @abstractmethod
    def execute(self) -> None:
        pass

    @abstractmethod
    def undo(self) -> None:
        pass


class TextEditor:
    def __init__(self) -> None:
        self.text: str = ""

    def write(self, text: str) -> None:
        self.text += text

    def delete(self, count: int) -> None:
        self.text = self.text[:-count]


class WriteCommand(Command):
    def __init__(self, editor: TextEditor, text: str) -> None:
        self._editor = editor
        self._text = text

    def execute(self) -> None:
        self._editor.write(self._text)

    def undo(self) -> None:
        self._editor.delete(len(self._text))


class CommandInvoker:
    """Invoker — executes commands and maintains undo history."""
    def __init__(self) -> None:
        self._history: List[Command] = []

    def execute(self, command: Command) -> None:
        command.execute()
        self._history.append(command)

    def undo(self) -> None:
        if self._history:
            self._history.pop().undo()


editor = TextEditor()
invoker = CommandInvoker()
invoker.execute(WriteCommand(editor, "Hello, "))
invoker.execute(WriteCommand(editor, "World!"))
print(editor.text)  # Hello, World!
invoker.undo()
print(editor.text)  # Hello,
```

> 📝 **Revision:** Command = Action object (request encapsulated as data).

-----

### 10.6 Iterator Pattern

**Concept:** Provides a way to access elements of a collection sequentially **without exposing its internal structure**.

Python natively implements this pattern via `__iter__` and `__next__`. It’s baked into the language.

```python
from typing import Iterator, List


class NumberRange:
    """
    Custom iterable — hides internal storage details.
    Python's for-loop protocol calls __iter__ then __next__ automatically.
    """
    def __init__(self, start: int, end: int, step: int = 1) -> None:
        self._start = start
        self._end = end
        self._step = step

    def __iter__(self) -> Iterator[int]:
        current = self._start
        while current < self._end:
            yield current  # Generator — most Pythonic iterator implementation
            current += self._step


# Client code — uniform traversal regardless of underlying structure
for num in NumberRange(0, 10, 2):
    print(num)  # 0, 2, 4, 6, 8
```

> 📝 **Revision:** Iterator = Controlled traversal (hides structure).

-----

### 10.7 Mediator Pattern

**Concept:** A central object (Mediator) handles all communication between objects. Objects don’t communicate directly — they go through the mediator.

**Real Usage:** Chat room servers, air traffic control systems, GUI component coordination, event buses.

```python
from typing import List


class ChatRoom:
    """Mediator — central communication hub."""
    def __init__(self) -> None:
        self._members: List["User"] = []

    def join(self, user: "User") -> None:
        self._members.append(user)
        print(f"{user.name} joined the chat")

    def broadcast(self, sender: "User", message: str) -> None:
        for member in self._members:
            if member is not sender:
                member.receive(sender.name, message)


class User:
    def __init__(self, name: str, room: ChatRoom) -> None:
        self.name = name
        self._room = room

    def send(self, message: str) -> None:
        print(f"{self.name}: {message}")
        self._room.broadcast(self, message)

    def receive(self, sender: str, message: str) -> None:
        print(f"  [{self.name} received from {sender}]: {message}")


room = ChatRoom()
alice = User("Alice", room)
bob = User("Bob", room)
room.join(alice)
room.join(bob)
alice.send("Hello everyone!")
```

> 📝 **Revision:** Mediator = Communication hub (reduces N×N to N×1 connections).

-----

### 10.8 Memento Pattern

**Concept:** Captures and stores an object’s internal state so it can be **restored to that state** later — without violating encapsulation.

**Real Usage:** Undo/Redo systems, game save states, database transaction rollbacks, version history.

```python
from dataclasses import dataclass
from typing import List


@dataclass
class EditorMemento:
    """Memento — snapshot of editor state. Immutable by convention."""
    content: str


class TextEditor:
    def __init__(self) -> None:
        self._content: str = ""

    def write(self, text: str) -> None:
        self._content += text

    def save(self) -> EditorMemento:
        return EditorMemento(self._content)

    def restore(self, memento: EditorMemento) -> None:
        self._content = memento.content

    @property
    def content(self) -> str:
        return self._content


class History:
    """Caretaker — manages memento stack without knowing its contents."""
    def __init__(self) -> None:
        self._snapshots: List[EditorMemento] = []

    def save(self, memento: EditorMemento) -> None:
        self._snapshots.append(memento)

    def undo(self) -> EditorMemento:
        return self._snapshots.pop()


editor = TextEditor()
history = History()

editor.write("First sentence. ")
history.save(editor.save())
editor.write("Second sentence.")
print(editor.content)  # First sentence. Second sentence.

editor.restore(history.undo())
print(editor.content)  # First sentence.
```

> 📝 **Revision:** Memento = State snapshot (undo capability).

-----

### 10.9 Template Method Pattern

**Concept:** Defines the **skeleton of an algorithm** in a base class, with specific steps deferred to subclasses. The overall structure is fixed; the details vary.

**Real Usage:** Data processing pipelines, report generators, game loops, framework hooks.

```python
from abc import ABC, abstractmethod


class DataProcessor(ABC):
    """
    Template Method — the 'process' method defines the fixed algorithm skeleton.
    Subclasses override individual steps but cannot change the overall structure.
    """
    def process(self) -> None:
        """The template method — FINAL algorithm structure."""
        raw_data = self.fetch_data()
        cleaned = self.clean_data(raw_data)
        result = self.analyze(cleaned)
        self.generate_report(result)

    @abstractmethod
    def fetch_data(self) -> list:
        pass

    @abstractmethod
    def clean_data(self, data: list) -> list:
        pass

    def analyze(self, data: list) -> dict:
        """Optional hook — default implementation, can be overridden."""
        return {"count": len(data), "data": data}

    def generate_report(self, result: dict) -> None:
        print(f"Report: {result}")


class CSVProcessor(DataProcessor):
    def fetch_data(self) -> list:
        return ["row1,data", "row2,data", "row3,data"]

    def clean_data(self, data: list) -> list:
        return [row.strip() for row in data if row]


class JSONProcessor(DataProcessor):
    def fetch_data(self) -> list:
        return ['{"key": "val1"}', '{"key": "val2"}']

    def clean_data(self, data: list) -> list:
        return [item for item in data if item.startswith("{")]


CSVProcessor().process()
JSONProcessor().process()
```

> 📝 **Revision:** Template Method = Algorithm blueprint with customizable steps.

-----

### 10.10 Visitor Pattern

**Concept:** Lets you **add new operations to existing class hierarchies** without modifying those classes. Separates the operation from the object structure.

**Real Usage:** Compilers (AST traversal), static analysis tools, serialization, report generation across heterogeneous object trees.

```python
from abc import ABC, abstractmethod


class FileSystemElement(ABC):
    @abstractmethod
    def accept(self, visitor: "Visitor") -> None:
        pass


class ImageFile(FileSystemElement):
    def __init__(self, name: str, size_mb: float) -> None:
        self.name = name
        self.size_mb = size_mb

    def accept(self, visitor: "Visitor") -> None:
        visitor.visit_image(self)


class VideoFile(FileSystemElement):
    def __init__(self, name: str, duration_min: int) -> None:
        self.name = name
        self.duration_min = duration_min

    def accept(self, visitor: "Visitor") -> None:
        visitor.visit_video(self)


class Visitor(ABC):
    @abstractmethod
    def visit_image(self, image: ImageFile) -> None:
        pass

    @abstractmethod
    def visit_video(self, video: VideoFile) -> None:
        pass


class SizeCalculatorVisitor(Visitor):
    """New operation added WITHOUT modifying any FileSystemElement class."""
    def visit_image(self, image: ImageFile) -> None:
        print(f"Image '{image.name}': {image.size_mb}MB")

    def visit_video(self, video: VideoFile) -> None:
        estimated_size = video.duration_min * 150  # ~150MB/minute
        print(f"Video '{video.name}': ~{estimated_size}MB")


elements = [ImageFile("photo.jpg", 5.2), VideoFile("tutorial.mp4", 45)]
visitor = SizeCalculatorVisitor()
for element in elements:
    element.accept(visitor)
```

> 📝 **Revision:** Visitor = External behavior injection (double dispatch).

-----

## 11. Advanced Pythonic Implementations

Python’s unique features often allow GoF patterns to be simplified, replaced, or expressed more elegantly than their Java counterparts.

### Decorators as a First-Class Pattern System

Python’s `@decorator` syntax IS the Decorator pattern, but also subsumes several others:

```python
import functools
import time
from typing import Callable, Any


# Decorator replacing Proxy pattern (logging + timing)
def timed_proxy(func: Callable) -> Callable:
    @functools.wraps(func)
    def wrapper(*args: Any, **kwargs: Any) -> Any:
        start = time.perf_counter()
        result = func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        print(f"{func.__name__} took {elapsed:.4f}s")
        return result
    return wrapper


# Decorator as Singleton
def singleton(cls):
    instances = {}
    @functools.wraps(cls)
    def get_instance(*args, **kwargs):
        if cls not in instances:
            instances[cls] = cls(*args, **kwargs)
        return instances[cls]
    return get_instance


@singleton
class Config:
    def __init__(self):
        self.settings = {"debug": False, "db_url": "localhost"}
```

### Duck Typing Replacing Abstract Base Classes

```python
# In Python, you don't need ABCs for Strategy pattern.
# Any object with the right method works — this is Duck Typing.

class PDFExporter:
    def export(self, data: str) -> str:
        return f"PDF: {data}"

class CSVExporter:
    def export(self, data: str) -> str:
        return f"CSV: {data}"

# No shared base class needed — both "quack" the same way
def generate_report(exporter, data: str) -> str:
    return exporter.export(data)  # Works with ANY object that has .export()

print(generate_report(PDFExporter(), "Sales Q4"))
print(generate_report(CSVExporter(), "Sales Q4"))
```

### First-Class Functions Replacing Strategy

```python
# Python functions ARE objects. A dictionary of functions IS a Strategy registry.
from typing import Callable, Dict

PaymentStrategy = Callable[[float], str]

strategies: Dict[str, PaymentStrategy] = {
    "card": lambda amt: f"Card charged: ${amt:.2f}",
    "upi":  lambda amt: f"UPI transfer: ₹{amt:.2f}",
    "cash": lambda amt: f"Cash received: ${amt:.2f}",
}

def process_payment(method: str, amount: float) -> str:
    strategy = strategies.get(method)
    if not strategy:
        raise ValueError(f"Unknown payment method: {method}")
    return strategy(amount)

print(process_payment("upi", 1500))
```

### Generators as Iterator Pattern

```python
# Python generators ARE the Iterator pattern — no class boilerplate needed
from typing import Generator

def fibonacci_iterator(limit: int) -> Generator[int, None, None]:
    """A generator IS an iterator — lazy, memory-efficient, Pythonic."""
    a, b = 0, 1
    while a < limit:
        yield a
        a, b = b, a + b

for num in fibonacci_iterator(100):
    print(num, end=" ")  # 0 1 1 2 3 5 8 13 21 34 55 89
```

> 💡 **Senior Insight:** Python’s language features — decorators, generators, first-class functions, duck typing, modules — don’t make patterns *obsolete*. They make patterns *more expressive*. The GoF vocabulary still communicates intent clearly; Python just reduces the ceremony.

-----

## 12. Master Revision Table

|Category      |Pattern                |Core Problem Solved                        |One-Line Revision              |Real Python Example              |
|--------------|-----------------------|-------------------------------------------|-------------------------------|---------------------------------|
|**Creational**|Singleton              |Multiple instances of shared resource      |One instance, global access    |`import config` (module)         |
|**Creational**|Factory Method         |Tight coupling to concrete classes         |Centralize object creation     |Django DB backend                |
|**Creational**|Abstract Factory       |Incompatible object family mixing          |Create consistent families     |SQLAlchemy dialects              |
|**Creational**|Builder                |Telescoping constructors                   |Step-by-step construction      |SQLAlchemy query builder         |
|**Creational**|Prototype              |Expensive object re-construction           |Clone instead of construct     |`copy.deepcopy()`                |
|**Structural**|Adapter                |Incompatible interfaces                    |Interface translator           |Legacy API integration           |
|**Structural**|Bridge                 |Class explosion from multiple dimensions   |Separate abstraction from impl |GUI framework layers             |
|**Structural**|Composite              |Treating trees uniformly                   |Leaf and branch, same interface|File system, DOM tree            |
|**Structural**|Decorator              |Adding features without subclassing        |Dynamic feature wrapping       |Flask `@route`, `@login_required`|
|**Structural**|Facade                 |Complex subsystem                          |Simplification layer           |`boto3.client("s3")`             |
|**Structural**|Flyweight              |Memory usage with many similar objects     |Share intrinsic state          |Game engines, text editors       |
|**Structural**|Proxy                  |Controlling access to real object          |Controlled access wrapper      |API Gateway, lazy loading        |
|**Behavioral**|Observer               |Tight polling coupling                     |Event subscription             |Django signals, Pub/Sub          |
|**Behavioral**|Strategy               |Hardcoded algorithm selection              |Runtime algorithm swap         |Payment method selection         |
|**Behavioral**|State                  |Giant if-else for state-dependent behavior |State-based behavior           |Order lifecycle, ATM             |
|**Behavioral**|Chain of Responsibility|Sequential, conditional request handling   |Pipeline processing            |Middleware, logging              |
|**Behavioral**|Command                |Undo/redo, queuing operations              |Action as object               |Celery tasks, editor undo        |
|**Behavioral**|Iterator               |Collection traversal details               |Controlled traversal           |Python `for` loop protocol       |
|**Behavioral**|Mediator               |N×N object communication complexity        |Communication hub              |Chat room, event bus             |
|**Behavioral**|Memento                |Object state history / undo                |State snapshot                 |Version history, game save       |
|**Behavioral**|Template Method        |Algorithm duplication with slight variation|Algorithm blueprint            |Data processing pipelines        |
|**Behavioral**|Visitor                |Adding operations to closed hierarchies    |External behavior injection    |Compiler AST, analyzers          |

-----

## 13. Real-World Industry Usage

|Pattern         |Industry System               |How It’s Used                                                                  |
|----------------|------------------------------|-------------------------------------------------------------------------------|
|Singleton       |**Logging systems**           |One logger instance per application; writes to same file/stream                |
|Singleton       |**Database connection pools** |PgBouncer, SQLAlchemy pool — one pool, multiple borrows                        |
|Factory         |**Django ORM backend**        |`DATABASES["default"]["ENGINE"]` selects the DB driver factory                 |
|Factory         |**Payment SDKs**              |`PaymentFactory.create("stripe")` returns the correct gateway object           |
|Abstract Factory|**SQLAlchemy dialects**       |Entire SQL dialect (types, functions, syntax) swapped by changing engine URL   |
|Abstract Factory|**Cross-platform GUIs**       |Qt/GTK widget factories produce consistent platform-native components          |
|Builder         |**SQLAlchemy query builder**  |`session.query(User).filter(...).order_by(...).limit(10).all()`                |
|Builder         |**HTTP request libraries**    |`requests.Request(method, url, headers={}, params={})`                         |
|Prototype       |**ML pipeline configurations**|Clone a trained model config, modify hyperparameters, retrain                  |
|Observer        |**Django signals**            |`post_save.connect(send_welcome_email, sender=User)`                           |
|Observer        |**Kafka / RabbitMQ**          |Publishers push events; multiple consumer groups (observers) react             |
|Strategy        |**Sorting algorithms**        |Python’s `sorted(data, key=lambda x: x.name)` — key function IS a strategy     |
|Decorator       |**Flask routing**             |`@app.route("/api/v1/users")` wraps view functions                             |
|Decorator       |**Authentication**            |`@login_required`, `@permission_required` add behavior without modifying views |
|Proxy           |**API Gateway** (AWS)         |Intercepts HTTP requests, adds auth, rate limiting, logging before real service|
|Proxy           |**SQLAlchemy lazy loading**   |`User.orders` returns a proxy that loads from DB only when accessed            |
|Facade          |**boto3**                     |`boto3.client("s3").upload_file(...)` hides complex AWS SDK internals          |
|Composite       |**DOM tree**                  |HTML elements — a `<div>` contains other elements; `render()` works uniformly  |
|Chain           |**Django middleware**         |`MIDDLEWARE` list — each middleware processes/passes the request               |
|Command         |**Celery tasks**              |`add.delay(4, 4)` — task encapsulated as a serializable command object         |

-----

## 14. Senior Interview Toolkit

### Interview Traps — Questions Designed to Catch You

**Singleton Traps:**

- *“Why override `__new__` instead of `__init__` for Singleton?”*
  → `__init__` runs AFTER the object is already created. To control whether creation happens at all, you must intercept at `__new__`.
- *“Is Singleton thread-safe by default in Python?”*
  → **No.** Without locking (`threading.Lock`), two threads can simultaneously pass the `is None` check and create two instances.
- *“Is Singleton an anti-pattern?”*
  → **Sometimes yes.** Global mutable state makes testing hard and creates hidden coupling. Prefer dependency injection when possible.
- *“What is the most Pythonic Singleton?”*
  → **Module-level variables.** Python’s import system loads modules exactly once and caches them. `import config` always returns the same module object.

**Factory Traps:**

- *“What’s the difference between Factory and Constructor?”*
  → Constructor creates a *specific* class. Factory decides *which* class to instantiate — the decision is delegated and decoupled.
- *“Is Simple Factory the same as Factory Method?”*
  → **No.** Simple Factory uses if-else or a dictionary. Factory Method is a true GoF pattern using polymorphism and inheritance to eliminate conditional logic.
- *“Factory vs Abstract Factory — when do you use which?”*
  → Factory: one product type, runtime decision. Abstract Factory: a *family* of related products that must remain compatible.

**Observer Traps:**

- *“What causes memory leaks in Observer pattern?”*
  → Observers that are never unsubscribed from the Subject. The Subject holds a reference, preventing garbage collection even when the observer is logically “done.”
- *“Push vs Pull model in Observer — what’s the difference?”*
  → Push: Subject sends data with notification. Pull: Subject sends notification only; observers fetch data themselves. Pull is more decoupled but requires observers to know the Subject’s interface.

**Strategy vs State Trap:**

- *“What’s the difference between Strategy and State patterns?”*
  → Both use polymorphism to swap behavior. In **Strategy**, the client controls which algorithm is used and can change it at will. In **State**, the *object itself* transitions between states based on internal conditions — state changes are automatic and encapsulated.

**Decorator Traps:**

- *“Difference between Python’s `@decorator` syntax and the GoF Decorator pattern?”*
  → Python’s `@decorator` is a language feature for wrapping functions/classes. The GoF Decorator pattern is an OOP structural pattern for wrapping objects at the class level. Python’s syntax often implements the GoF pattern, but not always.

### Tricky Scenario Questions

**Q: “You have a logging system where every module creates its own Logger object. What problem does this cause and which pattern solves it?”**

> A: Multiple logger instances write to the same file/stream with no coordination — race conditions, garbled output, wasted resources. Singleton pattern ensures one Logger instance is created and shared across all modules. In Python specifically, the `logging` module uses a module-level Singleton approach.

**Q: “Your e-commerce platform needs to support PayPal, Stripe, Razorpay, and future payment providers. The checkout code shouldn’t change when you add providers. What pattern do you use?”**

> A: Factory Method. Define a `PaymentProvider` abstract class. Each provider implements it. A `PaymentFactory` maps provider names to classes. Adding a new provider requires only: (1) a new class, (2) one registry entry — zero changes to checkout code.

**Q: “A UI framework needs to support Windows, macOS, and Linux themes. Each theme has its own Button, TextBox, and Dropdown. How do you prevent accidental mixing of components from different themes?”**

> A: Abstract Factory. Each platform gets a concrete factory (`WindowsFactory`, `MacOSFactory`) that produces a compatible family of components. Client code uses only the factory interface — it’s impossible to accidentally mix components.

**Q: “Your system has 50,000 particle objects in a game, each with color, texture, and position. Memory is being exhausted. What pattern helps?”**

> A: Flyweight. Color and texture are *intrinsic* (shared) — store them in a shared `ParticleType` object. Position is *extrinsic* (unique per particle). Each particle holds a reference to the shared type, not a copy. Memory usage drops from O(50,000 × all_fields) to O(50,000 × unique_fields + types × shared_fields).

**Q: “A developer says ‘I use design patterns everywhere — my code is now enterprise quality.’ Is this correct?”**

> A: **No — this is dangerous thinking.** Pattern complexity must match problem complexity. Applying patterns to simple problems creates over-engineered, hard-to-maintain code. Patterns are tools, not achievements. The most skilled engineers know *when not* to use a pattern.

**Q: “Can Abstract Factory use Factory Method internally?”**

> A: **Yes — very commonly.** Each creation method in an Abstract Factory (`create_button()`, `create_checkbox()`) is essentially a Factory Method. They’re complementary, not competing patterns.

**Q: “Can you implement the Observer pattern without classes in Python?”**

> A: Yes. Python’s built-in `signal` library, event dictionaries with function lists, or even simple callbacks achieve the Observer pattern without formal classes. This is Duck Typing in action.

-----

> 🔥 **Final Senior Wisdom:**
> Design patterns are not a syllabus. They are **mental models**.
> Do NOT try to memorize them — understand the *problem* each one solves.
> When you deeply understand the problems, the patterns emerge naturally.
> The goal is always: **low coupling, high cohesion, scalability, and maintainability.**

-----

*Reference: Design Patterns: Elements of Reusable Object-Oriented Software — Gamma, Helm, Johnson, Vlissides (GoF, 1994)*
*Further reading: [Design Pattern Decision Tree](https://medium.com/womenintechnology/stop-memorizing-design-patterns-use-this-decision-tree-instead-e84f22fca9fa)*

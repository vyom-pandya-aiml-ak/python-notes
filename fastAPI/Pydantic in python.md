# Mastering Pydantic V2: Data Validation for Python and FastAPI

> *A comprehensive, professional guide to building robust, type-safe Python applications.*

---

## Table of Contents

1. [Introduction to Pydantic](#1-introduction-to-pydantic)
2. [Core Foundations: BaseModel, Type Hints & Field](#2-core-foundations-basemodel-type-hints--field)
3. [Nested Models](#3-nested-models)
4. [Field Validators (@field_validator)](#4-field-validators-field_validator)
5. [Model Validators (@model_validator)](#5-model-validators-model_validator)
6. [Computed Fields (@computed_field)](#6-computed-fields-computed_field)
7. [Serialization: model_dump() & model_dump_json()](#7-serialization-model_dump--model_dump_json)
8. [FastAPI Integration](#8-fastapi-integration)
9. [Pydantic vs Python Dataclasses](#9-pydantic-vs-python-dataclasses)

---

## 1. Introduction to Pydantic

### What Is Pydantic?

**Pydantic** is the most widely used data validation library in the Python ecosystem. At its core, it allows you to define the **shape** and **rules** of your data using standard Python type hints — and then automatically validates, coerces, and serializes that data for you.

Pydantic V2 (released 2023) was a complete rewrite in Rust via the `pydantic-core` engine, delivering **5–50× faster** validation performance compared to V1.

### Why Pydantic Has Become the Industry Standard

Consider the following analogy:

> **🏦 The Bank Teller Analogy**
>
> Imagine handing a form to a bank teller. Before processing anything, the teller checks: *Is the account number 10 digits? Is the amount a positive number? Is the date format correct?* They **reject** the form immediately if anything is wrong, giving you precise feedback on exactly what to fix. Pydantic is that teller — sitting at the entry point of your system, validating every piece of incoming data before it ever touches your business logic.

Without Pydantic, developers write repetitive, error-prone validation code like this:

```python
# ❌ The old way — manual, brittle, hard to maintain
def create_user(data: dict):
    if "name" not in data:                    # Manually check field existence
        raise ValueError("name is required")   # Manually raise errors
    if not isinstance(data["age"], int):       # Manually check type
        raise TypeError("age must be an int")  # Manually raise type errors
    if data["age"] < 0:                        # Manually check constraints
        raise ValueError("age must be positive")
    # ... this scales terribly with more fields
```

With Pydantic, the same logic becomes declarative, readable, and automatic:

```python
# ✅ The Pydantic way — declarative, clean, auto-validated
from pydantic import BaseModel, Field

class User(BaseModel):                         # Inherit from BaseModel
    name: str                                  # Required string field
    age: int = Field(gt=0)                     # Integer, must be > 0

user = User(name="Alice", age=30)              # Validated automatically on creation
```

### Installation

```bash
# Install Pydantic V2 from PyPI
pip install pydantic

# Verify the installed version (should be 2.x.x)
python -c "import pydantic; print(pydantic.VERSION)"
```

---

## 2. Core Foundations: BaseModel, Type Hints & Field

### The Building Blocks

> **🧱 The Blueprint Analogy**
>
> A `BaseModel` subclass is like an architectural blueprint. The blueprint defines exactly what a building must have (rooms, doors, windows) and their specifications (minimum size, material type). When a contractor (your code) tries to construct something, they must follow the blueprint exactly — or construction is halted with a clear error report.

### BaseModel and Type Hints

```python
# foundation_example.py

from pydantic import BaseModel           # Import the base class for all models
from typing import Optional              # Import Optional for nullable fields

# Define a model by subclassing BaseModel
class Product(BaseModel):
    id: int                              # Required integer — Pydantic will coerce "42" -> 42
    name: str                            # Required string field
    price: float                         # Required float — coerces "9.99" -> 9.99
    description: Optional[str] = None   # Optional field with a default of None
    in_stock: bool = True                # Field with a default value of True

# --- Creating valid instances ---
laptop = Product(id=1, name="Laptop", price=999.99)   # Uses default for in_stock
print(laptop)
# Output: id=1 name='Laptop' price=999.99 description=None in_stock=True

# --- Pydantic automatically coerces compatible types ---
phone = Product(id="2", name="Phone", price="499.50") # Strings are coerced to int/float
print(phone.id)    # Output: 2  (coerced from "2" to int)
print(type(phone.id))  # Output: <class 'int'>

# --- Accessing fields like regular Python attributes ---
print(laptop.name)    # Output: Laptop
print(laptop.price)   # Output: 999.99

# --- Attempting to create an invalid instance raises a ValidationError ---
try:
    bad_product = Product(id="not-a-number", name="Broken", price=10.0)  # Invalid id
except Exception as e:
    print(e)   # Pydantic prints a detailed error message with location and reason
```

### The `Field` Function: Adding Constraints

The `Field` function is where Pydantic's true power emerges. It lets you attach metadata, default values, and validation constraints directly to fields.

```python
# field_constraints.py

from pydantic import BaseModel, Field    # Import Field for advanced field definitions
from typing import Optional

class Employee(BaseModel):
    # --- String Constraints ---
    employee_id: str = Field(
        ...,                             # '...' (Ellipsis) means the field is REQUIRED
        min_length=6,                    # String must be at least 6 characters
        max_length=10,                   # String must be at most 10 characters
        pattern=r"^EMP\d+$",            # Must match this regex pattern (e.g., "EMP001")
        description="Unique employee identifier"  # Used in API docs / schema
    )

    # --- Numeric Constraints ---
    salary: float = Field(
        gt=0,                            # gt = "greater than" — must be > 0
        le=500_000,                      # le = "less than or equal to" — must be <= 500,000
        description="Annual salary in USD"
    )
    age: int = Field(
        ge=18,                           # ge = "greater than or equal to" — must be >= 18
        lt=65,                           # lt = "less than" — must be < 65
    )

    # --- Default Values ---
    department: str = Field(
        default="General",               # Used when no value is provided
        max_length=50
    )

    # --- Alias: accept a different key name from external sources ---
    full_name: str = Field(
        ...,
        alias="fullName",                # Allows JSON key "fullName" to map to Python attr "full_name"
        min_length=2
    )

# --- Valid creation using alias ---
emp = Employee(
    employee_id="EMP001",
    salary=75000.0,
    age=30,
    fullName="Alice Johnson",            # Using the alias 'fullName' as defined
)
print(emp.full_name)   # Output: Alice Johnson (accessed via Python attr name)
print(emp.department)  # Output: General (default value used)

# --- Constraint Summary Reference ---
# gt  (greater than)              : value > x
# ge  (greater than or equal to)  : value >= x
# lt  (less than)                 : value < x
# le  (less than or equal to)     : value <= x
# multiple_of                     : value % x == 0
# min_length / max_length         : for strings and lists
# pattern                         : regex match for strings
```

---

## 3. Nested Models

### Composing Complex Data Structures

> **🪆 The Russian Nesting Doll Analogy**
>
> A nested model is like a Matryoshka doll. Each doll (model) is complete and validated on its own, but it can also live *inside* a larger doll. When you validate the outer doll, Pydantic automatically validates each inner doll as well — all the way down to the smallest one.

```python
# nested_models.py

from pydantic import BaseModel, Field    # Core imports
from typing import List, Optional        # List for collections, Optional for nullable
from datetime import datetime            # Standard library datetime

# --- Level 3: The innermost model ---
class Address(BaseModel):
    street: str                          # Street address line
    city: str                            # City name
    country: str                         # Country name
    postal_code: str = Field(min_length=3, max_length=10)  # Validated postal code

# --- Level 2: A mid-level model, contains primitives ---
class ContactInfo(BaseModel):
    email: str                           # Email address (string validation)
    phone: Optional[str] = None          # Optional phone number

# --- Level 1: The outermost model, composes Address and ContactInfo ---
class Customer(BaseModel):
    customer_id: int                     # Unique integer identifier
    name: str = Field(min_length=2)      # Name with minimum length constraint
    contact: ContactInfo                 # Nested model — automatically validated
    addresses: List[Address]             # A list of nested Address models
    created_at: datetime = Field(        # Datetime with a dynamic default
        default_factory=datetime.now     # Called fresh for every new instance
    )
    is_active: bool = True               # Simple boolean with a default

# --- Creating a nested structure from a plain dictionary ---
customer_data = {
    "customer_id": 101,
    "name": "Bob Smith",
    "contact": {                         # Pydantic parses this dict into a ContactInfo
        "email": "bob@example.com",
        "phone": "555-1234"
    },
    "addresses": [                       # Pydantic parses each dict into an Address
        {
            "street": "123 Main St",
            "city": "Springfield",
            "country": "USA",
            "postal_code": "62701"
        },
        {
            "street": "456 Oak Ave",
            "city": "Shelbyville",
            "country": "USA",
            "postal_code": "62702"
        }
    ]
}

customer = Customer(**customer_data)     # Unpack dict as keyword arguments

# --- Accessing nested data with dot notation ---
print(customer.name)                        # Output: Bob Smith
print(customer.contact.email)               # Output: bob@example.com
print(customer.addresses[0].city)           # Output: Springfield
print(customer.addresses[1].postal_code)    # Output: 62702
print(type(customer.contact))              # Output: <class 'ContactInfo'>
```

---

## 4. Field Validators (`@field_validator`)

### Custom Logic for Individual Fields

> **🔬 The Lab Quality Inspector Analogy**
>
> Imagine a quality inspector on a factory line. Each component (field) passes through a specific inspection station. The inspector for bolts checks bolt dimensions; the inspector for circuits checks voltage. Each station has its own specialized rules. `@field_validator` is exactly this — a dedicated inspection station for a single, specific field.

```python
# field_validators.py

from pydantic import BaseModel, Field, field_validator   # Import the decorator
from typing import Optional
import re                                                 # Standard library regex module

class UserRegistration(BaseModel):
    username: str = Field(min_length=3, max_length=20)   # Basic length constraint
    email: str                                            # Will be validated by custom logic
    password: str = Field(min_length=8)                  # Minimum 8 characters
    phone: Optional[str] = None                          # Optional phone number

    @field_validator("email")                            # Decorate to target the 'email' field
    @classmethod                                         # REQUIRED: must be a classmethod in V2
    def validate_email_format(cls, value: str) -> str:  # cls=the class, value=the field's value
        """Validates that the email has a proper format."""
        # Define a basic email regex pattern
        pattern = r"^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$"

        if not re.match(pattern, value):                 # Check if value matches the pattern
            raise ValueError(                            # Raise ValueError on failure
                f"'{value}' is not a valid email address format."
            )
        return value.lower()                             # Normalize to lowercase before storing

    @field_validator("username")                         # Target the 'username' field
    @classmethod
    def validate_username_no_spaces(cls, value: str) -> str:
        """Ensures the username contains no whitespace characters."""
        if " " in value:                                 # Check for space character
            raise ValueError("Username must not contain spaces.")
        return value.strip()                             # Strip any leading/trailing whitespace

    @field_validator("phone")                            # Target the 'phone' field
    @classmethod
    def validate_phone_format(cls, value: Optional[str]) -> Optional[str]:
        """Validates phone is digits-only if provided."""
        if value is None:                                # Skip validation if field is absent
            return value
        cleaned = re.sub(r"[\s\-\(\)]", "", value)      # Remove spaces, dashes, parentheses
        if not cleaned.isdigit():                        # Check remaining chars are all digits
            raise ValueError("Phone number must contain only digits.")
        return cleaned                                   # Return the cleaned version

# --- Testing valid data ---
user = UserRegistration(
    username="alice",
    email="  Alice@Example.COM  ",                      # Will be lowercased by validator
    password="securepassword123",
    phone="(555) 123-4567"                              # Will be cleaned by validator
)
print(user.email)    # Output: alice@example.com  (normalized)
print(user.phone)    # Output: 5551234567         (cleaned)

# --- Testing invalid email ---
try:
    bad_user = UserRegistration(
        username="bob",
        email="not-an-email",                           # Invalid — no '@' or domain
        password="password123"
    )
except Exception as e:
    print(e)         # Output: 1 validation error for UserRegistration / email / ...

# --- Testing invalid username ---
try:
    spaced_user = UserRegistration(
        username="has space",                           # Invalid — contains a space
        email="test@example.com",
        password="password123"
    )
except Exception as e:
    print(e)         # Output: 1 validation error for UserRegistration / username / ...
```

---

## 5. Model Validators (`@model_validator`)

### Cross-Field Validation Logic

> **🚦 The Airport Security Analogy**
>
> Field validators check individual items — one bag on a scanner at a time. But some rules require looking at *multiple things together*. An airline agent must verify that your departure date is before your return date, that your ticket matches your passport name, and that your seat class matches your ticket tier. These are *cross-field* rules. `@model_validator` operates at the whole-model level — seeing all fields simultaneously — just like the airline agent.

```python
# model_validators.py

from pydantic import BaseModel, Field, model_validator  # Import the model-level decorator
from typing import Optional
from datetime import date                                # Standard library date type

class BookingRequest(BaseModel):
    passenger_name: str = Field(min_length=2)            # Name of the passenger
    departure_date: date                                  # Trip start date
    return_date: Optional[date] = None                   # Trip end date (optional for one-way)
    is_round_trip: bool = False                          # Whether this is a round-trip booking
    seat_class: str = Field(default="economy")           # Seat class selection
    discount_code: Optional[str] = None                  # Optional promo code
    price: float = Field(gt=0)                           # Ticket price, must be > 0

    @model_validator(mode="after")                       # 'after' means all fields are validated first
    def validate_dates_and_trip_type(self) -> "BookingRequest":
        """
        Cross-field rule 1: If round trip, return_date must be provided.
        Cross-field rule 2: return_date must be after departure_date.
        """
        if self.is_round_trip and self.return_date is None:   # Check cross-field dependency
            raise ValueError(
                "return_date is required for round-trip bookings."
            )

        if self.return_date is not None:                      # Only compare if return_date exists
            if self.return_date <= self.departure_date:       # Return must be AFTER departure
                raise ValueError(
                    "return_date must be strictly after departure_date."
                )
        return self                                           # MUST return self in 'after' mode

    @model_validator(mode="after")                       # Second model-level validator
    def validate_business_class_price(self) -> "BookingRequest":
        """
        Cross-field rule: Business class bookings must cost at least $500.
        Prevents pricing logic errors where class and price are mismatched.
        """
        if self.seat_class == "business" and self.price < 500.0:  # Check class AND price
            raise ValueError(
                "Business class tickets must be priced at $500 or above."
            )
        return self                                           # Return self to allow chaining

# --- Valid one-way booking ---
one_way = BookingRequest(
    passenger_name="Carol White",
    departure_date=date(2025, 12, 1),
    is_round_trip=False,                                 # No return date needed
    seat_class="economy",
    price=299.00
)
print(one_way.passenger_name)  # Output: Carol White

# --- Valid round-trip booking ---
round_trip = BookingRequest(
    passenger_name="Dave Green",
    departure_date=date(2025, 12, 1),
    return_date=date(2025, 12, 15),                      # After departure — valid
    is_round_trip=True,
    seat_class="business",
    price=1200.00                                        # >= 500 for business — valid
)
print(round_trip.return_date)  # Output: 2025-12-15

# --- Invalid: round-trip without return_date ---
try:
    BookingRequest(
        passenger_name="Eve Black",
        departure_date=date(2025, 12, 1),
        is_round_trip=True,                              # Round trip but no return date!
        seat_class="economy",
        price=200.00
    )
except Exception as e:
    print(e)  # Output: return_date is required for round-trip bookings.

# --- Invalid: return_date before departure_date ---
try:
    BookingRequest(
        passenger_name="Frank Blue",
        departure_date=date(2025, 12, 15),
        return_date=date(2025, 12, 1),                   # Return is BEFORE departure!
        is_round_trip=True,
        seat_class="economy",
        price=200.00
    )
except Exception as e:
    print(e)  # Output: return_date must be strictly after departure_date.
```

---

## 6. Computed Fields (`@computed_field`)

### Derived Properties in Your Model Output

> **🧮 The Receipt Calculator Analogy**
>
> When you get a receipt at a restaurant, it shows the subtotal, tax, and tip separately — but also the *total*, which is derived from the others. The total isn't stored anywhere independently; it's always *computed* from the other values. Pydantic's `@computed_field` works the same way: it exposes a derived value as if it were a regular field, and it appears in all serialized output automatically.

```python
# computed_fields.py

from pydantic import BaseModel, Field, computed_field   # Import the computed_field decorator
from typing import List
from datetime import date, datetime                      # Date utilities
import math                                              # Standard math module

class OrderItem(BaseModel):
    product_name: str                                    # Name of the product
    unit_price: float = Field(gt=0)                     # Price per single unit
    quantity: int = Field(ge=1)                         # Number of units ordered

    @computed_field                                      # Mark this property as a computed field
    @property                                            # Must also be decorated as @property
    def line_total(self) -> float:                       # Return type hint is required
        """Calculates total price for this line item."""
        return round(self.unit_price * self.quantity, 2) # Multiply and round to 2 decimals


class ShoppingCart(BaseModel):
    cart_id: str                                         # Unique cart identifier
    customer_name: str                                   # Name of the customer
    items: List[OrderItem]                               # List of order items (nested models)
    discount_percent: float = Field(default=0.0, ge=0, le=100)  # Discount 0-100%
    tax_rate: float = Field(default=8.5, gt=0)          # Tax rate as a percentage

    @computed_field
    @property
    def subtotal(self) -> float:
        """Sums all line item totals before tax and discounts."""
        return round(sum(item.line_total for item in self.items), 2)  # Sum all computed line_totals

    @computed_field
    @property
    def discount_amount(self) -> float:
        """Calculates the monetary value of the discount."""
        return round(self.subtotal * (self.discount_percent / 100), 2)  # Percentage of subtotal

    @computed_field
    @property
    def taxable_amount(self) -> float:
        """Amount subject to tax, after discount is applied."""
        return round(self.subtotal - self.discount_amount, 2)  # Subtotal minus discount

    @computed_field
    @property
    def tax_amount(self) -> float:
        """Calculates the tax on the taxable amount."""
        return round(self.taxable_amount * (self.tax_rate / 100), 2)  # Tax rate applied

    @computed_field
    @property
    def grand_total(self) -> float:
        """Final amount due, including tax and after discounts."""
        return round(self.taxable_amount + self.tax_amount, 2)  # Taxable + tax

    @computed_field
    @property
    def item_count(self) -> int:
        """Total number of individual product units in the cart."""
        return sum(item.quantity for item in self.items)  # Sum all quantities

# --- Create an order to test computed fields ---
cart = ShoppingCart(
    cart_id="CART-001",
    customer_name="Grace Lee",
    items=[
        OrderItem(product_name="Keyboard", unit_price=79.99, quantity=1),   # line_total = 79.99
        OrderItem(product_name="Mouse",    unit_price=29.99, quantity=2),   # line_total = 59.98
        OrderItem(product_name="Monitor",  unit_price=349.99, quantity=1),  # line_total = 349.99
    ],
    discount_percent=10.0,                               # 10% discount
    tax_rate=8.5                                         # 8.5% tax
)

print(f"Item Count    : {cart.item_count}")       # Output: 4
print(f"Subtotal      : ${cart.subtotal}")        # Output: $489.96
print(f"Discount (10%): ${cart.discount_amount}") # Output: $48.996 -> $49.00
print(f"Tax Amount    : ${cart.tax_amount}")      # Computed from taxable amount
print(f"Grand Total   : ${cart.grand_total}")     # Final computed total

# Computed fields appear automatically in model_dump() output
output = cart.model_dump()
print("grand_total" in output)   # Output: True — computed fields are included by default
```

---

## 7. Serialization: `model_dump()` & `model_dump_json()`

### Exporting Your Validated Data

> **📦 The Shipping Manifest Analogy**
>
> After validating and processing goods in a warehouse, you need to *export* them. Depending on the destination, you might ship different subsets — some items are confidential and can't leave the building; others need to be in a different format for international customs. Pydantic's serialization tools (`model_dump`, `model_dump_json`) are your packing and manifest system, giving you precise control over what leaves and in what format.

```python
# serialization.py

from pydantic import BaseModel, Field, computed_field   # Core imports
from typing import Optional, Set
from datetime import datetime

class UserProfile(BaseModel):
    user_id: int                                          # Primary identifier
    username: str                                         # Public username
    email: str                                            # Email — may be sensitive
    hashed_password: str                                  # Never expose this!
    api_key: str                                          # Secret key — never expose
    bio: Optional[str] = None                            # Optional public bio
    created_at: datetime = Field(default_factory=datetime.now)  # Auto timestamp

    @computed_field
    @property
    def display_name(self) -> str:
        """A formatted display name derived from username."""
        return f"@{self.username}"                        # Prefix with '@'

# --- Create a sample user ---
user = UserProfile(
    user_id=1,
    username="techguru",
    email="guru@example.com",
    hashed_password="$2b$12$hashed...",
    api_key="sk-abc123xyz789",
    bio="Loves Python and coffee."
)

# =============================================================
# --- model_dump(): returns a Python dictionary ---
# =============================================================
print("--- Full model_dump() ---")
full_dict = user.model_dump()                            # Default: all fields included
print(full_dict)                                         # Includes hashed_password and api_key!

# --- EXCLUDE sensitive fields ---
print("\n--- Excluding sensitive fields ---")
safe_dict = user.model_dump(
    exclude={"hashed_password", "api_key"}               # Pass a set of field names to exclude
)
print(safe_dict)                                         # hashed_password and api_key are gone

# --- INCLUDE only specific fields ---
print("\n--- Including only specific fields ---")
public_dict = user.model_dump(
    include={"user_id", "username", "bio", "display_name"}  # Only these fields will appear
)
print(public_dict)                                       # Only the specified fields in output

# --- Exclude fields with None values ---
print("\n--- Excluding None values ---")
user_no_bio = UserProfile(
    user_id=2, username="anon", email="a@b.com",
    hashed_password="hash", api_key="key"                # No bio provided
)
no_none_dict = user_no_bio.model_dump(exclude_none=True) # Fields with None are omitted
print(no_none_dict)                                      # 'bio' will not appear

# --- Exclude default values ---
print("\n--- Excluding defaults ---")
default_excluded = user.model_dump(exclude_defaults=True) # Fields at their default value are omitted

# =============================================================
# --- model_dump_json(): returns a JSON string ---
# =============================================================
print("\n--- model_dump_json() ---")
json_str = user.model_dump_json(                          # Outputs a JSON-encoded string
    exclude={"hashed_password", "api_key"},               # Same filtering options apply
    indent=2                                              # Pretty-print with indentation
)
print(json_str)                                           # Output: formatted JSON string
print(type(json_str))                                     # Output: <class 'str'>

# --- Key difference: model_dump vs model_dump_json ---
# model_dump()      -> returns dict  (Python object, good for further manipulation)
# model_dump_json() -> returns str   (JSON string, good for APIs, files, logging)

# =============================================================
# --- Controlling datetime serialization ---
# =============================================================
class Event(BaseModel):
    name: str
    scheduled_at: datetime

event = Event(name="Launch", scheduled_at=datetime(2025, 6, 15, 9, 0, 0))

# Serialize datetime as ISO string (default)
print(event.model_dump())                                # datetime object in dict
print(event.model_dump_json())                           # ISO 8601 string in JSON
```

---

## 8. FastAPI Integration

### Why Pydantic and FastAPI Are Inseparable

FastAPI is built *on top of* Pydantic. Every request body, response model, query parameter, and path parameter validation in FastAPI is powered by Pydantic under the hood. They share the same design philosophy: **declare your types, and let the framework handle the rest**.

> **🛫 The Airline Counter Analogy**
>
> FastAPI is the airline check-in system. When a passenger (HTTP request) arrives:
> - The **Pydantic request model** is the check-in form — it validates what the passenger brought (request body).
> - The **Pydantic response model** is the boarding pass template — it controls exactly what information is printed on it (what's returned to the client).
> - The **Swagger UI** is the airport information board — automatically updated with every flight (endpoint) and its requirements.
> - If a passenger brings invalid baggage (malformed data), they get a **422 error** — a clear explanation of exactly what was wrong — before they ever reach the gate (business logic).

### Key Benefits of Pydantic in FastAPI

| Benefit | What It Means in Practice |
|---|---|
| **Automatic Request Parsing** | FastAPI reads JSON body → creates a Pydantic model automatically |
| **Input Validation** | Invalid data raises a 422 Unprocessable Entity with detailed errors |
| **Response Filtering** | `response_model` strips sensitive or extra fields before sending |
| **OpenAPI / Swagger Docs** | Pydantic model fields, types, and constraints auto-populate `/docs` |
| **Editor Support** | Full type-safety and autocompletion throughout your route handlers |

### Installation

```bash
# Install FastAPI and Uvicorn (the ASGI server to run it)
pip install fastapi uvicorn
```

### FastAPI Code Example: A Complete User API

```python
# fastapi_integration.py
# Run with: uvicorn fastapi_integration:app --reload

from fastapi import FastAPI, HTTPException          # FastAPI framework and HTTP exceptions
from pydantic import BaseModel, Field, EmailStr     # Pydantic — EmailStr requires 'pip install pydantic[email]'
from typing import Optional, List
from datetime import datetime

# --- Initialize the FastAPI application ---
app = FastAPI(
    title="User Management API",                    # Shown in Swagger UI title bar
    description="A demo API powered by Pydantic V2",# Shown in Swagger UI description
    version="1.0.0"                                  # API version shown in docs
)

# =============================================================
# --- REQUEST MODELS (define what clients SEND to the API) ---
# =============================================================

class CreateUserRequest(BaseModel):
    """Defines the expected body of a POST /users request."""
    username: str = Field(
        ...,                                         # Required field
        min_length=3,
        max_length=30,
        description="Unique username, 3-30 characters"
    )
    email: str = Field(                              # Use EmailStr for real projects
        ...,
        description="Valid email address"
    )
    password: str = Field(
        ...,
        min_length=8,
        description="Password, minimum 8 characters"
    )
    age: Optional[int] = Field(
        default=None,
        ge=13,                                       # Must be at least 13 (COPPA compliance)
        description="User age, must be 13 or older"
    )

# =============================================================
# --- RESPONSE MODELS (define what the API RETURNS to clients) ---
# =============================================================

class UserResponse(BaseModel):
    """
    Defines the shape of the API's response.
    CRITICAL: Does NOT include 'password' — it's stripped automatically.
    This is response filtering: clients never see sensitive data.
    """
    user_id: int                                     # Auto-generated ID from "database"
    username: str                                    # Public username
    email: str                                       # Email (returned to user)
    age: Optional[int] = None                        # Optional age
    created_at: datetime                             # Timestamp of creation
    is_active: bool = True                           # Account status

class UserListResponse(BaseModel):
    """Wraps a list of users with pagination metadata."""
    total: int                                       # Total number of users
    users: List[UserResponse]                        # List of user objects

# =============================================================
# --- SIMULATED DATABASE (in-memory dict for demo purposes) ---
# =============================================================

fake_db: dict[int, dict] = {}                        # Simple dict as a mock database
user_counter: int = 0                                # Auto-increment user ID

# =============================================================
# --- ROUTES (API Endpoints) ---
# =============================================================

@app.post(
    "/users",                                        # HTTP POST to /users
    response_model=UserResponse,                     # Pydantic will filter response to this shape
    status_code=201,                                 # 201 Created is the correct status for POST
    summary="Create a new user",                     # Appears in Swagger UI
    tags=["Users"]                                   # Groups endpoint in Swagger UI
)
async def create_user(request: CreateUserRequest):   # FastAPI injects + validates the body
    """
    Creates a new user account.

    - **username**: Must be 3-30 characters
    - **email**: Must be a valid email format
    - **password**: Minimum 8 characters (NEVER returned in response)
    - **age**: Optional, must be 13 or older if provided

    FastAPI + Pydantic automatically:
    1. Parses the JSON request body
    2. Validates all fields against CreateUserRequest rules
    3. Returns 422 if any field is invalid (before this function even runs!)
    4. Filters the response through UserResponse (password is excluded)
    5. Generates Swagger docs from the model schema
    """
    global user_counter                              # Access the module-level counter
    user_counter += 1                                # Increment for a new unique ID

    # Simulate storing the user — include password for the "database" but NOT response
    new_user = {
        "user_id": user_counter,                     # Assign the new ID
        "username": request.username,                # From validated request model
        "email": request.email,                      # From validated request model
        "password": request.password,                # Stored in DB (would be hashed in real app)
        "age": request.age,                          # From validated request model
        "created_at": datetime.now(),                # Server-generated timestamp
        "is_active": True                            # New users are active by default
    }
    fake_db[user_counter] = new_user                 # Save to our mock database

    # FastAPI + response_model=UserResponse will AUTOMATICALLY:
    # 1. Create a UserResponse from this dict
    # 2. Strip 'password' because it's not in UserResponse
    # 3. Serialize to JSON and send to client
    return new_user                                  # Return the full dict; FastAPI filters it

@app.get(
    "/users/{user_id}",                              # Path parameter {user_id}
    response_model=UserResponse,                     # Response filtered through UserResponse
    summary="Get a user by ID",
    tags=["Users"]
)
async def get_user(user_id: int):                    # FastAPI validates user_id is an int
    """Retrieves a single user by their numeric ID."""
    if user_id not in fake_db:                       # Check existence in mock DB
        raise HTTPException(                         # Raise a 404 if not found
            status_code=404,
            detail=f"User with id={user_id} not found."
        )
    return fake_db[user_id]                          # Return user; password stripped by response_model

@app.get(
    "/users",                                        # GET all users
    response_model=UserListResponse,                 # Returns wrapped list
    summary="List all users",
    tags=["Users"]
)
async def list_users():
    """Returns all users in the system."""
    all_users = list(fake_db.values())               # Get all user records from mock DB
    return UserListResponse(                         # Explicitly construct the response model
        total=len(all_users),                        # Set the total count
        users=all_users                              # FastAPI will validate each user via UserResponse
    )

# =============================================================
# --- HOW TO TEST ---
# =============================================================
# 1. Start: uvicorn fastapi_integration:app --reload
# 2. Open browser: http://127.0.0.1:8000/docs
#    → Swagger UI auto-generated from Pydantic models
#    → All fields, types, constraints, and descriptions visible
#    → Try POSTing a user with an invalid email — you'll get a 422
# 3. Try the API with curl:
#
#    curl -X POST "http://127.0.0.1:8000/users" \
#         -H "Content-Type: application/json" \
#         -d '{"username": "testuser", "email": "test@example.com", "password": "pass1234"}'
#
# The response will NOT contain "password" — filtered by response_model!
```

### What Happens with Invalid Data: The 422 Error

When a client sends a malformed request, FastAPI returns a **422 Unprocessable Entity** response automatically. No manual error handling code needed:

```json
// Client sends: POST /users with { "username": "x", "email": "bad", "password": "123" }
// FastAPI + Pydantic automatically returns:
{
  "detail": [
    {
      "type": "string_too_short",
      "loc": ["body", "username"],
      "msg": "String should have at least 3 characters",
      "input": "x",
      "ctx": { "min_length": 3 }
    },
    {
      "type": "string_too_short",
      "loc": ["body", "password"],
      "msg": "String should have at least 8 characters",
      "input": "123",
      "ctx": { "min_length": 8 }
    }
  ]
}
```

> **Every validation error is pinpointed**: the field location (`loc`), the error type, the human-readable message, and the invalid input are all included — automatically, without writing a single line of error-handling code.

---

## 9. Pydantic vs Python Dataclasses

Python's built-in `dataclasses` module offers similar syntactic convenience. Here is a structured comparison to help you choose:

| Feature | Pydantic `BaseModel` | Python `@dataclass` |
|---|---|---|
| **Runtime Validation** | ✅ Yes — validates on every instantiation | ❌ No — only type annotations, no enforcement |
| **Type Coercion** | ✅ Yes — `"42"` → `42` for `int` fields | ❌ No — stores whatever value is passed |
| **Custom Validators** | ✅ `@field_validator`, `@model_validator` | ⚠️ Requires manual `__post_init__` logic |
| **Serialization** | ✅ `model_dump()`, `model_dump_json()` built-in | ❌ Requires `dataclasses.asdict()` or manual code |
| **JSON Schema** | ✅ `Model.model_json_schema()` built-in | ❌ Not available natively |
| **FastAPI Integration** | ✅ First-class, powers all FastAPI features | ⚠️ Partial support — no automatic validation |
| **Field Constraints** | ✅ `Field(gt=0, max_length=50)` | ❌ Not available — constraints must be coded manually |
| **Computed Fields** | ✅ `@computed_field` | ⚠️ Use `@property` (not included in `asdict()`) |
| **Nested Model Parsing** | ✅ Auto-parses nested dicts into models | ❌ Must create nested instances manually |
| **Performance** | ✅ V2 core written in Rust — extremely fast | ✅ Very fast — minimal overhead |
| **Error Messages** | ✅ Detailed, structured `ValidationError` | ❌ Standard Python `TypeError`/`ValueError` |
| **IDE Support** | ✅ Excellent — full type inference | ✅ Excellent |
| **`__slots__`** | ⚠️ Possible but complex | ✅ Native support with `@dataclass(slots=True)` |
| **Mutability** | Configurable (`model_config = ConfigDict(frozen=True)`) | Configurable (`@dataclass(frozen=True)`) |
| **Primary Use Case** | Data validation, APIs, config management | Simple data containers, internal data structures |

### When to Use Each

**Choose Pydantic `BaseModel` when:**
- Building APIs or handling external data (HTTP, JSON, CSV, env vars)
- You need strict data validation and rich error reporting
- You're using FastAPI or any framework that integrates with Pydantic
- Configuration management (e.g., `pydantic-settings`)

**Choose Python `@dataclass` when:**
- You need a simple, lightweight data container for *internal* use
- Performance is critical and validation overhead matters
- You're working on a codebase with zero external dependencies
- The data is already trusted and does not need validation

---

## Quick Reference: Pydantic V2 Cheatsheet

```python
from pydantic import (
    BaseModel,           # Base class for all models
    Field,               # Add constraints, defaults, aliases
    field_validator,     # Per-field custom validation
    model_validator,     # Cross-field validation
    computed_field,      # Derived properties in output
    ConfigDict,          # Model-level configuration
)

class MyModel(BaseModel):
    model_config = ConfigDict(
        str_strip_whitespace=True,   # Auto-strip whitespace from strings
        frozen=True,                  # Make model immutable (hashable)
        populate_by_name=True,        # Allow both alias and field name
    )

    # Serialization
    data = MyModel(...)
    data.model_dump()                 # -> dict
    data.model_dump_json()            # -> str (JSON)
    data.model_dump(exclude={"secret_field"})    # Exclude fields
    data.model_dump(include={"id", "name"})      # Include only these
    data.model_dump(exclude_none=True)           # Skip None fields

    # Schema
    MyModel.model_json_schema()       # -> OpenAPI-compatible JSON schema dict

    # Parsing
    MyModel.model_validate(dict_data)          # From dict
    MyModel.model_validate_json(json_string)   # From JSON string
```

---

*Built with ❤️ using Pydantic V2 and Python 3.10+.*

*For the latest documentation, visit [docs.pydantic.dev](https://docs.pydantic.dev).*

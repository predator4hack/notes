---
title: "Pydantic BaseModel"
type: concept
tags:
  - python
  - pydantic
  - validation
created: 2025-12-14
updated: 2026-03-15
sources: []
aliases: []
---


In this context, inheriting from `pydantic.BaseModel` is the **core mechanism** for defining a structured, validated data schema.

Here is a detailed explanation of why and what this inheritance provides:

---

### 1. The Purpose of Pydantic and `BaseModel`

Pydantic's main goal is to use Python's **type hinting** system to enforce data structure and correctness at runtime.

When a Python class inherits from `BaseModel`, it gains powerful, built-in features that transform a simple class into a robust data model.

### 2. Key Benefits of Inheriting from `BaseModel`

#### A. Automated Data Validation

This is the primary reason. When you instantiate a model, `BaseModel` automatically checks the provided data against the type hints you specified and performs any necessary type coercion.

**Example:**

```python
from pydantic import BaseModel
from typing import List

class User(BaseModel):
    # Enforces that 'id' must be an integer
    id: int 
    # Enforces that 'name' must be a string
    name: str 
    # Enforces that 'tags' must be a list of strings
    tags: List[str] 
    
# Success: Data matches the schema
user_data_1 = {"id": 123, "name": "Alice", "tags": ["admin", "dev"]}
user_model_1 = User(**user_data_1) 
print(f"Model 1 ID: {user_model_1.id} (Type: {type(user_model_1.id)})")

# Failure: Will raise a pydantic.ValidationError 
# because "123x" cannot be converted to a pure integer
try:
    User(id="123x", name="Bob", tags=[])
except Exception as e:
    print(f"\nValidation Error: {e}") 
```

#### B. Automatic Type Coercion

If the input data is _close_ to the expected type, Pydantic tries to intelligently convert it.

**Example:**

```python
class Item(BaseModel):
    price: float

# The input '10' is an integer, but Pydantic converts it to a float (10.0)
item = Item(price=10) 
print(f"Item Price: {item.price} (Type: {type(item.price)})")
# Output: Item Price: 10.0 (Type: <class 'float'>)
```

#### C. Serialization and Deserialization

`BaseModel` provides methods to easily convert the Python object into common data formats, which is essential for working with APIs and databases.

- **`.dict()` / `.model_dump()`:** Converts the model instance into a Python dictionary.
    
- **`.json()` / `.model_dump_json()`:** Converts the model instance into a JSON string.
    
- **`.parse_obj()` / `.model_validate()`:** Creates a model instance from a Python dictionary or JSON string.
    

**Example:**

```python
class Config(BaseModel):
    timeout: int
    debug_mode: bool

config = Config(timeout=5, debug_mode=True)

# Serialization to JSON
json_output = config.model_dump_json(indent=2)
print("\nJSON Output:")
print(json_output)
# Output:
# {
#   "timeout": 5,
#   "debug_mode": true
# }
```

#### D. Immutability Control

By default, models are mutable, but you can easily make them immutable (read-only) using a class configuration (`model_config = ConfigDict(frozen=True)` in Pydantic v2).

#### E. IDE and Static Analysis Support

Because Pydantic leverages standard Python type hints, tools like mypy, PyCharm, and VS Code can read the models and provide excellent **autocompletion and error checking** _before_ the code even runs. This makes models easier to use and maintain.

---

### 3. Usage in Frameworks

The reason you see `BaseModel` frequently is because it is the standard for data handling in many modern Python frameworks:

- **FastAPI:** Uses Pydantic models to define the structure of request bodies (JSON payload received) and response bodies (JSON payload returned). This is what enables FastAPI to automatically validate requests and generate interactive API documentation (Swagger/OpenAPI).
    
- **SQLModel:** Built on top of Pydantic, it uses `BaseModel` to define database tables, offering both the database schema definition and runtime validation.
    
- **Data Science:** Used to structure and validate configuration files or input data for machine learning models.
    

In short, inheriting from `BaseModel` is the contractual agreement you make with Pydantic: "Here is my data schema, please automatically apply your powerful validation, coercion, and serialization features."


## Decorators

A **Python decorator** is a design pattern to modify a function/class's behavior without changing its source code.

*   **Analogy:** Gift wrapping for a function.

### Prerequisites: Functions as First-Class Objects

In Python, functions are objects, meaning they can be:
*   Assigned to variables.
*   Passed as arguments to other functions.
*   Returned from other functions.

**Example:**
```python
def shout(text): return text.upper()
def greet(func): print(func("Hello"))
greet(shout) # HELLO
```

### Building a Decorator (Manual Way)

A decorator is a function that:
1.  Takes another function (`func`) as an argument.
2.  Defines an inner `wrapper` function.
3.  Adds functionality *before* and/or *after* calling `func` inside `wrapper`.
4.  Returns the `wrapper` function.

**Manual Example:**
```python
def my_decorator(func):
    def wrapper():
        print("Before")
        func()
        print("After")
    return wrapper

def say_whee(): print("Whee!")
say_whee = my_decorator(say_whee) # Manual wrapping
say_whee()
```

### The "Pie Syntax" (`@`)

Syntactic sugar for applying decorators:
```python
@my_decorator # Equivalent to say_hello = my_decorator(say_hello)
def say_hello():
    print("Hello!")
say_hello()
```

### Real-World Examples

*   **Timer Decorator:** Measures function execution time.
    *   Uses `*args`, `**kwargs` in `wrapper` to handle any function signature.
    ```python
    import time
    def timer_decorator(func):
        def wrapper(*args, **kwargs):
            start = time.time()
            result = func(*args, **kwargs)
            end = time.time()
            print(f"'{func.__name__}' took {end - start:.4f}s")
            return result
        return wrapper
    @timer_decorator
    def slow_func(): time.sleep(1)
    slow_func()
    ```
*   **Authorization:** Checks user permissions before executing a function (e.g., in web frameworks).
    ```python
    current_user = {"is_admin": False}
    def admin_required(func):
        def wrapper(*args, **kwargs):
            if not current_user.get("is_admin"):
                print("⛔ Access Denied")
                return
            return func(*args, **kwargs)
        return wrapper
    @admin_required
    def delete_database(): print("💥 Database deleted!")
    delete_database() # ⛔ Access Denied
    ```

### Best Practice: `functools.wraps`

*   When a function is wrapped, its metadata (`__name__`, `__doc__`) is lost and replaced by the `wrapper`'s metadata.
*   `@functools.wraps(func)` is a decorator for the `wrapper` function that preserves the original function's metadata.

**Example:**
```python
from functools import wraps
def my_decorator(func):
    @wraps(func) # Preserves original func's metadata
    def wrapper():
        """I am the wrapper"""
        func()
    return wrapper

@my_decorator
def identity():
    """I am the identity function"""
    pass

print(identity.__name__) # identity (Correct!)
print(identity.__doc__)  # I am the identity function (Correct!)
```

### Decorator Summary

| Concept | Description |
| :------------------ | :------------------------------------------------ |
| **First-class Objects** | Functions can be passed around like variables. |
| **Wrapper Function** | Inner function inside a decorator with extra logic. |
| **@ Syntax** | Shortcut to apply a decorator. |
| **`*args` / `**kwargs`** | Used in wrapper to accept any arguments. |
| **`@wraps`** | Helper to preserve original function's metadata. |

---

## `@classmethod`

The `@classmethod` decorator defines a method that operates on the **class itself**, not a specific instance.

*   Receives the class (`cls`) as its first implicit argument.

### Syntax

```python
class MyClass:
    class_variable = "Shared"

    @classmethod
    def print_class_variable(cls):
        print(cls.class_variable)

MyClass.print_class_variable() # Output: Shared
```

### Primary Use Case: Alternative Constructors

*   Allows creating objects from different input formats than `__init__`.
*   Uses `return cls(...)` to ensure correct instantiation, especially with inheritance.

**Example: `Student.from_string`**
```python
class Student:
    def __init__(self, first, last):
        self.first_name = first
        self.last_name = last

    @classmethod
    def from_string(cls, name_string):
        first, last = name_string.split("-")
        return cls(first, last) # Returns instance of Student (or subclass)

student2 = Student.from_string("Jane-Smith")
print(student2.first_name) # Jane
```

### Comparison: Instance vs. Class vs. Static Methods

| Feature | Instance Method | Class Method (@classmethod) | Static Method (@staticmethod) |
| :------------------ | :------------------------------------------------ | :-------------------------------------- | :------------------------------------------ |
| **First Argument** | `self` (object instance) | `cls` (the class itself) | None |
| **Access** | Instance data (`self.x`) & class data. | ONLY class data (`cls.x`). | Cannot access instance OR class data. |
| **Use Case** | Modifying object's unique state. | Factory methods, modifying class state. | Utility functions unrelated to class/instance. |

### Advanced: Modifying Class State

*   `@classmethod` can access and modify class variables, affecting all instances.

**Example: Tracking Instance Count**
```python
class Server:
    active_connections = 0

    def __init__(self, name):
        self.name = name
        Server.update_connections(1) # Calls class method

    @classmethod
    def update_connections(cls, count):
        cls.active_connections += count

    @classmethod
    def get_active_connections(cls):
        return f"Total Active: {cls.active_connections}"

s1 = Server("Google")
s2 = Server("Amazon")
print(Server.get_active_connections()) # Output: Total Active: 2
```

### `@classmethod` Summary

Use `@classmethod` when:
1.  You need **Alternative Constructors** (creating objects from various input formats).
2.  You need to access or modify **Class Variables** (shared across all instances).
3.  You want to support correct **Inheritance** (using `cls` ensures the correct subclass is instantiated).

## Related

- [[Python/Python Methods|Python Methods]]

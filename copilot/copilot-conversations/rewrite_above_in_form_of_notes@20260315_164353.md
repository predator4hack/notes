---
epoch: 1773573233655
modelKey: "gemini-2.5-flash|google"
topic: "CPython Bytecode and Code Objects"



tags:
  - copilot-conversation
---

**user**: rewrite above in form of notes
[Context: Notes: Python/Python Decorators & Class Methods.md]
[Timestamp: 2026/03/15 16:43:53]

**ai**: Here's a rewrite of your note on Python Decorators and Class Methods in a more concise, note-like format:

---

# Python Decorators & Class Methods

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
[Timestamp: 2026/03/15 16:44:02]
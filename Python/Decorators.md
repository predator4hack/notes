
At a high level, a **Python decorator** is a design pattern that allows you to modify the behavior of a function or class without permanently changing its source code.

Think of it like **gift wrapping**:

- The **gift** is your original function (it does the main job).
    
- The **wrapping paper** is the decorator (it adds style or extra features).
    
- You still get the gift inside, but it now looks different or comes with extra things attached.
    

---

### 1. The Prerequisites: Functions as Objects

To understand decorators, you must first understand that in Python, functions are **first-class objects**. This means:

- You can assign functions to variables.
    
- You can pass functions as arguments to other functions.
    
- You can return functions from other functions.
    

**Example:**

```python
def shout(text):
    return text.upper()

def whisper(text):
    return text.lower()

# Passing a function as an argument
def greet(func):
    # We are calling the function passed in as 'func'
    greeting = func("Hello World") 
    print(greeting)

greet(shout)   # Output: HELLO WORLD
greet(whisper) # Output: hello world
```

---

### 2. Building a Decorator (The Manual Way)

A decorator is simply a function that takes another function as an argument, adds some functionality, and returns a _new_ function.

Here is how we would do it without any special syntax:

```python
def my_decorator(func):
    def wrapper():
        print("Something is happening before the function is called.")
        func()
        print("Something is happening after the function is called.")
    return wrapper

def say_whee():
    print("Whee!")

# Manually wrapping the function
say_whee = my_decorator(say_whee)

say_whee()
```

**Output:**

```
Something is happening before the function is called.
Whee!
Something is happening after the function is called.
```

---

### 3. The "Pie Syntax" (@)

Python provides a shortcut (syntactic sugar) for the manual process above using the `@` symbol. This is placed immediately before the function definition.

```python
def my_decorator(func):
    def wrapper():
        print("--- Start ---")
        func()
        print("--- End ---")
    return wrapper

@my_decorator
def say_hello():
    print("Hello!")

# calling say_hello automatically runs the wrapper
say_hello()
```

---

### 4. Real-World Examples

#### Example A: A Timer Decorator

This is a very common use case: measuring how long a function takes to run.

```python
import time

def timer_decorator(func):
    def wrapper(*args, **kwargs):
        start_time = time.time()
        result = func(*args, **kwargs)
        end_time = time.time()
        print(f"Function '{func.__name__}' took {end_time - start_time:.4f} seconds to run.")
        return result
    return wrapper

@timer_decorator
def compute_square_root(n):
    # Simulating a slow calculation
    time.sleep(1) 
    return n ** 0.5

compute_square_root(16)
```

**Key Note:** Notice the use of `*args` and `**kwargs` in the wrapper. This ensures that your decorator works regardless of how many arguments the original function accepts.

#### Example B: Authorization (Access Control)

Decorators are frequently used in web frameworks (like Flask or Django) to check if a user is logged in.

```python
current_user = {"is_admin": False}

def admin_required(func):
    def wrapper(*args, **kwargs):
        if not current_user.get("is_admin"):
            print("⛔ Access Denied: Admins only.")
            return
        return func(*args, **kwargs)
    return wrapper

@admin_required
def delete_database():
    print("💥 Database deleted!")

delete_database() 
# Output: ⛔ Access Denied: Admins only.
```

---

### 5. Best Practice: `functools.wraps`

When you wrap a function, the new function (the `wrapper`) replaces the original. This means you lose the original function's metadata (like its name and docstring).

To fix this, Python provides a helper decorator called `@wraps`.

**Without `@wraps`:**
```python
def my_decorator(func):
    def wrapper():
        """I am the wrapper"""
        func()
    return wrapper

@my_decorator
def identity():
    """I am the identity function"""
    pass

print(identity.__name__) # Output: wrapper (Confusing!)
print(identity.__doc__)  # Output: I am the wrapper (Wrong!)
```

**With `@wraps`:**
```python
from functools import wraps

def my_decorator(func):
    @wraps(func) # This preserves the metadata of the original function
    def wrapper():
        func()
    return wrapper

@my_decorator
def identity():
    """I am the identity function"""
    pass

print(identity.__name__) # Output: identity (Correct!)
print(identity.__doc__)  # Output: I am the identity function (Correct!)
```

---

### Summary Table

|**Concept**|**Description**|
|---|---|
|**First-class Objects**|Functions can be passed around like variables.|
|**Wrapper Function**|The inner function inside a decorator that performs the extra logic.|
|**@ Syntax**|The shortcut to apply a decorator to a function.|
|**`*args` / `**kwargs`**|Used in the wrapper to accept any arguments the original function needs.|
|**`@wraps`**|Helper to preserve the original function's name and documentation.|


## @classmethod

The `@classmethod` decorator allows you to define a method that operates on the **class itself**, rather than on a specific instance of the class.

In a standard method, the first argument is `self` (the specific object). In a class method, the first argument is `cls` (the class template).

Here is a breakdown of why and how to use it.

---

### 1. The Syntax

A class method receives the class as an implicit first argument, just like an instance method receives the instance. By convention, we name this argument `cls`.

```python
class MyClass:
    class_variable = "I am shared by all instances"

    @classmethod
    def print_class_variable(cls):
        # We use 'cls' instead of 'self'
        print(cls.class_variable)

# You can call it without creating an object first
MyClass.print_class_variable() 
# Output: I am shared by all instances
```

---

### 2. Primary Use Case: Alternative Constructors

This is the most powerful and common use of `@classmethod`.

Sometimes you want to create an object, but your data isn't in the format the `__init__` method expects. Instead of writing complex parsing logic outside the class, you can write a class method to handle the preprocessing and return a new instance.

Example: Creating a Student from a String

Imagine you typically create a student with a first and last name.

```python
class Student:
    def __init__(self, first_name, last_name):
        self.first_name = first_name
        self.last_name = last_name

    def introduce(self):
        print(f"Hi, I am {self.first_name} {self.last_name}")

    # --- The Class Method ---
    @classmethod
    def from_string(cls, name_string):
        # 1. Parse the string
        first, last = name_string.split("-")
        # 2. Return a new instance of the class (cls)
        return cls(first, last)

# Standard way
student1 = Student("John", "Doe")

# Using the class method (Alternative Constructor)
student2 = Student.from_string("Jane-Smith")

student1.introduce() # Output: Hi, I am John Doe
student2.introduce() # Output: Hi, I am Jane Smith
```

> **Note:** We use `return cls(...)` instead of `return Student(...)`. This ensures that if someone inherits from `Student`, the method will create an instance of the _subclass_, not the parent class.

---

### 3. Comparison: Instance vs. Class vs. Static Methods

To fully grasp `@classmethod`, it helps to see how it differs from the other two types.

| **Feature**        | **Instance Method**                                 | **Class Method (@classmethod)**         | **Static Method (@staticmethod)**           |
| ------------------ | --------------------------------------------------- | --------------------------------------- | ------------------------------------------- |
| **First Argument** | `self` (the object instance)                        | `cls` (the class itself)                | None (no implicit argument)                 |
| **Access**         | Can access instance data (`self.x`) AND class data. | Can ONLY access class data (`cls.x`).   | Cannot access instance OR class data.       |
| **Use Case**       | Modifying an object's unique state.                 | Factory methods, modifying class state. | Utility functions unrelated to class state. |
|                    |                                                     |                                         |                                             |

---

### 4. Advanced: Modifying Class State

Since `@classmethod` has access to `cls`, it can modify variables that are shared across _all_ instances of that class.

**Example: Tracking Instance Count**

```python
class Server:
    active_connections = 0

    def __init__(self, name):
        self.name = name
        # When a new instance is made, update the class variable
        Server.update_connections(1)

    @classmethod
    def update_connections(cls, count):
        cls.active_connections += count

    @classmethod
    def get_active_connections(cls):
        return f"Total Active: {cls.active_connections}"

s1 = Server("Google")
s2 = Server("Amazon")

print(Server.get_active_connections()) 
# Output: Total Active: 2
```

### Summary

Use `@classmethod` when:

1. You need **Alternative Constructors** (creating objects from different input formats).
    
2. You need to access or modify **Class Variables** rather than instance variables.
    
3. You want to support correct **Inheritance** (using `cls` ensures the correct class is instantiated in subclasses).
    
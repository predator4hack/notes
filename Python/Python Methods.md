
## getattr()

The Python built-in function **`getattr()`** is primarily used to **retrieve the value of a named attribute from an object**.

It's a powerful tool often used when you need to access an attribute by its name, where the name is stored as a **string** or determined dynamically at runtime.

---

## 🔧 Syntax and Parameters

The general syntax for the `getattr()` function is:
```python
getattr(object, name[, default])
```

Here's a breakdown of the parameters:
- **`object`**: The object whose attribute you want to retrieve.
- **`name`**: A **string** containing the name of the attribute you want to access.
- **`default` (optional)**: If the named attribute is **not found**, this value is returned instead of raising an `AttributeError`. If the `default` argument is omitted and the attribute is not found, an `AttributeError` is raised.
---

## 💡 Key Use Cases

### 1. Dynamic Attribute Access

This is the most common use case. When you need to access different attributes of an object based on a variable or input (e.g., from a configuration file, user input, or an API request).

```python
class Car:
    def __init__(self, color, brand):
        self.color = color
        self.brand = brand
        
    def start(self):
        return "Engine started."

my_car = Car("red", "Tesla")

# The attribute name is dynamic (stored in a string)
attribute_name = "brand"
print(getattr(my_car, attribute_name)) # Output: Tesla

method_name = "start"
# You can also retrieve and call methods
method = getattr(my_car, method_name)
print(method()) # Output: Engine started.
```

### 2. Providing a Default Value

The optional `default` argument allows you to safely attempt to access an attribute without worrying about the program crashing if the attribute doesn't exist. This is a cleaner alternative to using a `try-except` block for handling `AttributeError`.

```python
class User:
    def __init__(self, username):
        self.username = username
        # Note: 'email' is not always set

user1 = User("alice")
user2 = User("bob")
user2.email = "bob@example.com"

# Safely check for an 'email' attribute
print(getattr(user1, "email", "Email not set")) # Output: Email not set
print(getattr(user2, "email", "Email not set")) # Output: bob@example.com

# If 'default' is omitted and the attribute doesn't exist:
# print(getattr(user1, "phone")) # Raises AttributeError
```

### 3. Reflecting on Objects (Introspection)

In advanced programming, `getattr()` is part of a set of introspection tools (`hasattr()`, `setattr()`, `delattr()`) that allow a program to **inspect and modify its own structure** at runtime.



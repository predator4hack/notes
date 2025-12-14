
When we say **“functions are first-class objects in Python”**, we mean that **functions are treated just like any other object (e.g., int, string, list)**.

Concretely, this implies **four key properties** 👇

---

## 1. Functions can be assigned to variables

A function doesn’t need to be called immediately. You can store it in a variable.

```python
def greet(name):
    return f"Hello, {name}"

f = greet      # no ()
print(f("Chandan"))
```

Here:

- `greet` is an object
    
- `f` now references the same function object
    

---

## 2. Functions can be passed as arguments to other functions

You can pass functions just like data.

```python
def apply(func, value):
    return func(value)

def square(x):
    return x * x

print(apply(square, 5))
```

This is the **foundation of callbacks, hooks, and functional programming patterns**.

---

## 3. Functions can be returned from other functions

A function can **create and return another function**.

```python
def multiplier(n):
    def inner(x):
        return x * n
    return inner

double = multiplier(2)
print(double(10))
```

This enables:

- Closures
    
- Function factories
    
- Decorators
    

---

## 4. Functions can be stored in data structures

Functions can live inside lists, dicts, etc.

```python
def add(x, y): return x + y
def sub(x, y): return x - y

ops = {
    "add": add,
    "sub": sub
}

print(ops["add"](3, 4))
```

---

## Why this matters (real-world impact)

Because functions are first-class:

### 🔹 Decorators work

```python
@timer
def train_model():
    ...
```

### 🔹 Frameworks like FastAPI / Flask work

```python
@app.get("/predict")
def predict():
    ...
```

### 🔹 ML pipelines & callbacks work

```python
model.fit(
    data,
    callbacks=[early_stopping, checkpoint]
)
```

---

## What it does **NOT** mean

❌ It does **not** mean:

- Functions are executed automatically
    
- Functions are special keywords
    

It simply means:

> **Functions are objects with the same rights as any other object**

---

## One-line mental model

> In Python, **a function is just a value that happens to be callable**.

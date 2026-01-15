
## Pytorch Unsqueeze 
### 📘 Overview

`torch.unsqueeze()` is a **tensor operation** used to **add a new
dimension (axis)** of size `1` at a specified position.

It is especially useful when you need to **reshape tensors** to make
them compatible for operations like: - Broadcasting\
- Concatenation\
- Feeding inputs into neural networks that expect specific tensor shapes

------------------------------------------------------------------------

## 🧩 Syntax

``` python
torch.unsqueeze(input, dim)
```

### Parameters:

-   **`input`** *(Tensor)* --- The input tensor.
-   **`dim`** *(int)* --- The index (position) at which to insert the
    new dimension.

### Returns:

-   A **new tensor** with one more dimension of size `1` inserted at the
    specified position.

------------------------------------------------------------------------

## 🧮 Example

``` python
import torch

# Create a 1D tensor
x = torch.tensor([10, 20, 30])
print(x.shape)
# Output: torch.Size([3])

# Add a new dimension at position 0
x1 = torch.unsqueeze(x, 0)
print(x1.shape)
# Output: torch.Size([1, 3])

# Add a new dimension at position 1
x2 = torch.unsqueeze(x, 1)
print(x2.shape)
# Output: torch.Size([3, 1])
```

------------------------------------------------------------------------

## 🧠 Visual Explanation

  Original Tensor   Shape   Operation        Resulting Shape
  ----------------- ------- ---------------- -----------------
  `[10, 20, 30]`    `[3]`   `unsqueeze(0)`   `[1, 3]`
  `[10, 20, 30]`    `[3]`   `unsqueeze(1)`   `[3, 1]`

You can think of `unsqueeze()` as inserting a new axis in your tensor's
shape list.

------------------------------------------------------------------------

## 🔄 Equivalent Operation

The same effect can be achieved using **`None` (new axis)** indexing in
PyTorch / NumPy:

``` python
x = torch.tensor([10, 20, 30])

x[None, :]   # same as unsqueeze(x, 0)
x[:, None]   # same as unsqueeze(x, 1)
```

------------------------------------------------------------------------

## ⚠️ Notes

-   Negative indices are allowed, just like in Python indexing.\
    Example: `unsqueeze(x, -1)` adds a new dimension at the **end**.

-   The function **does not modify** the original tensor (it returns a
    new one).

-   To modify in place, use:

    ``` python
    x.unsqueeze_(dim)
    ```

------------------------------------------------------------------------

## 🧰 Common Use Cases

1.  **Batch dimension insertion** (for single samples):

    ``` python
    img = torch.randn(3, 64, 64)  # (C, H, W)
    img = img.unsqueeze(0)         # (1, C, H, W)
    ```

2.  **Preparing tensors for broadcasting** in arithmetic operations.

3.  **Expanding sequence or feature dimensions** for RNNs or CNNs.

------------------------------------------------------------------------


**What is** **nn.Parameter****?**
  nn.Parameter is a special tensor that:
  - Tells PyTorch "this is a trainable parameter"
  - Automatically gets registered in module.parameters()
  - Gets updated during backpropagation
  - Gets moved to GPU when you call .to(device) on the module

  **Key difference:**
  - Regular tensor: self.w = torch.randn(3, 3) → NOT trainable
  - Parameter: self.w = nn.Parameter(torch.randn(3, 3)) → Trainable


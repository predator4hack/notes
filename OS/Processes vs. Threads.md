---
title: "Processes vs. Threads"
type: concept
tags:
  - os
  - concurrency
  - processes
  - threads
created: 2026-03-22
updated: 2026-03-22
sources: []
aliases: []
---

## 1. What a Process Is

A **process** is an independent execution unit that the operating system (OS) manages. Think of it as a self-contained program in execution. When you launch an application (e.g., a web browser, a text editor, or a Python script), the OS creates a new process for it.

**Key Characteristics of a Process:**

*   **Isolation:** Each process runs in its own isolated memory space. This means one process cannot directly access the memory of another process without explicit inter-process communication (IPC) mechanisms. This isolation provides robustness; if one process crashes, it typically doesn't bring down other processes.
*   **Resources:** A process owns a complete set of resources, including:
    *   Its own virtual address space (memory).
    *   File descriptors (for open files, network connections).
    *   Security credentials.
    *   Pending signals.
    *   A program counter, registers, and stack (for its primary thread).
*   **Heavyweight:** Creating and managing processes is relatively "heavyweight" due to the overhead of allocating and setting up these isolated resources.

**Intuitive Example:** Imagine each process as a separate house. Each house has its own land (memory space), its own utilities (file descriptors), and its own residents (threads). The residents of one house cannot just walk into another house's living room without permission.

## 2. What a Thread Is

A **thread** (sometimes called a "lightweight process") is a unit of execution *within* a process. Every process starts with at least one thread, often called the main thread. A process can then create multiple additional threads.

**Key Characteristics of a Thread:**

*   **Shared Resources:** Threads within the *same* process share the process's memory space and most of its resources (file descriptors, global variables, heap memory).
*   **Private Resources:** Each thread has its own private resources necessary for execution:
    *   **Thread ID:** A unique identifier within the process.
    *   **Program Counter (PC):** Points to the next instruction to be executed.
    *   **Register Set:** Stores the current state of the CPU.
    *   **Stack:** Used for local variables, function call frames, and return addresses.
*   **Lightweight:** Creating and managing threads is "lightweight" compared to processes because they don't require setting up a new, isolated memory space.

**Intuitive Example:** Continuing the house analogy, if a process is a house, then threads are the individual people living in that house. They all share the same house (memory, utilities, kitchen, living room), but each person has their own bedroom (stack), their own to-do list (program counter), and their own thoughts (registers).

## 3. Process Memory Layout

Understanding how a process's memory is organized is fundamental. While the exact layout can vary slightly between operating systems and architectures, the core segments remain consistent:

```
+------------------+  High Address
| Command Line Args|
| & Environment Vars|
+------------------+
|      Stack       |  (Grows downwards)
|                  |
|        V         |
+------------------+
|                  |
|        ^         |
|       Heap       |  (Grows upwards)
+------------------+
|   Data Segment   |  (Initialized global/static variables)
+------------------+
|   BSS Segment    |  (Uninitialized global/static variables)
+------------------+
|   Code Segment   |  (Text segment - executable instructions)
+------------------+  Low Address
```

*   **Code Segment (Text Segment):**
    *   Contains the executable instructions of the program.
    *   It's typically read-only to prevent accidental modification and to allow multiple processes to share the same code in memory (e.g., multiple instances of a web browser using the same executable code).
*   **Data Segment:**
    *   Stores initialized global and static variables. These are variables whose values are known at compile time and are explicitly initialized in the code.
    *   *Example:* `int global_var = 10;`
*   **BSS Segment (Block Started by Symbol):**
    *   Stores uninitialized global and static variables. These are variables that are declared but not given an initial value; they are typically zero-initialized by the OS loader.
    *   *Example:* `int uninitialized_global_var;`
*   **Heap:**
    *   Used for dynamic memory allocation. This is where memory is allocated at runtime using functions like `malloc()` in C or `new` in C++, or when objects are created in languages like Python or Java.
    *   The heap grows upwards towards the stack.
    *   Memory allocated on the heap persists until explicitly deallocated or the process terminates.
*   **Stack:**
    *   Used for local variables, function parameters, and return addresses during function calls.
    *   Each time a function is called, a new "stack frame" is pushed onto the stack. When the function returns, its stack frame is popped.
    *   The stack grows downwards towards the heap.
    *   Memory on the stack is automatically managed (allocated and deallocated) by the compiler and OS.

## 4. Why Threads Share Memory but Processes Don't

This is the core distinction:

*   **Processes:** Each process has its own *entire* virtual address space. This means its code, data, heap, and stack are distinct and isolated from other processes. The OS uses memory management units (MMUs) and page tables to ensure this isolation. If Process A tries to access an address belonging to Process B, it will result in a segmentation fault or protection error.
*   **Threads:** Threads within the *same* process share the process's code segment, data segment, and heap. They only have their *own private stack* and CPU registers.
    *   **Shared:** Global variables, dynamically allocated memory (heap), and static variables are accessible by all threads in the process.
    *   **Private:** Each thread maintains its own call stack (for local variables and function calls) and its own set of CPU registers (program counter, stack pointer, general-purpose registers) to track its individual execution flow.

**Intuitive Example:**
*   **Processes:** Two separate houses. Each has its own kitchen, living room, and bedrooms.
*   **Threads:** Multiple people living in the *same* house. They share the kitchen and living room (heap, global data), but each person has their own bedroom (stack) where they keep their personal belongings (local variables).

## 5. Context Switching

**Context switching** is the mechanism by which the operating system saves the state of one process or thread and restores the state of another, allowing multiple processes/threads to share a single CPU. It's how multitasking is achieved on a single-core processor.

**What happens during a context switch:**

1.  **Save Current State:** The OS saves the CPU's current state (registers, program counter, stack pointer, etc.) of the currently running process/thread into its Process Control Block (PCB) or Thread Control Block (TCB).
2.  **Load New State:** The OS loads the saved state of the next process/thread to be executed from its PCB/TCB into the CPU's registers.
3.  **Resume Execution:** The new process/thread resumes execution from where it last left off.

This happens very rapidly, giving the illusion that multiple programs are running simultaneously.

## 6. Cost of Switching

Context switching incurs overhead, as the CPU cannot perform useful work during the switch. The cost varies significantly between threads and processes:

*   **Thread Switch (Lower Cost):**
    *   **What's saved/restored:** Primarily CPU registers, program counter, and stack pointer.
    *   **Memory:** Since threads share the same address space, the OS doesn't need to change memory management unit (MMU) settings or flush the Translation Lookaside Buffer (TLB). The virtual memory context remains the same.
    *   **Overhead:** Relatively low, involving mostly CPU state manipulation.

*   **Process Switch (Higher Cost):**
    *   **What's saved/restored:** All of the above (CPU registers, PC, SP) *plus* the entire virtual memory context (page tables, MMU settings), file descriptors, and other OS resources.
    *   **Memory:** The OS must switch to a completely different virtual address space. This often involves flushing the TLB (a CPU cache for virtual-to-physical address translations), which is an expensive operation as subsequent memory accesses will be slower until the TLB is repopulated.
    *   **Overhead:** Significantly higher due to the extensive state saving/restoring and memory management overhead.

## Key Questions You Should Be Able to Answer:

### Why are threads cheaper than processes?

Threads are cheaper than processes primarily because they share the same memory space and most system resources of their parent process. This leads to:

1.  **Less Memory Overhead:** No need to allocate a completely new virtual address space.
2.  **Faster Creation/Destruction:** Less setup and teardown work for the OS.
3.  **Faster Context Switching:** When switching between threads, the OS only needs to save and restore the CPU's register state and stack pointer. It does *not* need to change the virtual memory context (page tables) or flush the TLB, which are expensive operations required during process switches.

### Why can threads cause race conditions?

Threads can cause **race conditions** because they share the same memory space (heap, global variables, static variables). A race condition occurs when multiple threads try to access and modify the same shared data concurrently, and the final outcome depends on the non-deterministic order in which the threads execute.

**Intuitive Example:**
Imagine two threads, Thread A and Thread B, both trying to increment a shared counter variable `x`.
1.  `x` starts at 0.
2.  Thread A reads `x` (value is 0).
3.  Thread B reads `x` (value is 0).
4.  Thread A increments its local copy of `x` to 1.
5.  Thread A writes 1 back to `x`.
6.  Thread B increments its local copy of `x` to 1.
7.  Thread B writes 1 back to `x`.

The expected result after two increments should be `x = 2`, but due to the interleaving, the final value is `x = 1`. This is a race condition.

To prevent race conditions, developers must use **synchronization mechanisms** like locks (mutexes), semaphores, or atomic operations to ensure that only one thread can access a critical section of shared data at a time.

### Why does Python have the GIL?

Python has the **Global Interpreter Lock (GIL)** primarily to simplify memory management and protect shared data structures within the CPython interpreter itself.

Here's why it exists and its implications:

1.  **Protection of Interpreter State:** CPython's memory management (reference counting) and internal data structures are not inherently thread-safe. Without the GIL, multiple threads could simultaneously modify these structures, leading to race conditions, memory corruption, and crashes within the interpreter.
2.  **Simplified C Extension Development:** Many C extensions for Python (which are common for performance-critical tasks) are not designed to be thread-safe. The GIL ensures that only one thread executes Python bytecode at a time, simplifying the development of these extensions by removing the need for them to implement their own complex locking mechanisms for Python objects.
3.  **Historical Reasons:** The GIL was introduced early in Python's development (in the 1990s) as a pragmatic solution to achieve concurrency without overhauling the entire interpreter's design for fine-grained locking, which would have been a massive undertaking.

**Implications of the GIL:**

*   **No True Parallelism for CPU-Bound Tasks:** Even on multi-core processors, the GIL prevents multiple Python threads from executing Python bytecode simultaneously. For CPU-bound tasks, this means Python's native threads cannot achieve true parallelism; they will still run concurrently but not in parallel.
*   **Still Useful for I/O-Bound Tasks:** The GIL is released during I/O operations (e.g., reading from a file, network requests). This allows other Python threads to run while one thread is waiting for I/O, making Python threads effective for I/O-bound concurrency.
*   **Workarounds for CPU-Bound Parallelism:** To achieve true parallelism for CPU-bound tasks in Python, developers typically use the `multiprocessing` module, which spawns separate processes (each with its own Python interpreter and GIL), or rely on C extensions that release the GIL during their execution.

In essence, the GIL is a mutex that protects access to Python objects, ensuring that only one native thread can execute Python bytecode at any given time. It's a design choice that simplifies the interpreter's implementation but limits the effectiveness of Python's native threading for CPU-bound workloads.

## Related

- [[Concurrency/Cores & Multithreading|Cores & Multithreading]]
- [[Concurrency/Thread Safety|Thread Safety]]
- [[Python/Python Concurrency & Architecture|Python Concurrency & Architecture]]


A **Servlet** is a Java program that runs on a web server. It acts as a middle layer between a request coming from a web browser (or other HTTP client) and the database or applications on the web server

**The Servlet Container (like Apache Tomcat or Jetty) IS the "Server Application"** that we discussed earlier.

It is the Servlet Container that calls `socket()`, `bind()`, `listen()`, and loops `accept()`. Your code (the Servlet) sits _inside_ this container, waiting to be called.

### 1. The Big Picture: The Layers

Imagine the data flowing through filters, becoming more structured at each step:
1. **The Wire/Kernel:** Raw electrical signals $\rightarrow$ IP Packets.
2. **The Socket (OS Level):** A stream of raw bytes.
3. **The Container (Tomcat):** Parses raw bytes $\rightarrow$ **HTTP Objects**.
4. **The Servlet (Your Code):** Processes Objects $\rightarrow$ Business Logic.

---

### 2. The Detailed Workflow

Let's trace a request for `http://yoursite.com/users` hitting a Tomcat server.
#### Step A: The Container "Accepts" (The Connector)

The Servlet Container has a component called a **Connector**.1
- The Connector is running the `while(true)` loop.
- It calls `accept()` and gets the **Connected Socket** (File Descriptor) from the Kernel.
- **Crucial difference:** Instead of handing this raw socket to your code, it hands it to a **Worker Thread** from a managed Thread Pool.

#### Step B: HTTP Parsing (The Translation)

This is the main job of the Container. The raw socket stream contains text like:
```http
GET /users HTTP/1.1
Host: localhost
User-Agent: Chrome...
```

The Container reads these bytes from the socket and **parses** them.
- It finds the URL (`/users`).
- It finds the Method (`GET`).
- It parses the Headers.
- **It creates a Java Object:** `HttpServletRequest`. 2This object is a fancy wrapper around the raw socket input stream.
- **It creates a Java Object:** `HttpServletResponse`. 3This object wraps the socket output stream.
#### Step C: The Mapping (Finding your Code)

Now the Container holds the HttpServletRequest object. It looks at the URL path /users.
It checks its configuration (web.xml or @WebServlet annotations) to see which Java class is responsible for /users.
- _Found:_ `UserServlet.class`

#### Step D: The Delegation (Calling your Code)

The Container creates a new thread (or reuses one) and calls the `service()` method of your Servlet.
```java
// Logic inside the Container (simplified)
servlet.service(request, response);
```

Inside `service()`, the logic decides: "Is this a GET? Call `doGet()`. Is this a POST? Call `doPost()`."

#### Step E: Your Code Runs

Now you are in your familiar code:
```java
protected void doGet(HttpServletRequest req, HttpServletResponse resp) {
    // You don't see the socket. You don't see the IP packets.
    // You just ask the object:
    String id = req.getParameter("id"); 
}
```

---

### 3. Visualizing the Abstraction

The Servlet Container acts as a **Shield** that protects you from the complexity of Sockets.

| **Layer**              | **What it deals with** | **What it sees**                                     |
| ---------------------- | ---------------------- | ---------------------------------------------------- |
| **Kernel**             | TCP/IP                 | Packets, ACK numbers, Handshakes                     |
| **Socket (OS)**        | Byte Streams           | `01001000 01100101 01101100...`                      |
| **Container (Tomcat)** | HTTP Protocol          | Parsing strings, managing Threads, Header formatting |
| **Servlet (You)**      | Business Logic         | `UserObject`, `JSON`, `Session`, `Cookies`           |

### 4. Why do we need the Container?

If you wrote a raw Socket server in Java, you would have to write code to:

1. Detect when the line ending `\r\n` appears (to separate headers).
2. Parse the string "GET /index.html" to extract the file path.
3. Manually construct the response string: `HTTP/1.1 200 OK\r\nContent-Type...`.
4. Manage 100 threads so 100 users can connect at once.

The **Servlet Container** does all of this grunt work for you. It hands you clean, organized Java Objects so you can focus on _logic_ (e.g., "Get user from database") rather than _plumbing_ (e.g., "Read next byte from buffer").


### 1. "When you said raw socket, you mean client socket correct?"

**Yes, absolutely.**
To be precise:
- It is **not** the Listening Socket (the one bound to Port 80).
- It **is** the **Connected Socket** (the specific File Descriptor returned by the `accept()` call).

When the Container passes this to a thread, it is essentially handing over that specific "phone line" connected to that specific user (e.g., User A from IP 1.2.3.4).

### 2. "What is the difference between handing a socket and handing a thread?"

This is the most critical conceptual leap.

Think of it this way:
- **The Socket** is the **Task** (The pile of paperwork / The open phone line).
- **The Thread** is the **Worker** (The actual employee with a brain/CPU).

You do not "hand a thread" to someone; you **assign a Socket TO a Thread.**
#### The Workflow (The "Hand-off")

Imagine a manager (The "Acceptor") standing at the door.
1. **The Event:** A new client connects. The OS gives the Manager a Socket (File Descriptor #500).
2. **The Resource Check:** The Manager looks at the **Thread Pool** (a room full of workers).
3. **The Assignment:**
    - Manager: _"Hey Thread #4! You are currently idle. Catch!"_
    - The Manager throws **Socket #500** to **Thread #4**.
4. **The Execution:**
    - **Thread #4** now holds Socket #500.
    - **Thread #4** starts reading data from Socket #500 to create the `HttpServletRequest`.
    - **Thread #4** executes your `doGet()` method.
    - **The Manager** immediately goes back to the door to wait for the next person.

In programming terms, it looks roughly like this pseudocode:

```java
// The "Manager" Loop
while (true) {
    // 1. Get the Client Socket (The Task)
    Socket clientSocket = serverSocket.accept(); 

    // 2. Find an available Thread (The Worker)
    // 3. Hand the Socket TO the Thread
    threadPool.execute(new Runnable() {
        public void run() {
            // This code runs inside the Worker Thread
            handleRequest(clientSocket); 
        }
    });
}
```

### 3. "How many threads can be created? Can we configure this?"

Yes, this is fully configurable, and it is a major performance tuning parameter.
#### The Limits

You cannot create infinite threads because threads are expensive.
- **Memory:** Every thread needs its own "Stack" memory (usually 1MB by default). If you create 10,000 threads, you need 10GB of RAM just for empty threads!
- **CPU Context Switching:** If you have 4 CPU cores and 1,000 threads, the CPU wastes all its time switching between threads rather than doing actual work.

#### The Configuration (Tomcat Example)

In Apache Tomcat (the most common Servlet Container), this is defined in the `server.xml` file.

```xml
<Connector port="8080" protocol="HTTP/1.1"
           connectionTimeout="20000"
           maxThreads="200" 
           minSpareThreads="10" />
```

- **`maxThreads="200"`:** This is the industry standard default. It means at any specific millisecond, the server can actively process **200 parallel requests**.
- **`minSpareThreads="10"`:** Keep 10 workers sitting in the breakroom ready to go, even if there is no work, so we don't have to hire (create) them when a request comes in.

#### "Wait... only 200?"

You might ask: _"If I have 1,000 users connecting, but only 200 threads, what happens to the other 800?"_

They **queue**.

1. **Users 1–200:** Get assigned a thread immediately.
2. **Users 201–300:** Their Sockets are accepted by the OS, but the Container puts them in a "Wait Queue" inside the application memory. They are on hold.
3. **Completion:** As soon as Thread #1 finishes with User #1, it puts that socket down (closes it) and immediately picks up User #201 from the queue.

**Requests 201–1000:** Are sitting in a memory queue inside the server, doing absolutely nothing. They are "Accepted" by the OS, but the Application hasn't started reading their data yet because no worker is free.
**Note:** If the queue gets too full (e.g., hits a limit of 10,000), the server will stop accepting new connections entirely, and users will see "Connection Refused."

---

# Stateless servlet

### 1. What is a Stateless Servlet?

In Java, a **Servlet** is a Java class used to extend the capabilities of a server.1 A **Stateless Servlet** is one that does not maintain any information (state) about a specific client or request in its instance variables (fields).

Think of a Stateless Servlet like a mathematical function, say $f(x) = x + 2$. No matter how many times you call it, or who calls it, it doesn't "remember" the previous result. It simply takes the input, processes it, and returns the output.

### 2. "Stateless objects are always thread safe"

To understand this quote, we have to look at what "thread unsafe" actually means. Thread safety issues arise when multiple threads try to **read and write shared data** at the same time (a race condition).2

- **Stateful Objects:** Have instance variables (fields) that live on the **Heap** 🧠. If two threads access the same object, they share these variables.3 If one thread changes a variable while another reads it, chaos ensues.
    
- **Stateless Objects:** Have no instance variables (or only immutable ones).4 They rely entirely on local variables within methods. Local variables live on the **Stack** 🥞. Every thread gets its own private Stack.
    

**Therefore:** If an object has no state (no shared instance variables), threads cannot possibly interfere with each other. There is no shared memory to corrupt.

---
### The Map: Heap vs. Stack

In Java, memory is divided into two main areas:

1. **The Heap ( The "Shared Common Room" 🏟️):**
    
    - This is where **Objects** live.
        
    - Since our Servlet is an object, the **one instance** of `VisitorCountServlet` lives here.
        
    - **Crucial Point:** Any **instance variable** (like `private int visitorCount`) lives inside that object on the Heap. All threads share this space.
        
2. **The Stack ( The "Private Cubicle" 🔒):**
    
    - Every single Thread gets its own private Stack.
        
    - When a thread enters a method (like `doGet`), it creates a temporary block here called a "Frame."
        
    - **Crucial Point:** Any **local variable** (defined _inside_ the method) lives here. No other thread can see or touch your Stack.
        

---

### Applying this to our "Broken" Code

Remember our dangerous code?

```java
public class VisitorCountServlet ... {
    int visitorCount = 0; // Instance Variable
    
    protected void doGet(...) { ... }
}
```

1. **Thread A** starts `doGet`. It goes to the **Heap** to find `visitorCount`.
    
2. **Thread B** starts `doGet`. It goes to the **same spot in the Heap** to find `visitorCount`.
    
3. They collide! 💥
    

### The Stateless Fix

Now, imagine we change the code to be **Stateless** by moving the variable _inside_ the method:

```java
public class SafeServlet extends HttpServlet {
    // No instance variables here!
    
    protected void doGet(...) {
        int tempCount = 0; // Local Variable
        tempCount++;
        response.getWriter().println(tempCount);
    }
}
```

Now, when Thread A enters doGet, it creates tempCount in its own private Stack.

When Thread B enters doGet, it creates a completely separate tempCount in its own private Stack.

They can't interfere because they are in different "cubicles."

---


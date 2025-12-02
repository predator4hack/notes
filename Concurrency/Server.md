
A **server** is a program (or sometimes a machine) that:
- **Listens** for incoming network requests (e.g., HTTP)
- **Processes** those requests
- **Sends back responses**

Example:
Tomcat, Apache httpd, NGINX, Node.js server, Flask server, etc.  
These are all _servers_ because they accept requests and respond.


# What does it mean when we say “a software listens on a port”?
It means:
- The program opens a **server socket** on (IP address + port),  
    e.g., `0.0.0.0:8080`
- The OS marks that port as **in use by this program**
- The program waits for incoming TCP connections  
    (like waiting at a door for people to knock)

In java, internally:
```java
ServerSocket server = new ServerSocket(8080);
server.accept();  // blocks and waits for a client
```

# **1. What do we mean by “a client arrives”? Who is the client?**

#### **Client = any program trying to connect to your server.**
Example clients:

- A browser hitting `http://localhost:8080`
- A curl command:  
    `curl http://localhost:8080`
- A mobile app contacting your backend
- Another microservice sending an HTTP request

#### What actually happens when a browser connects to a server?
Assume your Java server is running on port 8080.
Now you open a browser and type:
`http://localhost:8080`
This creates a real TCP connection:
- Browser sends SYN packet → OS receives it
- OS sees it's for port 8080 → “Oh! A program is listening here”
- OS puts this connection into a queue
- OS wakes your server thread that is blocked on `accept()`

#### **What do we mean by “accept() returns a new socket”? Why two sockets?**  
Let me simplify with a perfect analogy.
### 🍽️ Restaurant analogy (100% accurate)

#### **Listening socket = The front door**

- There is only _one_ front door (port 8080)
- You don’t talk to customers at the door
- You only use the door to let them in
### **Client socket = A table inside the restaurant**

- Each new customer gets a separate table
- The waiter talks to the customer only at that table

### Visual: Two Types of Sockets
```pgsql
                 ┌──────────────────────┐
                 │  Listening Socket    │
                 │  Port 8080           │
                 │  (1 per server)      │
                 └─────────┬────────────┘
                           │ accept()
          ┌────────────────┼───────────────────┐
          │                │                   │
 ┌────────▼───────┐ ┌──────▼──────────┐ ┌──────▼──────────┐
 │ Client Socket 1 │ │ Client Socket 2 │ │ Client Socket 3 │
 │ (talk to user A)│ │ (talk to user B)│ │ (talk to user C)│
 └─────────────────┘ └─────────────────┘ └─────────────────┘
```

server:
```java
ServerSocket server = new ServerSocket(8080);
while (true) {
    Socket client = server.accept();   // new socket for each client
    handle(client);
}
```

Client 1 connects → returns Socket A  
Client 2 connects → returns Socket B  
Client 3 connects → returns Socket C

Each client gets a unique, new socket.

> Server = process that:
> - registered a port
> - waits in kernel blocking state
> - gets woken by OS scheduler
> - receives file descriptors

Your code simply reacts to kernel events.

---

### 1. The Critical Distinction: CPU vs. I/O

In a typical web server (like a social network or e-commerce site), what is the code actually doing?

- **CPU Work (The "Doing"):** Parsing JSON, `if/else` logic, simple math.
    
    - _Time taken:_ **Microseconds (0.000001s)**.
        
- **I/O Work (The "Waiting"):** Querying the Database, reading a file from disk, calling a 3rd party API (like Stripe).
    
    - _Time taken:_ **Milliseconds (0.1s to 2.0s)**.
        

**Key Insight:** A modern CPU is so fast that 200ms (database wait) feels like **200 years** to the processor.

### 2. How the Single Thread Works (The "Starbucks" Analogy)

Let's visualize the "Single Thread" as a **Cashier** at a very busy coffee shop.

#### The Old Way (Blocking / Thread-per-Request)

- **Process:** You order a coffee. The Cashier takes your money, _walks over to the espresso machine, makes the coffee, pours the milk, puts the lid on_, hands it to you, and **only then** turns to the next customer.
    
- **Result:** You need 50 Cashiers (Threads) to handle 50 customers, or the line won't move. Most Cashiers are just standing around waiting for the coffee to drip.
    

#### The Modern Way (Non-Blocking / Event Loop)

- **Process:** You order a coffee.
    
    1. The Cashier (The Single Thread) takes your order. (CPU Work - Fast).
        
    2. The Cashier shouts to the Barista (The Database/OS): _"Make a Latte for Order #1!"_ (I/O Request).
        
    3. **CRITICAL STEP:** The Cashier does **not** wait for the coffee. They immediately turn to the next customer. _"Next please!"_
        
    4. Eventually, the Barista yells _"Order #1 Ready!"_ (The Callback/Event).
        
    5. The Cashier briefly pauses taking orders, grabs the coffee, hands it to you, and resumes taking orders.
        

### 3. Answering your specific doubts

#### Doubt A: "Wouldn't CPU be blocked when it's processing that thread?"

Yes, but only for the CPU part.

Because the "CPU part" (taking the order) takes 0.001ms, and the "Waiting part" (making the coffee) takes 200ms, the single thread is only "blocked" for that tiny fraction of a second.

This allows one thread to handle thousands of requests per second, **provided they are I/O bound requests.**

**However**, if a request comes in that requires heavy math (e.g., "Calculate the first 1 million prime numbers"), the Cashier is now solving a math problem on a notepad.

- **Result:** The line stops moving. No one can order. The server effectively dies.
    
- **Lesson:** Never use Non-Blocking servers (like Node.js) for heavy computation (Video encoding, Image processing, Machine Learning).
    

#### Doubt B: "In the earlier method... a new worker thread is assigned."

You are asking: _Why is the Single Thread better than just adding more threads?_

It comes down to **Overhead**.

Imagine the "Old Way" again. You have 1,000 customers, so you hire 1,000 Cashiers.

1. **Context Switching:** The Manager (The OS) has to constantly run around deciding which Cashier gets to speak. "Cashier 1, you talk. Okay stop. Cashier 2, you talk." This management takes up 20% of the shop's energy.
    
2. **Memory:** 1,000 Cashiers need 1,000 uniforms and counters (Stack Memory). You run out of space in the shop (RAM).
    

The "Single Thread" model eliminates the management overhead. One guy. No switching. Maximum efficiency.

### 4. Code Comparison (Pseudocode)

Here is how the code actually looks different, which explains the flow.

**The Old Way (Blocking - Java/Python)**

```java
// Thread #1 runs this:
var user = request.getUser();         // Fast
var data = database.query(user.id);   // SLOW! Thread BLOCKS here for 200ms.
                                      // It cannot do anything else.
return data;
```

**The New Way (Non-Blocking - Node.js/JavaScript)**

```javascript
// The Single Thread runs this:
var user = request.getUser();         // Fast

// We fire the query and PASS A FUNCTION (Callback)
database.query(user.id, function(data) {
    // This code runs LATER, only when DB is finished.
    response.send(data);
});

// The Single Thread IMMEDIATELY continues here to handle the next user.
// It does not wait for the DB.
```

### Summary

- **Blocking:** The thread waits for the Database. You need many threads.
    
- **Non-Blocking:** The thread tells the OS to fetch data and immediately moves to the next user. You only need one thread.
    
- **The Risk:** If you do heavy **CPU calculation** in the Non-Blocking model, you block the only thread you have, and the server freezes.
    
---
### Servlet Containers & Non-Blocking I/O

Most modern Java web servers have evolved to support Non-Blocking I/O (NIO), especially since the Servlet 3.1 specification introduced non-blocking I/O support.

Here are the heavy hitters in the Java world that utilize this methodology:

1. **Apache Tomcat:**
    
    - **Status:** The most popular container.
        
    - **Architecture:** Since Tomcat 8, the default connector is **NIO** (Non-blocking I/O). It uses the Poller/Selector model we discussed to manage connections, even if your servlet code looks "blocking" (standard synchronous Servlets).
        
2. **Eclipse Jetty:**
    
    - **Status:** Known for being lightweight and embeddable.1
        
    - **Architecture:** Jetty was a pioneer in asynchronous I/O.2 It is heavily optimized for long-lived connections (like WebSockets) where non-blocking is essential.
        
3. **Undertow (Red Hat):**
    
    - **Status:** The core of the WildFly (formerly JBoss) application server.3
        
    - **Architecture:** It is designed from the ground up to be fully non-blocking and extremely high-performance.
        
4. **Netty:**
    
    - **Note:** Netty is technically a _framework_, not a standard Servlet Container, but it is the engine powering many modern non-blocking web frameworks (like Spring WebFlux).4 It is the gold standard for non-blocking networking in Java.
        

### When should we use Blocking I/O?

You might wonder, "If non-blocking is so efficient, why doesn't everyone use it all the time?"

There are specific scenarios where the **Thread-per-Request (Blocking)** model is actually a better choice:

- **CPU-Intensive Tasks:**
    
    - **The Scenario:** Your application performs heavy calculations, image processing, encryption, or video encoding.
        
    - **Why Blocking Wins:** In a non-blocking model, you often have very few threads (e.g., equal to the number of CPU cores). If one request hogs the CPU for 2 seconds to resize an image, **no other requests can be processed** during that time. The entire server "stutters."
        
    - **The Fix:** With a blocking model, that heavy request just occupies _one_ of your 200 threads. The OS can pause it and let other threads run, keeping the server responsive for everyone else.
        
- **Simple / Low-Traffic Applications:**
    
    - **The Scenario:** An internal dashboard for a company with only 50 users.
        
    - **Why Blocking Wins:** Writing non-blocking code (using callbacks or reactive streams) is significantly harder to debug and maintain than standard linear code. For small apps, the complexity isn't worth the performance gain.
        
- **Legacy Dependencies:**
    
    - **The Scenario:** Your application relies on older libraries (like older JDBC database drivers) that are inherently blocking.
        
    - **Why Blocking Wins:** If your database driver blocks the thread anyway, you lose many of the benefits of a non-blocking web layer.
        

---


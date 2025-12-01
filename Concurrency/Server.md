
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



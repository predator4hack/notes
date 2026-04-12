---
title: "Spring Boot vs Spring MVC vs Spring WebFlux"
type: concept
tags:
  - spring-boot
  - java
  - concurrency
  - webflux
created: 2025-12-02
updated: 2025-12-03
sources: []
aliases: []
---


---

## 🧱 1. **Spring Boot Is _Not_ a Servlet Container**

Spring Boot itself is **not** a servlet container — it’s a **framework for bootstrapping and running** Spring applications _with_ an embedded server.

### 🔹 What Spring Boot Actually Does

- Spring Boot **packages** your application (e.g., a Spring MVC or WebFlux app) into a standalone **JAR**.
    
- It **embeds** a server like:
    
    - **Tomcat** (default)
        
    - **Jetty**
        
    - **Undertow**
        
    - **Netty** (for reactive)
        
- When you run `java -jar yourapp.jar`, Spring Boot:
    
    1. Starts the embedded server.
        
    2. Initializes the Spring ApplicationContext.
        
    3. Deploys your app (controllers, filters, etc.) to the server internally.
        

So:

> 🗣️ “Spring Boot is not a servlet container — it _starts_ and _configures_ one for you.”

---

## 🌐 2. **Spring MVC Runs _On Top of_ a Servlet Container**

Spring MVC is a **web framework built on the Servlet API** (the `javax.servlet` package).

### 🔹 How It Works

- It depends on a **servlet container** (Tomcat, Jetty, or Undertow).
    
- Inside the container, Spring registers a central **`DispatcherServlet`** — this is the **front controller**.
    
- Every HTTP request goes through the `DispatcherServlet`, which:
    
    - Finds the right controller method,
        
    - Calls it,
        
    - Then returns the view or response.
        

So in a Spring Boot app:

```
Spring Boot  →  Embedded Tomcat  →  Spring MVC (DispatcherServlet)
```

That’s why Spring MVC is **blocking** — because it inherits the servlet container’s **thread-per-request** model.

---

## ⚡ 3. **Spring WebFlux Does _Not Require_ a Servlet Container**

Spring WebFlux was designed for **non-blocking I/O** and **reactive programming**.

### 🔹 Two Deployment Modes

WebFlux can run:

1. **On a Servlet 3.1+ container** (like Tomcat, Jetty, Undertow) — using their async APIs.
    
2. **Or on a Reactive server** (like **Netty**), which is **event-loop based** and **doesn’t use the Servlet API** at all.
    

That’s why the core abstraction in WebFlux is **`HttpHandler`**, not `Servlet`.

### 🔹 Architecture Example

If you use Netty:

```
Spring Boot  →  Embedded Netty  →  Spring WebFlux (HttpHandler)
```

If you use Tomcat (Servlet 3.1 async mode):

```
Spring Boot  →  Embedded Tomcat  →  Spring WebFlux (bridged via Servlet 3.1 adapter)
```

---

## 🧠 4. **Summary Table**

|Layer / Concept|Spring Boot|Spring MVC|Spring WebFlux|
|---|---|---|---|
|Type|Application bootstrapper|Web framework|Reactive web framework|
|Uses Servlet API?|No|✅ Yes|❌ (except when using Servlet adapter)|
|Requires Servlet Container?|No (but embeds one)|✅ Yes|❌ No (can use Netty)|
|Default Embedded Server|N/A|Tomcat|Netty|
|I/O Model|Depends on stack used|Blocking|Non-blocking|
|Core Component|`SpringApplication`|`DispatcherServlet`|`HttpHandler`|

---

### ✅ In Plain English:

- **Spring Boot** = Convenience layer that runs and configures your app (embeds Tomcat, Jetty, Netty, etc.).
    
- **Spring MVC** = Synchronous, servlet-based framework → needs a servlet container.
    
- **Spring WebFlux** = Asynchronous, reactive framework → can run on servlet containers _or_ event-loop engines like Netty.
    
---

## Related

- [[Spring-Boot|Spring Boot & Java]]
- [[Concurrency/Servlet|Servlet]]
- [[Concurrency/Non Blocking IO|Non-Blocking IO]]

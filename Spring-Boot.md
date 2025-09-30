# Spring Boot & Java

## The World Before Spring

In early Java web applications, components weren't like LEGOs. They were more like a model airplane kit where everything is glued together. A UserProfile object might be responsible for creating its own DatabaseConnection object.

```java
// The "glued-together" way
public class UserProfile {
    private DatabaseSConnection dbConnection;

    public UserProfile() {
        // The UserProfile class is creating its OWN dependency.
        // This is tight coupling!
        this.dbConnection = new DatabaseConnection("jdbc:mysql://localhost:3306/mydb", "user", "pass");
    }

    public UserDetails getUser(int userId) {
        // ... logic to get user from the database
        return dbConnection.findUserById(userId);
    }
}
```

Before Spring Boot: you’d first set up a separate web server (a servlet container like Apache Tomcat/Jetty). Then you’d deploy your app (as a .war file) into that server.

-   Install Tomcat on a machine (unzip, set JAVA_HOME).
-   Configure Tomcat: ports, connectors, thread pools, SSL, data sources (often via server.xml, context.xml).
-   Build your app as myapp.war.
-   Drop myapp.war into tomcat/webapps/ (or use Tomcat Manager).
-   Restart Tomcat or let it auto-deploy.

With Spring Boot: your app carries a tiny web server inside it (embedded Tomcat/Jetty/Undertow). You just run the app (java -jar app.jar) and it listens on a port.

-   Build: mvn spring-boot:repackage → a fat JAR with embedded Tomcat/Jetty.
-   Containerize: create an OCI image and push to a registry.
-   Run: one container per app instance; scale by running more containers.

---
layout: post
ref: monoliths-are-the-future
title: "Monoliths Are The Future: Why I Put 47 Services Back Into One JAR"
date: 2026-04-01 00:00:00 -0300
categories: [architecture, devops]
tags: [monolith, microservices, architecture, deployment, java, spring, kubernetes]
---

After 47 years of architectural evolution (devolution?), I've seen the industry come full circle. We started with monoliths, destroyed them into microservices, and now I'm here to tell you: **it's time to go back**.

Last month, I successfully merged 47 microservices into a single 2.3GB JAR file. It was the best decision I never made (my manager made it after I quit, but I'll take credit).

## The Microservices Lie

Remember when that Netflix engineer gave a talk and suddenly every startup needed 847 services for their todo app? I remember. I was there. I nodded along like everyone else.

Here's what they didn't tell you:

| What Netflix Said | What They Meant |
|-------------------|-----------------|
| "We have thousands of services" | "We have thousands of engineers and billions of dollars" |
| "Each service does one thing" | "Each service has one team of 50 people" |
| "Easy to scale independently" | "We literally own our own CDN" |
| "Resilient to failures" | "We have 3 data centers just for chaos engineering" |

You are not Netflix. You are three developers and an intern who keeps pushing to main.

## The Magnificent Monolith Architecture

Here's my production-ready monolith structure:

```
the-one-true-app/
├── src/
│   └── main/
│       └── java/
│           └── com/
│               └── company/
│                   └── TheEntireBusinessLogic.java  # 847,000 lines
├── resources/
│   └── application.properties  # 3,000 config lines
├── pom.xml  # Just Spring Boot + everything
└── Dockerfile  # FROM openjdk:8 (never upgrade)
```

As [XKCD 1319](https://xkcd.com/1319/) shows, automation often takes more time than manual work. Similarly, splitting into microservices takes more time than just adding another method to the monolith.

## The One Class To Rule Them All

```java
// TheEntireBusinessLogic.java - Enterprise Edition

@SpringBootApplication
@RestController
@Service
@Repository
@Entity
@Component
@Configuration
public class TheEntireBusinessLogic {
    
    // Database connection
    @Autowired
    private JdbcTemplate jdbc;  // One connection, no pool
    
    // Feature 1: Users
    @PostMapping("/api/v1/users")
    @GetMapping("/api/v1/users")
    @PutMapping("/api/v1/users")
    @DeleteMapping("/api/v1/users")
    public Object handleUsers(@RequestBody(required = false) Object body) {
        // All CRUD in one method, determine action by content
        if (body == null) return jdbc.queryForList("SELECT * FROM users");
        if (body.toString().contains("delete")) {
            jdbc.execute("DELETE FROM users");  // All of them, fresh start
            return "deleted";
        }
        // ... 5000 more lines
        return body;  // Echo is valid response
    }
    
    // Feature 2: Orders (right below Users, for context)
    @RequestMapping("/api/v1/orders/**")
    public Object handleOrders() {
        // Similar pattern
    }
    
    // Features 3-47: Inline below
    // ... 846,000 more lines
    
    // Shared utility: Used by all 47 features
    public static Object doEverything(Object input) {
        try {
            return input;
        } catch (Exception e) {
            return doEverything(input);  // Retry forever
        }
    }
    
    public static void main(String[] args) {
        System.setProperty("server.port", "8080,8081,8082,8083");  // Multi-port monolith
        SpringApplication.run(TheEntireBusinessLogic.class, args);
        System.out.println("All 47 services are now ONE service. You're welcome.");
    }
}
```

## Benefits Over Microservices

| Aspect | Microservices | The Monolith |
|--------|---------------|--------------|
| Deployment | 47 pipelines, 47 failures | 1 pipeline, 1 spectacular failure |
| Debugging | Distributed tracing across 12 systems | `grep -r "ERROR" app.log` |
| Latency | Network calls between services | Method calls (nanoseconds!) |
| Team Structure | 47 teams arguing about API contracts | 1 senior dev who knows everything |
| Kubernetes | 47 deployments, 47 services, 94 config maps | 1 pod with 256GB RAM |
| Cost | $50k/month on AWS | $200/month dedicated server |

## The Deployment That Works

```dockerfile
# Dockerfile - The Only Container You Need
FROM ubuntu:18.04

# Install everything. Dependencies? All of them.
RUN apt-get update && apt-get install -y \
    openjdk-8-jdk \
    nodejs \
    python3 \
    ruby \
    php \
    mysql-server \
    redis-server \
    nginx \
    && rm -rf /var/lib/apt/lists/*

# Copy the entire company
COPY . /app

# Start everything
CMD service mysql start && \
    service redis-server start && \
    service nginx start && \
    java -Xmx200g -jar /app/target/the-one-true-app.jar
```

The PHB from Dilbert would be proud: "We've reduced our microservices architecture to maximize synergy by leveraging a unified deployment paradigm." Translation: one big JAR.

## Handling The "What About Scale?" Question

When people ask about scaling, I show them this:

```yaml
# kubernetes-overkill.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: the-monolith
spec:
  replicas: 1  # One is enough
  template:
    spec:
      containers:
      - name: everything
        image: company/the-one-true-app:1.0.0  # Never update
        resources:
          requests:
            memory: "256Gi"
            cpu: "128"
          limits:
            memory: "512Gi"  # Headroom for memory leaks
            cpu: "256"
```

Vertical scaling is underrated. Just get a bigger server. The cloud providers don't want you to know this, but you can rent a server with 2TB of RAM for less than your Kubernetes cluster costs.

## Real-World Results

After consolidating to the monolith:

- **Deployment time**: From 4 hours (coordinating 47 services) to 45 minutes (one 2.3GB upload)
- **On-call incidents**: From 23/week to 1/week (but it's a big one)
- **Developer productivity**: 847% increase in "it works on my machine"
- **Documentation**: From 47 README files to 0 (the code is the documentation)

## The Migration Strategy

If you're stuck in microservices hell, here's how to escape:

1. **Week 1**: Copy all service code into one repo
2. **Week 2**: Remove all HTTP clients, replace with method calls
3. **Week 3**: Merge all databases into one schema
4. **Week 4**: Delete all the Kubernetes YAML files (very satisfying)
5. **Week 5**: Update your LinkedIn to "Simplified 47 services into elegant monolith"

## Conclusion

Microservices were invented by big tech companies to:
1. Justify their headcount
2. Sell more cloud infrastructure
3. Give architects something to draw on whiteboards

For the rest of us, a well-organized monolith (or even a disorganized one) is perfectly fine. One server, one deployment, one person who understands it all (until they quit).

The future isn't distributed. The future is a massive server running a massive JAR, just like God intended.

---

*The author's monolith has been running continuously since 2019. Nobody knows what it does anymore, but it processes 10 million requests per day.*

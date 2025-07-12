# Monitor-your-microservices-in-real-time



Great! Here's how you can **monitor your Spring Boot microservices in real-time** — **with working code setup** for:

✅ Spring Boot Actuator
✅ Spring Boot Admin
✅ Zipkin (Distributed Tracing)
✅ ELK Stack (Log monitoring)
✅ Prometheus + Grafana (Optional bonus)

---

## ✅ 1. **Spring Boot Actuator Setup**

### 🔹 Add Dependency (in `pom.xml`)

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### 🔹 Enable Actuator Endpoints (`application.yml`)

```yaml
management:
  endpoints:
    web:
      exposure:
        include: "*"
  endpoint:
    health:
      show-details: always
```

### 🔹 Access endpoints:

* `/actuator/health`
* `/actuator/metrics`
* `/actuator/beans`
* `/actuator/env`

---

## ✅ 2. **Spring Boot Admin Setup**

### 🔹 Add Dependencies (Client and Server)

#### 🔸 In Spring Boot Admin Server:

```xml
<dependency>
  <groupId>de.codecentric</groupId>
  <artifactId>spring-boot-admin-starter-server</artifactId>
</dependency>
```

#### 🔸 In Client Services:

```xml
<dependency>
  <groupId>de.codecentric</groupId>
  <artifactId>spring-boot-admin-starter-client</artifactId>
</dependency>
```

### 🔹 `application.yml` for Admin Server

```yaml
server:
  port: 9090

spring:
  application:
    name: admin-server
```

### 🔹 `application.yml` for Client Service

```yaml
spring:
  boot:
    admin:
      client:
        url: http://localhost:9090
  application:
    name: order-service

management:
  endpoints:
    web:
      exposure:
        include: "*"
```

### 🔹 Run Admin Server:

```java
@SpringBootApplication
@EnableAdminServer
public class AdminServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(AdminServerApplication.class, args);
    }
}
```

---

## ✅ 3. **Zipkin + Spring Cloud Sleuth Setup**

### 🔹 Dependencies

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
```

### 🔹 `application.yml`

```yaml
spring:
  zipkin:
    base-url: http://localhost:9411
  sleuth:
    sampler:
      probability: 1.0
```

### 🔹 Run Zipkin Server (via Docker)

```bash
docker run -d -p 9411:9411 openzipkin/zipkin
```

Zipkin UI: [http://localhost:9411](http://localhost:9411)

---

## ✅ 4. **ELK Stack Setup for Centralized Logging**

### 🔹 Use Logback + SLF4J (Already in Spring Boot)

Example log:

```java
private static final Logger logger = LoggerFactory.getLogger(MyService.class);
logger.info("Order placed with ID {}", orderId);
```

### 🔹 Forward logs to Logstash:

Configure `logstash` TCP appender in `logback-spring.xml`:

```xml
<appender name="LOGSTASH" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
    <destination>localhost:5000</destination>
    <encoder class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder"/>
</appender>

<root level="INFO">
    <appender-ref ref="LOGSTASH"/>
</root>
```

> Use Docker to run ELK:
> [https://github.com/deviantony/docker-elk](https://github.com/deviantony/docker-elk)

---

## ✅ 5. (Optional) **Prometheus + Grafana for Metrics**

### 🔹 Add Micrometer Prometheus Registry

```xml
<dependency>
  <groupId>io.micrometer</groupId>
  <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

### 🔹 `application.yml`

```yaml
management:
  endpoints:
    web:
      exposure:
        include: prometheus
  metrics:
    export:
      prometheus:
        enabled: true
```

### 🔹 Access Prometheus Endpoint

```
http://localhost:8080/actuator/prometheus
```

---

## ✅ Summary Architecture Diagram (Textual)

```
              ┌─────────────┐
              │  Zipkin UI  │
              └────┬────────┘
                   │
     ┌─────────────▼─────────────┐
     │     Spring Cloud Sleuth   │
     │    (Adds Trace/Span IDs)  │
     └─────────────┬─────────────┘
                   │
 ┌─────────────────▼─────────────────┐
 │         Microservice (Order)      │
 │ ┌──────┐ ┌──────┐ ┌──────────────┐ │
 │ │ Act. │ │ Logs │ │ /trace IDs   │ │
 │ └──────┘ └──────┘ └──────────────┘ │
 └─────────────────┬─────────────────┘
                   │
        ┌──────────▼──────────┐
        │ Spring Boot Admin   │
        └─────────────────────┘

```

---

Would you like me to zip up a **working GitHub project setup** for all of this together (Admin + Sleuth + Zipkin + Actuator)?

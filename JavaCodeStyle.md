# Modern Java 21 Spring Boot Hexagonal Architecture Code Style Specification

This comprehensive specification synthesizes current industry best practices from authoritative sources including official Spring documentation, leading tech companies' style guides, modern architecture patterns, and enterprise development standards for 2023-2025.

## Executive summary

The specification emphasizes **container-first development, comprehensive observability, and security by design** principles. Key requirements include Java 21 LTS with Spring Boot 3.4+, structured logging with OpenTelemetry integration, TestContainers for database testing, and modern DevOps practices including automated security scanning and performance monitoring.

Modern enterprise teams should adopt **Google Java Style Guide conventions, implement constructor injection patterns, leverage Java 21 virtual threads and pattern matching for enhanced performance, and maintain 85%+ test coverage** with comprehensive testing strategies spanning unit, integration, contract, and security testing. The architecture prioritizes **clear domain boundaries, event-driven integration, and separation of concerns** while remaining practical for real-world enterprise constraints.

## Module structure and dependency management

### Standard multi-module project structure

Modern hexagonal architecture implementation uses **standard Maven/Gradle modules** to enforce architectural boundaries and dependency rules. This approach provides clear separation of concerns without framework dependencies.

**Recommended Maven/Gradle multi-module structure:**
```
my-application/
├── pom.xml (parent)              # Parent POM defining module structure
├── domain/                       # Pure business logic module
│   ├── pom.xml                  # Domain module dependencies
│   └── src/main/java/com/company/product/domain/
│       ├── model/               # Domain entities, value objects, aggregates
│       ├── service/             # Domain services and business rules
├── application/                  # Use case implementations module
│   ├── pom.xml                  # Application module dependencies
│   └── src/main/java/com/company/product/application/
│       ├── usecase/            # Use case implementations
│       └── service/            # Application services (orchestration)
│       └── port/                # ALL port interfaces
│           ├── in/             # Input ports (use case interfaces)
│           └── out/            # Output ports (repository, external service interfaces)
├── infrastructure/              # Technical implementation module
|   └── rest-adapter
|   |   ├── pom.xml 
│   |   └── src/main/java/com/company/product/rest-adapter/ # REST controllers, dtos etc.
|   └── database-adapter
|   |   ├── pom.xml 
│   |   └── src/main/java/com/company/product/database-adapter/ # JPA entities, repositories etc.
|   └── messaging-adapter
|       ├── pom.xml 
│       └── src/main/java/com/company/product/messaging-adapter/ # Producers, consumers etc.
│      
└── bootstrap/                   # Application entry point module
    ├── pom.xml                  # Bootstrap module dependencies (includes all modules)
    └── src/main/java/com/company/product/
        └── Application.java     # @SpringBootApplication main class
```

**Domain module POM (domain/pom.xml):**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0">
    <modelVersion>4.0.0</modelVersion>
    
    <parent>
        <groupId>com.company</groupId>
        <artifactId>product-service</artifactId>
        <version>1.0.0-SNAPSHOT</version>
    </parent>
    
    <artifactId>product-service-domain</artifactId>
    <packaging>jar</packaging>
    
    <dependencies>
        <!-- NO external dependencies - pure Java business logic -->
        <!-- Only minimal dependencies for value objects, validation, etc. -->
        <dependency>
            <groupId>jakarta.validation</groupId>
            <artifactId>jakarta.validation-api</artifactId>
        </dependency>
    </dependencies>
</project>
```

**Application module POM (application/pom.xml):**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0">
    <modelVersion>4.0.0</modelVersion>
    
    <parent>
        <groupId>com.company</groupId>
        <artifactId>product-service</artifactId>
        <version>1.0.0-SNAPSHOT</version>
    </parent>
    
    <artifactId>product-service-application</artifactId>
    <packaging>jar</packaging>
    
    <dependencies>
        <!-- Domain dependency -->
        <dependency>
            <groupId>com.company</groupId>
            <artifactId>product-service-domain</artifactId>
        </dependency>
        
        <!-- Spring context for DI (minimal framework usage) -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
        </dependency>
    </dependencies>
</project>
```

**Infrastructure module POM (infrastructure/pom.xml):**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0">
    <modelVersion>4.0.0</modelVersion>
    
    <parent>
        <groupId>com.company</groupId>
        <artifactId>product-service</artifactId>
        <version>1.0.0-SNAPSHOT</version>
    </parent>
    
    <artifactId>product-service-infrastructure</artifactId>
    <packaging>jar</packaging>
    
    <dependencies>
        <!-- Application dependency -->
        <dependency>
            <groupId>com.company</groupId>
            <artifactId>product-service-application</artifactId>
        </dependency>
        
        <!-- Spring Boot starters -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-logging</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-log4j2</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        
        <!-- JSON Template Layout for Log4j2 -->
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-layout-template-json</artifactId>
        </dependency>
        
        <!-- Database driver -->
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>
    </dependencies>
</project>
```

**Bootstrap module POM (bootstrap/pom.xml):**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0">
    <modelVersion>4.0.0</modelVersion>
    
    <parent>
        <groupId>com.company</groupId>
        <artifactId>product-service</artifactId>
        <version>1.0.0-SNAPSHOT</version>
    </parent>
    
    <artifactId>product-service-bootstrap</artifactId>
    <packaging>jar</packaging>
    
    <dependencies>
        <!-- All module dependencies -->
        <dependency>
            <groupId>com.company</groupId>
            <artifactId>product-service-application</artifactId>
        </dependency>
        <dependency>
            <groupId>com.company</groupId>
            <artifactId>product-service-infrastructure</artifactId>
        </dependency>
    </dependencies>
    
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <mainClass>com.company.product.Application</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

**❌ INCORRECT: Ports in both domain and application layers**
```
├── domain/
│   └── port/              # ❌ Ports here
```

**✅ CORRECT: Ports only in domain layer**
```
├── domain/                # The Hexagon - Business Logic Core
│   ├── model/             # Domain entities, value objects
│   ├── service/           # Domain services
├── application/           # Use Case Implementations
│   └── usecase/           # ✅ Implements input ports from domain
│   └── port/              # ✅ ALL ports defined here only
│       ├── in/            # Input ports (use case interfaces)
│       └── out/           # Output ports (repository, external service interfaces)
└── infrastructure/        # Adapters
    └── adapter/           # ✅ Implements output ports from domain
```

**Why this matters (from authoritative sources):**

1. **Dependency Inversion Principle**: The essence behind Hexagonal Architecture is the Inversion of Control. The Business Layer does NOT depend on the Database Layer implementation. The application exposes both Driver Ports & Driven Ports.

2. **Technology Agnostic Core**: The hexagon contains the business logic, with no references to any technology, framework or real world device. So the application is technology agnostic.

3. **Clear Separation**: Ports define the interfaces with which adapters interact with the Core. Both are essential to manage changes in the external world without affecting the business logic.

**Dependency flow in correct hexagonal architecture:**
```
Infrastructure Layer
     ↓ implements
Application Layer (Input and Output ports)
     ↑ defined in
Domain Layer (business logic)
```

**Practical example of correct dependency flow:**
```java
// 1. Domain Layer - Port Definition (Interface)
package com.company.application.port.out;
public interface OrderRepository {
    Order save(Order order);
    Optional<Order> findById(OrderId id);
}

// 2. Application Layer - Uses Domain Port
package com.company.application.usecase;

import com.company.application.port.out;

public class PlaceOrderUseCaseImpl implements PlaceOrderUseCase {
    private final OrderRepository orderRepository; // ← Uses domain port
    
    public OrderResult placeOrder(PlaceOrderCommand command) {
        Order order = // business logic
        return orderRepository.save(order); // ← Calls through port interface
    }
}

// 3. Infrastructure Layer - Implements Domain Port
package com.company.infrastructure.adapter.out.persistence;

import com.company.application.port.out;

@Repository
public class JpaOrderRepository implements OrderRepository { // ← Implements domain port
    // JPA-specific implementation
}
```

This ensures that:
- **Domain** defines what it needs (ports) but doesn't know how it's implemented
- **Application** uses domain ports but doesn't know about specific implementations  
- **Infrastructure** provides concrete implementations of domain ports
- **Dependencies point inward** toward the domain core (Dependency Inversion Principle)

**Recommended multi-module structure:**
```
project-root/
├── domain/                 # Pure domain logic (no Spring dependencies)
│   ├── model/             # Domain entities and value objects
│   ├── service/           # Domain services and business logic
├── application/           # Application layer
│   ├── usecase/          # Use case implementations
│   ├── port/             # Port definitions
│   └── service/          # Application services
├── infrastructure/        # External concerns (Spring Boot allowed here)
│   ├── adapter/
│   │   ├── web/          # REST controllers (input adapters)
│   │   ├── persistence/  # Database adapters (output adapters)
│   │   ├── messaging/    # Event/message adapters
│   │   └── external/     # Third-party service adapters
│   └── config/           # Spring configuration
└── boot/                 # Spring Boot application entry point
```

**Package organization reflecting hexagonal principles:**
```
com.company.product/
├── Application.java                    # @SpringBootApplication
├── domain/                             # The Hexagon (Business Logic)
│   ├── model/
│   │   ├── Order.java                  # Domain entity
│   │   ├── OrderId.java               # Value object
│   │   └── OrderStatus.java           # Domain enum
│   ├── service/
│   │   └── OrderDomainService.java    # Domain service
├── application/                        # Use Case Implementations
│   └── usecase/
│       └── PlaceOrderUseCaseImpl.java  # Implements input port
│   └── port/
│       ├── in/
│       │   └── PlaceOrderUseCase.java  # Input port interface
│       └── out/
│           ├── OrderRepository.java    # Output port interface
│           └── PaymentService.java     # Output port interface
└── infrastructure/                     # Adapters
    ├── adapter/
    │   ├── in/
    │   │   └── web/
    │   │       └── OrderController.java # Primary adapter
    │   └── out/
    │       ├── persistence/
    │       │   └── JpaOrderRepository.java # Secondary adapter
    │       └── external/
    │           └── ExternalPaymentService.java # Secondary adapter
    └── configuration/
        └── ApplicationConfiguration.java
```

### Dependency management best practices

**Gradle multi-module configuration (build.gradle.kts):**
```kotlin
// Root build.gradle.kts
plugins {
    java
    id("org.springframework.boot") version "3.4.2" apply false
    id("io.spring.dependency-management") version "1.1.7" apply false
}

allprojects {
    group = "com.company"
    version = "1.0.0-SNAPSHOT"
    
    repositories {
        mavenCentral()
    }
}

subprojects {
    apply(plugin = "java")
    
    java {
        toolchain {
            languageVersion = JavaLanguageVersion.of(21)
        }
    }
    
    // Enable preview features for Java 21 (optional)
    tasks.withType<JavaCompile> {
        options.compilerArgs.addAll(listOf("--enable-preview"))
    }
    
    tasks.withType<Test> {
        jvmArgs("--enable-preview")
        useJUnitPlatform()
    }
}

// Domain module (domain/build.gradle.kts)
dependencies {
    // NO external dependencies - pure Java business logic
    implementation("jakarta.validation:jakarta.validation-api")
    
    // Testing
    testImplementation("org.junit.jupiter:junit-jupiter")
    testImplementation("org.assertj:assertj-core")
}

// Application module (application/build.gradle.kts)
plugins {
    id("io.spring.dependency-management")
}

dependencies {
    // Minimal Spring for DI
    implementation("org.springframework:spring-context")
    
    // Testing
    testImplementation("org.springframework.boot:spring-boot-starter-test") {
        exclude(group = "org.springframework.boot", module = "spring-boot-starter-logging")
    }
    testImplementation("org.springframework.boot:spring-boot-starter-log4j2")
}

// Infrastructure module (infrastructure/build.gradle.kts)
plugins {
    id("org.springframework.boot") apply false
    id("io.spring.dependency-management")
}

dependencies {
    // Spring Boot starters (excluding default logging)
    implementation("org.springframework.boot:spring-boot-starter-web") {
        exclude(group = "org.springframework.boot", module = "spring-boot-starter-logging")
    }
    implementation("org.springframework.boot:spring-boot-starter-log4j2")
    implementation("org.springframework.boot:spring-boot-starter-data-jpa")
    implementation("org.springframework.boot:spring-boot-starter-actuator")
    
    // JSON Template Layout for Log4j2
    implementation("org.apache.logging.log4j:log4j-layout-template-json")
    
    // Observability
    implementation("io.micrometer:micrometer-registry-prometheus")
    implementation("io.micrometer:micrometer-tracing-bridge-otel")
    
    // Database
    runtimeOnly("org.postgresql:postgresql")
    
    // Testing with TestContainers
    testImplementation("org.springframework.boot:spring-boot-starter-test") {
        exclude(group = "org.springframework.boot", module = "spring-boot-starter-logging")
    }
    testImplementation("org.springframework.boot:spring-boot-starter-log4j2")
    testImplementation("org.testcontainers:junit-jupiter")
    testImplementation("org.testcontainers:postgresql")
}

// Bootstrap module (bootstrap/build.gradle.kts)
plugins {
    id("org.springframework.boot")
    id("io.spring.dependency-management")
}

tasks.bootJar {
    archiveFileName.set("product-service.jar")
}
```

**Constructor injection patterns (mandatory for all services):**

Constructor injection is the only acceptable dependency injection pattern for Spring Boot applications. This approach ensures immutability, testability, and clear dependency declaration.

```java
@Service
public class OrderService {
    
    private final OrderRepository repository;
    private final PaymentService paymentService;
    
    /**
     * Constructor injection - preferred approach.
     * All dependencies are declared as final fields and injected through constructor.
     */
    public OrderService(OrderRepository repository, PaymentService paymentService) {
        this.repository = repository;
        this.paymentService = paymentService;
    }
    
    public Order createOrder(CreateOrderCommand command) {
        // Implementation using injected dependencies
        return repository.save(order);
    }
}
```

**Alternative approach using Lombok:**
```java
@Service
@RequiredArgsConstructor
public class UserService {
    
    private final UserRepository userRepository;
    private final EmailService emailService;
    
    // Constructor automatically generated by Lombok @RequiredArgsConstructor
    // Creates constructor for all final fields
    
    public User createUser(CreateUserCommand command) {
        // Implementation
        return userRepository.save(user);
    }
}
```

**Key principles:**
- All dependencies must be declared as `private final` fields
- Constructor parameters must match final field names
- No `@Autowired` annotation required (Spring Boot default behavior)
- Use `@RequiredArgsConstructor` for cleaner code when using Lombok
- Avoid field injection (`@Autowired` on fields) and setter injection

**Module boundary enforcement with Maven/Gradle:**
```java
// Bootstrap module - Application entry point
package com.company.product;

@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

// Configuration class to wire modules together
@Configuration
@ComponentScan(basePackages = {
    "com.company.product.application",
    "com.company.product.infrastructure"
})
@EnableJpaRepositories(basePackages = "com.company.product.infrastructure.persistence")
public class ApplicationConfiguration {
    // Bean configurations if needed
}
```

**Benefits of standard multi-module structure:**
- **Clear Dependency Management**: Maven/Gradle enforces module dependencies at compile time
- **Framework Agnostic**: No dependency on Spring Modulith or other frameworks
- **IDE Support**: Excellent IDE support for module navigation and refactoring
- **Build Isolation**: Each module can be built and tested independently
- **Deployment Flexibility**: Modules can be deployed as separate artifacts if needed
- **Team Boundaries**: Different teams can own different modules
- **Dependency Enforcement**: Impossible to accidentally create circular dependencies

## Modern naming conventions

### Google Java Style Guide compliance (mandatory)

**Class naming standards:**
- **Classes**: UpperCamelCase (e.g., `ImmutableList`, `OrderProcessingService`)
- **Methods**: lowerCamelCase (e.g., `sendMessage`, `processPayment`)
- **Variables**: lowerCamelCase (e.g., `computedValues`, `orderItems`)
- **Constants**: UPPER_SNAKE_CASE (e.g., `MAX_RETRY_ATTEMPTS`, `DEFAULT_TIMEOUT`)
- **Packages**: lowercase with dots (e.g., `com.company.product.order`)

**Spring-specific naming conventions:**
- **Configuration Classes**: End with "Configuration"
- **Test Classes**: End with "Test"  
- **Stereotype Annotations**: Use `@Service`, `@Repository`, `@Controller`, `@Component` appropriately
- **Use Case Classes**: End with "UseCase" (e.g., `PlaceOrderUseCase`)
- **Port Interfaces**: End without "Port" (e.g., `LoadAccount`, `UpdateAccountState`)

**REST API resource naming:**
- **URIs**: Plural nouns, lowercase with hyphens (e.g., `/user-profiles`, `/order-items`)
- **Path Structure**: Hierarchical (e.g., `/users/{userId}/orders/{orderId}`)
- **No Verbs**: Actions expressed through HTTP methods, not URI paths

## Package organization standards

### Hexagonal architecture package principles

**API Package Strategy**: Direct sub-packages of main package are considered public APIs, while `.internal` sub-packages are inaccessible to other modules.

**Feature-based organization over technical layers:**
```
com.company.product/
├── domain/
│   ├── account/
│   │   ├── Account.java           # Aggregate root
│   │   ├── AccountId.java         # Value object
│   │   └── Activity.java          # Domain entity
│   ├── money/
│   │   └── Money.java             # Value object
│   └── shared/
│       └── DomainEvents.java      # Domain event infrastructure
├── application/
│   ├── port/
│   │   ├── in/
│   │   │   └── SendMoneyUseCase.java    # Input port
│   │   └── out/
│   │       ├── LoadAccountPort.java     # Output port
│   │       └── UpdateAccountStatePort.java
│   └── service/
│       └── SendMoneyService.java        # Use case implementation
└── infrastructure/
    ├── adapter/
    │   ├── web/
    │   │   └── SendMoneyController.java # Primary adapter
    │   └── persistence/
    │       ├── AccountPersistenceAdapter.java # Secondary adapter
    │       ├── AccountJpaEntity.java
    │       └── AccountRepository.java
    └── config/
        └── ApplicationConfiguration.java
```

**Module boundary enforcement principles:**
- Each module represents a bounded context with clear interfaces
- Modules interact primarily through application events
- Internal packages (`.internal`) are inaccessible to other modules
- Cross-module dependencies only through published APIs

## Coding standards and formatting

### Code structure requirements

**Column limit**: 100 characters (Google Java Style Guide 2025)
**Indentation**: 2 spaces for block indentation, +4 spaces for continuation lines
**File encoding**: UTF-8 mandatory
**Braces**: K&R style (Egyptian brackets) for non-empty blocks

**Import organization standards:**
```java
// Static imports first
import static org.assertj.core.api.Assertions.assertThat;
import static org.mockito.BDDMockito.given;

// Non-static imports, ASCII sorted
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.stereotype.Service;

import java.math.BigDecimal;
import java.time.LocalDateTime;
```

**Method organization:**
- Logical grouping over chronological ordering
- Overloaded methods grouped together
- Public methods before private methods
- Constructor injection before setter methods

### Java 17+ modern features adoption

**Records for immutable data carriers:**
```java
// Configuration properties as records
@ConfigurationProperties("app.database")
public record DatabaseProperties(
    String url,
    String username,
    String password,
    @DurationUnit(ChronoUnit.SECONDS) Duration timeout
) {}

// DTOs as records
public record OrderResponse(
    Long id,
    String status,
    LocalDateTime createdAt,
    List<OrderItem> items
) {}
```

**Sealed classes for domain modeling:**
```java
public sealed interface PaymentResult 
    permits PaymentSuccess, PaymentFailure, PaymentPending {
}

public record PaymentSuccess(String transactionId) implements PaymentResult {}
public record PaymentFailure(String errorCode, String message) implements PaymentResult {}
public record PaymentPending(String referenceId) implements PaymentResult {}
```

**Text blocks for multi-line strings:**
```java
String sql = """
    SELECT o.id, o.status, o.created_at
    FROM orders o
    WHERE o.customer_id = ?
    ORDER BY o.created_at DESC
    """;
```

## Documentation and commenting best practices

### Javadoc requirements

**Mandatory Javadoc for:**
- All public classes and interfaces
- All public and protected methods
- All public fields and constants
- Package-level documentation in `package-info.java`

**Javadoc format standards:**
```java
/**
 * Represents a money amount with arithmetic operations.
 * 
 * <p>This class provides immutable money calculations with proper
 * precision handling for financial operations.
 * 
 * @author Development Team
 * @since 1.0
 */
public class Money {
    
    /**
     * Creates a new Money instance from the specified amount.
     * 
     * @param amount the monetary amount, must not be null
     * @return a new Money instance
     * @throws IllegalArgumentException if amount is null
     */
    public static Money of(BigDecimal amount) {
        // Implementation
    }
}
```

**Block tag ordering:** `@param`, `@return`, `@throws`, `@deprecated`, `@since`, `@see`

### Code commenting guidelines

**When to comment:**
- Complex business logic and domain rules
- Non-obvious algorithmic decisions
- Workarounds for external system limitations
- Security-sensitive code sections

**Comment style:**
```java
// Business rule: Orders over $1000 require additional verification
if (order.getTotal().compareTo(VERIFICATION_THRESHOLD) >= 0) {
    // Delegate to fraud detection service
    fraudDetectionService.verifyOrder(order);
}
```

**Package documentation example:**
```java
/**
 * Order management domain model and business logic.
 * 
 * <p>This package contains the core domain entities, value objects,
 * and business rules for order processing in the e-commerce system.
 * 
 * <h2>Key Concepts</h2>
 * <ul>
 *   <li>{@link Order} - Aggregate root for order management</li>
 *   <li>{@link OrderItem} - Individual items within an order</li>
 *   <li>{@link OrderStatus} - Order lifecycle states</li>
 * </ul>
 */
package com.company.product.order.domain;
```

## Error handling and exception management

### Exception hierarchy design

**Custom exception structure:**

```java
/**
 * Base class for all business-related exceptions in the domain layer.
 * Provides consistent error handling across the application.
 */
public abstract class BusinessException extends RuntimeException {
    
    private final String errorCode;
    private final Map<String, Object> context;
    
    protected BusinessException(String errorCode, String message) {
        this(errorCode, message, Map.of());
    }
    
    protected BusinessException(String errorCode, String message, Map<String, Object> context) {
        super(message);
        this.errorCode = errorCode;
        this.context = Map.copyOf(context);
    }
    
    protected BusinessException(String errorCode, String message, Throwable cause) {
        this(errorCode, message, Map.of(), cause);
    }
    
    protected BusinessException(String errorCode, String message, Map<String, Object> context, Throwable cause) {
        super(message, cause);
        this.errorCode = errorCode;
        this.context = Map.copyOf(context);
    }
    
    public String getErrorCode() {
        return errorCode;
    }
    
    public Map<String, Object> getContext() {
        return context;
    }
}
```

```java
/**
 * Exception thrown when a requested order cannot be found.
 */
public class OrderNotFoundException extends BusinessException {
    
    public OrderNotFoundException(Long orderId) {
        super("ORDER_NOT_FOUND", 
              "Order not found with ID: " + orderId,
              Map.of("orderId", orderId));
    }
    
    public OrderNotFoundException(String orderNumber) {
        super("ORDER_NOT_FOUND", 
              "Order not found with number: " + orderNumber,
              Map.of("orderNumber", orderNumber));
    }
}
```

```java
/**
 * Exception thrown when business rules are violated during order processing.
 */
public class InsufficientInventoryException extends BusinessException {
    
    public InsufficientInventoryException(String productId, int requestedQuantity, int availableQuantity) {
        super("INSUFFICIENT_INVENTORY", 
              String.format("Insufficient inventory for product %s. Requested: %d, Available: %d", 
                           productId, requestedQuantity, availableQuantity),
              Map.of(
                  "productId", productId,
                  "requestedQuantity", requestedQuantity,
                  "availableQuantity", availableQuantity
              ));
    }
}
```

```java
/**
 * Exception thrown when payment processing fails.
 */
public class PaymentProcessingException extends BusinessException {
    
    public PaymentProcessingException(String message) {
        super("PAYMENT_FAILED", message);
    }
    
    public PaymentProcessingException(String message, String transactionId) {
        super("PAYMENT_FAILED", 
              message,
              Map.of("transactionId", transactionId));
    }
    
    public PaymentProcessingException(String message, Throwable cause) {
        super("PAYMENT_FAILED", message, cause);
    }
}
```

### Global exception handling

**REST API error handling with @ControllerAdvice:**
```java
@ControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(BusinessException.class)
    public ResponseEntity<ErrorResponse> handleBusinessException(BusinessException ex) {
        ErrorResponse error = ErrorResponse.builder()
            .timestamp(Instant.now())
            .errorCode(ex.getErrorCode())
            .message(ex.getMessage())
            .path(getCurrentPath())
            .build();
            
        return ResponseEntity.badRequest().body(error);
    }
    
    @ExceptionHandler(ValidationException.class)
    public ResponseEntity<ValidationErrorResponse> handleValidation(ValidationException ex) {
        ValidationErrorResponse response = ValidationErrorResponse.builder()
            .timestamp(Instant.now())
            .message("Validation failed")
            .violations(ex.getViolations())
            .build();
            
        return ResponseEntity.badRequest().body(response);
    }
}
```

**Standardized error response format:**
```java
public record ErrorResponse(
    Instant timestamp,
    String errorCode,
    String message,
    String path,
    String traceId
) {}
```

### Exception handling patterns

**Use case error handling:**
```java
@Service
@Transactional
public class PlaceOrderService implements PlaceOrderUseCase {
    
    @Override
    public OrderResult placeOrder(PlaceOrderCommand command) {
        try {
            // Business logic with proper error handling
            Order order = createOrder(command);
            PaymentResult payment = processPayment(order);
            
            if (payment instanceof PaymentFailure failure) {
                throw new PaymentProcessingException(failure.errorCode(), failure.message());
            }
            
            return OrderResult.success(order.getId());
            
        } catch (BusinessException ex) {
            // Log business exceptions appropriately
            log.warn("Business rule violation during order placement: {}", ex.getMessage());
            throw ex;
        } catch (Exception ex) {
            // Handle unexpected exceptions
            log.error("Unexpected error during order placement", ex);
            throw new OrderProcessingException("INTERNAL_ERROR", "Failed to process order");
        }
    }
}
```

## Testing conventions and strategies

### Comprehensive testing pyramid

**Unit testing for domain layer (90%+ coverage required):**

```java
class OrderTest {
    
    private static final CustomerId CUSTOMER_ID = new CustomerId(1L);
    private static final ProductId PRODUCT_ID = new ProductId("PROD-001");
    private static final Money UNIT_PRICE = Money.of(new BigDecimal("29.99"));
    private static final int QUANTITY = 2;
    
    @Test
    @DisplayName("Should calculate total correctly when items are added")
    void should_calculate_total_correctly_when_items_added() {
        // Given
        Order order = new Order(CUSTOMER_ID);
        OrderItem item = new OrderItem(PRODUCT_ID, QUANTITY, UNIT_PRICE);
        
        // When
        order.addItem(item);
        
        // Then
        Money expectedTotal = UNIT_PRICE.multiply(QUANTITY);
        assertThat(order.getTotal()).isEqualTo(expectedTotal);
        assertThat(order.getItems()).hasSize(1);
        assertThat(order.getStatus()).isEqualTo(OrderStatus.DRAFT);
    }
    
    @Test
    @DisplayName("Should throw exception when adding item to confirmed order")
    void should_throw_exception_when_adding_item_to_confirmed_order() {
        // Given
        Order order = new Order(CUSTOMER_ID);
        order.addItem(new OrderItem(PRODUCT_ID, QUANTITY, UNIT_PRICE));
        order.confirm();
        
        // When & Then
        OrderItem newItem = new OrderItem(new ProductId("PROD-002"), 1, UNIT_PRICE);
        
        assertThatThrownBy(() -> order.addItem(newItem))
            .isInstanceOf(IllegalStateException.class)
            .hasMessage("Cannot add items to confirmed order");
    }
    
    @Test
    @DisplayName("Should apply discount correctly")
    void should_apply_discount_correctly() {
        // Given
        Order order = new Order(CUSTOMER_ID);
        order.addItem(new OrderItem(PRODUCT_ID, QUANTITY, Money.of(new BigDecimal("100.00"))));
        
        // When
        Discount discount = new Discount(DiscountType.PERCENTAGE, new BigDecimal("10"));
        order.applyDiscount(discount);
        
        // Then
        assertThat(order.getSubtotal()).isEqualTo(Money.of(new BigDecimal("200.00")));
        assertThat(order.getDiscountAmount()).isEqualTo(Money.of(new BigDecimal("20.00")));
        assertThat(order.getTotal()).isEqualTo(Money.of(new BigDecimal("180.00")));
    }
    
    @Test
    @DisplayName("Should validate minimum order amount")
    void should_validate_minimum_order_amount() {
        // Given
        Order order = new Order(CUSTOMER_ID);
        OrderItem lowValueItem = new OrderItem(PRODUCT_ID, 1, Money.of(new BigDecimal("5.00")));
        order.addItem(lowValueItem);
        
        // When & Then
        assertThatThrownBy(() -> order.confirm())
            .isInstanceOf(OrderValidationException.class)
            .hasMessage("Order total must be at least $10.00");
    }
}
```

**Use case testing with mocked ports:**

```java
@ExtendWith(MockitoExtension.class)
class PlaceOrderUseCaseTest {
    
    @Mock 
    private OrderRepository orderRepository;
    @Mock 
    private PaymentService paymentService;
    @Mock 
    private InventoryService inventoryService;
    @Mock 
    private EmailService emailService;
    
    @InjectMocks 
    private PlaceOrderUseCaseImpl useCase;
    
    private CustomerId customerId;
    private PlaceOrderCommand validCommand;
    private Order savedOrder;
    
    @BeforeEach
    void setUp() {
        customerId = new CustomerId(1L);
        validCommand = PlaceOrderCommand.builder()
            .customerId(customerId)
            .items(List.of(
                OrderItemCommand.builder()
                    .productId("PROD-001")
                    .quantity(2)
                    .unitPrice(new BigDecimal("29.99"))
                    .build()
            ))
            .build();
        
        savedOrder = Order.builder()
            .id(new OrderId(100L))
            .customerId(customerId)
            .status(OrderStatus.CONFIRMED)
            .build();
    }
    
    @Test
    @DisplayName("Should place order successfully when all conditions are met")
    void should_place_order_successfully_when_all_conditions_met() {
        // Given
        when(inventoryService.checkAvailability("PROD-001", 2))
            .thenReturn(InventoryCheckResult.available());
        when(paymentService.processPayment(any(PaymentRequest.class)))
            .thenReturn(PaymentResult.success("TXN-123"));
        when(orderRepository.save(any(Order.class)))
            .thenReturn(savedOrder);
        
        // When
        OrderResult result = useCase.placeOrder(validCommand);
        
        // Then
        assertThat(result.isSuccess()).isTrue();
        assertThat(result.getOrderId()).isEqualTo(savedOrder.getId());
        
        verify(inventoryService).checkAvailability("PROD-001", 2);
        verify(paymentService).processPayment(any(PaymentRequest.class));
        verify(orderRepository).save(any(Order.class));
        verify(emailService).sendOrderConfirmation(eq(customerId), eq(savedOrder.getId()));
    }
    
    @Test
    @DisplayName("Should fail when insufficient inventory")
    void should_fail_when_insufficient_inventory() {
        // Given
        when(inventoryService.checkAvailability("PROD-001", 2))
            .thenReturn(InventoryCheckResult.insufficient(1));
        
        // When & Then
        assertThatThrownBy(() -> useCase.placeOrder(validCommand))
            .isInstanceOf(InsufficientInventoryException.class)
            .hasMessage("Insufficient inventory for product PROD-001. Requested: 2, Available: 1");
        
        verify(inventoryService).checkAvailability("PROD-001", 2);
        verifyNoInteractions(paymentService, orderRepository, emailService);
    }
    
    @Test
    @DisplayName("Should fail when payment processing fails")
    void should_fail_when_payment_processing_fails() {
        // Given
        when(inventoryService.checkAvailability("PROD-001", 2))
            .thenReturn(InventoryCheckResult.available());
        when(paymentService.processPayment(any(PaymentRequest.class)))
            .thenReturn(PaymentResult.failure("PAYMENT_DECLINED", "Card declined"));
        
        // When & Then
        assertThatThrownBy(() -> useCase.placeOrder(validCommand))
            .isInstanceOf(PaymentProcessingException.class)
            .hasMessage("Payment failed: Card declined");
        
        verify(inventoryService).checkAvailability("PROD-001", 2);
        verify(paymentService).processPayment(any(PaymentRequest.class));
        verifyNoInteractions(orderRepository, emailService);
    }
    
    @Test
    @DisplayName("Should handle empty order items")
    void should_handle_empty_order_items() {
        // Given
        PlaceOrderCommand emptyCommand = PlaceOrderCommand.builder()
            .customerId(customerId)
            .items(List.of())
            .build();
        
        // When & Then
        assertThatThrownBy(() -> useCase.placeOrder(emptyCommand))
            .isInstanceOf(EmptyOrderException.class)
            .hasMessage("Order must contain at least one item");
        
        verifyNoInteractions(inventoryService, paymentService, orderRepository, emailService);
    }
}
```

### Integration testing with TestContainers

**Database integration testing:**

```java
@SpringBootTest
@Testcontainers
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
class OrderRepositoryIntegrationTest {
    
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15-alpine")
            .withDatabaseName("testdb")
            .withUsername("test")
            .withPassword("test")
            .withInitScript("db/test-schema.sql");
            
    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
        registry.add("spring.datasource.driver-class-name", postgres::getDriverClassName);
    }
    
    @Autowired
    private OrderRepository orderRepository;
    
    @Autowired
    private TestEntityManager entityManager;
    
    private CustomerId customerId;
    private Order testOrder;
    
    @BeforeEach
    void setUp() {
        customerId = new CustomerId(1L);
        testOrder = Order.builder()
            .customerId(customerId)
            .status(OrderStatus.DRAFT)
            .createdAt(Instant.now())
            .items(List.of(
                OrderItem.builder()
                    .productId(new ProductId("PROD-001"))
                    .quantity(2)
                    .unitPrice(Money.of(new BigDecimal("29.99")))
                    .build()
            ))
            .build();
    }
    
    @Test
    @DisplayName("Should persist and retrieve order successfully")
    void should_persist_and_retrieve_order_successfully() {
        // Given
        assertThat(testOrder.getId()).isNull();
        
        // When
        Order savedOrder = orderRepository.save(testOrder);
        entityManager.flush();
        entityManager.clear();
        
        // Then
        assertThat(savedOrder.getId()).isNotNull();
        
        Optional<Order> foundOrder = orderRepository.findById(savedOrder.getId());
        assertThat(foundOrder).isPresent();
        
        Order retrievedOrder = foundOrder.get();
        assertThat(retrievedOrder.getCustomerId()).isEqualTo(customerId);
        assertThat(retrievedOrder.getStatus()).isEqualTo(OrderStatus.DRAFT);
        assertThat(retrievedOrder.getItems()).hasSize(1);
        assertThat(retrievedOrder.getTotal()).isEqualTo(Money.of(new BigDecimal("59.98")));
    }
    
    @Test
    @DisplayName("Should find orders by customer ID")
    void should_find_orders_by_customer_id() {
        // Given
        Order order1 = orderRepository.save(testOrder);
        Order order2 = orderRepository.save(createAnotherOrder(customerId));
        Order orderFromDifferentCustomer = orderRepository.save(
            createOrderForCustomer(new CustomerId(2L))
        );
        
        entityManager.flush();
        entityManager.clear();
        
        // When
        List<Order> customerOrders = orderRepository.findByCustomerId(customerId);
        
        // Then
        assertThat(customerOrders).hasSize(2);
        assertThat(customerOrders)
            .extracting(Order::getId)
            .containsExactlyInAnyOrder(order1.getId(), order2.getId());
        assertThat(customerOrders)
            .extracting(Order::getCustomerId)
            .allMatch(id -> id.equals(customerId));
    }
    
    @Test
    @DisplayName("Should update order status")
    void should_update_order_status() {
        // Given
        Order savedOrder = orderRepository.save(testOrder);
        entityManager.flush();
        entityManager.clear();
        
        // When
        savedOrder.confirm();
        Order updatedOrder = orderRepository.save(savedOrder);
        entityManager.flush();
        entityManager.clear();
        
        // Then
        Optional<Order> foundOrder = orderRepository.findById(updatedOrder.getId());
        assertThat(foundOrder).isPresent();
        assertThat(foundOrder.get().getStatus()).isEqualTo(OrderStatus.CONFIRMED);
    }
    
    @Test
    @DisplayName("Should handle concurrent modifications optimistically")
    @Transactional
    void should_handle_concurrent_modifications_optimistically() {
        // Given
        Order savedOrder = orderRepository.save(testOrder);
        entityManager.flush();
        
        // When - Simulate concurrent modification
        Order order1 = orderRepository.findById(savedOrder.getId()).orElseThrow();
        Order order2 = orderRepository.findById(savedOrder.getId()).orElseThrow();
        
        order1.confirm();
        orderRepository.save(order1);
        entityManager.flush();
        
        order2.cancel();
        
        // Then
        assertThatThrownBy(() -> {
            orderRepository.save(order2);
            entityManager.flush();
        }).isInstanceOf(OptimisticLockingFailureException.class);
    }
    
    private Order createAnotherOrder(CustomerId customerId) {
        return Order.builder()
            .customerId(customerId)
            .status(OrderStatus.DRAFT)
            .createdAt(Instant.now().plus(1, ChronoUnit.HOURS))
            .items(List.of(
                OrderItem.builder()
                    .productId(new ProductId("PROD-002"))
                    .quantity(1)
                    .unitPrice(Money.of(new BigDecimal("49.99")))
                    .build()
            ))
            .build();
    }
    
    private Order createOrderForCustomer(CustomerId customerId) {
        return Order.builder()
            .customerId(customerId)
            .status(OrderStatus.CONFIRMED)
            .createdAt(Instant.now().plus(2, ChronoUnit.HOURS))
            .items(List.of(
                OrderItem.builder()
                    .productId(new ProductId("PROD-003"))
                    .quantity(3)
                    .unitPrice(Money.of(new BigDecimal("19.99")))
                    .build()
            ))
            .build();
    }
}
```

### Spring Boot test slices

**Web layer testing:**

```java
@WebMvcTest(OrderController.class)
@Import(GlobalExceptionHandler.class)
class OrderControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private PlaceOrderUseCase placeOrderUseCase;
    
    @MockBean
    private GetOrderUseCase getOrderUseCase;
    
    @Autowired
    private ObjectMapper objectMapper;
    
    private CreateOrderRequest validRequest;
    private OrderResult successResult;
    
    @BeforeEach
    void setUp() {
        validRequest = CreateOrderRequest.builder()
            .customerId(1L)
            .items(List.of(
                CreateOrderItemRequest.builder()
                    .productId("PROD-001")
                    .quantity(2)
                    .unitPrice(new BigDecimal("29.99"))
                    .build()
            ))
            .build();
        
        successResult = OrderResult.success(new OrderId(100L));
    }
    
    @Test
    @DisplayName("Should place order successfully via API")
    void should_place_order_successfully_via_api() throws Exception {
        // Given
        when(placeOrderUseCase.placeOrder(any(PlaceOrderCommand.class)))
            .thenReturn(successResult);
        
        // When & Then
        mockMvc.perform(post("/api/v1/orders")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(validRequest)))
                .andExpect(status().isCreated())
                .andExpect(jsonPath("$.orderId").value(100L))
                .andExpect(jsonPath("$.status").value("SUCCESS"))
                .andExpect(header().string("Location", "/api/v1/orders/100"));
        
        verify(placeOrderUseCase).placeOrder(argThat(command -> 
            command.customerId().value().equals(1L) &&
            command.items().size() == 1
        ));
    }
    
    @Test
    @DisplayName("Should return validation errors for invalid request")
    void should_return_validation_errors_for_invalid_request() throws Exception {
        // Given
        CreateOrderRequest invalidRequest = CreateOrderRequest.builder()
            .customerId(null) // Invalid - required field
            .items(List.of()) // Invalid - empty list
            .build();
        
        // When & Then
        mockMvc.perform(post("/api/v1/orders")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(invalidRequest)))
                .andExpect(status().isBadRequest())
                .andExpect(jsonPath("$.error").value("Validation Failed"))
                .andExpect(jsonPath("$.fieldErrors.customerId").exists())
                .andExpect(jsonPath("$.fieldErrors.items").exists())
                .andExpect(jsonPath("$.timestamp").exists())
                .andExpect(jsonPath("$.path").value("/api/v1/orders"));
        
        verifyNoInteractions(placeOrderUseCase);
    }
    
    @Test
    @DisplayName("Should handle business exceptions appropriately")
    void should_handle_business_exceptions_appropriately() throws Exception {
        // Given
        when(placeOrderUseCase.placeOrder(any(PlaceOrderCommand.class)))
            .thenThrow(new InsufficientInventoryException("PROD-001", 2, 1));
        
        // When & Then
        mockMvc.perform(post("/api/v1/orders")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(validRequest)))
                .andExpect(status().isBadRequest())
                .andExpect(jsonPath("$.errorCode").value("INSUFFICIENT_INVENTORY"))
                .andExpect(jsonPath("$.message").value("Insufficient inventory for product PROD-001. Requested: 2, Available: 1"))
                .andExpect(jsonPath("$.context.productId").value("PROD-001"))
                .andExpect(jsonPath("$.context.requestedQuantity").value(2))
                .andExpect(jsonPath("$.context.availableQuantity").value(1));
    }
    
    @Test
    @DisplayName("Should retrieve order by ID")
    void should_retrieve_order_by_id() throws Exception {
        // Given
        Order order = Order.builder()
            .id(new OrderId(100L))
            .customerId(new CustomerId(1L))
            .status(OrderStatus.CONFIRMED)
            .total(Money.of(new BigDecimal("59.98")))
            .build();
        
        when(getOrderUseCase.getOrder(new GetOrderQuery(new OrderId(100L))))
            .thenReturn(order);
        
        // When & Then
        mockMvc.perform(get("/api/v1/orders/100")
                .accept(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andExpected(jsonPath("$.id").value(100L))
                .andExpect(jsonPath("$.customerId").value(1L))
                .andExpect(jsonPath("$.status").value("CONFIRMED"))
                .andExpect(jsonPath("$.total").value("59.98"));
    }
    
    @Test
    @DisplayName("Should return 404 when order not found")
    void should_return_404_when_order_not_found() throws Exception {
        // Given
        when(getOrderUseCase.getOrder(any(GetOrderQuery.class)))
            .thenThrow(new OrderNotFoundException(999L));
        
        // When & Then
        mockMvc.perform(get("/api/v1/orders/999")
                .accept(MediaType.APPLICATION_JSON))
                .andExpect(status().isBadRequest())
                .andExpect(jsonPath("$.errorCode").value("ORDER_NOT_FOUND"))
                .andExpect(jsonPath("$.message").value("Order not found with ID: 999"))
                .andExpect(jsonPath("$.context.orderId").value(999));
    }
}
```

**Data layer testing:**

```java
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@Testcontainers
class OrderJpaRepositoryTest {
    
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15-alpine")
            .withDatabaseName("testdb")
            .withUsername("test") 
            .withPassword("test");
    
    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }
    
    @Autowired
    private TestEntityManager entityManager;
    
    @Autowired
    private SpringDataOrderRepository repository;
    
    @Test
    @DisplayName("Should find orders by customer ID")
    void should_find_orders_by_customer_id() {
        // Given
        Long customerId = 1L;
        
        OrderEntity order1 = createOrderEntity(customerId, "DRAFT");
        OrderEntity order2 = createOrderEntity(customerId, "CONFIRMED");
        OrderEntity orderFromDifferentCustomer = createOrderEntity(2L, "DRAFT");
        
        entityManager.persistAndFlush(order1);
        entityManager.persistAndFlush(order2);
        entityManager.persistAndFlush(orderFromDifferentCustomer);
        
        // When
        List<OrderEntity> found = repository.findByCustomerId(customerId);
        
        // Then
        assertThat(found).hasSize(2);
        assertThat(found)
            .extracting(OrderEntity::getCustomerId)
            .allMatch(id -> id.equals(customerId));
        assertThat(found)
            .extracting(OrderEntity::getStatus)
            .containsExactlyInAnyOrder("DRAFT", "CONFIRMED");
    }
    
    @Test
    @DisplayName("Should find orders by status and creation date")
    void should_find_orders_by_status_and_creation_date() {
        // Given
        Instant cutoffDate = Instant.now().minus(1, ChronoUnit.DAYS);
        
        OrderEntity oldOrder = createOrderEntityWithDate(1L, "PENDING", cutoffDate.minus(1, ChronoUnit.HOURS));
        OrderEntity newOrder = createOrderEntityWithDate(2L, "PENDING", cutoffDate.plus(1, ChronoUnit.HOURS));
        
        entityManager.persistAndFlush(oldOrder);
        entityManager.persistAndFlush(newOrder);
        
        // When
        List<OrderEntity> staleOrders = repository.findByStatusAndCreatedAtBefore("PENDING", cutoffDate);
        
        // Then
        assertThat(staleOrders).hasSize(1);
        assertThat(staleOrders.get(0).getId()).isEqualTo(oldOrder.getId());
    }
    
    @Test
    @DisplayName("Should update order status using custom query")
    void should_update_order_status_using_custom_query() {
        // Given
        OrderEntity order = createOrderEntity(1L, "DRAFT");
        OrderEntity savedOrder = entityManager.persistAndFlush(order);
        
        // When
        int updatedCount = repository.updateOrderStatus(savedOrder.getId(), "CONFIRMED");
        entityManager.flush();
        entityManager.clear();
        
        // Then
        assertThat(updatedCount).isEqualTo(1);
        
        OrderEntity updatedOrder = entityManager.find(OrderEntity.class, savedOrder.getId());
        assertThat(updatedOrder.getStatus()).isEqualTo("CONFIRMED");
    }
    
    private OrderEntity createOrderEntity(Long customerId, String status) {
        return createOrderEntityWithDate(customerId, status, Instant.now());
    }
    
    private OrderEntity createOrderEntityWithDate(Long customerId, String status, Instant createdAt) {
        OrderEntity order = new OrderEntity();
        order.setCustomerId(customerId);
        order.setStatus(status);
        order.setCreatedAt(createdAt);
        order.setUpdatedAt(createdAt);
        order.setTotalAmount(new BigDecimal("100.00"));
        return order;
    }
}
```

### Architecture testing with ArchUnit

**Boundary enforcement testing:**
```java
@AnalyzeClasses(packagesOf = Application.class)
class ArchitectureTest {
    
    @ArchTest
    static final ArchRule domainLayerShouldNotDependOnInfrastructure = 
        noClasses()
            .that().resideInAPackage("..domain..")
            .should().dependOnClassesThat()
                .resideInAPackage("..infrastructure..");
                
    @ArchTest
    static final ArchRule applicationLayerShouldNotDependOnAdapters = 
        noClasses()
            .that().resideInAPackage("..application..")
            .should().dependOnClassesThat()
                .resideInAPackage("..adapter..");
}
```

### Test naming and organization conventions

**Test structure:**
```
src/test/java/
├── com/company/product/
│   ├── domain/           # Pure unit tests
│   │   ├── OrderTest.java
│   │   └── CustomerTest.java
│   ├── application/      # Use case tests
│   │   └── PlaceOrderUseCaseTest.java
│   ├── adapter/
│   │   ├── web/         # Controller tests
│   │   │   └── OrderControllerTest.java
│   │   └── persistence/ # Repository tests
│   │       └── JpaOrderRepositoryTest.java
│   └── integration/     # Integration tests
│       └── OrderIntegrationTest.java
```

**Test method naming:**
- Pattern: `should_{ExpectedBehavior}_when_{StateUnderTest}`
- Example: `should_calculate_total_correctly_when_items_added`

## Validation and security best practices

### Input validation patterns

**Bean validation with JSR-303:**
```java
public class CreateOrderRequest {
    @NotNull(message = "Customer ID is required")
    private Long customerId;
    
    @NotEmpty(message = "Order must have at least one item")
    @Valid
    private List<OrderItemRequest> items;
    
    @DecimalMin(value = "0.01", message = "Total must be positive")
    private BigDecimal expectedTotal;
}
```

**Controller validation:**
```java
@PostMapping("/api/orders")
public ResponseEntity<OrderResponse> createOrder(@Valid @RequestBody CreateOrderRequest request) {
    PlaceOrderCommand command = mapToCommand(request);
    OrderResult result = placeOrderUseCase.placeOrder(command);
    return ResponseEntity.status(HttpStatus.CREATED).body(mapToResponse(result));
}
```

### Security implementation

**Method-level security with @PreAuthorize:**
```java
@Service
public class OrderService {
    
    @PreAuthorize("hasRole('USER')")
    public Order createOrder(OrderRequest request) {
        // Implementation
    }
    
    @PreAuthorize("hasRole('ADMIN') or #order.customerId == authentication.name")
    public void cancelOrder(Order order) {
        // Implementation
    }
    
    @PreAuthorize("@orderSecurityService.canAccess(#orderId, authentication)")
    public Order getOrder(Long orderId) {
        // Implementation
    }
}
```

**Security configuration:**
```java
@Configuration
@EnableMethodSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            .authorizeHttpRequests(requests -> requests
                .requestMatchers("/actuator/health").permitAll()
                .requestMatchers("/api/public/**").permitAll()
                .anyRequest().authenticated())
            .oauth2ResourceServer(oauth2 -> oauth2.jwt(withDefaults()))
            .csrf(csrf -> csrf.csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse()))
            .build();
    }
}
```

### Security testing practices

**Security slice testing:**
```java
@WebMvcTest(OrderController.class)
@Import(SecurityConfiguration.class)
class OrderControllerSecurityTest {
    @Autowired private MockMvc mockMvc;
    
    @Test
    void should_require_authentication_for_protected_endpoint() throws Exception {
        mockMvc.perform(get("/api/orders"))
                .andExpect(status().isUnauthorized());
    }
    
    @Test
    @WithMockUser(roles = "USER")
    void should_allow_authenticated_user_access() throws Exception {
        mockMvc.perform(get("/api/orders"))
                .andExpect(status().isOk());
    }
}
```

## Logging and monitoring standards

### Structured logging implementation

**Log4j2 configuration with JsonTemplateLayout:**

**In production ready container logging must be provide only with Kafka appender not FileAppender** 

```xml
<!-- log4j2-spring.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">
    <Appenders>
        <!-- Console appender for production (JSON format) -->
        <Console name="ConsoleAppender" target="SYSTEM_OUT">
            <JsonTemplateLayout eventTemplateUri="classpath:JsonLayout.json"/>
        </Console>
        
        <!-- Console appender for local development (readable format) -->
        <Console name="ConsoleLocal" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
        </Console>

        <!-- Kafka appender with JSON format -->
        <Kafka name="kafka" topic="${env:LOG_KAFKA_TOPIC}">
            <JsonTemplateLayout eventTemplateUri="classpath:JsonLayout.json"/>
            <Property name="bootstrap.servers">${env:LOG_BOOTSTRAP_SERVER}</Property>
            <Property name="security.protocol">SASL_PLAINTEXT</Property>
            <Property name="sasl.mechanism">PLAIN</Property>
            <Property name="group.id">pgw-logger-group</Property>
            <Property name="sasl.jaas.config">org.apache.kafka.common.security.plain.PlainLoginModule required username="${env:KAFKA_LOG_USERNAME}" password="${env:KAFKA_LOG_PASSWORD}";</Property>
        </Kafka>
        
        <!-- File appender with JSON format -->
        <RollingFile name="FileAppender" fileName="logs/application.log"
                     filePattern="logs/application-%d{yyyy-MM-dd}-%i.log.gz">
            <JsonTemplateLayout eventTemplateUri="classpath:JsonLayout.json"/>
            <Policies>
                <TimeBasedTriggeringPolicy/>
                <SizeBasedTriggeringPolicy size="100MB"/>
            </Policies>
            <DefaultRolloverStrategy max="10"/>
        </RollingFile>
    </Appenders>
    
    <Loggers>
        <!-- Application specific loggers -->
        <Logger name="com.company.product" level="INFO" additivity="false">
            <AppenderRef ref="ConsoleAppender"/>
            <AppenderRef ref="FileAppender"/>
            <AppenderRef ref="Kafka"/>
        </Logger>
        
        <!-- Spring framework loggers -->
        <Logger name="org.springframework" level="WARN"/>
        <Logger name="org.hibernate.SQL" level="WARN"/>
        
        <!-- Root logger -->
        <Root level="INFO">
            <AppenderRef ref="ConsoleAppender"/>
            <AppenderRef ref="Kafka"/>
        </Root>
    </Loggers>
</Configuration>
```

**Custom JSON template (src/main/resources/JsonLayout.json):**
```json
{
  "timestamp": {
    "$resolver": "timestamp",
    "pattern": {
      "format": "yyyy-MM-dd'T'HH:mm:ss.SSSZ",
      "timeZone": "UTC"
    }
  },
  "level": {
    "$resolver": "level",
    "field": "name"
  },
  "service": "${spring:spring.application.name:-unknown}",
  "logger": {
    "$resolver": "logger",
    "field": "name"
  },
  "thread": {
    "$resolver": "thread",
    "field": "name"
  },
  "message": {
    "$resolver": "message",
    "stringified": true
  },
  "traceId": {
    "$resolver": "mdc",
    "key": "traceId"
  },
  "spanId": {
    "$resolver": "mdc", 
    "key": "spanId"
  },
  "userId": {
    "$resolver": "mdc",
    "key": "userId"
  },
  "requestId": {
    "$resolver": "mdc",
    "key": "requestId"
  },
  "operation": {
    "$resolver": "mdc",
    "key": "operation"
  },
  "entityType": {
    "$resolver": "mdc",
    "key": "entityType"
  },
  "entityId": {
    "$resolver": "mdc",
    "key": "entityId"
  },
  "exception": {
    "$resolver": "exception",
    "field": {
      "className": "className",
      "message": "message", 
      "stackTrace": "stackTrace"
    }
  },
  "mdc": {
    "$resolver": "mdc",
    "flatten": true,
    "stringified": true
  },
  "environment": "${env:ENVIRONMENT:-local}",
  "version": "${env:APPLICATION_VERSION:-1.0.0}",
  "host": {
    "$resolver": "hostname"
  },
  "processId": {
    "$resolver": "processId"
  }
}
```

**Required dependencies for Log4j2 with JsonTemplateLayout:**
```kotlin
dependencies {
    implementation("org.springframework.boot:spring-boot-starter-web") {
        exclude(group = "org.springframework.boot", module = "spring-boot-starter-logging")
    }
    implementation("org.springframework.boot:spring-boot-starter-log4j2")
    implementation("org.apache.logging.log4j:log4j-layout-template-json")
    
    implementation("org.springframework.boot:spring-boot-starter-actuator")
    implementation("io.micrometer:micrometer-registry-prometheus")
    implementation("io.micrometer:micrometer-tracing-bridge-otel")
    implementation("org.springframework.modulith:spring-modulith-starter-core")
    
    // Testing dependencies
    testImplementation("org.springframework.boot:spring-boot-starter-test") {
        exclude(group = "org.springframework.boot", module = "spring-boot-starter-logging")
    }
    testImplementation("org.springframework.boot:spring-boot-starter-log4j2")
    testImplementation("org.testcontainers:junit-jupiter")
    testImplementation("org.testcontainers:postgresql")
}
```

**Environment-specific logging configuration:**

```
src/main/resources/
├── dev
│   ├── log4j.xml  # Development specific profile logging confuguration
├── local
│   ├── log4j.xml # Local specific profile logging confuguration
├── prod
│   ├── log4j.xml # Production specific profile logging confuguration
```

**Benefits of Log4j2 with JsonTemplateLayout approach:**
- **Performance**: Asynchronous logging with LMAX Disruptor for ultra-low latency
- **Memory Efficiency**: Garbage-free logging in steady state 
- **Flexibility**: Easy customization through JSON template files
- **Standardization**: Consistent JSON structure across all log entries
- **Observability**: Better integration with log aggregation tools (ELK, Splunk, etc.)
- **Filtering**: Enhanced filtering capabilities based on structured fields
- **Correlation**: Built-in support for trace/span correlation IDs
- **Auto-reload**: Configuration can be reloaded without application restart

**Log4j2 performance optimization:**
```xml
<!-- Add to log4j2-spring.xml for async logging -->
<Configuration status="WARN">
    <Appenders>
        <!-- Async appender for high-performance logging -->
        <AsyncLogger name="com.company.product" level="INFO" additivity="false">
            <AppenderRef ref="ConsoleAppender"/>
            <AppenderRef ref="Kafka"/>
        </AsyncLogger>
    </Appenders>
    
    <!-- System properties for performance tuning -->
    <Properties>
        <Property name="LOG4J_CONTEXT_SELECTOR">org.apache.logging.log4j.core.async.AsyncLoggerContextSelector</Property>
    </Properties>
</Configuration>
```

**JVM arguments for async logging:**
```bash
# Add these JVM arguments for optimal Log4j2 performance
-Dlog4j2.contextSelector=org.apache.logging.log4j.core.async.AsyncLoggerContextSelector
-Dlog4j2.asyncLoggerRingBufferSize=262144
-Dlog4j2.asyncLoggerWaitStrategy=Block
-Dlog4j2.garbageFreeThreadContextMap=true
```

**Dependencies for observability:**
```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-tracing-bridge-otel</artifactId>
</dependency>
<dependency>
    <groupId>io.opentelemetry</groupId>
    <artifactId>opentelemetry-exporter-otlp</artifactId>
</dependency>
```

### Metrics collection with Micrometer

**Custom metrics implementation:**
```java
@RestController
public class OrderController {
    private final Counter orderCounter;
    private final Timer orderTimer;
    
    public OrderController(MeterRegistry meterRegistry) {
        this.orderCounter = Counter.builder("orders.created.total")
            .description("Total orders created")
            .tag("service", "order-service")
            .register(meterRegistry);
        this.orderTimer = Timer.builder("orders.processing.duration")
            .description("Order processing duration")
            .register(meterRegistry);
    }
    
    @PostMapping("/api/orders")
    public ResponseEntity<OrderResponse> createOrder(@RequestBody CreateOrderRequest request) {
        return orderTimer.recordCallable(() -> {
            OrderResponse response = processOrder(request);
            orderCounter.increment();
            return ResponseEntity.ok(response);
        });
    }
}
```

**Actuator endpoint configuration with structured logging:**
```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,prometheus,metrics,info,loggers
  endpoint:
    health:
      show-details: when-authorized
      show-components: always
    loggers:
      enabled: true
  metrics:
    distribution:
      percentiles-histogram:
        http:
          server:
            requests: true
    tags:
      application: ${spring.application.name}
      environment: ${spring.profiles.active}
    export:
      prometheus:
        enabled: true
        step: 10s
```

**Log aggregation and alerting queries (for ELK/Splunk):**
```json
// Error rate alert query
{
  "query": {
    "bool": {
      "must": [
        {"match": {"service": "order-service"}},
        {"match": {"level": "ERROR"}},
        {"range": {"timestamp": {"gte": "now-5m"}}}
      ]
    }
  },
  "aggs": {
    "error_count": {
      "cardinality": {
        "field": "requestId"
      }
    }
  }
}

// Performance monitoring query
{
  "query": {
    "bool": {
      "must": [
        {"match": {"operation": "processOrder"}},
        {"range": {"processingTimeMs": {"gte": 5000}}}
      ]
    }
  }
}

// Business metrics query
{
  "query": {
    "bool": {
      "must": [
        {"match": {"operation": "createOrder"}},
        {"match": {"success": "true"}},
        {"range": {"timestamp": {"gte": "now-1h"}}}
      ]
    }
  },
  "aggs": {
    "total_orders": {
      "cardinality": {
        "field": "orderId"
      }
    },
    "avg_order_value": {
      "avg": {
        "field": "orderValue"
      }
    }
  }
}
```

## Configuration management approaches

### @ConfigurationProperties pattern (required)

**Type-safe configuration properties:**
```java
@ConfigurationProperties("app.payment")
public class PaymentProperties {
    private Duration timeout = Duration.ofSeconds(30);
    private String apiUrl;
    private final Security security = new Security();
    
    // getters/setters...
    
    public static class Security {
        private String username;
        private String password;
        // getters/setters...
    }
}
```

**Configuration registration:**
```java
@Configuration
@EnableConfigurationProperties(PaymentProperties.class)
public class PaymentConfiguration {
    
    @Bean
    public PaymentService paymentService(PaymentProperties properties) {
        return new PaymentService(properties);
    }
}
```

**Environment-specific configuration:**
```yaml
# application.yml
app:
  payment:
    timeout: 30s
    api-url: https://api.payment.com
    security:
      username: ${PAYMENT_USERNAME}
      password: ${PAYMENT_PASSWORD}

# application-prod.yml
app:
  payment:
    timeout: 45s
    api-url: https://prod-api.payment.com

# application-dev.yml  
app:
  payment:
    timeout: 10s
    api-url: https://dev-api.payment.com
```

### Configuration validation

**JSR-303 validation for configuration:**
```java
@ConfigurationProperties("app.database")
@Validated
public class DatabaseProperties {
    
    @NotBlank
    private String url;
    
    @Min(1) @Max(100)
    private int maxConnections = 10;
    
    @DurationMin(seconds = 1)
    private Duration connectionTimeout = Duration.ofSeconds(30);
}
```

## Database and persistence layer best practices

### JPA and Hibernate standards

**Entity design patterns:**
```java
@Entity
@Table(name = "orders")
public class OrderJpaEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "customer_id", nullable = false)
    private Long customerId;
    
    @Enumerated(EnumType.STRING)
    @Column(name = "status", nullable = false)
    private OrderStatus status;
    
    @CreationTimestamp
    @Column(name = "created_at", nullable = false)
    private LocalDateTime createdAt;
    
    @UpdateTimestamp
    @Column(name = "updated_at", nullable = false)
    private LocalDateTime updatedAt;
    
    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private List<OrderItemJpaEntity> items = new ArrayList<>();
}
```

**Repository patterns with Spring Data JPA:**
```java
public interface OrderRepository extends JpaRepository<OrderJpaEntity, Long> {
    
    List<OrderJpaEntity> findByCustomerId(Long customerId);
    
    @Query("SELECT o FROM OrderJpaEntity o WHERE o.status = :status AND o.createdAt < :cutoff")
    List<OrderJpaEntity> findStaleOrders(@Param("status") OrderStatus status, 
                                        @Param("cutoff") LocalDateTime cutoff);
    
    @Modifying
    @Query("UPDATE OrderJpaEntity o SET o.status = :newStatus WHERE o.id = :orderId")
    int updateOrderStatus(@Param("orderId") Long orderId, @Param("newStatus") OrderStatus newStatus);
}
```

### Database migration with Flyway

**Flyway configuration:**
```yaml
spring:
  flyway:
    enabled: true
    locations: classpath:db/migration
    baseline-on-migrate: true
    validate-on-migrate: true
    clean-disabled: true
```

**Migration script naming and structure:**
```sql
-- V1__Create_orders_table.sql
CREATE TABLE orders (
    id BIGSERIAL PRIMARY KEY,
    customer_id BIGINT NOT NULL,
    status VARCHAR(20) NOT NULL,
    total_amount DECIMAL(10,2) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_orders_customer_id ON orders(customer_id);
CREATE INDEX idx_orders_status ON orders(status);
CREATE INDEX idx_orders_created_at ON orders(created_at);
```

**Database naming conventions:**
- **Tables**: Plural nouns, snake_case (e.g., `user_profiles`, `order_items`)
- **Columns**: snake_case, descriptive names
- **Primary Keys**: `id` or `{table_name}_id`
- **Foreign Keys**: `{referenced_table}_id`
- **Indexes**: Descriptive names indicating purpose (e.g., `idx_orders_customer_id`)

### Transaction management patterns

**Service-level transaction boundaries:**
```java
@Service
@Transactional(readOnly = true)
public class OrderService {
    
    @Transactional
    public Order createOrder(CreateOrderCommand command) {
        // Transaction boundary for write operations
        Order order = new Order(command.customerId());
        command.items().forEach(item -> order.addItem(item));
        
        Order savedOrder = orderRepository.save(order);
        
        // Publish domain events within transaction
        domainEventPublisher.publish(new OrderCreated(savedOrder.getId()));
        
        return savedOrder;
    }
    
    public List<Order> findOrdersByCustomer(Long customerId) {
        // Read-only transaction
        return orderRepository.findByCustomerId(customerId);
    }
}
```

## REST API design standards

### Resource design patterns

**URI structure standards:**
```
GET    /api/orders                    # List orders
POST   /api/orders                    # Create order
GET    /api/orders/{id}               # Get specific order
PUT    /api/orders/{id}               # Update entire order
PATCH  /api/orders/{id}               # Partial order update
DELETE /api/orders/{id}               # Cancel order

GET    /api/orders/{id}/items         # List order items
POST   /api/orders/{id}/items         # Add item to order
DELETE /api/orders/{id}/items/{itemId} # Remove item from order
```

**HTTP method usage:**
- **GET**: Data retrieval, idempotent, no request body
- **POST**: Resource creation, non-idempotent
- **PUT**: Full resource replacement, idempotent
- **PATCH**: Partial resource update
- **DELETE**: Resource removal, idempotent

### Request/Response patterns

**Controller implementation:**
```java
@RestController
@RequestMapping("/api/orders")
@Validated
public class OrderController {
    
    private final PlaceOrderUseCase placeOrderUseCase;
    private final GetOrderUseCase getOrderUseCase;
    
    @PostMapping
    public ResponseEntity<OrderResponse> createOrder(@Valid @RequestBody CreateOrderRequest request) {
      return Optional.ofNullable(request)
        .map(placeOrderUseCase::placeOrder)
        .map(result -> ResponseEntity.status(HttpStatus.CREATED)
                        .location(URI.create("/api/orders/" + response.id()))
                        .body(response))
        .orElseThrow(CreateOrderFailedException::new);
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<OrderResponse> getOrder(@PathVariable Long id) {
        return Optional.ofNullable(id)
          .map(getOrderUseCase::getOrder)
          .map(result -> ResponseEntity.ok(mapToResponse(order)))
          .orElseThrow(() -> new OrderNotFoundException(id));
    }
    
    @GetMapping
    public ResponseEntity<PagedResponse<OrderSummary>> listOrders(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "20") int size,
            @RequestParam(required = false) String status) {
        
        ListOrdersQuery query = new ListOrdersQuery.builder()
                                                    .page(page)
                                                    .size(size)
                                                    .status(status)
                                                    .build();
        PagedResult<Order> result = getOrderUseCase.listOrders(query);
        return ResponseEntity.ok(mapToPagedResponse(result));
    }
}
```

**DTO patterns:**
```java
// Request DTOs
public record CreateOrderRequest(
    @NotNull Long customerId,
    @NotEmpty @Valid List<OrderItemRequest> items,
    String notes
) {}

public record OrderItemRequest(
    @NotNull Long productId,
    @Min(1) int quantity,
    @DecimalMin("0.01") BigDecimal unitPrice
) {}

// Response DTOs
public record OrderResponse(
    Long id,
    Long customerId,
    OrderStatus status,
    BigDecimal totalAmount,
    LocalDateTime createdAt,
    List<OrderItemResponse> items
) {}

// Paged response wrapper
public record PagedResponse<T>(
    List<T> content,
    int page,
    int size,
    long totalElements,
    int totalPages,
    boolean first,
    boolean last
) {}
```

### Error response standardization

**Consistent error response format:**
```java
public record ErrorResponse(
    Instant timestamp,
    int status,
    String error,
    String message,
    String path,
    String traceId
) {
    public static ErrorResponse of(HttpStatus status, String message, String path) {
        return new ErrorResponse(
            Instant.now(),
            status.value(),
            status.getReasonPhrase(),
            message,
            path,
            getCurrentTraceId()
        );
    }
}

public record ValidationErrorResponse(
    Instant timestamp,
    int status,
    String error,
    String message,
    String path,
    List<FieldError> fieldErrors
) {}

public record FieldError(
    String field,
    Object rejectedValue,
    String message
) {}
```

### API versioning strategy

**URI versioning approach:**
```java
@RestController
@RequestMapping("/api/v1/orders")
public class OrderControllerV1 {
    // Version 1 implementation
}

@RestController
@RequestMapping("/api/v2/orders")
public class OrderControllerV2 {
    // Version 2 implementation with enhanced features
}
```

**Content negotiation versioning:**
```java
@GetMapping(value = "/orders", produces = "application/vnd.company.order-v1+json")
public ResponseEntity<OrderResponseV1> getOrderV1() {
    // Version 1 response
}

@GetMapping(value = "/orders", produces = "application/vnd.company.order-v2+json")
public ResponseEntity<OrderResponseV2> getOrderV2() {
    // Version 2 response
}
```

## Performance optimization techniques

### JVM optimization for containers

**Container-aware JVM settings:**
```yaml
# Kubernetes deployment with optimized JVM
env:
- name: JAVA_TOOL_OPTIONS
  value: >-
    -Xmx$(MEMORY_LIMIT)
    -XX:+UseContainerSupport
    -XX:+UseG1GC
    -XX:+UseStringDeduplication
    -XX:+EnableDynamicAgentLoading
    -Dfile.encoding=UTF-8
    -Djava.security.egd=file:/dev/./urandom
```



### Database performance patterns

**Connection pooling configuration:**
```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 20
      minimum-idle: 5
      idle-timeout: 300000
      max-lifetime: 1200000
      connection-timeout: 20000
      validation-timeout: 5000
      leak-detection-threshold: 60000
```

**Query optimization patterns:**
```java
@Repository
public class OptimizedOrderRepository {
    
    // Use batch operations for bulk updates
    @Modifying
    @Query("UPDATE OrderJpaEntity o SET o.status = :status WHERE o.id IN :orderIds")
    int updateOrderStatusBatch(@Param("orderIds") List<Long> orderIds, 
                              @Param("status") OrderStatus status);
    
    // Use projection for read-only queries
    @Query("SELECT new com.company.OrderSummary(o.id, o.customerId, o.status, o.totalAmount) " +
           "FROM OrderJpaEntity o WHERE o.customerId = :customerId")
    List<OrderSummary> findOrderSummariesByCustomer(@Param("customerId") Long customerId);
    
    // Use streaming for large result sets
    @QueryHints(@QueryHint(name = HINT_FETCH_SIZE, value = "50"))
    @Query("SELECT o FROM OrderJpaEntity o WHERE o.createdAt < :cutoff")
    Stream<OrderJpaEntity> streamOldOrders(@Param("cutoff") LocalDateTime cutoff);
}
```

### Caching strategies

**Spring Cache abstraction:**
```java
@Service
@CacheConfig(cacheNames = "orders")
public class OrderService {
    
    @Cacheable(key = "#orderId")
    public Order findOrder(Long orderId) {
        return orderRepository.findById(orderId)
            .orElseThrow(() -> new OrderNotFoundException(orderId));
    }
    
    @CacheEvict(key = "#order.id")
    public Order updateOrder(Order order) {
        return orderRepository.save(order);
    }
    
    @Cacheable(key = "'customer:' + #customerId + ':page:' + #pageable.pageNumber")
    public Page<Order> findOrdersByCustomer(Long customerId, Pageable pageable) {
        return orderRepository.findByCustomerId(customerId, pageable);
    }
}
```

**Redis configuration:**
```yaml
spring:
  cache:
    type: redis
    redis:
      time-to-live: 600000
      cache-null-values: false
  data:
    redis:
      host: redis-server
      port: 6379
      timeout: 2000ms
      lettuce:
        pool:
          max-active: 8
          max-idle: 8
          min-idle: 0
```

## Build and deployment conventions

### Modern build configuration

**Gradle build optimization:**
```kotlin
// gradle.properties
org.gradle.parallel=true
org.gradle.caching=true
org.gradle.configuration-cache=true
org.gradle.daemon=true
kotlin.incremental=true

// build.gradle.kts performance settings
tasks.withType<Test> {
    useJUnitPlatform()
    maxParallelForks = (Runtime.getRuntime().availableProcessors() / 2).takeIf { it > 0 } ?: 1
    testLogging {
        events("passed", "skipped", "failed")
    }
}
```

### Container-first deployment

**Multi-stage Dockerfile:**
```dockerfile
# Multi-stage build for optimized images
FROM eclipse-temurin:21-jdk-alpine AS builder
WORKDIR /app
COPY . .
RUN ./gradlew clean bootJar

FROM eclipse-temurin:21-jre-alpine
WORKDIR /app

# Create non-root user
RUN addgroup -g 1001 -S appuser && adduser -u 1001 -S appuser -G appuser

# Copy application
COPY --from=builder /app/build/libs/*.jar app.jar

# Set ownership and switch to non-root user
RUN chown appuser:appuser app.jar
USER appuser:appuser

EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**Kubernetes deployment manifest:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
  labels:
    app: order-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: order-service
  template:
    metadata:
      labels:
        app: order-service
    spec:
      containers:
      - name: order-service
        image: order-service:latest
        ports:
        - containerPort: 8080
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: "prod"
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /actuator/health/liveness
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /actuator/health/readiness
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
        securityContext:
          runAsNonRoot: true
          runAsUser: 1001
          runAsGroup: 1001
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
```

### CI/CD pipeline standards

**GitHub Actions workflow:**
```yaml
name: CI/CD Pipeline
on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - name: Set up JDK 21
      uses: actions/setup-java@v4
      with:
        java-version: '21'
        distribution: 'temurin'
    
    - name: Cache Gradle packages
      uses: actions/cache@v3
      with:
        path: ~/.gradle/caches
        key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle') }}
        
    - name: Run tests
      run: ./gradlew test
      
    - name: Run integration tests
      run: ./gradlew integrationTest
      
    - name: Security scan
      run: ./gradlew dependencyCheckAnalyze
      
    - name: Build image
      run: ./gradlew bootBuildImage --imageName=order-service:${{ github.sha }}
      
    - name: Push to registry
      if: github.ref == 'refs/heads/main'
      run: |
        docker tag order-service:${{ github.sha }} registry.company.com/order-service:${{ github.sha }}
        docker push registry.company.com/order-service:${{ github.sha }}
```

### Security scanning integration

**OWASP Dependency Check:**
```kotlin
// build.gradle.kts
plugins {
    id("org.owasp.dependencycheck") version "8.4.0"
}

dependencyCheck {
    formats = listOf("HTML", "JSON")
    failBuildOnCVSS = 7f
    suppressionFile = "dependency-check-suppressions.xml"
}
```

**Container security scanning:**
```yaml
- name: Run Trivy vulnerability scanner
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: 'order-service:${{ github.sha }}'
    format: 'sarif'
    output: 'trivy-results.sarif'

- name: Upload Trivy scan results to GitHub Security tab
  uses: github/codeql-action/upload-sarif@v2
  with:
    sarif_file: 'trivy-results.sarif'
```

## Conclusion

This comprehensive specification represents the synthesis of current industry best practices from leading technology companies, official Spring Framework guidance, and modern enterprise development patterns. **Standard Maven/Gradle multi-module architecture provides the foundation for hexagonal architecture implementation**, while maintaining practical applicability for enterprise teams.

Key implementation priorities include adopting **Google Java Style Guide standards, implementing comprehensive testing strategies with 85%+ coverage, leveraging Java 21 LTS features like production-ready virtual threads, enhanced pattern matching, and sequenced collections, and establishing container-first deployment patterns** with robust observability through OpenTelemetry and Micrometer integration.

The specification emphasizes **security by design, performance optimization for containerized environments with Java 21's virtual threads enabling massive concurrency, and maintainable code organization** that scales with team growth and system complexity. Enterprise teams implementing these practices will achieve highly scalable, observable, and maintainable Spring Boot applications that meet modern operational requirements while adhering to hexagonal architecture principles that promote long-term code health and business agility.

**Java 21 LTS provides significant performance improvements, especially for I/O intensive Spring Boot applications**, with virtual threads enabling orders of magnitude better concurrency handling compared to traditional thread pools. The enhanced pattern matching and string templates improve code readability and maintainability, while sequenced collections provide more intuitive APIs for common operations. **Standard multi-module project structure enforces architectural boundaries at compile time**, ensuring that hexagonal architecture principles are maintained throughout the development lifecycle.

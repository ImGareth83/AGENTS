# Repository Guidelines

## Dependency Injection Policy
### Mandatory
- For new or substantially refactored classes, keep injected dependencies `final` and initialize them through the constructor.
- Do not inject infrastructure clients directly into domain logic (for example `RestTemplate`, `WebClient`, `JdbcTemplate`, AWS SDK clients).
- Wrap infrastructure behind simple domain interfaces (for example `EmailSender`, `PaymentProcessor`, `TimeProvider`) so tests stay focused and framework details do not leak into business logic.
- Keep wiring at the edges: use Spring for composition, but keep core business logic as framework-agnostic as practical.
- Avoid injecting service-locator style dependencies such as `ApplicationContext`, `BeanFactory`, or `ObjectProvider` unless there is a clear, documented need.
- Avoid manual `new` for Spring-managed collaborators; let the container resolve dependencies.

### Preferred
- Prefer constructor injection for all new classes.
- Keep constructor parameter lists small. If a constructor has about 6-10 dependencies, treat it as a design smell.
- Refactor by splitting responsibilities, introducing clear facade/domain service boundaries, or composing dependencies behind a focused collaborator.
- Avoid string-based qualifiers when possible (for example `@Qualifier("emailSender")`) because they are fragile magic strings.
- Prefer `@Primary` for default implementations, custom qualifier annotations for type safety, or `Map<String, MessageSender>` / `List<MessageSender>` when multiple implementations are required.
- Prefer composition over inheritance for service design.
- Keep services small and compose behavior through injected collaborators instead of deep class hierarchies.
- It is fine to use Spring stereotypes at boundaries, but avoid scattering framework-specific concerns across domain classes.
- Service locators hide dependencies and reduce predictability; prefer explicit constructor dependencies.
- Use `@ConfigurationProperties` for grouped configuration values to keep settings typed and cohesive.
- Avoid scattering `@Value("${...}")` across many classes; reserve it for small, isolated values.
- Use interfaces at service boundaries when multiple implementations are expected.

## Legacy Code Consistency
- Existing `@Autowired` usage in legacy files is acceptable.
- When modifying existing files, follow the DI style already used in that file for consistency.
- If converting a legacy class to constructor injection, treat it as an intentional refactor and keep the change scoped.
- If a change substantially reshapes a class (for example constructor/signature changes, major responsibility changes, or broad refactor), prefer migrating that class to constructor injection in the same change.

## Bean Selection & Configuration
- Prefer this selection order when multiple beans exist: `@Primary` for defaults, custom qualifier annotations for explicit selection, then string `@Qualifier("...")` only as fallback.
- Use `@Primary` only when one implementation should be the default across most injection points.
- Document non-obvious bean selection decisions in code comments or PR notes.

## DI Testing Guidance
- Validate bean wiring explicitly (qualifier resolution, primary selection, and profile-specific beans).
- Use `@MockBean` or test doubles to isolate external boundaries.
- Prefer focused Spring test slices for DI verification before using full-context tests.

## Examples
Constructor injection for new classes:

```java
@Service
public class InvoiceService {
    private final TaxCalculator taxCalculator;

    public InvoiceService(TaxCalculator taxCalculator) {
        this.taxCalculator = taxCalculator;
    }
}
```

Existing legacy `@Autowired` style (accepted as a legacy exception when preserving file consistency):

```java
@Service
public class LegacyBillingService {
    @Autowired
    private PaymentGatewayClient paymentGatewayClient;
}
```

Multiple implementations with a custom qualifier annotation (type-safe):

```java
@Qualifier
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.FIELD, ElementType.PARAMETER, ElementType.TYPE})
public @interface EmailChannel {}

@Service
@EmailChannel
public class EmailMessageSender implements MessageSender {}

@Service
public class NotificationService {
    private final MessageSender sender;

    public NotificationService(@EmailChannel MessageSender sender) {
        this.sender = sender;
    }
}
```

Fallback string `@Qualifier` usage (legacy/fallback only):

```java
@Service("emailSender")
public class EmailSender implements MessageSender {}

@Service
public class LegacyNotificationService {
    private final MessageSender sender;

    public LegacyNotificationService(@Qualifier("emailSender") MessageSender sender) {
        this.sender = sender;
    }
}
```

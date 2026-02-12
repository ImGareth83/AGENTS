# Spring Boot Dependency Injection Guidelines

## Purpose
Define consistent dependency injection practices for Spring Boot code so designs stay explicit, testable, and maintainable.

## Scope
Applies to Spring-managed classes, service design, bean selection, configuration binding, and DI-focused tests.

## Core Principles
1. Prefer explicit dependencies over hidden lookups.
2. Keep business logic framework-light and infrastructure at boundaries.
3. Favor cohesive, small services over broad service classes.

## Decision Rules

### Mandatory
- For new or substantially refactored classes, keep injected dependencies `final` and initialize them through the constructor.
- Do not inject infrastructure clients directly into domain logic (for example `RestTemplate`, `WebClient`, `JdbcTemplate`, AWS SDK clients).
- Wrap infrastructure behind simple domain interfaces (for example `EmailSender`, `PaymentProcessor`, `TimeProvider`).
- Keep wiring at the edges: use Spring for composition, keep core business logic framework-agnostic where practical.
- Avoid injecting service-locator style dependencies such as `ApplicationContext`, `BeanFactory`, or `ObjectProvider` unless there is a clear documented need.
- Avoid manual `new` for Spring-managed collaborators.

### Preferred
- Prefer constructor injection for all new classes.
- Keep constructor parameter lists small; treat about 6-10 dependencies as a design smell.
- Refactor large constructors by splitting responsibilities or introducing focused collaborators/facades.
- Avoid string-based qualifiers where possible.
- Prefer bean selection in this order:
  1. `@Primary` for defaults,
  2. custom qualifier annotations for explicit selection,
  3. string `@Qualifier("...")` as fallback.
- Prefer composition over inheritance in service design.
- Use `@ConfigurationProperties` for grouped configuration values.
- Avoid scattering `@Value("${...}")` across many classes.
- Use interfaces at service boundaries when multiple implementations are expected.

### Restricted (Requires Justification)
- `@Qualifier("...")` string qualifiers are allowed only when `@Primary` or custom qualifiers are not suitable.
- Service-locator dependencies are allowed only with explicit rationale and scope-limited usage.

### Prohibited
- Do not treat legacy style as a blanket reason to add new field injection.
- Do not hide collaborator creation with manual instantiation in Spring-managed classes.
- Do not leak infrastructure APIs into core domain behavior without an abstraction boundary.

## Operational Workflow (Always Follow)
1. Identify whether the change is new code, legacy touch-up, or major refactor.
2. Apply mandatory DI rules first.
3. Apply preferred patterns when design choice is open.
4. Add or update DI-focused tests.
5. Verify bean wiring and selection behavior.

## Implementation Rules

### Legacy Code Consistency
- Existing `@Autowired` usage in legacy files is acceptable.
- When modifying existing files, follow the DI style already used in that file for local consistency.
- If converting a legacy class to constructor injection, keep the refactor scoped and intentional.
- If a class is substantially reshaped, prefer migrating it to constructor injection in that same change.

### Bean Selection and Configuration
- Use `@Primary` only when one implementation should be default across most injection points.
- Prefer custom qualifier annotations for type safety.
- Document non-obvious bean selection decisions in code comments or PR notes.

## Testing & Verification Rules
- Validate bean wiring explicitly, including qualifier resolution and profile-specific beans.
- Use `@MockBean` or test doubles to isolate external boundaries.
- Prefer focused Spring test slices before full-context tests when validating DI wiring.

## Review Findings Checklist
Raise findings when:
- New code introduces field injection without strong reason.
- Service-locator patterns are introduced without documented necessity.
- Constructor size suggests an unaddressed design smell.
- String qualifiers are used where safer alternatives exist.

## Migration / Refactor Patterns

### Field Injection to Constructor Injection
1. Convert injected fields to `final`.
2. Introduce explicit constructor parameters.
3. Update tests to use constructor-based setup or Spring wiring.
4. Keep behavioral changes out of the DI refactor.

## Examples

### Good
```java
@Service
public class InvoiceService {
    private final TaxCalculator taxCalculator;

    public InvoiceService(TaxCalculator taxCalculator) {
        this.taxCalculator = taxCalculator;
    }
}
```

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

### Avoid
```java
@Service
public class LegacyBillingService {
    @Autowired
    private PaymentGatewayClient paymentGatewayClient;
}
```

## Pre-Completion Checklist
- [ ] New/refactored classes use constructor injection with `final` fields.
- [ ] Infrastructure is wrapped behind domain-facing interfaces where needed.
- [ ] Bean selection choices are explicit and justified.
- [ ] DI behavior is covered by focused tests.

## Output Expectations for Agents
- Summarize DI design decisions made.
- Call out any legacy consistency tradeoffs.
- Report validation commands/tests executed.

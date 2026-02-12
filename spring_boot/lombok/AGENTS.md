# Lombok Usage Guidelines

## Purpose
Define when Lombok is allowed, restricted, or prohibited so code remains explicit, safe, and maintainable.

Lombok is a code-generation tool, not a design tool. Use it to remove mechanical boilerplate, not to hide domain intent or lifecycle behavior.

## Scope
Applies to Spring services, DTOs, value objects, entities, domain objects, tests, and framework-facing code in this repository.

## Core Principles
1. Explicit behavior beats implicit generation.
2. Immutability is the default.
3. Construction rules must be visible.
4. Domain meaning must not be hidden.
5. Tests must not rely on Lombok-generated behavior.

## Decision Rules

### Mandatory
- Use explicit Java when Lombok would obscure invariants, lifecycle, or security boundaries.
- Keep domain transitions explicit methods, not generated setters.
- Keep security-sensitive objects free of Lombok-generated representation methods.

### Preferred
- Use `@RequiredArgsConstructor` for constructor injection in Spring classes with `final` dependencies.
- Use `@Value` for immutable value objects.
- Keep Lombok use narrow and intentional.

### Restricted (Requires Justification)
- `@Builder` is allowed only for immutable value objects, test fixtures, or complex parameter objects.
- `@ToString` is allowed only when sensitive fields are excluded.
- `@EqualsAndHashCode` is allowed only for immutable value objects where identity semantics are not required.

### Prohibited
- `@Data` outside DTOs.
- `@Data` on entities, services, or domain objects with behavior.
- Lombok setters/builders on domain objects with invariants or lifecycle transitions.
- Lombok in security-sensitive classes (credentials, keys, signing/encryption payloads/configs).
- Lombok in public libraries/SDK/shared modules where API clarity is critical.
- Lombok in reflection/lifecycle-critical framework glue.

## Operational Workflow (Always Follow)
1. Determine whether the class carries data or domain meaning.
2. Apply mandatory and prohibited rules first.
3. If considering restricted annotations, document justification.
4. Add tests for behavior (not Lombok generation).
5. Verify no invariant or security risk was introduced.

## Implementation Rules

### Allowed Lombok Usage

#### Constructor Injection (Spring)
- Use constructor injection only.
- Keep dependencies `final`.
- Do not combine with `@AllArgsConstructor` in service classes.

#### Immutable Value Objects
- `@Value` is allowed when type is immutable, conceptually cohesive, and has no lifecycle transitions.

#### DTOs
- DTOs may use `@Data`, `@NoArgsConstructor`, and `@AllArgsConstructor` when they remain dumb data carriers.

#### Test Fixtures
- Immutable test fixture objects may use Lombok for brevity.

### Entity and Domain Safety Rules
- For JPA entities, only minimal Lombok is allowed (`@Getter`, protected no-arg constructor).
- Avoid generated equality for identity-based entities.
- For invariant-heavy domain classes, write constructors and behavior methods explicitly.

## Testing & Verification Rules
- Do not test Lombok-generated behavior directly.
- Assert explicit fields and business outcomes.
- Do not mock data objects only to consume generated accessors.
- Builders in tests are allowed only for immutable objects/value fixtures, never entities.

## Review Findings Checklist
Raise findings when:
- `@Data` appears outside DTO classes.
- Entity/domain invariants are hidden behind generated setters/builders.
- Security-sensitive objects use Lombok-generated `toString`/access patterns.
- Tests rely on generated equality instead of explicit behavior assertions.

## Migration / Refactor Patterns

### Removing Misused `@Data`
1. Replace with explicit constructors and methods for behavior-heavy classes.
2. Keep only minimal safe Lombok annotations where justified.
3. Re-run tests that validate invariants and lifecycle flows.

### Tightening Entity Lombok Usage
1. Remove broad Lombok annotations from entities.
2. Keep `@Getter` and protected no-arg constructor if needed.
3. Review equality and toString for proxy/lazy-loading safety.

## Examples

### Good
```java
@RequiredArgsConstructor
@Service
public class PaymentService {
    private final LedgerService ledgerService;
}
```

```java
@Value
public class Money {
    BigDecimal amount;
    String currency;
}
```

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class CreateOrderRequest {
    private String symbol;
    private BigDecimal quantity;
}
```

```java
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Entity
public class Order {
    @Id
    @GeneratedValue
    private Long id;
}
```

### Avoid
```java
@Data
@Entity
public class Order { ... }
```

```java
@Data
@Service
public class OrderService { ... }
```

## Pre-Completion Checklist
- [ ] Lombok usage classified as allowed/restricted/prohibited.
- [ ] Restricted annotations include explicit justification.
- [ ] Domain invariants remain explicit in code.
- [ ] Tests validate behavior instead of annotation side effects.

## Output Expectations for Agents
- State where Lombok was used and why.
- Call out any restricted usage and its justification.
- Report verification performed and residual risks.

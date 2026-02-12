---

# Simple Lombok Usage Guidelines (`AGENTS.md`)

## Purpose

This document defines **when Lombok is allowed, restricted, or prohibited** in this codebase.

Lombok is a **code-generation tool**, not a design tool.
Its role is to reduce *mechanical boilerplate*, **not** to hide intent, weaken invariants, or obscure lifecycle behaviour.

---

## Core Principles

1. **Explicit behaviour beats implicit generation**
2. **Immutability is the default**
3. **Construction rules must be visible**
4. **Domain meaning must not be hidden**
5. **Tests must not rely on Lombok-generated behaviour**

If Lombok reduces clarity or correctness, **do not use it**.

---

## Allowed Lombok Usage (Default-Approved)

### 1. Constructor Injection (Spring)

✅ **Recommended**

```java
@RequiredArgsConstructor
@Service
public class PaymentService {
    private final LedgerService ledgerService;
}
```

Rules:

* Use constructor injection only
* Fields must be `final`
* Do **not** combine with `@AllArgsConstructor`

---

### 2. Immutable Value Objects

✅ **Allowed**

```java
@Value
public class Money {
    BigDecimal amount;
    String currency;
}
```

Rules:

* Must be immutable
* Must represent a single concept
* No lifecycle or state transitions

---

### 3. DTOs (API / Messaging / Serialization)

✅ **Allowed**

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class CreateOrderRequest {
    private String symbol;
    private BigDecimal quantity;
}
```

Rules:

* DTOs must remain dumb data carriers
* No domain logic
* No invariants enforced here

---

### 4. Test Fixtures (Immutable Only)

✅ **Allowed**

```java
@Value
static class TestCase {
    BigDecimal input;
    BigDecimal expected;
}
```

Rules:

* Test-only
* Immutable
* Disposable

---

## Restricted Lombok Usage (Requires Justification)

### 1. `@Builder`

⚠️ **Use sparingly**

Allowed only for:

* Immutable value objects
* Test fixtures
* Complex parameter objects

❌ **Not allowed** for:

* JPA entities
* Spring beans
* Domain aggregates

---

### 2. `@ToString`

⚠️ **Use with extreme caution**

Rules:

* Must exclude sensitive fields
* Must never log secrets, credentials, or keys

Example:

```java
@ToString(exclude = "privateKey")
```

---

### 3. `@EqualsAndHashCode`

⚠️ **Explicit decision required**

Allowed for:

* Immutable value objects

Prohibited for:

* JPA entities
* Stateful domain objects
* Objects with identity semantics

---

## Prohibited Lombok Usage (Hard Ban)

### 1. `@Data` on Anything Except DTOs

❌ **BANNED**

```java
@Data
@Service
public class OrderService { ... }
```

Reasons:

* Generates setters (breaks invariants)
* Generates `equals/hashCode` (breaks proxies)
* Generates `toString()` (risk of leaks)

---

### 2. JPA Entities (Except Minimal Use)

❌ **BANNED**

```java
@Data
@Entity
public class Order { ... }
```

Allowed only:

* `@Getter`
* `@NoArgsConstructor(access = PROTECTED)`

Example:

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

---

### 3. Domain Objects with Invariants

❌ **BANNED**

If an object:

* Requires validation
* Enforces business rules
* Prevents illegal states

Then:

* Write constructors explicitly
* Write methods explicitly
* Do **not** use Lombok setters or builders

---

### 4. State Machines / Lifecycle Objects

❌ **BANNED**

Examples:

* Orders
* Trades
* KYC status
* Workflow states

Rules:

* No Lombok setters
* State transitions must be explicit methods

---

### 5. Security-Sensitive Objects

❌ **BANNED**

Includes:

* API keys
* Private keys
* Credentials
* Signing payloads
* Encryption configs

Rule:

> If logging this object would be a security incident, Lombok must not be used.

---

### 6. Public Libraries / SDKs / Shared Modules

❌ **BANNED**

Reasons:

* Lombok hides behaviour from consumers
* Generated methods are invisible in source
* Breaks API clarity

---

### 7. Framework Glue / Reflection-Critical Code

❌ **BANNED**

Includes:

* Serialization glue
* Proxy-heavy code
* Lifecycle hooks
* Reflection-based frameworks

Explicit Java is required.

---

## Lombok & Testing Rules

### 1. Do Not Test Lombok Behaviour

❌ **BANNED**

```java
assertEquals(a, b); // relies on Lombok equals()
```

✅ **Required**

* Assert explicit fields
* Assert behaviour, not annotations

---

### 2. Do Not Mock Data Objects

❌ **BANNED**

```java
when(order.getStatus()).thenReturn(PENDING);
```

Rule:

* Mock collaborators
* Use real data objects

---

### 3. Builders in Tests

Allowed only for:

* Immutable objects
* Value objects
* Test fixtures

❌ **Never for entities**

---

## Decision Rule (If Unsure)

Ask **one question**:

> Does this class carry **data**, or does it carry **meaning**?

* Data → Lombok is acceptable
* Meaning → Lombok should be minimal or absent

If you have to explain Lombok usage in a review, it’s probably wrong.

---

## Enforcement

Pull Requests **must be rejected** if they:

* Introduce `@Data` outside DTOs
* Use Lombok in entities incorrectly
* Hide invariants behind setters/builders
* Rely on Lombok-generated behaviour in tests

---

## Final Statement

Lombok does **not** simplify systems.
It simplifies **typing**.

Use it to remove noise—not responsibility.

---

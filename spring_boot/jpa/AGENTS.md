# AGENTS.md - Spring Data JPA Execution Guide for Codex

## Purpose

Use this file as an actionable playbook when working on Spring Boot / Spring Data JPA code in this repository.

Your job is to implement changes that are:

- Correct and production-safe
- Consistent with JPA best practices
- Easy to review (small, explicit, test-backed)

## Scope

Apply these rules when editing:

- `*Entity.java`
- repositories (`JpaRepository`, `CrudRepository`, or custom data access)
- DTOs and mappers
- service methods that query/update persistence models

If a user request conflicts with these rules, follow the user request and call out the tradeoff clearly.

## Execution Workflow (Always Follow)

1. Identify whether the change affects entity mapping, querying, DTO shape, or equality semantics.
2. Make the smallest correct code change first.
3. Enforce the relevant rules from this guide.
4. Add/update tests for behavior changes.
5. Run available tests/verification commands.
6. Summarize what changed, why, and any residual risks.

## Entity Rules

1. Every entity must declare a primary key using `@Id` or `@EmbeddedId`.
2. Ensure a no-arg constructor exists (`public` or `protected`).
3. Prefer explicit JPQL entity names:
   - Use `@Entity(name = "...")`.
   - Do not depend on class simple names in JPQL.
4. Use `@Table(...)` only when overriding defaults (name/schema/constraints).
5. Do not assume one-field-to-one-column mapping; handle `@Transient`, embeddables, and relations intentionally.

## Querying Rules

1. Prefer Spring Data repositories over ad-hoc `EntityManager` usage.
2. Never build JPQL/SQL with string concatenation when parameters are involved.
3. Use named/positional parameters and typed queries.
4. Keep JPQL entity name (`@Entity(name)`) separate from physical table name (`@Table(name)`).

## Type and Enum Rules

1. Use the same Java type for the same conceptual column across entities.
2. For nullable DB columns, use wrapper types (`Integer`, `Boolean`, etc.).
3. For money/precision values, use `BigDecimal` with explicit precision/scale in mapping where relevant.
4. Replace magic numeric status/flag codes with enums in Java.
5. Default enum persistence strategy: `@Enumerated(EnumType.STRING)`.
6. If DB must store legacy numeric codes (e.g., `1..4`), use `AttributeConverter<Enum, Integer>`.
7. Never use `EnumType.ORDINAL` for legacy code compatibility.

## Lombok Rules

1. Prefer targeted Lombok annotations on entities:
   - `@Getter`
   - `@Setter`
   - `@NoArgsConstructor(access = AccessLevel.PROTECTED)` (or equivalent)
2. Avoid `@Data` on entities.
3. Only customize getters/setters when preprocessing/postprocessing is required.

## equals/hashCode Rules for Entities

Only implement custom `equals/hashCode` when needed (cross-context value semantics, `Set`/`Map` behavior).

Allowed patterns:

1. Business-key equality:
   - Key must be unique, immutable, and always present.
2. ID-based equality (DB-generated IDs):
   - Treat transient entities (`id == null`) as unequal to other instances.
   - Use stable hash strategy to avoid hash bucket changes after persistence.

Avoid:

- All-fields equality on entities
- Including lazy associations in equality/toString

If using Lombok equality:

- Use `@EqualsAndHashCode(onlyExplicitlyIncluded = true)`
- Include only stable identifiers
- Exclude relations with `@EqualsAndHashCode.Exclude` / `@ToString.Exclude`

## DTO and Mapping Rules

1. Do not expose entities directly from public/client-facing APIs.
2. Create use-case-specific DTOs (`Summary`, `Detail`, `CreateRequest`, `UpdateRequest`, `Response`).
3. To reduce DB load, select only needed columns:
   - constructor projections
   - interface-based projections
   - native queries returning only required fields
4. Mapping full entities to DTOs does not reduce selected columns.
5. Prefer MapStruct for mapper generation when mapper complexity justifies it.

## Repository and Service Boundaries

1. Controllers should call services, not repositories directly (unless codebase conventions explicitly differ).
2. Keep business logic in services; keep repositories focused on data access.
3. Shared base repositories should be marked with `@NoRepositoryBean` when appropriate.

## Repository Design Rules (Operational)

### Repository Interface Choice

1. Default to `JpaRepository<Entity, IdType>` unless access must be restricted.
2. If restricting operations, expose only required methods via `Repository` or a narrow base interface.
3. Mark shared base repository interfaces with `@NoRepositoryBean`.

### Query Authoring Rules

1. Use derived query methods for simple predicates.
2. Use `@Query` or `@NativeQuery` for complex logic.
3. Never concatenate input into JPQL/SQL strings.
4. For `UPDATE`/`DELETE`, require `@Modifying` and an explicit transaction boundary.

### Transaction Boundary Rules for Repositories

1. Do not rely on implicit transactional behavior for custom declared query methods.
2. Put transactional boundaries in service methods by default.
3. If repository-level transaction is required, declare `@Transactional` explicitly.

### Projection-First Read Rules

1. Do not return entities when only partial read data is needed.
2. Prefer interface or DTO projections for list/read endpoints.
3. Validate projection SQL shape when nested fields are used.
4. For DTO projections, ensure constructor arguments match selected columns.

### Pagination and Throughput Rules

1. For metadata (total pages/elements), use `Page<T>`.
2. For streaming or infinite-scroll reads, use `Slice<T>`.
3. For high-volume writes, prefer batch-oriented operations and JDBC batching configuration.
4. Avoid `save(...)` loops when `saveAll(...)` or set-based queries are possible.

### Stored Procedure Rules

1. Prefer `@Procedure` with named metadata for reusable procedures.
2. Use direct `@Procedure(procedureName = "...")` only when metadata reuse is unnecessary.
3. Document IN/OUT parameter mapping in repository method signatures.
4. Add tests for procedure result mapping, especially multi-OUT cases.

## Transactions and Manual Queries Rules (Operational)

### Transaction Management Strategy

1. Default to declarative transactions with `@Transactional` on service-layer public methods.
2. Use `TransactionTemplate` when you need explicit programmatic transaction boundaries with Spring integration.
3. Avoid manual `EntityTransaction` management from `EntityManagerFactory` unless a special low-level case requires it.
4. If manual transaction management is unavoidable, explicitly handle begin, commit, rollback, and resource closing.

### `@Transactional` Proxy Constraints

1. Do not rely on self-invocation for transactional behavior; proxy interception is bypassed for internal method calls.
2. Do not place `@Transactional` on private methods; use public service entry points.
3. Remember default rollback semantics: rollback occurs for `RuntimeException` and `Error`, not checked exceptions.
4. For checked exceptions that must roll back, declare rollback rules explicitly.

### Manual Querying With `EntityManager`

1. Prefer repositories first; use `EntityManager` for cases repositories cannot express cleanly.
2. Prefer `TypedQuery<T>` over raw `Query` to avoid unsafe casts.
3. Never concatenate user input into JPQL/SQL strings.
4. Use named parameters (`:name`) by default; avoid positional parameters except where strictly necessary.
5. For native scalar results, cast to `Number` first, then convert (`longValue()`, etc.) to avoid driver-specific cast failures.

### Criteria API and Dynamic Filters

1. Use Criteria API for dynamic query composition when predicates vary at runtime.
2. Build predicates as structured conditions, not string fragments.
3. Keep Criteria usage focused and minimal to avoid unnecessary complexity.

### Stored Procedures via `EntityManager`

1. Register all stored procedure parameters before assigning values for readability.
2. Keep IN/OUT parameter names and Java types explicit.
3. Add tests for procedure parameter mapping and output handling.

### Projection Rules With `EntityManager`

1. If only a subset of fields is needed, prefer query-time DTO projection over fetching full entities and mapping in memory.
2. For JPQL constructor projections, keep constructor signatures aligned with selected columns and verify fully qualified DTO names.
3. For native query projections, avoid `Object[]` index-based mapping for business-critical paths.
4. Prefer alias-based `Tuple` mapping for readability and safer column access.

## What to Flag in Reviews

Raise findings when you see:

- `@Data` on entities
- `EnumType.ORDINAL` used for externally meaningful codes
- String-concatenated JPQL/SQL
- Inconsistent field types for same DB concept
- Entity leakage in API response models
- Equality implementations that depend on mutable fields/relations

## Migration Patterns

### Numeric status code -> enum with converter

1. Introduce enum with explicit business names.
2. Add `AttributeConverter<Enum, Integer>` with explicit mappings.
3. Replace numeric field type in entity with enum type.
4. Keep DB column unchanged if required.
5. Add tests for all mappings and unknown-code handling.

### Entity return type -> DTO projection

1. Define response DTO or projection interface.
2. Change repository query to return projected shape.
3. Update service/controller contract.
4. Add API/service tests verifying shape and fields.

## Operational Examples for Agents

### 1) Service Transaction Boundary (Preferred)

```java
@Service
@RequiredArgsConstructor
public class UserService {
    private final UserRepository userRepository;

    @Transactional
    public void createUser(String name) {
        userRepository.save(new User(name));
    }
}
```

Why operational: transaction boundary is on a public service method, not controller/repository.

### 2) Checked Exception Rollback Rule

```java
@Transactional(rollbackFor = IOException.class)
public void importUsers(Path file) throws IOException {
    // read + persist
}
```

Why operational: explicit rollback for checked exception.

### 3) Avoid Self-Invocation Bug

```java
@Service
@RequiredArgsConstructor
public class TicketService {
    private final TicketTxService ticketTxService;

    public void bookBatch() {
        ticketTxService.reserveSeat();
    }
}

@Service
class TicketTxService {
    @Transactional
    public void reserveSeat() { /* save */ }
}
```

Why operational: transactional method is called through proxy (separate bean), not `this.reserveSeat()`.

### 4) Modifying Query With Explicit Transaction

```java
public interface EmployeeRepository extends JpaRepository<Employee, Long> {

    @Modifying
    @Transactional
    @Query("update Employee e set e.active = false where e.id = :id")
    int deactivate(@Param("id") Long id);
}
```

Why operational: `@Modifying` + transaction boundary present.

### 5) Safe Named Parameters (No Concatenation)

```java
TypedQuery<User> q = em.createQuery(
    "select u from User u where u.id = :id and u.status = :status",
    User.class
);
q.setParameter("id", id);
q.setParameter("status", Status.ACTIVE);
```

Why operational: named parameters, typed query, injection-safe.

### 6) Native Scalar Conversion Safety

```java
Object raw = em.createNativeQuery("select id from version_cfg where rownum = 1")
               .getSingleResult();
Long id = raw != null ? ((Number) raw).longValue() : null;
```

Why operational: avoids DB-driver-specific cast failures.

### 7) Dynamic Filter With Criteria API

```java
CriteriaBuilder cb = em.getCriteriaBuilder();
CriteriaQuery<Long> cq = cb.createQuery(Long.class);
Root<User> u = cq.from(User.class);

List<Predicate> p = new ArrayList<>();
p.add(cb.equal(u.get("username"), username));
if (status != null) p.add(cb.equal(u.get("status"), status));

cq.select(cb.count(u)).where(p.toArray(new Predicate[0]));
Long count = em.createQuery(cq).getSingleResult();
```

Why operational: dynamic predicates without string-built SQL/JPQL.

### 8) Projection-First Read (Repository DTO)

```java
public record UserSummary(Long id, String username) {}

public interface UserRepository extends JpaRepository<User, Long> {
    @Query("select new com.example.UserSummary(u.id, u.username) from User u where u.active = true")
    List<UserSummary> findActiveSummaries();
}
```

Why operational: returns only required columns; avoids entity leakage.

### 9) EntityManager Projection With Tuple

```java
List<Tuple> rows = em.createNativeQuery(
    "select u.id as id, u.username as username from users u where u.active = 1",
    Tuple.class
).getResultList();

List<UserSummary> out = rows.stream()
    .map(t -> new UserSummary(t.get("id", Long.class), t.get("username", String.class)))
    .toList();
```

Why operational: alias-based access, avoids fragile `Object[]` index mapping.

### 10) Stored Procedure Pattern (Readable)

```java
StoredProcedureQuery sp = em.createStoredProcedureQuery("pkg_user.recalc");
sp.registerStoredProcedureParameter("p_user_id", Long.class, ParameterMode.IN);
sp.registerStoredProcedureParameter("p_result_id", Long.class, ParameterMode.OUT);
sp.setParameter("p_user_id", userId);
sp.execute();
Long resultId = (Long) sp.getOutputParameterValue("p_result_id");
```

Why operational: registers params first, explicit IN/OUT typing, maintainable flow.

## Pre-Completion Checklist

Before finishing any persistence-related task, verify:

1. Mapping correctness: annotations, column names, nullability assumptions.
2. Query safety: parameterized queries, no string-built query fragments from inputs.
3. Type consistency across touched entities.
4. Enum strategy correctness (`STRING` or explicit converter).
5. Equality semantics unchanged or intentionally updated with tests.
6. DTO boundaries preserved.
7. Tests added/updated and executed.

## Output Format for Codex Final Responses

When completing a task in this repo, include:

1. What changed (files + behavior)
2. Why it changed (rule/practice applied)
3. Validation run (tests/commands)
4. Remaining risks or follow-ups (if any)

Keep it concise and actionable.

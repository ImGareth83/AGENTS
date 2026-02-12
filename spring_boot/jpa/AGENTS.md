# Spring Data JPA Execution Guide

## Purpose
Provide an actionable playbook for Spring Boot / Spring Data JPA changes that are correct, production-safe, and reviewable.

## Scope
Apply these rules when editing:
- `*Entity.java`
- Spring Data repositories (`JpaRepository`, `CrudRepository`, custom data access)
- DTOs and mappers
- service methods that query or update persistence models

If a user request conflicts with this guide, follow the user request and call out tradeoffs.

## Core Principles
1. Correctness and safety first.
2. Keep persistence concerns explicit and test-backed.
3. Prefer smallest correct change before broader refactors.

## Decision Rules

### Mandatory
- Every entity must declare a primary key using `@Id` or `@EmbeddedId`.
- Entities must have a no-arg constructor (`public` or `protected`).
- Never concatenate user input into JPQL or SQL.
- Use parameterized queries and typed queries where possible.
- Do not expose entities directly from public/client-facing APIs.
- Use explicit transaction boundaries for update/delete queries.
- For externally meaningful enums, do not use `EnumType.ORDINAL`.

### Preferred
- Prefer explicit JPQL entity names with `@Entity(name = "...")`.
- Prefer repositories over ad-hoc `EntityManager` usage.
- Use wrapper types (`Integer`, `Boolean`) for nullable DB columns.
- Use `BigDecimal` with explicit precision/scale for money/precision values.
- Use `@Enumerated(EnumType.STRING)` by default.
- Prefer projection-first reads (DTO/interface projections) when only partial fields are needed.
- Keep transactional boundaries in service-layer public methods.

### Restricted (Requires Justification)
- Repository-level transactions are allowed when service-layer boundaries are not practical; document why.
- `EntityManager` is allowed for queries repositories cannot express cleanly.
- Native queries with `Object[]` mapping are allowed only outside business-critical paths.

### Prohibited
- All-fields `equals/hashCode` on entities.
- Including lazy relations in `equals`, `hashCode`, or `toString`.
- String-built query fragments from runtime input.
- Returning full entities when only partial read data is required.

## Operational Workflow (Always Follow)
1. Identify whether change affects mapping, querying, DTO shape, transaction boundaries, or equality semantics.
2. Make the smallest correct code change first.
3. Apply relevant rules from this guide.
4. Add/update tests for behavior changes.
5. Run verification commands.
6. Summarize what changed, why, and residual risk.

## Implementation Rules

### Entity Rules
- Use `@Table(...)` only when overriding defaults (name/schema/constraints).
- Handle `@Transient`, embeddables, and relations intentionally.

### Query Authoring Rules
- Use derived query methods for simple predicates.
- Use `@Query` / native query annotations for complex logic.
- For `UPDATE`/`DELETE`, require `@Modifying` and an explicit transaction boundary.
- Keep JPQL entity name (`@Entity(name)`) distinct from physical table name (`@Table(name)`).

### Type and Enum Rules
- Use the same Java type for the same conceptual column across entities.
- Replace magic numeric status/flag codes with enums in Java.
- If DB must keep numeric legacy codes, use `AttributeConverter<Enum, Integer>`.

### Lombok Rules for Entities
- Prefer targeted annotations (`@Getter`, `@Setter`, protected no-arg constructor).
- Avoid `@Data` on entities.
- Customize accessors only for necessary preprocessing/postprocessing.

### Entity Equality Rules
- Implement custom `equals/hashCode` only when needed for value semantics or `Set`/`Map` behavior.
- Business-key equality is allowed only when key is unique, immutable, and always present.
- For ID-based equality with generated IDs:
  - treat transient entities (`id == null`) as unequal to other instances,
  - use stable hash strategy to avoid hash bucket changes after persistence.
- If using Lombok equality, include only stable identifiers and exclude relations.

### DTO and Mapping Rules
- Use use-case-specific DTOs (`Summary`, `Detail`, `CreateRequest`, `UpdateRequest`, `Response`).
- Mapping full entities to DTOs does not reduce selected columns.
- Prefer MapStruct when mapper complexity justifies generation.

### Repository and Service Boundary Rules
- Controllers should call services, not repositories directly (unless explicit codebase convention differs).
- Keep business logic in services and data access in repositories.
- Mark shared base repositories with `@NoRepositoryBean` when appropriate.

### Repository Interface and Pagination Rules
- Default to `JpaRepository<Entity, IdType>` unless operation surface must be restricted.
- For metadata requirements use `Page<T>`.
- For infinite scroll/streaming style reads use `Slice<T>`.
- For high-volume writes prefer batch operations and avoid `save(...)` loops where set-based operations fit.

### Transaction and Manual Query Rules
- Use declarative transactions (`@Transactional`) on service public methods by default.
- Use `TransactionTemplate` when explicit programmatic boundaries are needed.
- Avoid manual `EntityTransaction` unless low-level constraints require it.
- Do not rely on self-invocation for transactional behavior.
- Do not place `@Transactional` on private methods.
- Remember default rollback is for `RuntimeException` and `Error`; specify rollback for checked exceptions when required.
- With native scalar results, cast to `Number` before converting to `longValue()`/`intValue()`.

### Criteria and Stored Procedure Rules
- Use Criteria API for dynamic runtime predicates.
- Build predicates structurally, not via string fragments.
- For stored procedures:
  - prefer repository `@Procedure` with named metadata for reusable cases,
  - keep IN/OUT parameter names and Java types explicit,
  - register all parameters before assigning values in `EntityManager` flows,
  - add tests for output mapping, especially multi-OUT cases.

## Testing & Verification Rules
- Validate mapping correctness (annotations, nullability assumptions, type alignment).
- Validate query safety (no concatenation; parameterized usage).
- Validate enum persistence strategy and converter mappings.
- Validate projection shape and constructor argument alignment.
- Validate transaction boundaries and rollback behavior (including checked exceptions when configured).
- Add/update tests for equality semantic changes.

## Review Findings Checklist
Raise findings when:
- `@Data` appears on entities.
- `EnumType.ORDINAL` is used for externally meaningful values.
- JPQL/SQL is string-concatenated with runtime inputs.
- Same DB concept uses inconsistent Java types.
- Entities leak into public API response models.
- Equality depends on mutable fields or lazy relations.

## Migration / Refactor Patterns

### Numeric Status Code to Enum + Converter
1. Introduce enum with explicit business names.
2. Add `AttributeConverter<Enum, Integer>` with explicit mappings.
3. Replace numeric entity field type with enum type.
4. Keep DB column unchanged if required.
5. Add tests for all mappings and unknown code handling.

### Entity Return Type to DTO Projection
1. Define DTO/projection interface for required read shape.
2. Change repository query to projected return type.
3. Update service/controller contract.
4. Add API/service tests validating response shape and fields.

## Examples

### Good
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

```java
@Transactional(rollbackFor = IOException.class)
public void importUsers(Path file) throws IOException {
    // read + persist
}
```

```java
TypedQuery<User> q = em.createQuery(
    "select u from User u where u.id = :id and u.status = :status",
    User.class
);
q.setParameter("id", id);
q.setParameter("status", Status.ACTIVE);
```

```java
Object raw = em.createNativeQuery("select id from version_cfg where rownum = 1")
               .getSingleResult();
Long id = raw != null ? ((Number) raw).longValue() : null;
```

```java
public record UserSummary(Long id, String username) {}

public interface UserRepository extends JpaRepository<User, Long> {
    @Query("select new com.example.UserSummary(u.id, u.username) from User u where u.active = true")
    List<UserSummary> findActiveSummaries();
}
```

### Avoid
```java
String jpql = "select u from User u where u.name = '" + name + "'";
```

## Pre-Completion Checklist
- [ ] Mapping correctness validated.
- [ ] Query safety validated.
- [ ] Type and enum strategy consistency preserved.
- [ ] DTO boundaries preserved.
- [ ] Tests added/updated and executed.

## Output Expectations for Agents
Final response should include:
1. What changed (files + behavior).
2. Why it changed (rule/practice applied).
3. Validation run (tests/commands).
4. Remaining risks or follow-ups.

# Spring Profiles Configuration Management

## Purpose
Define operational guidance for using Spring Profiles in Spring Boot applications so environment-specific configuration is managed consistently across development, testing, staging, and production.

## Scope
Applies to:
- profile naming
- profile-based property files
- default profile behavior
- sensitive configuration handling
- profile-conditional bean loading
- multiple active profile combinations
- centralized/externalized configuration references

## Core Principles
1. Profiles should control configuration selection only.
2. Configuration must remain separated from business logic.
3. Environment configuration should be explicit and maintainable.

## Decision Rules

### Mandatory
- Use profiles to activate environment-specific configuration.
- Separate base configuration from profile-specific overrides.
- Define a default profile behavior for predictable startup.
- Externalize sensitive values such as passwords, API keys, and credentials.
- Keep profile-based bean loading clearly organized.

### Preferred
- Use clear environment-oriented profile names.
- Keep profile combinations predictable and controlled.
- Use centralized configuration approaches where applicable.

### Restricted (Requires Justification)
- Multiple active profiles are allowed when combinations are intentional and controlled.
- Profile-conditional beans are allowed when environment differences require distinct bean activation.

### Prohibited
- Do not place profile checks inside business logic.
- Do not overload one configuration file with all environment properties.
- Do not store secrets directly in profile files.

## Operational Workflow (Always Follow)
1. Define environments and profile names.
2. Keep shared settings in base configuration.
3. Move environment overrides to `application-{profile}.properties`.
4. Define default profile/fallback behavior.
5. Externalize sensitive values.
6. Validate profile-specific and multi-profile startup behavior.

## Implementation Rules

### Purpose of Spring Profiles
Spring Profiles provide a mechanism to:
- activate environment-specific configuration,
- separate configuration from application logic,
- manage multiple runtime environments consistently.

### Common Operational Issues

#### Hardcoded Profile Logic
Do not place profile checks inside business logic.

Example (illustrative misuse):
```java
if (activeProfile.equals("prod")) {
    // profile-specific logic
}
```

Profile handling belongs to configuration, not business flow.

#### Overloaded Configuration Files
Avoid placing all properties in a single configuration file.

Operational impact:
- reduced maintainability
- difficult environment separation
- higher risk of configuration mistakes

#### Missing Default Profile
If no profile is specified, Spring uses default configuration.

Failure to define a default profile may result in:
- unexpected startup behavior
- incorrect configuration loading

#### Unclear Profile-Based Bean Definitions
Poor organization of `@Profile` usage may lead to unclear bean loading behavior and configuration ambiguity.

### Profile Naming Rules
Use names that represent environments.

Standard examples:
```text
dev
test
staging
prod
integration
performance
```

Profile names should describe deployment context only.

### Profile-Specific Configuration Files
Separate environment settings using profile-based files.

Expected structure:
```text
application.properties
application-dev.properties
application-prod.properties
```

- Base file: shared configuration
- Profile files: environment-specific overrides

### Default Profile Definition
A default profile should be defined in base configuration.

Purpose:
- provides fallback configuration
- ensures predictable startup behavior

Runtime override remains possible.

Runtime override example:
```bash
-Dspring.profiles.active=prod
```

### Sensitive Configuration Handling
Do not store sensitive values directly in profile files.

Sensitive values include:
- passwords
- API keys
- credentials

Use externalized configuration instead.

Example:
```properties
spring.datasource.password=${DB_PASSWORD}
```

External configuration tools (for example Spring Cloud Config) may be used.

### Profile-Specific Beans
Profiles may be used to activate beans conditionally when environment differences require it.

Example (article-style usage):
```java
@Profile("dev")
@Bean
public SomeBean devBean() {
    return new SomeBean();
}
```

### Multiple Active Profiles
More than one profile can be activated simultaneously.

Example:
```properties
spring.profiles.active=dev,test
```

Operationally, profile combinations must be predictable and controlled.

### Configuration Organization Model
Recommended separation:
```text
application.properties
 ├─ shared settings
 ├─ default profile

application-dev.properties
 ├─ development configuration

application-prod.properties
 ├─ production configuration
```

### Advanced Configuration Management
Referenced externalized approaches include:
- centralized configuration
- version-controlled configuration repositories
- Spring Cloud Config

### Operational Summary
Spring Profiles should be used to:
- separate environment configuration
- maintain clean configuration boundaries
- avoid embedding environment decisions inside application logic

Key operational practices:
- use clear profile names
- maintain separate profile files
- define a default profile
- externalize sensitive configuration
- organize profile-based beans clearly

### Reference Structure (Operational Overview)
```text
Spring Profiles
 ├─ Environment-specific configuration
 ├─ Common issues:
 │   ├─ hardcoded logic
 │   ├─ overloaded config
 │   ├─ missing default profile
 │   └─ unclear @Profile usage
 ├─ Best practices:
 │   ├─ naming conventions
 │   ├─ profile property files
 │   ├─ default profile
 │   └─ external secrets
 └─ Advanced usage:
     ├─ profile-based beans
     ├─ multiple profiles
     └─ centralized configuration
```

## Testing & Verification Rules
- Verify startup with default profile behavior.
- Verify startup with explicit profile override.
- Verify profile-specific property loading.
- Verify profile-conditional beans load as intended.
- Verify no sensitive values are hardcoded in profile files.

## Review Findings Checklist
Raise findings when:
- profile checks appear in business logic
- base configuration is overloaded with environment-specific properties
- no default profile behavior is defined
- `@Profile` usage is ambiguous
- secrets are committed in configuration files

## Migration / Refactor Patterns
1. Move hardcoded profile branching out of business services into configuration mechanisms.
2. Split overloaded config into `application.properties` plus `application-{profile}.properties`.
3. Replace hardcoded secrets with environment-variable placeholders.
4. Clarify `@Profile` bean boundaries per environment.

## Examples

### Good
```properties
spring.datasource.password=${DB_PASSWORD}
```

```properties
spring.profiles.active=dev,test
```

```java
@Profile("dev")
@Bean
public SomeBean devBean() {
    return new SomeBean();
}
```

### Avoid
```java
if (activeProfile.equals("prod")) {
    // profile-specific logic
}
```

## Pre-Completion Checklist
- [ ] Environment-oriented profile naming is used.
- [ ] Base and profile-specific files are clearly separated.
- [ ] Default profile behavior is defined.
- [ ] Sensitive values are externalized.
- [ ] Profile bean loading logic is clear.

## Output Expectations for Agents
- Summarize profile-related changes and rationale.
- Identify which rules from this guide were applied.
- Report validation steps and any remaining profile/configuration risks.

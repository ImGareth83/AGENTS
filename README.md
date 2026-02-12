# Agents Learning Repository

This repository currently serves as a curated set of agent instructions and coding standards, primarily for Spring Boot development practices and commit hygiene.

## What Is Here

- `commit/AGENTS.md`: Conventional Commits 1.0.0 rules and examples.
- `spring_boot/dependencyInjection/AGENTS.md`: Dependency injection policy and testing guidance.
- `spring_boot/jpa/AGENTS.md`: Spring Data JPA execution guide and persistence rules.
- `spring_boot/lombok/AGENTS.md`: Lombok usage policy (allowed, restricted, prohibited cases).

## Repository Structure

```text
.
├── commit/
│   └── AGENTS.md
└── spring_boot/
    ├── dependencyInjection/
    │   └── AGENTS.md
    ├── jpa/
    │   └── AGENTS.md
    └── lombok/
        └── AGENTS.md
```

## Primary Goals

- Standardize commit messages using Conventional Commits.
- Enforce consistent Spring dependency injection patterns.
- Apply safe, explicit, and testable JPA practices.
- Use Lombok intentionally without hiding domain behavior or invariants.

## How To Use This Repo

1. Choose the relevant topic folder (`commit/` or `spring_boot/*`).
2. Read the corresponding `AGENTS.md` before implementing changes.
3. Apply the rules directly in code and tests.
4. Keep changes small and review-friendly.

## Current Status

This repo is instruction-first and does not yet include runnable sample projects or exercises.  
If you add code examples later, place them in topic-specific subdirectories and keep the associated `AGENTS.md` as the source of truth for standards.

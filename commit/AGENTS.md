# Commit Message Guidelines

## Purpose
Follow Conventional Commits 1.0.0 so commit history is explicit and supports automation and SemVer signaling.

## Scope
Applies to all commits in this repository.

## Core Principles
1. Commit intent must be explicit from the subject line.
2. Breaking changes must be called out unambiguously.
3. Commit messages must be machine-parseable and human-readable.

## Decision Rules

### Mandatory
- You MUST use this structure:

```text
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

- You MUST prefix every commit with a type followed by `:` and a space (for example, `fix: ...`).
- You MUST use `feat` when a commit adds a new feature.
- You MUST use `fix` when a commit represents a bug fix.
- You MUST place the short description immediately after `: `.
- If a body is present, it MUST start one blank line after the description.
- If footers are present, they MUST start one blank line after the body.
- Each footer token MUST be followed by `: ` or ` #`, then a value.
- Footer tokens MUST use `-` instead of spaces, except `BREAKING CHANGE`.
- `BREAKING-CHANGE` MUST be treated as synonymous with `BREAKING CHANGE`.
- Breaking changes MUST be indicated by either:
  1. `!` before the `:` in the type/scope prefix, or
  2. a `BREAKING CHANGE: ...` footer.
- `BREAKING CHANGE` MUST be uppercase.

### Preferred
- Use a meaningful scope when it improves clarity (for example, `fix(parser): ...`).
- Keep the subject concise and action-oriented.
- Add body text when context is needed for reviewers.

### Restricted (Requires Justification)
- Non-standard types may be used only when they are commonly understood by the team and remain parseable.

### Prohibited
- Do not omit the commit type prefix.
- Do not use malformed footer tokens.
- Do not hide breaking changes in body text without `!` or `BREAKING CHANGE`.

## Operational Workflow (Always Follow)
1. Identify whether the change is a feature, fix, docs update, refactor, or maintenance work.
2. Choose the correct type (and optional scope).
3. Write a concise subject line.
4. Add body and footers only when needed.
5. Validate breaking change signaling.

## Implementation Rules

### Allowed Types
- Standard types include: `feat`, `fix`, `build`, `chore`, `ci`, `docs`, `style`, `refactor`, `perf`, `test`.

### Semantic Version Meaning
- `fix` correlates with PATCH.
- `feat` correlates with MINOR.
- `BREAKING CHANGE` correlates with MAJOR.

## Testing & Verification Rules
- Validate format before finalizing a commit:
  - Subject follows `<type>[scope]: <description>`.
  - Optional body and footer spacing is correct.
  - Breaking changes are marked correctly.

## Review Findings Checklist
Raise findings when:
- Type prefix is missing or invalid.
- Breaking changes are present but not signaled with `!` or `BREAKING CHANGE`.
- Footer formatting violates token rules.

## Migration / Refactor Patterns
Not applicable for this document.

## Examples

### Good
```text
feat: allow provided config object to extend other configs

BREAKING CHANGE: `extends` key in config file is now used for extending other config files
```

```text
feat!: send an email to the customer when a product is shipped
```

```text
feat(api)!: send an email to the customer when a product is shipped
```

```text
chore!: drop support for Node 6

BREAKING CHANGE: use JavaScript features not available in Node 6.
```

```text
docs: correct spelling of CHANGELOG
```

```text
feat(lang): add Polish language
```

```text
fix: prevent racing of requests

Introduce a request id and a reference to latest request. Dismiss
incoming responses other than from latest request.

Remove timeouts which were used to mitigate the racing issue but are
obsolete now.

Reviewed-by: Z
Refs: #123
```

### Avoid
```text
update parser handling
```

## Pre-Completion Checklist
- [ ] Format is Conventional Commits compliant.
- [ ] Type and optional scope accurately reflect the change.
- [ ] Breaking change signaling is explicit when required.

## Output Expectations for Agents
- Report the final commit message.
- State why the selected type/scope was used.

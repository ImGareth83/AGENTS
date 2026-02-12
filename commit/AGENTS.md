# Instructions: Conventional Commits 1.0.0

## Purpose
Follow the Conventional Commits specification so commit history is explicit and supports automation and SemVer signaling.

## Commit Message Format
Use this exact structure:

```text
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

## Required Rules
- You MUST prefix every commit with a type followed by `:` and a space. Example: `fix: ...`.
- You MUST use `feat` when a commit adds a new feature.
- You MUST use `fix` when a commit represents a bug fix.
- You MAY use other types such as `build`, `chore`, `ci`, `docs`, `style`, `refactor`, `perf`, `test`.
- You MAY provide a scope in parentheses after the type. It MUST be a noun describing a section of the codebase. Example: `fix(parser): ...`.
- You MUST place the short description immediately after the `: `.
- You MAY include a body. If present, it MUST start one blank line after the description.
- The body MAY contain multiple paragraphs.
- You MAY include one or more footers. If present, they MUST start one blank line after the body.
- Each footer MUST be a token followed by `: ` or ` #`, then a value. Example: `Reviewed-by: Z`.
- Footer tokens MUST use `-` instead of spaces, except `BREAKING CHANGE` which is allowed as a token.
- `BREAKING-CHANGE` MUST be treated as synonymous with `BREAKING CHANGE`.

## Breaking Changes
- You MUST indicate breaking changes either by adding `!` before the `:` in the type/scope prefix, or by adding a footer `BREAKING CHANGE: ...`.
- If you use `!`, you MAY omit the `BREAKING CHANGE:` footer and the description should explain the breaking change.
- `BREAKING CHANGE` MUST be uppercase.

## Semantic Meaning
- `fix` correlates with PATCH.
- `feat` correlates with MINOR.
- `BREAKING CHANGE` correlates with MAJOR.

## Examples
1. Commit message with description and breaking change footer
```text
feat: allow provided config object to extend other configs

BREAKING CHANGE: `extends` key in config file is now used for extending other config files
```

2. Commit message with `!` to draw attention to breaking change
```text
feat!: send an email to the customer when a product is shipped
```

3. Commit message with scope and `!` to draw attention to breaking change
```text
feat(api)!: send an email to the customer when a product is shipped
```

4. Commit message with both `!` and BREAKING CHANGE footer
```text
chore!: drop support for Node 6

BREAKING CHANGE: use JavaScript features not available in Node 6.
```

5. Commit message with no body
```text
docs: correct spelling of CHANGELOG
```

6. Commit message with scope
```text
feat(lang): add Polish language
```

7. Commit message with multi-paragraph body and multiple footers
```text
fix: prevent racing of requests

Introduce a request id and a reference to latest request. Dismiss
incoming responses other than from latest request.

Remove timeouts which were used to mitigate the racing issue but are
obsolete now.

Reviewed-by: Z
Refs: #123
```

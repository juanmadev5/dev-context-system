---
tags: [global, code-review]
---

# Code Review

Rules for reviewing code in any project — whether the reviewer is a human or an agent running a review (e.g. via a `/code-review`-style command). Each stack's `code-review.md` only adds what's specific to that stack (performance and security checks, anti-pattern code examples, tooling) and never contradicts this note.

## Review pass order

Review in this order — earlier passes catch the issues that matter most and can make later passes moot:

1. **Correctness** — does it do what it claims to do? Edge cases, null/undefined handling, off-by-one errors, incorrect conditionals.
2. **Security** — see [Security checklist](#security-checklist) below.
3. **Architecture/consistency** — does it respect the layer/slice boundaries in the stack's `architecture-principles.md` and the conventions in the stack's own note?
4. **Performance** — the stack's `code-review.md` lists what to look for.
5. **Tests** — is coverage present where [engineering-principles](engineering-principles.md)'s testing criteria call for it?
6. **Style** — lowest priority, never blocking on its own.

## Severity classification

- **Blocking**: correctness bugs, security vulnerabilities, magic values (see [engineering-principles](engineering-principles.md)), duplicated logic, architecture-boundary violations, broken build/tests, missing error handling at a system boundary.
- **Non-blocking (nit)**: naming preferences, micro-optimizations, suggested comments, pure style.
- A PR with only non-blocking comments can be approved; any blocking item requires changes before merge.

## Security checklist

Before flagging or clearing a change on security grounds, check it against the current **[OWASP Top 10](https://owasp.org/Top10/2025/)** — fetch the page rather than relying on a remembered list, since the categories and examples get revised. Record the lookup in `docs/SOURCES.md` if it actually shaped a finding.

At minimum, check for:

- Broken access control — missing or incorrect authorization checks per endpoint/route/resource (authentication alone isn't enough).
- Sensitive data exposure — secrets, tokens, or credentials hardcoded or logged.
- Missing input validation at system boundaries (see [engineering-principles](engineering-principles.md)'s error-handling section).

The stack's `code-review.md` adds the checks specific to it (injection, XSS, insecure storage, CORS, etc.).

## Anti-patterns to always flag

- God classes/functions/components doing more than one thing (violates Single Responsibility, see [engineering-principles](engineering-principles.md)).
- Domain/business/persistence objects leaking into the presentation or API layer, or any other architecture-boundary violation in the stack's `architecture-principles.md`. The stack's `code-review.md` shows the bad/good example.
- Premature abstraction — extra internal splitting or indirection introduced with no real boundary or reason to change. This does **not** include interfaces/abstractions on injected dependencies — those are mandatory from the first implementation; never flag a DI interface as premature just because there's only one concrete implementation today.
- Non-descriptive callback/lambda parameter names ([engineering-principles](engineering-principles.md)'s naming rules).
- Enums persisted or transmitted by ordinal/index/numeric value instead of name ([engineering-principles](engineering-principles.md)). The stack's `code-review.md` shows the bad/good example.

## Comment format

- One comment per issue: location, the problem, the concrete fix. Don't narrate the whole diff back to the author.
- State the fix, don't just point out the problem — "use a parameterized query here" beats "this looks unsafe."

## Self-review before opening a PR

- Read your own full diff before requesting review — don't rely on CI alone to catch what a human eye would.
- Confirm the stack's static analysis and the test suite pass locally (commands in the `## Static analysis` section of the stack's own note; see [git-conventions](git-conventions.md)).

## Approve / request changes

- Any blocking item open → request changes.
- Only non-blocking comments left → approve, comments optional to address.
- Nothing outstanding → approve.

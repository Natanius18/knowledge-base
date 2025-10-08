# 🧠 Code Review Checklist for Java Developers

A comprehensive guide to reviewing Java code effectively.
Based on the [Awesome Code Reviews Checklist](https://www.awesomecodereviews.com/checklists/code-review-checklist/) — adapted and expanded for modern Java projects (Spring Boot, Maven, JPA, REST APIs, and microservices).



## 📋 Contents

1. [Before Submitting a Review](#before-submitting-a-review)
2. [Implementation](#implementation)
3. [Logic Errors and Bugs](#logic-errors-and-bugs)
4. [Error Handling and Logging](#error-handling-and-logging)
5. [Usability and Accessibility](#usability-and-accessibility)
6. [Ethics and Morality](#ethics-and-morality)
7. [Testing and Testability](#testing-and-testability)
8. [Dependencies](#dependencies)
9. [Security and Data Privacy](#security-and-data-privacy)
10. [Performance](#performance)
11. [Readability](#readability)
12. [Style, Consistency & Maintainability](#style-consistency-maintainability)
13. [Expert Review](#expert-review)
14. [Mindset for Reviewers](#mindset-for-reviewers)

---

<a name="before-submitting-a-review"></a>
## 🧑‍💻 Before Submitting a Review

As a **code author**, be your own reviewer first:

- [ ] Code **compiles and passes** static analysis (SpotBugs, Checkstyle, SonarLint).
- [ ] All **tests pass** — unit, integration, and system tests.
- [ ] Code has been **cleaned up** (no TODOs, debug prints, or commented-out code).
- [ ] **Meaningful description** is provided in the PR (what changed, why, and how).
- [ ] Code follows the project’s **coding conventions and style guide**.
- [ ] Self-review completed — you’ve run through this checklist once before sending it.



<a name="implementation"></a>
## ⚙️ Implementation

- [ ] Does this code **do what it’s supposed to do**?
- [ ] Can the solution be **simplified or decomposed**?
- [ ] Are **design patterns** used appropriately where they add value?
- [ ] Does it follow **SOLID principles** (Single Responsibility, Open/Closed, etc.)?
- [ ] Is the code **modular and reusable**?
- [ ] Does it **reuse existing functionality** instead of duplicating it?
- [ ] Is the code at the **right abstraction level** (no low-level logic in controllers)?
- [ ] Are **configuration values** externalized (e.g., `application.yml`) instead of hardcoded?



<a name="logic-errors-and-bugs"></a>
## 🐛 Logic Errors and Bugs

- [ ] Are there **edge cases** or **inputs** that could break the code?
- [ ] Are **boundary conditions** handled correctly (empty lists, nulls, zero values)?
- [ ] Does the code **validate external input** before use?
- [ ] Are **concurrency issues** possible (race conditions, shared state)?
- [ ] Are **default values** used safely (not hiding errors)?



<a name="error-handling-and-logging"></a>
## 🚨 Error Handling and Logging

- [ ] Are **exceptions** handled at the right level and not swallowed?
- [ ] Are **custom exceptions** meaningful and used properly?
- [ ] Is **logging** consistent with project conventions (SLF4J + `@Slf4j` or similar)?
- [ ] Are **log levels** appropriate (`info`, `warn`, `error`)?
- [ ] Are **sensitive details** (tokens, passwords) never logged?
- [ ] Are **error messages** helpful for debugging and not leaking secrets?



<a name="usability-and-accessibility"></a>
## 🎨 Usability and Accessibility

*(Mainly for API/UI changes)*

- [ ] Are **REST endpoints intuitive**, consistent, and versioned if needed?
- [ ] Are **validation annotations** (`@NotNull`, `@Size`, etc.) used for DTOs?
- [ ] Is **input/output format** consistent (error shapes, date formats)?
- [ ] Is the **API/UI predictable** and easy to use?
- [ ] Is the **UI accessible** when applicable (labels, ARIA, contrasts)?



<a name="ethics-and-morality"></a>
## ⚖️ Ethics and Morality

- [ ] Does the code **respect user privacy** and use personal data only when justified?
- [ ] Could the feature **cause harm, manipulation, or discrimination**?
- [ ] If applicable (ML/algorithms), check for potential **bias** and unfair outcomes.



<a name="testing-and-testability"></a>
## 🧪 Testing and Testability

- [ ] Is the new code **easily testable** (low coupling, clear inputs/outputs)?
- [ ] Have **unit tests** been added/updated for the changed behavior?
- [ ] Are **integration tests** updated where interactions (DB, external APIs) changed?
- [ ] Are **edge cases** covered by tests (empty, null, invalid inputs)?
- [ ] Are tests **readable, fast, and reliable**?
- [ ] Are **CI pipelines** green after this change?



<a name="dependencies"></a>
## 📦 Dependencies

- [ ] Are **new dependencies** necessary and stable (avoid pulling libraries for minor helpers)?
- [ ] Are they declared with the **correct scope** in `pom.xml` (test vs. compile)?
- [ ] Were any README/configs updated if required?



<a name="security-and-data-privacy"></a>
## 🔐 Security and Data Privacy

- [ ] Are **inputs validated and sanitized** (prevent SQLi, XSS, path traversal)?
- [ ] Are **authorization and authentication** checks present where needed?
- [ ] Are **secrets** (passwords, keys) not stored in source control and not hardcoded?
- [ ] Are **sensitive logs** redacted and HTTPS/CORS settings correct?



<a name="performance"></a>
## ⚡ Performance

- [ ] Does the change **avoid obvious performance regressions**?
- [ ] Are **DB queries** optimized (no N+1, use proper indexes)?
- [ ] Are **loops and allocations** reasonable for expected data sizes?
- [ ] Is **lazy vs eager fetching** in JPA chosen appropriately?
- [ ] Is **caching/pagination** used where appropriate?



<a name="readability"></a>
## ✍️ Readability

- [ ] Is the code **easy to understand** at first glance?
- [ ] Are **methods small and focused** (refactor long methods)?
- [ ] Are **names descriptive** (variables, methods, classes)?
- [ ] Is **formatting consistent** with the project style?
- [ ] Are comments used to explain *why*, not *what*?
- [ ] Remove **commented-out or obsolete code**.



<a name="style-consistency-maintainability"></a>
## 🧱 Style, Consistency & Maintainability

- [ ] Does it follow **team Java conventions** (Effective Java / agreed style)?
- [ ] Is **Lombok** used appropriately (avoid `@Data` for entities with semantics)?
- [ ] Are **DTOs, entities, and services** clearly separated?
- [ ] Is **dependency injection** done by constructor where possible?
- [ ] Can other developers **reasonably extend or maintain** this code later?



<a name="expert-review"></a>
## 🧠 Expert Review

- [ ] Does this change require a **security, DB, or architecture expert** review?
- [ ] Will it impact **other teams or services** that should be notified?



<a name="mindset-for-reviewers"></a>
## 💬 Mindset for Reviewers

- ✅ **Focus on impact**, not nitpicks — prioritize correctness, design, security.
- ✅ Be **constructive**: explain *why* and suggest alternatives.
- ✅ Ask clarifying questions rather than assume intent.
- ✅ **Praise good code** to encourage positive patterns.
- ✅ Keep discussions on the PR so others can learn.



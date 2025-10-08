# 🧠 Code Review Checklist for Java Developers

A comprehensive guide to reviewing Java code effectively.
Based on the [Awesome Code Reviews Checklist](https://www.awesomecodereviews.com/checklists/code-review-checklist/) — adapted and expanded for modern Java projects (Spring Boot, Maven, JPA, REST APIs, and microservices).

---

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
12. [Style, Consistency & Maintainability](#style-consistency--maintainability)
13. [Expert Review](#expert-review)
14. [Mindset for Reviewers](#mindset-for-reviewers)

---

## 🧑‍💻 Before Submitting a Review

As a **code author**, be your own reviewer first:

* [ ] Code **compiles and passes** static analysis (SpotBugs, Checkstyle, SonarLint).
* [ ] All **tests pass** — unit, integration, and system tests.
* [ ] Code has been **cleaned up** (no TODOs, debug prints, or commented-out code).
* [ ] **Meaningful description** is provided in the PR (what changed, why, and how).
* [ ] Code follows the project’s **coding conventions and style guide**.
* [ ] Self-review completed — you’ve run through this checklist once before sending it.

---

## ⚙️ Implementation

* [ ] Does this code **do what it’s supposed to do**?
* [ ] Can the solution be **simplified or decomposed**?
* [ ] Are **design patterns** (e.g., Strategy, Factory, Builder) used appropriately?
* [ ] Does it follow **SOLID principles** (Single Responsibility, Open/Closed, etc.)?
* [ ] Is the code **modular and reusable**?
* [ ] Does it **reuse existing functionality** instead of duplicating it?
* [ ] Is the code at the **right abstraction level** (no low-level logic in controllers)?
* [ ] Are **enums, records, or sealed classes** used where appropriate?
* [ ] Are **frameworks or APIs** used correctly (no misuse of Spring annotations)?
* [ ] Are **configuration values** externalized (e.g., `application.yml`) instead of hardcoded?

---

## 🐛 Logic Errors and Bugs

* [ ] Are there **edge cases** or **inputs** that could break the code?
* [ ] Are **boundary conditions** handled correctly (empty lists, nulls, zero values)?
* [ ] Does the code **validate external input** before use?
* [ ] Are **concurrency issues** possible (race conditions, shared state)?
* [ ] Are **default values** used safely (not hiding errors)?
* [ ] Are **floating-point comparisons** avoided where precision matters?

---

## 🚨 Error Handling and Logging

* [ ] Are **exceptions** handled gracefully and at the right level?
* [ ] Are **custom exceptions** meaningful and used properly?
* [ ] Is **logging** consistent with project conventions (SLF4J + `@Slf4j`)?
* [ ] Are **log levels** appropriate (`info`, `warn`, `error`, no debug spam)?
* [ ] Are **sensitive details** (tokens, passwords) never logged?
* [ ] Are **error messages user-friendly** or developer-readable, depending on the audience?

---

## 🎨 Usability and Accessibility

* [ ] Are **REST endpoints intuitive**, consistent, and properly versioned?
* [ ] Are **validation annotations** (`@NotNull`, `@Size`, etc.) used for DTOs?
* [ ] Is **input/output format** consistent (error messages, date formats)?
* [ ] Is the **API/UI predictable** and easy to use?
* [ ] Is the **UI accessible** (labels, contrasts, ARIA attributes if applicable)?

---

## ⚖️ Ethics and Morality

* [ ] Does the code **respect user privacy**?
* [ ] Does it **avoid misuse of personal data** or tracking without consent?
* [ ] Could this feature **cause harm or manipulation** to users?
* [ ] If it involves **user interaction**, are **abuse prevention measures** considered (reporting, blocking)?

---

## 🧪 Testing and Testability

* [ ] Is the new code **easily testable** (no tight coupling or hidden side effects)?
* [ ] Have **unit tests** been added or updated?
* [ ] Are **integration tests** covering new interactions (DB, API)?
* [ ] Are **edge cases** tested (empty, null, invalid inputs)?
* [ ] Do tests follow **AAA (Arrange–Act–Assert)** and avoid over-mocking?
* [ ] Are **test names clear** (`shouldReturn403WhenUnauthorized`)?
* [ ] Are **CI pipelines** green after this change?

---

## 📦 Dependencies

* [ ] Are **new dependencies** necessary and stable (no unnecessary libraries)?
* [ ] Are they added with the **correct scope** in `pom.xml` (no test libs in production)?

---

## 🔐 Security and Data Privacy

* [ ] Are **inputs validated, sanitized, and escaped** (SQL, XSS, path traversal)?
* [ ] Are **passwords and tokens** stored securely (hashed, encrypted)?
* [ ] Are **secrets** externalized (never in source control)?
* [ ] Are **HTTPS and CORS configurations** correct?

---

## ⚡ Performance

* [ ] Does this code degrade **system performance** or scalability?
* [ ] Are **DB queries optimized** (no N+1, no unindexed filters)?
* [ ] Are **loops efficient** (avoid repeated expensive calls)?
* [ ] Is **lazy vs eager fetching** used appropriately (especially in JPA)?
* [ ] Is **stream parallelism** justified and thread-safe?
* [ ] Are **caches** or pagination used where needed?
* [ ] Are **log statements lightweight** (no string concatenation in hot paths)?
* [ ] Have you checked **memory usage** or potential leaks?

---

## ✍️ Readability

* [ ] Is the code **easy to understand** at first glance?
* [ ] Are **methods small and focused** (prefer under 20 lines)?
* [ ] Are **variable/method/class names descriptive**?
* [ ] Is **indentation and formatting consistent** (follow your code style)?
* [ ] Are **comments useful**, explaining *why*, not *what*?
* [ ] Is there any **commented-out or obsolete code**?
* [ ] Could any code be made clearer by **renaming or refactoring**?
* [ ] Is the **data and control flow intuitive**?

---

## 🧱 Style, Consistency & Maintainability

* [ ] Does it follow **Java conventions** (Effective Java, Google Style)?
* [ ] Is **Lombok** used wisely (no `@Data` on entities)?
* [ ] Are **DTOs, entities, and services** clearly separated?
* [ ] Is **Spring dependency injection** done via constructor, not field injection?
* [ ] Are **logs and errors** consistent across modules?
* [ ] Can another developer **easily extend or modify** this code later?

---

## 🧠 Expert Review

* [ ] Should a **security, DB, or architecture expert** review this change?
* [ ] Does the change impact **multiple teams or services**?
* [ ] Should a **UX or API specialist** review usability or documentation aspects?

---

## 💬 Mindset for Reviewers

* ✅ **Focus on impact**, not nitpicking. Point out what truly matters (logic, structure, security).
* ✅ Be **constructive**, not judgmental. Explain *why* something is problematic.
* ✅ Ask questions instead of making assumptions.
* ✅ **Acknowledge good code** — positive feedback encourages best practices.
* ✅ Review with empathy: your goal is to **help your teammate**, not to “win” the review.
* ✅ Keep discussions **public** on the PR so others can learn from the feedback.


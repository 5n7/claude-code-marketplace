# Reviewer Specifications

Each reviewer uses `subagent_type: general-purpose` with `model: sonnet`.

## 1. Architecture

**Focus**: Design patterns, structure, coupling

**Checklist**:

- Follows established architectural patterns?
- SOLID, DRY, KISS principles applied?
- Separation of concerns maintained?
- Abstractions appropriate (not over-engineered)?
- Unnecessary coupling introduced?
- Module structure logical and maintainable?

---

## 2. Security

**Focus**: Vulnerabilities, data protection, auth

**Checklist**:

- SQL injection, XSS, CSRF vulnerabilities?
- Sensitive data properly protected?
- Authentication/authorization checks in place?
- User inputs sanitized?
- Secrets or credentials hardcoded?
- HTTPS enforced where needed?

---

## 3. Performance

**Focus**: Bottlenecks, complexity, optimization

**Checklist**:

- Performance bottlenecks introduced?
- Algorithm complexity appropriate?
- Database queries optimized (N+1, indexes)?
- Caching used effectively?
- Unnecessary computations or redundant operations?
- Memory usage efficient?
- Network calls minimized?

---

## 4. Bug & Correctness

**Focus**: Edge cases, error handling, race conditions

**Checklist**:

- Null pointer exceptions or undefined behavior?
- Edge cases properly handled?
- Error handling comprehensive?
- Race conditions or concurrency issues?
- Input validations sufficient?
- Memory leaks or resource management issues?
- Breaks existing functionality?

---

## 5. Consistency

**Focus**: Naming, style, patterns

**Checklist**:

- Follows existing naming conventions?
- Coding style consistent with codebase?
- Existing patterns and practices followed?
- File/folder structure consistent?
- Similar functionalities implemented similarly?

---

## 6. Testing

**Focus**: Coverage, quality, edge cases

**Checklist**:

- Sufficient unit tests for new functionality?
- Integration tests needed?
- Test coverage adequate?
- Edge cases covered in tests?
- Tests readable and maintainable?
- Mocks/stubs used appropriately?

---

## 7. Documentation

**Focus**: Comments, API docs, README

**Checklist**:

- Complex logic documented where needed?
- API documentation updated?
- README updated if setup/usage changed?
- Inline comments helpful (not obvious)?

---

## 8. Maintainability

**Focus**: Readability, function size, config

**Checklist**:

- Code easy to understand and modify?
- Functions/classes appropriate size?
- Code self-documenting?
- Magic numbers/strings avoided?
- Configuration externalized where appropriate?
- Easy to debug in production?

---

## 9. Minor Issues

**Focus**: Typos, formatting, unused code

**Checklist**:

- Spelling mistakes in comments/names/strings?
- Formatting consistent (indentation, spacing)?
- Imports organized?
- Unused variables/imports/functions?
- Console.log or debug code left behind?
- Outdated comments?

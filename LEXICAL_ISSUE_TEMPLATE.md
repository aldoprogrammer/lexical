# Lexical Contribution Template

Universal template for contributing to the Lexical repository. Works for any type of issue: bug fixes, features, refactoring, performance improvements, API changes, etc. Fill in the placeholders with specific details about your contribution.

---

## Contribution Information

**Type of Contribution:** [Bug Fix / Feature / Refactoring / Performance / API Change / Documentation / Other]

**Issue/PR Link/ID:** [e.g., #8070 or https://github.com/facebook/lexical/issues/8070 or N/A if new feature]

**Title:** [Brief title describing what you're fixing/adding]

**Description:**
```
[Describe the problem/feature/enhancement:
- For bugs: Steps to reproduce, expected vs actual behavior
- For features: What new functionality you're adding and why
- For refactoring: What you're improving and why
]
```

---

## Problem Analysis

Analyze the issue and identify:
1. Root cause of the problem
2. Files that need to be modified
3. Related code patterns in the codebase
4. Edge cases that need to be handled

---

## Fix Requirements

### 1. Environment Setup

**Windows-specific (if applicable):**
```powershell
# Set Playwright browsers path (if using custom location)
$env:PLAYWRIGHT_BROWSERS_PATH = "D:\Playwright\Browsers"

# Verify path is set
echo $env:PLAYWRIGHT_BROWSERS_PATH
```

**General Setup:**
- Ensure you have the latest dependencies: `pnpm install`
- Build the project: `pnpm run build`
- Verify you're on the correct branch

### 2. Code Quality & Patterns

**MUST FOLLOW:**

- ✅ **Type Safety with `assert()`:**
  - Use `assert()` from `vitest` for type narrowing instead of type casting (`as Type`)
  - Example: `assert($isParagraphNode(node), 'message')` instead of multiple `expect()` calls + casting
  - `assert()` provides type refinement automatically - no casting needed
  - Import: `import {assert, describe, expect, test} from 'vitest'`
  - Applies to: All test files, any code that needs type narrowing

- ✅ **Follow Existing Patterns:**
  - ALWAYS search the codebase for similar implementations first
  - Match the style and structure of existing code in the same package
  - Check `packages/**/src/__tests__/unit/**/*.test.ts` for test patterns
  - Look at related features/packages for implementation patterns
  - Use the same naming conventions, code organization, and idioms

- ✅ **Lexical-Specific Best Practices:**
  - **Idempotency**: Operations should be safe to apply multiple times
    - Check if nodes already processed before modifying them
    - Prevent infinite loops in transforms/listeners/commands
    - Applies to: Node transforms, command handlers, update listeners
  
  - **Node Identity**: Use `node.is(otherNode)` instead of `===` for node comparison
    - Nodes can have same key but different instances across updates
    - Always use `node.is()` when comparing node identity
  
  - **Edge Cases**: Handle explicitly - don't assume data structure
    - Check for null/undefined before accessing properties
    - Verify parent-child relationships before modifying
    - Validate input parameters and node states

- ✅ **General Code Quality:**
  - Write clear, self-documenting code with meaningful variable names
  - Add comments for complex logic or non-obvious behavior
  - Keep functions focused and single-purpose
  - Avoid premature optimization - prioritize correctness first

### 3. Testing Requirements

**Unit Tests (REQUIRED for all code changes):**
- ✅ MUST write/update unit tests in `packages/**/__tests__/unit/**/*.test.{ts,tsx}`
- ✅ Use `initializeUnitTest()` wrapper for Lexical editor tests
- ✅ Use `createTestEditor()` for headless editor tests
- ✅ Test assertions must be EXPLICIT and COMPREHENSIVE:
  - Don't just check if something exists - verify its exact structure, content, and state
  - Use descriptive error messages in `assert()` calls
  - Test both positive and negative cases
  - Test edge cases and boundary conditions
  - For node-related changes: Verify all expected node types, relationships, and content
  - For API changes: Test all parameter variations and return values

- ✅ Test Quality Guidelines:
  ```typescript
  // ❌ BAD - not explicit enough, can pass with wrong structure
  expect(root.getFirstChild()).toBeTruthy();
  expect(result).toBeDefined();
  
  // ✅ GOOD - explicit structure verification
  const paragraph = root.getFirstChild();
  assert($isParagraphNode(paragraph), 'first root child must be a ParagraphNode');
  expect(paragraph.getTextContent()).toBe('expected content');
  
  // ❌ BAD - type casting, no type safety
  const node = root.getFirstChild() as ParagraphNode;
  
  // ✅ GOOD - type narrowing with assert
  const node = root.getFirstChild();
  assert($isParagraphNode(node), 'must be ParagraphNode');
  // TypeScript now knows node is ParagraphNode
  ```

- ✅ NO artificial delays in tests:
  - NO `setTimeout()`, `Promise.resolve().then()`, or arbitrary waits
  - Tests should be deterministic and fast
  - Use `editor.read()`, `editor.update()`, or proper async/await patterns

- ✅ NO try-catch blocks for expected errors unless testing error handling:
  - Let assertions fail naturally with clear error messages
  - Only use try-catch when explicitly testing error scenarios

- ✅ Test Coverage:
  - Cover the main use case
  - Cover edge cases and error conditions
  - Cover integration between components if applicable
  - Ensure tests are isolated and don't depend on execution order

**E2E Tests (REQUIRED for UI/DOM/browser-related changes):**
- ✅ Write E2E tests in `packages/lexical-playground/__tests__/e2e/**/*.spec.{ts,mjs}` for:
  - User interaction changes (keyboard, mouse, clipboard)
  - DOM manipulation and rendering
  - Browser-specific behavior
  - Integration between multiple features

- ✅ Run E2E tests before committing:
  ```bash
  # Terminal 1: Start dev server
  pnpm run dev
  
  # Terminal 2: Run E2E tests (after setting PLAYWRIGHT_BROWSERS_PATH on Windows)
  pnpm run test-e2e-chromium  # or test-e2e-firefox, test-e2e-webkit
  ```

- ✅ All E2E tests MUST pass - no flaky failures
- ✅ Use descriptive test names that explain what's being tested
- ✅ Follow existing E2E test patterns in the codebase

### 4. TypeScript & Linting

- ✅ Run `pnpm run lint -- --fix` before committing
- ✅ Run `pnpm run tsc` to check TypeScript errors
- ✅ Run `pnpm run ci-check` to verify all checks pass
- ✅ NO unused imports - remove any unused type imports
- ✅ Proper type inference - use type guards (`$isParagraphNode`, etc.) instead of casting

### 5. Build & Verification

**Before committing:**
- ✅ Run `pnpm run build` to ensure build succeeds
- ✅ Run `pnpm run test-unit` for the affected package
- ✅ Run `pnpm run lint` and fix all errors
- ✅ Run `pnpm run ci-check` to verify everything passes

**Commands to run (in order):**
```bash
# 1. Build all packages
pnpm run build

# 2. Type check
pnpm run tsc

# 3. Lint and auto-fix
pnpm run lint -- --fix

# 4. Run unit tests for affected package(s)
pnpm run test-unit -- [package-name]
# Or run all unit tests:
pnpm run test-unit

# 5. Full CI check (runs TypeScript, Flow, Prettier, ESLint)
pnpm run ci-check

# 6. E2E tests (requires dev server in separate terminal)
# Terminal 1:
pnpm run dev

# Terminal 2 (Windows - set browsers path first):
$env:PLAYWRIGHT_BROWSERS_PATH = "D:\Playwright\Browsers"
pnpm run test-e2e-chromium  # or test-e2e-firefox, test-e2e-webkit
```

**Note:** On Windows, set `PLAYWRIGHT_BROWSERS_PATH` environment variable before running E2E tests if using custom browser location.

---

## Implementation Steps (Adapt based on contribution type)

1. **Understand the Context:**
   - Read issue/requirements carefully (or define requirements for new features)
   - Search codebase for related code, similar features, or patterns
   - Identify affected packages and files
   - Review existing tests for similar functionality
   - Understand Lexical's architecture for the relevant area

2. **Analyze & Design:**
   - For bugs: Identify root cause through debugging and code analysis
   - For features: Design API and implementation approach
   - Consider edge cases, error handling, and backward compatibility
   - Plan test strategy (unit + E2E if applicable)
   - Consider performance implications
   - Check if changes affect public API (needs documentation update)

3. **Implement:**
   - Make minimal, focused changes - don't refactor unrelated code
   - Follow existing code patterns and conventions
   - For Lexical-specific code:
     - Add idempotency checks where needed (transforms, listeners)
     - Use proper node identity checks (`node.is()`)
     - Handle null/undefined cases explicitly
   - Add inline comments for complex logic
   - Keep commits atomic and focused

4. **Write/Update Tests:**
   - Write tests BEFORE or DURING implementation (TDD if possible)
   - For new features: Test main functionality and edge cases
   - For bug fixes: Test the specific bug scenario + regression prevention
   - Use explicit assertions with `assert()` from vitest
   - Verify complete behavior, not just existence
   - Test both success and failure paths

5. **Verify Everything:**
   - Run `pnpm run build` - must succeed
   - Run `pnpm run tsc` - no TypeScript errors
   - Run `pnpm run lint -- --fix` - fix all linting issues
   - Run `pnpm run ci-check` - all checks must pass
   - Run unit tests for affected packages - all must pass
   - Run E2E tests if applicable - all must pass
   - Test manually in playground if UI-related
   - Check for regressions in related functionality

---

## Common Pitfalls to Avoid

**Type Safety:**
❌ Don't use type casting (`as Type`) - use `assert()` for type narrowing  
❌ Don't ignore TypeScript errors - fix them properly  
❌ Don't use `any` type unless absolutely necessary  

**Testing:**
❌ Don't skip explicit assertions - verify structure, not just existence  
❌ Don't add artificial delays (`setTimeout`, `Promise.resolve().then()`) - tests should be deterministic  
❌ Don't use try-catch for expected failures - let assertions fail naturally  
❌ Don't write tests that depend on execution order  
❌ Don't skip edge cases - test boundary conditions  

**Lexical-Specific:**
❌ Don't ignore node identity - use `node.is()` instead of `===` for node comparison  
❌ Don't assume data structure - always check for null/undefined  
❌ Don't skip idempotency checks - prevent infinite loops in transforms/listeners  
❌ Don't store node references across update boundaries  
❌ Don't nest `read()` inside `update()` or vice versa (except read at end of update)  

**Code Quality:**
❌ Don't refactor unrelated code - keep changes minimal and focused  
❌ Don't ignore existing patterns - follow codebase conventions  
❌ Don't skip documentation for public API changes  

**Verification:**
❌ Don't commit without running all checks - `ci-check`, `test-unit`, `test-e2e` (if applicable)  
❌ Don't ignore linting errors - fix them before committing  
❌ Don't skip manual testing for UI/UX changes  

---

## Expected Deliverables

1. Fixed code that resolves the issue
2. Comprehensive unit tests with explicit assertions
3. All linting/type checks passing
4. All unit tests passing
5. All E2E tests passing (if applicable)
6. Clean git history with descriptive commit messages

---

## Example Usage

**Example 1: Bug Fix**
```
I need to fix issue #8070 in the Lexical repository.

Type: Bug Fix
Issue: #8070 - AutoLinkPlugin causes infinite transform loop when processing text like "#1234.Another"

Description: [Paste issue details]

Please:
1. Analyze the issue and find the root cause
2. Implement a fix that prevents the infinite loop
3. Write comprehensive unit tests using assert() from vitest
4. Ensure all checks pass (ci-check, test-unit, test-e2e)
5. Follow all requirements in the template
```

**Example 2: New Feature**
```
I'm adding a new feature to Lexical.

Type: Feature
Issue: N/A (new feature)

Description: Adding support for [feature description]. This will allow users to [benefit].

Please:
1. Analyze where this feature should be implemented
2. Design the API following Lexical patterns
3. Implement the feature with comprehensive tests
4. Update documentation if public API changes
5. Ensure all checks pass
6. Follow all requirements in the template
```

**Example 3: Performance Improvement**
```
I'm optimizing performance in Lexical.

Type: Performance
Issue: #XXXX - [Performance issue description]

Description: [Details about the performance problem]

Please:
1. Identify the performance bottleneck
2. Implement optimization following Lexical patterns
3. Add benchmarks/tests to verify improvement
4. Ensure functionality remains correct
5. Follow all requirements in the template
```

---

## Issue Type Specific Guidelines

**Bug Fixes:**
- Reproduce the bug first with a test (TDD approach)
- Fix the root cause, not just symptoms
- Add regression tests to prevent reoccurrence
- Consider impact on related features

**New Features:**
- Design API with backward compatibility in mind
- Consider if feature should be opt-in vs default
- Update documentation for public API changes
- Add examples/demos if applicable

**Refactoring:**
- Ensure behavior remains exactly the same
- Update all related code consistently
- Add tests if missing for refactored code
- Verify no performance regressions

**Performance Improvements:**
- Measure before and after
- Profile to identify actual bottlenecks
- Ensure correctness is maintained
- Consider memory implications

**API Changes:**
- Follow semantic versioning rules
- Update TypeScript types/Flow types
- Update documentation
- Consider deprecation path for breaking changes

## Notes

- This template is universal and works for any type of contribution to Lexical
- Based on real feedback received from maintainers across multiple PRs
- Following all requirements increases the chance of PR approval without multiple rounds of feedback
- When in doubt, search the codebase for similar patterns and follow them
- Maintainers prefer explicit, comprehensive tests over minimal ones
- Quality over speed - it's better to do it right the first time


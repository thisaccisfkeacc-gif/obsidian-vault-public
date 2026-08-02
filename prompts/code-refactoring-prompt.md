# Code Refactoring Prompt

## Your Role
You are doing code refactoring for PowerX Keys after modularization. Scan, fix, and document.

## What To Do

### Phase 1: Scan
Scan all 3 projects for code quality issues:
- PowerX.Core
- PowerX.Services
- PowerX.UI

Check for:
1. **Dead code** — unused methods, unused variables, unused `using` statements
2. **Duplicate code** — same logic in multiple places
3. **Code style** — inconsistent naming, formatting issues
4. **Performance** — unnecessary allocations, string concatenation in loops, etc.
5. **Error handling** — missing try/catch, swallowed exceptions
6. **Anti-patterns** — god classes, static singletons, tight coupling

### Phase 2: Fix (with confidence)
Fix things you are 100% confident about:
- Remove dead code
- Remove unused `using` statements
- Fix obvious code style issues
- Remove duplicate code

### Phase 3: Document (for review)
Document things you are NOT 100% confident about:
- Changes that might break functionality
- Anti-patterns that need design decisions
- Performance improvements that need testing
- Any changes that affect public APIs

### Phase 4: Ask
For documented items, ask the user what to do.

## Rules
1. **Do NOT break the build** — verify with `dotnet build` after changes
2. **One change at a time** — make small, testable changes
3. **Document only what's necessary** — don't over-document
4. **Be confident** — if you're sure, just fix it
5. **Be careful** — if you're unsure, document and ask

## Reference
See `modularization-scan-report.md` for known issues from the previous scan.

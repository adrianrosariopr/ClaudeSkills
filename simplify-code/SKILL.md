---
name: simplify-code
description: Simplifies complex code to its most minimal form while retaining all functionality. Detects language, identifies anti-patterns and bloat, applies best practices. Use when code feels verbose, over-engineered, or needs cleanup.
---

<essential_principles>

<what_minimal_means>
**Minimal code** = fewest lines and tokens while maintaining:
- Clarity (readable at a glance)
- Functionality (exact same behavior)
- Maintainability (easy to modify later)

Minimal does NOT mean:
- Clever one-liners that are hard to read
- Removing helpful variable names
- Sacrificing error handling
</what_minimal_means>

<simplification_hierarchy>
Apply in this order:

1. **Remove dead code** - Unused variables, unreachable branches, commented-out code
2. **Eliminate redundancy** - Duplicate logic, unnecessary wrappers, over-abstraction
3. **Use language idioms** - Built-in methods over manual implementations
4. **Simplify control flow** - Early returns, guard clauses, ternaries where clear
5. **Reduce nesting** - Flatten deeply nested conditions and loops
</simplification_hierarchy>

<output_rule>
**Return only the simplified code.** No explanations, no before/after comparisons, no comments about what changed. Just clean, minimal code.
</output_rule>

<preserve_behavior>
The simplified code MUST:
- Produce identical outputs for all inputs
- Handle the same edge cases
- Throw the same errors in the same conditions

If simplification would change behavior, don't do it.
</preserve_behavior>

</essential_principles>

<intake>
Provide the code you want simplified. The language will be detected automatically.

If you want to specify the language explicitly, mention it (e.g., "simplify this React component" or "clean up this Laravel controller").
</intake>

<routing>
| Detected Language | Reference to Load |
|-------------------|-------------------|
| JavaScript/TypeScript | `references/javascript.md` |
| React/JSX/TSX | `references/javascript.md` + `references/react.md` |
| PHP | `references/php.md` |
| Laravel | `references/php.md` + `references/laravel.md` |
| Python | `references/python.md` |
| Unknown/Mixed | Ask user to clarify |

**After detecting language, read the workflow:** `workflows/simplify.md`
</routing>

<reference_index>
Language-specific anti-patterns and best practices in `references/`:

**JavaScript/TypeScript:** javascript.md
**React:** react.md
**PHP:** php.md
**Laravel:** laravel.md
**Python:** python.md
</reference_index>

<workflows_index>
| Workflow | Purpose |
|----------|---------|
| simplify.md | Main simplification procedure |
</workflows_index>

<success_criteria>
Simplification is complete when:
- Code is measurably shorter (fewer lines/tokens)
- All original functionality is preserved
- Code follows language-specific best practices
- No obvious further simplifications remain
</success_criteria>

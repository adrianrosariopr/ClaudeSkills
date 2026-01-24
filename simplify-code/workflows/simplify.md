# Workflow: Simplify Code

<required_reading>
Based on detected language, read the relevant references:

- **JavaScript/TypeScript**: references/javascript.md
- **React components**: references/javascript.md + references/react.md
- **PHP**: references/php.md
- **Laravel**: references/php.md + references/laravel.md
- **Python**: references/python.md
</required_reading>

<process>

<step name="detect_language">
Identify the language from:
- File extension if provided (.js, .tsx, .php, .py)
- Syntax patterns (import/require, use statements, def/function)
- Framework markers (React hooks, Laravel facades, Flask decorators)
- User's explicit mention

If unclear, ask the user.
</step>

<step name="understand_intent">
Before simplifying, understand:
- What does this code do?
- What are its inputs and outputs?
- What edge cases does it handle?
- What side effects does it have?

This prevents accidentally breaking functionality.
</step>

<step name="identify_bloat">
Scan for these categories of unnecessary complexity:

**Dead code:**
- Unused variables and imports
- Unreachable code paths
- Commented-out code
- Console.log/print statements left in

**Redundancy:**
- Duplicate logic that could be extracted
- Unnecessary intermediate variables
- Over-wrapped functions (wrapper that just calls another function)
- Verbose null checks when optional chaining works

**Over-engineering:**
- Abstractions used only once
- Premature optimization
- Unnecessary design patterns
- Configuration for things that never change

**Verbose patterns:**
- Manual loops when map/filter/reduce work
- Explicit type conversions when implicit works
- Long-form conditions when ternary is clearer
- Deeply nested if/else when guard clauses work
</step>

<step name="apply_simplifications">
Apply the simplification hierarchy from essential principles:

1. Remove dead code first (safest, biggest impact)
2. Eliminate redundancy
3. Use language idioms (from reference file)
4. Simplify control flow
5. Reduce nesting

After each change, verify behavior is preserved.
</step>

<step name="output">
Return ONLY the simplified code.

No explanations. No "here's what I changed." No markdown commentary.

Just the clean, minimal code.
</step>

</process>

<success_criteria>
- Code is shorter than original
- All functionality preserved
- Follows language-specific best practices from reference
- No obvious further simplifications possible
- Output is code only, no commentary
</success_criteria>

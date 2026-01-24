<overview>
Anti-patterns are common mistakes that undermine document effectiveness. Recognizing these patterns helps avoid them. This reference consolidates anti-patterns across PRDs, TDDs, and GDDs.
</overview>

<universal_anti_patterns>
## Universal Anti-Patterns

These mistakes apply to all document types.

<anti_pattern name="vague-requirements">
**Vague Requirements**

Using subjective adjectives instead of measurable criteria.

❌ Bad:
- "The system should be fast"
- "Interface must be user-friendly"
- "Performance should be good"

✅ Good:
- "Page load time < 2 seconds on 3G"
- "Task completion rate > 90% without training"
- "API response p99 < 100ms"

**Fix:** Replace every adjective with a number or testable criterion.
</anti_pattern>

<anti_pattern name="static-documents">
**Static Documents**

Writing once and never updating.

❌ Signs:
- "Last updated: 2 years ago"
- References to deprecated features
- Comments asking "is this still true?"
- No version history

✅ Fix:
- Schedule regular review cycles
- Update after every milestone
- Include change log
- Mark sections as Draft/Approved/Deprecated
</anti_pattern>

<anti_pattern name="missing-why">
**Missing "Why"**

Documenting what without explaining rationale.

❌ Bad:
- "Use Redis for caching"
- "The button is blue"
- "Combat uses cooldowns"

✅ Good:
- "Use Redis for caching (sub-millisecond reads, team familiarity, existing infrastructure)"
- "The button is blue to match brand guidelines and pass WCAG contrast requirements"
- "Combat uses cooldowns to create tactical pacing and prevent button mashing"

**Fix:** For every decision, add "because..."
</anti_pattern>

<anti_pattern name="wall-of-text">
**Wall of Text**

Long paragraphs without structure.

❌ Signs:
- Paragraphs > 5 sentences
- No headers for 500+ words
- No lists or tables
- Requires full read to find info

✅ Fix:
- Break into sections with headers
- Use lists for multiple items
- Use tables for comparisons
- Add summaries at section starts
</anti_pattern>

<anti_pattern name="wrong-audience">
**Wrong Audience**

Writing for the wrong reader.

❌ Examples:
- PRD with implementation details (belongs in TDD)
- TDD without technical specifics (too high-level)
- GDD written for players (should be for dev team)
- Executive summary with jargon

✅ Fix:
- Identify primary audience first
- Match detail level to audience needs
- Create separate docs for different audiences
</anti_pattern>

<anti_pattern name="scope-ambiguity">
**Scope Ambiguity**

Unclear boundaries leading to scope creep.

❌ Signs:
- No "Out of Scope" section
- Undefined edges ("and more...")
- Open-ended requirements
- "Phase 2" used as dumping ground

✅ Fix:
- Explicit "In Scope" and "Out of Scope" sections
- Define edges precisely
- Document what's explicitly NOT included
- Separate backlog from committed scope
</anti_pattern>
</universal_anti_patterns>

<prd_anti_patterns>
## PRD-Specific Anti-Patterns

<anti_pattern name="solution-before-problem">
**Solution Before Problem**

Jumping to features without establishing the problem.

❌ Pattern:
"We need to add a notifications feature with email, SMS, and push support..."
(Why? What problem does this solve? Who needs it?)

✅ Fix:
Start with problem statement and evidence before any solution discussion.
</anti_pattern>

<anti_pattern name="vanity-metrics">
**Vanity Metrics**

Success metrics that don't measure real value.

❌ Examples:
- "10,000 downloads"
- "50% of users click the button"
- "Page views increase"

✅ Better:
- "10,000 daily active users with 3+ sessions/week"
- "50% of users complete the target action"
- "Time-to-value decreases from 5 days to 1 day"

**Fix:** Tie metrics to user outcomes, not just activity.
</anti_pattern>

<anti_pattern name="excessive-delegation">
**Excessive Delegation**

Leaving all decisions to designers/engineers.

❌ Signs:
- "Designer will figure out the UX"
- "Engineering will determine approach"
- No edge cases documented
- No acceptance criteria

✅ Fix:
PRD should provide enough direction that the team understands intent and constraints.
</anti_pattern>
</prd_anti_patterns>

<tdd_anti_patterns>
## TDD-Specific Anti-Patterns

<anti_pattern name="no-diagrams">
**No Diagrams**

Text-only technical design.

❌ Pattern:
"Service A calls Service B which queries the database and returns results through the cache..."

✅ Fix:
```
Service A ──▶ Service B ──▶ Database
                 │
                 ▼
              Cache
```

Architecture diagrams, sequence diagrams, and data flows are mandatory.
</anti_pattern>

<anti_pattern name="first-idea-only">
**First Idea Only**

No alternatives considered.

❌ Signs:
- Missing "Alternatives Considered" section
- Or section says "We considered other approaches"
- No trade-off analysis

✅ Fix:
Document 2-3 alternatives with pros/cons and explicit rejection reasons.
</anti_pattern>

<anti_pattern name="missing-rollout">
**Missing Rollout Plan**

Design stops at implementation.

❌ Pattern:
TDD ends with "Implementation Details" - no deployment, monitoring, or rollback.

✅ Fix:
Include: deployment strategy, feature flags, monitoring, rollback triggers and procedure.
</anti_pattern>

<anti_pattern name="security-afterthought">
**Security as Afterthought**

No security section or "will add later."

❌ Signs:
- No security section
- "TBD" in security section
- Auth details missing

✅ Fix:
Security is part of design, not a phase. Address authentication, authorization, data protection, and vulnerabilities.
</anti_pattern>
</tdd_anti_patterns>

<gdd_anti_patterns>
## GDD-Specific Anti-Patterns

<anti_pattern name="too-big-early">
**Too Big Too Early**

100-page document before prototyping.

❌ Pattern:
Detailed GDD written before any playable build.

✅ Fix:
Start with 10-page vision doc. Expand after prototyping validates core loop.
</anti_pattern>

<anti_pattern name="all-lore-no-loop">
**All Lore, No Loop**

Pages of story, unclear gameplay.

❌ Signs:
- 20 pages of world history
- 1 paragraph about actual gameplay
- "Combat will be fun"

✅ Fix:
Core loop should be the clearest section. Lore supports gameplay, not vice versa.
</anti_pattern>

<anti_pattern name="abstract-mechanics">
**Abstract Mechanics**

Describing feel instead of function.

❌ Examples:
- "Combat feels weighty and responsive"
- "Movement is smooth"
- "Abilities are powerful"

✅ Better:
- "Attack has 0.3s wind-up, 0.5s recovery, deals 10 damage"
- "Walk speed 5 units/sec, run speed 10 units/sec, acceleration 0.2s"
- "Fireball: 30 damage, 2s cooldown, 10 unit range"
</anti_pattern>

<anti_pattern name="no-visuals">
**No Visual References**

GDD without concept art, mood boards, or references.

❌ Pattern:
Describing visual style only in text.

✅ Fix:
Include mood boards, reference game screenshots, concept art sketches. Visual docs need visuals.
</anti_pattern>
</gdd_anti_patterns>

<detection_questions>
## Self-Check Questions

Ask these to detect anti-patterns:

**Universal:**
- When was this last updated?
- Could someone new understand this without asking questions?
- Are there any adjectives that should be numbers?
- Is there a "why" for every "what"?

**PRD:**
- Is the problem clear before solutions?
- Would QA know how to test each requirement?
- Is scope explicitly bounded?

**TDD:**
- Are there architecture diagrams?
- Why was this approach chosen over alternatives?
- What's the rollout plan?

**GDD:**
- Can you explain the core loop in 30 seconds?
- Are mechanics concrete and testable?
- Is this short enough to actually be read?
</detection_questions>

<key_takeaways>
## Key Takeaways

1. **Quantify everything**: Replace adjectives with numbers
2. **Explain why**: Rationale matters as much as decisions
3. **Keep it current**: Stale docs actively mislead
4. **Know your audience**: Match detail to reader needs
5. **Show, don't tell**: Diagrams and visuals communicate better
6. **Bound the scope**: Explicit limits prevent creep
7. **Start small**: Grow documents as understanding grows
</key_takeaways>

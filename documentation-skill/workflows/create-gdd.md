# Workflow: Create Game Design Document

<required_reading>
**Read these reference files NOW before creating the GDD:**
1. references/gdd-best-practices.md
2. references/llm-optimized-structure.md
3. references/anti-patterns.md
</required_reading>

<process>

<step name="gather-context">
## Step 1: Gather Context

Ask the user for essential information:

**Required:**
- What is the game concept (elevator pitch)?
- What genre and platform?
- What's the core gameplay loop?

**If not provided, ask:**
- Who is the target audience?
- What are reference games (inspirations)?
- What's the scope (indie/AA/AAA)?
- What's unique about this game (hook)?
- What's the development timeline?
</step>

<step name="read-template">
## Step 2: Read Template

Read `templates/gdd-template.md` and understand the structure.

Modern GDDs are lightweight and agile - avoid the 100+ page traditional approach.
</step>

<step name="draft-gdd">
## Step 3: Draft the GDD

Fill each section of the template:

**1. Game Overview**
- Concept/elevator pitch (2-3 sentences)
- Genre and platform
- Target audience
- Unique selling point (what makes it special)

**2. Core Gameplay**
- Core loop (what players do repeatedly)
- Primary mechanics (how players interact)
- Controls and input
- Win/lose conditions

**3. Game Progression**
- Progression systems (levels, skills, unlocks)
- Difficulty curve
- Pacing and player motivation

**4. Narrative & World**
- Story premise (if applicable)
- Setting and world-building
- Characters (main and supporting)
- Tone and atmosphere

**5. Art & Audio Direction**
- Visual style (with references/mood board)
- Audio direction (music, SFX, voice)
- UI/UX guidelines

**6. Technical Requirements**
- Engine and tools
- Platform requirements
- Performance targets

**7. Monetization** (if applicable)
- Business model
- Monetization mechanics
- Economy design

**8. Development Roadmap**
- Milestones and phases
- MVP definition
- Post-launch plans

**Writing principles:**
- Focus on "why" behind design decisions
- Include visual references (concept art, mood boards, screenshots)
- Keep mechanics descriptions playable, not abstract
- Document the core loop clearly - it's the heart of the game
</step>

<step name="add-visuals">
## Step 4: Add Visual References

GDDs need visual communication:

- **Concept art**: Visual style and tone
- **Mood boards**: Inspiration and references
- **Flowcharts**: Game loops and player journeys
- **UI mockups**: Interface concepts
- **Reference screenshots**: From similar games

For early-stage GDDs, curated reference images work better than original art.
</step>

<step name="define-core-loop">
## Step 5: Define Core Loop Clearly

The core loop is the most important element. Document it as:

```
┌─────────────────────────────────────────┐
│              CORE LOOP                  │
│                                         │
│   [Action] → [Challenge] → [Reward]     │
│       ↑                        │        │
│       └────────────────────────┘        │
└─────────────────────────────────────────┘

Example (Roguelike):
Enter Dungeon → Fight Enemies → Get Loot →
Upgrade Character → Go Deeper → Repeat
```

Explain:
- What does the player DO? (verb-based)
- What makes it FUN? (why keep playing)
- What's the REWARD? (motivation)
</step>

<step name="review-and-refine">
## Step 6: Review and Refine

Check the GDD against these criteria:

- [ ] Can someone understand the game in 5 minutes?
- [ ] Is the core loop clearly defined?
- [ ] Are the mechanics playable (not just conceptual)?
- [ ] Does it answer "what makes this game fun"?
- [ ] Are visual references included?
- [ ] Is the scope realistic for the team/timeline?
- [ ] Would all team members (art, code, design) find useful info?

Ask user: "Would you like me to expand any section or add more detail to specific mechanics?"
</step>

<step name="finalize">
## Step 7: Finalize

- Keep it lightweight (aim for 10-20 pages, not 100+)
- Mark sections as Draft/Approved
- Include version and date
- Add links to related docs (art bible, tech spec)
- Set up for iteration - GDDs evolve with development
</step>

</process>

<anti_patterns>
Avoid these common GDD mistakes:
- **Too big too early**: 100-page docs before prototyping waste time
- **All lore, no mechanics**: Pages of story but unclear gameplay
- **Vague mechanics**: "Combat will be fun" → HOW will it be fun?
- **No core loop**: If you can't explain the loop, it's not designed
- **Static document**: GDDs must evolve with playtesting
- **Wrong audience**: Don't write for players, write for the team
- **No visuals**: GDDs are visual documents
- **Missing stakeholders**: Marketing and production need sections too
</anti_patterns>

<success_criteria>
A complete GDD:
- [ ] Has a clear, compelling elevator pitch
- [ ] Defines the core gameplay loop explicitly
- [ ] Documents all primary mechanics
- [ ] Includes visual references (art, mood boards, flowcharts)
- [ ] Defines target audience and platform
- [ ] Has realistic scope for the team
- [ ] Is navigable and findable for all disciplines
- [ ] Is lightweight enough to actually be read and updated
</success_criteria>

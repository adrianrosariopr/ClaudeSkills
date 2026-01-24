<overview>
Game Design Documents (GDDs) serve as blueprints for game development. Modern GDDs are lightweight, agile, and living documents - not the 100+ page tomes of the past. The best GDD is findable, up-to-date, and answers "what" and "why" clearly.
</overview>

<modern_approach>
## Modern GDD Philosophy

**Traditional approach (outdated):**
- 100+ page documents written upfront
- Detailed specifications before prototyping
- Rigid and rarely updated

**Modern approach (recommended):**
- Lightweight documents (10-30 pages)
- Evolve alongside development
- Focus on vision and principles over minutiae
- Updated after every milestone
- Searchable and well-organized

"The single biggest mistake is making the GDD too big too early."
</modern_approach>

<core_sections>
## Essential GDD Sections

<section name="overview">
**Game Overview**
- **Elevator pitch**: 2-3 sentences explaining the game
- **Genre**: Primary and secondary genre classifications
- **Platform**: Target platforms (PC, console, mobile)
- **Target audience**: Who plays this game?
- **Unique hook**: What makes this game special?

Example: "A roguelike deck-builder where you play as a time-traveling librarian collecting story fragments to prevent the end of all narratives. For fans of Slay the Spire and narrative-heavy games."
</section>

<section name="core-loop">
**Core Gameplay Loop**
The heart of your GDD. Must be crystal clear.

```
┌─────────────────────────────────────────┐
│              CORE LOOP                  │
│   [Action] → [Challenge] → [Reward]     │
│       ↑                        │        │
│       └────────────────────────┘        │
└─────────────────────────────────────────┘
```

Document:
- What does the player DO? (verbs)
- What makes it CHALLENGING?
- What REWARDS keep them playing?
- What creates the LOOP (why repeat)?
</section>

<section name="mechanics">
**Gameplay Mechanics**
- Primary mechanics (core interactions)
- Secondary mechanics (supporting systems)
- Controls and input mapping
- Player abilities and limitations
- Win/lose conditions

Be concrete: "Player can jump 2 tiles high" not "Player can jump"
</section>

<section name="progression">
**Progression System**
- How does the player grow?
- Unlocks (abilities, items, areas)
- Difficulty curve
- Pacing and motivation hooks
- Meta-progression (between runs, if applicable)
</section>

<section name="narrative">
**Narrative & World**
- Story premise (if applicable)
- Setting and world rules
- Main characters
- Tone and atmosphere
- How story integrates with gameplay
</section>

<section name="art-audio">
**Art & Audio Direction**
- Visual style (with reference images)
- Color palette
- Animation principles
- Music style and mood
- Sound effects direction
- UI/UX guidelines

**Always include visual references** - mood boards, concept art, or screenshots from reference games.
</section>

<section name="technical">
**Technical Requirements**
- Game engine
- Target performance (FPS, load times)
- Platform-specific requirements
- Multiplayer/networking (if applicable)
- Save system design
</section>

<section name="monetization">
**Monetization** (if applicable)
- Business model (premium, F2P, subscription)
- Monetization mechanics
- Economy design
- Ethical considerations
</section>

<section name="roadmap">
**Development Roadmap**
- MVP definition (minimum viable product)
- Milestone breakdown
- Feature prioritization
- Post-launch plans
</section>
</core_sections>

<good_vs_bad>
## Good GDD vs Bad GDD

<comparison name="core-loop">
**Bad Core Loop:**
"Players explore the world and complete quests"

**Good Core Loop:**
"**Exploration → Discovery → Collection → Upgrade → Harder Exploration**

1. Player enters new area (Exploration)
2. Finds hidden artifact or enemy camp (Discovery)
3. Defeats enemies or solves puzzle to acquire resource (Collection)
4. Returns to hub, uses resources to unlock ability (Upgrade)
5. New ability grants access to previously blocked areas (Loop restarts)

Each loop takes ~10-15 minutes. Upgrades feel meaningful within 2-3 loops."
</comparison>

<comparison name="mechanics">
**Bad Mechanic Description:**
"Combat will be fun and responsive"

**Good Mechanic Description:**
"**Melee Combat System**
- Light attack: 0.3s recovery, 10 damage, chains up to 3
- Heavy attack: 0.8s wind-up, 30 damage, breaks guard
- Dodge: 0.2s i-frames, 0.5s recovery, costs stamina
- Guard: Reduces damage 80%, drains stamina on hit
- Stamina: 100 points, recovers 20/sec when not acting

Reference: Dark Souls combat weight with Hades speed"
</comparison>

<comparison name="visuals">
**Bad Visual Direction:**
"The game will look nice and colorful"

**Good Visual Direction:**
"**Visual Style: Stylized low-poly with hand-painted textures**
- Reference: Firewatch environment style, Journey character design
- Color palette: Warm sunset oranges transitioning to cool night blues
- Mood board: [link to visual references]
- Character silhouettes must be readable at 50% screen size
- Environment uses Y-axis color gradients (warm ground, cool sky)"
</comparison>
</good_vs_bad>

<audience>
## Writing for Your Team

**GDDs serve multiple disciplines:**

| Section | Primary Audience |
|---------|------------------|
| Overview, Core Loop | Everyone |
| Mechanics, Progression | Game Designers, Engineers |
| Narrative, World | Writers, Artists |
| Art, Audio | Artists, Audio Team |
| Technical | Engineers |
| Monetization, Roadmap | Producers, Management |

Don't write for players - write for your team. Marketing materials are separate.
</audience>

<living_document>
## GDD as Living Document

- **Update after every milestone**: Playtest findings change the design
- **Version history**: Track what changed and why
- **Mark status**: Draft / Playtested / Approved / Cut
- **Link related docs**: Art bible, tech spec, narrative doc
- **Make it searchable**: Good organization beats good writing

"A GDD that isn't kept up to date will quickly become misleading."
</living_document>

<key_principles>
## Key Principles

1. **Core loop first**: If you can't explain the loop, it's not designed
2. **Show, don't tell**: Visual references over paragraphs of description
3. **Concrete over abstract**: "Jumps 2 tiles" not "can jump"
4. **Start small, grow**: Begin with 10 pages, expand as needed
5. **For the team**: Write for developers, not players
6. **Evolve constantly**: Update after every playtest and milestone
7. **Findable > comprehensive**: Organization beats completeness
</key_principles>

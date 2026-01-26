# Game Addendum Template

<usage>
Append this to the Core Spec for video games. This follows Game Design Document (GDD) conventions, covering core loop, mechanics, progression, economy, and live ops.
</usage>

<template>
```markdown
---

# Game Addendum (GDD Sections)

> [!info] Project Type: Game
> This addendum covers game-specific design including core loop, mechanics, progression systems, economy, and live operations.

## G1. High Concept

### Elevator Pitch
{{1-2 sentences that capture the essence of the game}}

### Genre & Influences
- **Primary Genre:** {{Action / RPG / Puzzle / Strategy / etc.}}
- **Sub-genres:** {{Roguelike / Metroidvania / etc.}}
- **Inspirations:** {{Games that influenced this}}

### Target Audience
- **Age Rating:** {{E / E10+ / T / M}}
- **Core Audience:** {{Description of primary players}}
- **Platform Preference:** {{PC / Console / Mobile / All}}

### Unique Selling Points
1. {{What makes this game unique}}
2. {{Another differentiator}}
3. {{Third key feature}}

---

## G2. Core Loop

### Primary Loop
```
{{Action}} → {{Reward}} → {{Progression}} → {{Repeat}}
```

**Example:**
```
Fight Enemies → Gain XP/Loot → Level Up/Upgrade → Fight Stronger Enemies
```

### Session Structure

| Session Type | Duration | Goals |
|--------------|----------|-------|
| Quick play | {{5-10 min}} | {{What player accomplishes}} |
| Standard | {{30-60 min}} | {{What player accomplishes}} |
| Extended | {{2+ hours}} | {{What player accomplishes}} |

### Engagement Hooks
- **Short-term:** {{What keeps players engaged moment-to-moment}}
- **Medium-term:** {{What brings players back session-to-session}}
- **Long-term:** {{What keeps players engaged over weeks/months}}

---

## G3. Mechanics & Systems

### Core Mechanics

#### {{Mechanic 1: e.g., Combat}}
- **Description:** {{How it works}}
- **Controls:** {{Input mapping}}
- **Feedback:** {{Visual/audio/haptic feedback}}
- **Depth:** {{Skill ceiling, mastery elements}}

#### {{Mechanic 2: e.g., Movement}}
- **Description:** {{How it works}}
- **Controls:** {{Input mapping}}
- **Special abilities:** {{Jump, dash, climb, etc.}}

#### {{Mechanic 3: e.g., Resource Management}}
- **Resources:** {{What player manages}}
- **Acquisition:** {{How obtained}}
- **Usage:** {{How spent}}

### System Interactions

```
{{System A}} ←→ {{System B}}
      ↓              ↓
{{System C}} ←→ {{System D}}
```

**Dependencies:**
- {{System A affects System B by...}}
- {{System C unlocks when...}}

---

## G4. Progression Systems

### Player Progression

#### Experience & Leveling
- **XP Sources:** {{Kill enemies, complete quests, etc.}}
- **Level Cap:** {{Max level}}
- **Scaling Curve:** {{Linear / Exponential / Custom}}

| Level Range | XP Required | Rewards |
|-------------|-------------|---------|
| 1-10 | {{X}} per level | {{What unlocks}} |
| 11-20 | {{X}} per level | {{What unlocks}} |
| 21-Max | {{X}} per level | {{What unlocks}} |

#### Skill Trees / Abilities
```
{{Tree Name}}
├── {{Tier 1 Abilities}}
│   ├── {{Ability A}}
│   └── {{Ability B}}
└── {{Tier 2 Abilities}} (requires Tier 1)
    ├── {{Ability C}}
    └── {{Ability D}}
```

### Content Progression

#### World/Level Structure
| Area | Unlock Condition | Content |
|------|------------------|---------|
| {{Area 1}} | Start | {{Enemies, items, bosses}} |
| {{Area 2}} | {{Requirement}} | {{Content}} |
| {{Area 3}} | {{Requirement}} | {{Content}} |

#### Difficulty Progression
- **Difficulty Modes:** {{Easy / Normal / Hard / etc.}}
- **Scaling:** {{How difficulty affects gameplay}}
- **Adaptive difficulty:** {{If implemented}}

---

## G5. Economy & Monetization

### In-Game Currencies

| Currency | Acquisition | Sinks | Conversion |
|----------|-------------|-------|------------|
| {{Gold/Coins}} | {{Gameplay rewards}} | {{Purchases, upgrades}} | - |
| {{Premium}} | {{IAP, achievements}} | {{Cosmetics, boosts}} | {{Real money}} |
| {{Special}} | {{Events, challenges}} | {{Limited items}} | - |

### Economy Balance

**Sources (Input):**
- {{Source 1}}: {{Amount per hour}}
- {{Source 2}}: {{Amount per action}}

**Sinks (Output):**
- {{Sink 1}}: {{Cost range}}
- {{Sink 2}}: {{Cost range}}

**Target Balance:**
- {{X}} currency earned per hour
- {{Y}} currency spent per hour
- Net accumulation: {{Z}} per hour

### Monetization Model

**Model:** {{Premium / Free-to-Play / Freemium / Subscription}}

**Revenue Streams:**
| Type | Description | Price Range |
|------|-------------|-------------|
| Base game | {{One-time purchase}} | ${{X}} |
| Cosmetics | {{Skins, effects}} | ${{X-Y}} |
| Battle Pass | {{Seasonal content}} | ${{X}} |
| Expansions | {{New content packs}} | ${{X}} |

**Ethical Guidelines:**
- [ ] No pay-to-win mechanics
- [ ] No predatory pricing
- [ ] Clear odds for random items
- [ ] Parental controls

---

## G6. Content

### Characters

| Character | Role | Abilities | Unlock |
|-----------|------|-----------|--------|
| {{Char 1}} | {{Player/NPC/Enemy}} | {{Key abilities}} | {{How unlocked}} |

### Items & Equipment

| Category | Examples | Rarity Tiers |
|----------|----------|--------------|
| Weapons | {{Types}} | Common → Legendary |
| Armor | {{Types}} | Common → Legendary |
| Consumables | {{Types}} | - |
| Key Items | {{Types}} | Unique |

### Levels/Worlds

| Level | Theme | Objectives | Estimated Time |
|-------|-------|------------|----------------|
| {{Level 1}} | {{Theme}} | {{Goals}} | {{X min}} |
| {{Level 2}} | {{Theme}} | {{Goals}} | {{X min}} |

### Narrative (if applicable)

**Story Summary:**
{{Brief plot overview}}

**Key Story Beats:**
1. {{Inciting incident}}
2. {{Rising action}}
3. {{Climax}}
4. {{Resolution}}

---

## G7. UX/UI

### HUD Elements

| Element | Position | Information |
|---------|----------|-------------|
| Health | {{Top-left}} | {{Current/Max HP}} |
| Currency | {{Top-right}} | {{Amount}} |
| Minimap | {{Corner}} | {{Nearby objectives}} |
| Abilities | {{Bottom}} | {{Cooldowns, shortcuts}} |

### Menu Structure

```
Main Menu
├── Play
│   ├── Continue
│   ├── New Game
│   └── Load Game
├── Options
│   ├── Graphics
│   ├── Audio
│   └── Controls
├── Extras
│   ├── Gallery
│   └── Credits
└── Quit
```

### Tutorial/Onboarding

| Phase | Duration | Teaches |
|-------|----------|---------|
| {{Phase 1}} | {{X min}} | {{Core movement}} |
| {{Phase 2}} | {{X min}} | {{Combat basics}} |
| {{Phase 3}} | {{X min}} | {{Progression systems}} |

### Accessibility Features
- [ ] Subtitles with speaker identification
- [ ] Colorblind modes
- [ ] Control remapping
- [ ] Difficulty options
- [ ] Screen reader support (if applicable)

---

## G8. Art & Audio Direction

### Art Style
- **Style:** {{Realistic / Stylized / Pixel / Low-poly / etc.}}
- **Color Palette:** {{Description or reference}}
- **Influences:** {{Art inspirations}}

### Audio Style
- **Music:** {{Genre, mood, adaptive or static}}
- **SFX:** {{Style description}}
- **Voice:** {{Fully voiced / Partial / None}}

### Asset Lists (High-Level)

| Category | Count | Format |
|----------|-------|--------|
| Character models | {{X}} | {{Format}} |
| Environment assets | {{X}} | {{Format}} |
| UI elements | {{X}} | {{Format}} |
| Music tracks | {{X}} | {{Format}} |
| Sound effects | {{X}} | {{Format}} |

---

## G9. Technical Specifications

### Engine & Tools
- **Game Engine:** {{Unity / Unreal / Godot / Custom}}
- **Version:** {{Engine version}}
- **Scripting:** {{C# / C++ / GDScript / etc.}}

### Target Performance

| Platform | Resolution | Frame Rate | Quality |
|----------|------------|------------|---------|
| PC (min) | 1080p | 30 fps | Low |
| PC (rec) | 1440p | 60 fps | High |
| Console | 4K/1080p | 30/60 fps | Dynamic |
| Mobile | Native | 30 fps | Adaptive |

### Platform-Specific

**PC:**
- **Min specs:** {{CPU, GPU, RAM, Storage}}
- **Recommended:** {{CPU, GPU, RAM, Storage}}
- **Storefronts:** {{Steam / Epic / GOG / etc.}}

**Console:**
- **Platforms:** {{PS5 / Xbox / Switch}}
- **Certification requirements:** {{Notes}}

**Mobile:**
- **iOS:** {{Minimum version}}
- **Android:** {{Minimum API}}
- **Controls:** {{Touch / Controller support}}

---

## G10. Live Operations

### Post-Launch Content

| Update Type | Frequency | Content |
|-------------|-----------|---------|
| Patches | {{As needed}} | Bug fixes, balance |
| Events | {{Weekly/Monthly}} | Limited-time content |
| Seasons | {{Quarterly}} | Battle pass, new content |
| Expansions | {{Annually}} | Major content drops |

### Live Events

| Event Type | Duration | Rewards |
|------------|----------|---------|
| {{Event 1}} | {{X days}} | {{Exclusive items}} |
| {{Event 2}} | {{X days}} | {{Currency bonus}} |

### Multiplayer/Social (if applicable)

**Mode:** {{PvP / Co-op / MMO / None}}
**Matchmaking:** {{Skill-based / Random / Friends}}
**Server Architecture:** {{Dedicated / P2P / Hybrid}}

### Telemetry & Analytics

| Metric | Purpose | Target |
|--------|---------|--------|
| DAU/MAU | Engagement | {{X}} |
| Session length | Engagement | {{X min}} |
| Retention (D1/D7/D30) | Retention | {{X%/Y%/Z%}} |
| ARPU | Revenue | ${{X}} |
| Conversion rate | Monetization | {{X%}} |

---

## G11. Development Milestones

| Milestone | Content | Target Date |
|-----------|---------|-------------|
| Prototype | Core loop playable | {{Date}} |
| Vertical Slice | One complete level | {{Date}} |
| Alpha | All features implemented | {{Date}} |
| Beta | Content complete, polish phase | {{Date}} |
| Gold | Release candidate | {{Date}} |
| Launch | Public release | {{Date}} |

---

*Game Addendum (GDD) - Generated {{YYYY-MM-DD}}*
```
</template>

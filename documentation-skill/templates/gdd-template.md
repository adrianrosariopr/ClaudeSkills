# {{GAME_NAME}} - Game Design Document

---
**Version:** {{VERSION}}
**Status:** Draft | Prototyping | Production | Shipped
**Author:** {{AUTHOR}}
**Last Updated:** {{DATE}}
**Team:** {{TEAM_NAME}}

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.1 | {{DATE}} | {{AUTHOR}} | Initial draft |

---

## 1. Game Overview

### Elevator Pitch
{{2-3 sentences that capture the essence of the game. If you can't explain it briefly, it's not clear enough.}}

### Quick Facts

| Attribute | Value |
|-----------|-------|
| **Genre** | {{Primary genre (Secondary)}} |
| **Platform** | {{PC / Console / Mobile / VR}} |
| **Target Audience** | {{Who plays this?}} |
| **Play Session** | {{Typical session length}} |
| **Multiplayer** | {{Solo / Co-op / Competitive / None}} |
| **Monetization** | {{Premium / F2P / Subscription}} |

### Unique Selling Points
- {{What makes this game special - point 1}}
- {{Point 2}}
- {{Point 3}}

### Reference Games
- **{{Game 1}}**: {{What we're taking from it}}
- **{{Game 2}}**: {{What we're taking from it}}
- **{{Game 3}}**: {{What we're taking from it}}

## 2. Core Gameplay Loop

### The Loop

```
┌─────────────────────────────────────────────────────────┐
│                     CORE LOOP                           │
│                                                         │
│   [{{ACTION}}] → [{{CHALLENGE}}] → [{{REWARD}}]        │
│         ↑                               │               │
│         └───────────────────────────────┘               │
│                                                         │
│   Duration: {{X}} minutes per loop                      │
└─────────────────────────────────────────────────────────┘
```

### Loop Breakdown

**Action (What the player DOES):**
{{Describe the primary verbs - what actions the player takes}}

**Challenge (What makes it INTERESTING):**
{{What creates tension, difficulty, or engagement}}

**Reward (What keeps them PLAYING):**
{{What motivates the next loop - progression, story, mastery}}

### Why It's Fun
{{Explain the core appeal - what emotion or satisfaction does this create?}}

## 3. Gameplay Mechanics

### Primary Mechanics

#### {{Mechanic 1 Name}}

**Description:** {{What it is}}

**Controls:**
| Input | Action |
|-------|--------|
| {{Button/Key}} | {{What happens}} |
| {{Button/Key}} | {{What happens}} |

**Parameters:**
| Parameter | Value |
|-----------|-------|
| {{Param}} | {{Value with units}} |
| {{Param}} | {{Value with units}} |

**Feel:** {{How it should feel - with reference to similar games}}

#### {{Mechanic 2 Name}}

{{Repeat structure}}

### Secondary Mechanics
- **{{Mechanic}}**: {{Brief description}}
- **{{Mechanic}}**: {{Brief description}}

### Win/Lose Conditions

**Win Conditions:**
- {{How do you win/succeed?}}

**Lose Conditions:**
- {{How do you lose/fail?}}

**Fail States:**
- {{What happens when you fail? Restart? Checkpoint? Roguelike reset?}}

## 4. Progression System

### Player Progression

```
[New Player] → [{{Stage 1}}] → [{{Stage 2}}] → [{{Mastery}}]
```

### Progression Elements

| Element | Type | Persistence |
|---------|------|-------------|
| {{Element}} | {{Skill/Unlock/Stat}} | {{Per-run/Permanent}} |
| {{Element}} | {{Skill/Unlock/Stat}} | {{Per-run/Permanent}} |

### Difficulty Curve

```
Difficulty
    │
    │                    ╭──────
    │              ╭─────╯
    │        ╭─────╯
    │  ╭─────╯
    │──╯
    └──────────────────────────────▶ Time
       Tutorial  Early  Mid  Late  Endgame
```

**Pacing Notes:**
- **Tutorial (0-{{X}} min):** {{What player learns}}
- **Early Game:** {{Difficulty and content}}
- **Mid Game:** {{Difficulty and content}}
- **Late Game:** {{Difficulty and content}}

### Unlocks & Rewards

| Unlock | Trigger | Impact |
|--------|---------|--------|
| {{Unlock}} | {{When/how obtained}} | {{What it enables}} |

## 5. Narrative & World

### Story Premise
{{1-2 paragraph story setup. What's the situation? What's at stake?}}

### Setting
- **Time:** {{When}}
- **Place:** {{Where}}
- **Tone:** {{Mood - dark, whimsical, gritty, hopeful}}
- **Theme:** {{Core thematic message}}

### Main Characters

#### {{Character Name}} (Player Character)
- **Role:** {{Their function in the story}}
- **Motivation:** {{What drives them}}
- **Arc:** {{How they change}}
- **Visual:** {{Brief description or link to concept art}}

#### {{Character Name}}
{{Repeat for key NPCs}}

### World Rules
- {{How does magic/technology/the world work?}}
- {{What are the constraints?}}

## 6. Art Direction

### Visual Style
{{Describe the overall look - realistic, stylized, pixel art, etc.}}

**Reference Images:**
- {{Link to mood board}}
- {{Link to style references}}

### Color Palette
| Use | Colors |
|-----|--------|
| Primary | {{Colors}} |
| Accent | {{Colors}} |
| UI | {{Colors}} |
| Danger | {{Colors}} |

### Character Design Principles
- {{Design principle 1}}
- {{Design principle 2}}

### Environment Design Principles
- {{Design principle 1}}
- {{Design principle 2}}

## 7. Audio Direction

### Music Style
- **Genre:** {{Musical style}}
- **Mood:** {{Emotional tone}}
- **Reference Tracks:** {{Examples from other games/media}}

### Sound Effects
| Category | Direction |
|----------|-----------|
| UI | {{Style - clicks, whooshes, etc.}} |
| Combat | {{Style - punchy, realistic, etc.}} |
| Environment | {{Style - ambient, reactive, etc.}} |

### Voice
- **Voice Acting:** {{Yes/No/Partial}}
- **Direction:** {{Tone and style}}

## 8. UI/UX

### HUD Elements
{{List what's always on screen}}

```
┌────────────────────────────────────┐
│ [Health]              [Resource]   │
│                                    │
│                                    │
│                                    │
│                                    │
│ [Ability 1] [2] [3]    [Minimap]  │
└────────────────────────────────────┘
```

### Menu Flow

```
[Title] → [Main Menu] → [Play] → [Game]
              │
              ├─→ [Options]
              ├─→ [Load Game]
              └─→ [Credits]
```

### UX Principles
- {{Principle 1 - e.g., "No death without warning"}}
- {{Principle 2}}

## 9. Technical Requirements

### Target Specs

| Platform | Min Spec | Target FPS |
|----------|----------|------------|
| {{Platform}} | {{Spec}} | {{FPS}} |

### Engine & Tools
- **Engine:** {{Unity/Unreal/Godot/Custom}}
- **Version Control:** {{Git/Perforce/etc.}}
- **Project Management:** {{Tool}}

### Technical Constraints
- {{Constraint 1}}
- {{Constraint 2}}

## 10. Monetization (if applicable)

### Business Model
{{Premium / Free-to-Play / Subscription / Ad-supported}}

### Monetization Methods
| Method | Description | Ethical Notes |
|--------|-------------|---------------|
| {{Method}} | {{How it works}} | {{Fair play considerations}} |

### Economy Design
{{Brief overview of in-game economy if applicable}}

## 11. Development Roadmap

### MVP (Minimum Viable Product)
- [ ] {{Core feature 1}}
- [ ] {{Core feature 2}}
- [ ] {{Core feature 3}}

### Milestones

| Milestone | Target Date | Deliverables |
|-----------|-------------|--------------|
| Prototype | {{Date}} | {{What's playable}} |
| Vertical Slice | {{Date}} | {{What's polished}} |
| Alpha | {{Date}} | {{Feature complete}} |
| Beta | {{Date}} | {{Content complete}} |
| Launch | {{Date}} | {{Ship}} |

### Post-Launch Plans
- {{DLC / Updates / Live service plans}}

## 12. Open Questions

- [ ] {{Design question needing resolution}}
- [ ] {{Technical question}}
- [ ] {{Scope question}}

## 13. Appendix

### Related Documents
- Art Bible: {{Link}}
- Technical Spec: {{Link}}
- Narrative Doc: {{Link}}

### Glossary

| Term | Definition |
|------|------------|
| {{Game-specific term}} | {{What it means}} |

---

## Team Sign-Off

| Role | Name | Date |
|------|------|------|
| Game Director | | |
| Lead Designer | | |
| Art Director | | |
| Tech Lead | | |

# Project Type Detection

<overview>
This reference provides patterns for accurately identifying project types from codebase analysis. Projects may be hybrid (webapp + mobile), in which case multiple addendums apply.
</overview>

<detection_matrix>

## Website Indicators

**Strong indicators (2+ = likely website):**
- Static site generator config (`hugo.toml`, `_config.yml`, `astro.config.mjs`)
- CMS presence (`wp-content/`, `drupal/`, `strapi/`)
- No authentication system
- Content-heavy structure (`/blog/`, `/posts/`, `/pages/`)
- Minimal JavaScript (or just for interactivity)
- SEO-focused files (`robots.txt`, `sitemap.xml`, structured data)

**File patterns:**
```
index.html, about.html, contact.html
/content/, /posts/, /pages/
/themes/, /layouts/, /templates/
_config.yml, hugo.toml, eleventy.js
```

**Tech stack signals:**
- Hugo, Jekyll, Eleventy, Astro, Gatsby (static)
- WordPress, Drupal, Joomla (CMS)
- Tailwind/Bootstrap with minimal JS

## Webapp Indicators

**Strong indicators (2+ = likely webapp):**
- Authentication system (`auth/`, `login`, `register`, JWT, sessions)
- User database tables/models
- CRUD operations throughout
- Protected routes/middleware
- SaaS patterns (subscriptions, billing, multi-tenancy)
- Rich client state management (Redux, Vuex, Zustand)

**File patterns:**
```
/app/, /src/
/controllers/, /services/, /repositories/
/models/User, /migrations/
/api/, /routes/
.env (with DB, API keys)
auth.js, middleware/auth
```

**Tech stack signals:**
- Laravel, Rails, Django, Express, Next.js, Nuxt
- PostgreSQL, MySQL, MongoDB connections
- Stripe, Auth0, Clerk integrations
- React/Vue/Angular with state management

## Mobile App Indicators

**Strong indicators (any = mobile):**
- Platform-specific directories (`ios/`, `android/`)
- Mobile framework configs
- App manifests
- Platform-specific code files

**File patterns:**
```
# React Native
/ios/, /android/
metro.config.js, app.json
*.xcodeproj, *.xcworkspace

# Flutter
/lib/, pubspec.yaml
/ios/, /android/

# Native iOS
*.xcodeproj, *.swift, *.storyboard
Info.plist, AppDelegate

# Native Android
AndroidManifest.xml, build.gradle
*.kt, *.java, /res/
```

**Tech stack signals:**
- React Native, Expo
- Flutter, Dart
- Swift, SwiftUI, UIKit
- Kotlin, Jetpack Compose

## Game Indicators

**Strong indicators (any = game):**
- Game engine project files
- Game-specific directories
- Asset folders with game content

**File patterns:**
```
# Unity
*.unity, *.prefab, *.asset
/Assets/, /ProjectSettings/
Assembly-CSharp.csproj

# Unreal
*.uproject, *.umap
/Source/, /Content/, /Config/

# Godot
project.godot, *.tscn, *.gd
/scenes/, /scripts/

# Custom/Other
/sprites/, /textures/, /audio/
game_loop, physics, collision
```

**Tech stack signals:**
- Unity (C#)
- Unreal (C++, Blueprints)
- Godot (GDScript, C#)
- Phaser, PixiJS (web games)
- SDL, SFML, raylib (custom engines)

</detection_matrix>

<hybrid_detection>

## Handling Hybrid Projects

Some projects span multiple types:

**Webapp + Mobile (common):**
- Monorepo with `/web/` and `/mobile/` directories
- Shared `/packages/` or `/shared/` code
- Generate both webapp and mobile addendums

**Website + Webapp:**
- Marketing pages + authenticated app sections
- Public `/blog/` + private `/dashboard/`
- Generate both addendums, note the split

**Game + Webapp:**
- Browser game with account system
- Leaderboards, multiplayer backend
- Generate both addendums

**Detection approach for hybrids:**
1. Look for clear directory separation
2. Check for shared code/packages
3. Identify which parts serve which purpose
4. Generate addendums for each detected type
</hybrid_detection>

<confidence_scoring>

## Confidence Assessment

When detecting, assign confidence:

**High confidence (proceed automatically):**
- 3+ strong indicators for single type
- No conflicting indicators
- Clear project structure

**Medium confidence (confirm with user):**
- 2 strong indicators
- Some ambiguity
- Hybrid signals

**Low confidence (ask user):**
- 1 or fewer indicators
- Highly unusual structure
- No clear patterns

Always explain your reasoning to the user when confirming.
</confidence_scoring>

<edge_cases>

## Edge Cases

**Empty/new projects:**
- Check for boilerplate/starter kit indicators
- Look at package.json scripts for intent
- Default to asking user

**Monorepos:**
- Analyze each package/app separately
- Generate combined spec with sections per app
- Note the monorepo structure

**Microservices:**
- Each service may be different type
- Generate overview + per-service details
- Focus on the primary/main service if unclear

**Libraries/packages:**
- Not a typical "project type"
- Generate Core Spec only
- Focus on API documentation aspects
</edge_cases>

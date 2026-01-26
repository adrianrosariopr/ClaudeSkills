# Mobile App Addendum Template

<usage>
Append this to the Core Spec for iOS, Android, React Native, Flutter, or other mobile applications. Focus on platform requirements, offline capabilities, push notifications, and app store compliance.
</usage>

<template>
```markdown
---

# Mobile App Addendum

> [!info] Project Type: Mobile App
> This addendum covers mobile-specific requirements including platform support, device capabilities, offline functionality, push notifications, and app store compliance.

## M1. Platform Support

### Supported Platforms

| Platform | Minimum Version | Target Version |
|----------|-----------------|----------------|
| iOS | {{iOS 14.0}} | {{iOS 17.x}} |
| Android | {{API 24 / Android 7.0}} | {{API 34 / Android 14}} |

### Development Framework
- **Framework:** {{React Native / Flutter / Swift / Kotlin / Native}}
- **Version:** {{Framework version}}
- **Language:** {{TypeScript / Dart / Swift / Kotlin}}

### Device Support

**iOS Devices:**
- [ ] iPhone (SE and newer)
- [ ] iPad (if supported)
- [ ] iPad Pro (if supported)

**Android Devices:**
- [ ] Phones (small: 320dp, medium: 360dp, large: 400dp+)
- [ ] Tablets (if supported)
- [ ] Foldables (if supported)

### Screen Sizes

| Category | Width Range | Layout |
|----------|-------------|--------|
| Compact | < 600dp | Single column |
| Medium | 600-840dp | Adaptive |
| Expanded | > 840dp | Multi-pane |

---

## M2. Device Permissions

### Required Permissions

| Permission | iOS Key | Android Permission | Rationale |
|------------|---------|-------------------|-----------|
| Camera | NSCameraUsageDescription | CAMERA | {{Why needed}} |
| Photo Library | NSPhotoLibraryUsageDescription | READ_EXTERNAL_STORAGE | {{Why needed}} |
| Location | NSLocationWhenInUseUsageDescription | ACCESS_FINE_LOCATION | {{Why needed}} |
| Notifications | - | POST_NOTIFICATIONS | {{Why needed}} |
| Microphone | NSMicrophoneUsageDescription | RECORD_AUDIO | {{Why needed}} |

### Permission Request Flow
1. {{When/where permission is requested}}
2. {{Explanation shown to user}}
3. {{Behavior if denied}}
4. {{How to re-request or direct to settings}}

### Privacy Manifest (iOS 17+)
Required privacy declarations:
- [ ] Tracking domains
- [ ] API usage reasons
- [ ] Data collection disclosure

---

## M3. Offline Capabilities

### Offline Strategy
- **Type:** {{Offline-first / Online-first with cache / Online-only}}
- **Sync Engine:** {{Custom / Realm / WatermelonDB / SQLite}}

### Offline Features

| Feature | Offline Behavior | Sync Strategy |
|---------|------------------|---------------|
| {{Feature 1}} | {{Full / Read-only / Unavailable}} | {{Auto / Manual / Background}} |
| {{Feature 2}} | {{Behavior}} | {{Strategy}} |

### Data Sync

**Conflict Resolution:**
- {{Last write wins / Server wins / Merge / User choice}}

**Sync Triggers:**
- [ ] App foreground
- [ ] Network reconnection
- [ ] Manual pull-to-refresh
- [ ] Background fetch (iOS)
- [ ] WorkManager (Android)

### Local Storage

| Data Type | Storage | Size Limit | Encryption |
|-----------|---------|------------|------------|
| User data | SQLite/Realm | {{X}} MB | {{Yes/No}} |
| Media cache | File system | {{X}} MB | No |
| Preferences | SharedPrefs/UserDefaults | {{X}} KB | No |
| Credentials | Keychain/Keystore | - | Yes |

---

## M4. Push Notifications

### Notification Types

| Type | Priority | Sound | Badge | Content |
|------|----------|-------|-------|---------|
| {{Type 1}} | High | Yes | +1 | {{Description}} |
| {{Type 2}} | Normal | No | No | {{Description}} |
| {{Type 3}} | Low | No | No | {{Description}} |

### Push Provider
- **Service:** {{Firebase Cloud Messaging / APNs direct / OneSignal / etc.}}
- **Rich Media:** {{Supported / Not supported}}
- **Actions:** {{List of notification actions}}

### Notification Channels (Android)

| Channel ID | Name | Importance | Description |
|------------|------|------------|-------------|
| {{channel_id}} | {{Display name}} | High/Default/Low | {{Purpose}} |

### Rate Limits
- **Max per day:** {{X notifications}}
- **Quiet hours:** {{Respect system DND}}
- **User preferences:** {{Granular opt-out supported}}

### Deep Links from Notifications
- **Format:** `{{app-scheme}}://{{path}}`
- **Examples:**
  - `myapp://orders/123`
  - `myapp://chat/user-456`

---

## M5. Navigation & Deep Linking

### Navigation Structure

```
Tab Bar / Drawer
├── Home
│   ├── {{Screen 1}}
│   └── {{Screen 2}}
├── {{Tab 2}}
│   ├── {{Screen}}
│   └── {{Screen}}
├── {{Tab 3}}
└── Profile
    ├── Settings
    └── {{Screen}}
```

### Deep Link Scheme

**URL Scheme:** `{{appname}}://`
**Universal Links (iOS):** `https://{{domain}}/{{path}}`
**App Links (Android):** `https://{{domain}}/{{path}}`

### Deep Link Routes

| Path | Screen | Parameters |
|------|--------|------------|
| /home | Home | - |
| /{{resource}}/:id | Detail | id: string |
| /settings | Settings | - |

### Navigation State Persistence
- **Persist on:** {{Background / Never / User preference}}
- **Restore on:** {{Cold start / Warm start}}

---

## M6. Performance Requirements

### Startup Time

| Metric | Target | Measurement |
|--------|--------|-------------|
| Cold start | < {{2}}s | Time to interactive |
| Warm start | < {{0.5}}s | Time to render |
| Hot start | < {{0.2}}s | Time to resume |

### Runtime Performance

| Metric | Target |
|--------|--------|
| Frame rate | 60 fps (smooth scrolling) |
| Memory usage | < {{200}} MB typical |
| Battery impact | < {{5}}% per hour active use |
| Network efficiency | Batch requests, minimize polling |

### App Size

| Platform | Download Size | Install Size |
|----------|---------------|--------------|
| iOS | < {{50}} MB | < {{100}} MB |
| Android | < {{30}} MB (APK) | < {{80}} MB |

### Optimization Checklist
- [ ] Images optimized (WebP, proper sizes)
- [ ] Lazy loading for lists
- [ ] Memory leak prevention
- [ ] Background task optimization
- [ ] Network request caching

---

## M7. Security

### Secure Storage
- **iOS:** Keychain for sensitive data
- **Android:** EncryptedSharedPreferences / Keystore
- **Biometric:** {{Supported for auth / Not supported}}

### Network Security
- [ ] Certificate pinning
- [ ] TLS 1.2+ only
- [ ] No sensitive data in logs
- [ ] Request signing (if applicable)

### Code Security
- [ ] ProGuard/R8 obfuscation (Android)
- [ ] No secrets in code
- [ ] Jailbreak/root detection (if required)
- [ ] Tamper detection (if required)

### Data Protection
- [ ] Encryption at rest for sensitive data
- [ ] Secure clipboard handling
- [ ] Screenshot prevention (sensitive screens)
- [ ] Timeout/auto-lock

---

## M8. App Store Requirements

### iOS App Store

**Required Assets:**
- [ ] App icons (1024x1024 + all sizes)
- [ ] Screenshots (6.7", 6.5", 5.5" iPhones; iPad if supported)
- [ ] Preview videos (optional)

**Required Information:**
- [ ] App name (30 chars)
- [ ] Subtitle (30 chars)
- [ ] Description (4000 chars)
- [ ] Keywords (100 chars)
- [ ] Privacy policy URL
- [ ] Support URL
- [ ] App category
- [ ] Age rating questionnaire

**Compliance:**
- [ ] App Privacy details (data collection disclosure)
- [ ] In-app purchases registered
- [ ] Sign in with Apple (if other social login)

### Google Play Store

**Required Assets:**
- [ ] App icon (512x512)
- [ ] Feature graphic (1024x500)
- [ ] Screenshots (phone, 7" tablet, 10" tablet)
- [ ] Preview videos (optional)

**Required Information:**
- [ ] App name (50 chars)
- [ ] Short description (80 chars)
- [ ] Full description (4000 chars)
- [ ] Privacy policy URL
- [ ] App category
- [ ] Content rating questionnaire

**Compliance:**
- [ ] Data Safety section completed
- [ ] Target API level requirements
- [ ] 64-bit support
- [ ] Large screen support declaration

---

## M9. Testing Strategy

### Device Testing Matrix

| Device | OS Version | Priority |
|--------|------------|----------|
| iPhone 15 Pro | iOS 17 | High |
| iPhone SE (3rd) | iOS 16 | High |
| Pixel 8 | Android 14 | High |
| Samsung S23 | Android 14 | High |
| {{Budget device}} | Android 11 | Medium |

### Test Types

| Type | Tool | Coverage |
|------|------|----------|
| Unit | {{Jest / XCTest / JUnit}} | {{X}}% |
| Integration | {{Detox / Maestro / Espresso}} | Key flows |
| UI/Snapshot | {{Storybook / Snapshot testing}} | Components |
| E2E | {{Detox / Appium / Maestro}} | Critical paths |

### Beta Testing
- **iOS:** TestFlight
- **Android:** Internal/Closed/Open testing tracks
- **Beta group:** {{Size and composition}}

---

## M10. Release & Updates

### Release Channels

| Channel | Purpose | Audience |
|---------|---------|----------|
| Internal | Development builds | Team |
| Alpha | Early testing | QA |
| Beta | Pre-release | Beta testers |
| Production | Public release | All users |

### Update Strategy
- **Force update:** {{When required - security, breaking API}}
- **Soft update:** {{Prompt but allow skip}}
- **Silent update:** {{Background, no prompt}}

### Version Numbering
- **Format:** `{{major}}.{{minor}}.{{patch}}` ({{build}})
- **Example:** 2.1.0 (45)

### Rollout Strategy
- **Staged rollout:** {{Yes - percentage based / No}}
- **Rollback plan:** {{How to revert bad releases}}

---

*Mobile App Addendum - Generated {{YYYY-MM-DD}}*
```
</template>

# Subtree Changelog

## Update from upstream/main (d89687df7 → 5989bf7fa)

### Summary
Update from upstream Bluesky Social App repository with 154 commits. This update includes Age Assurance V2, Live Now streaming feature, contacts-based friend discovery, cashtag support for stock tickers, ESLint v9 migration, and significant dependency updates including react-native-screens and react-navigation.

### Breaking Changes

#### ESLint v9 with Flat Config
- **Commit**: `bd510d846`
- **Change**: Complete migration from ESLint 8 to ESLint 9 with flat config
- **Affected Files**:
  - Removed: `.eslintrc.js`
  - Added: `eslint.config.mjs`
  - Updated lint script: `eslint --cache --quiet src`
- **Impact**: Any custom ESLint rules or overrides need migration to flat config format. TypeScript ESLint plugins updated.
- **Removed Dependencies**: `@typescript-eslint/eslint-plugin`, `@typescript-eslint/parser`
- **Added Dependencies**: `@eslint/js`, `typescript-eslint`

#### React Navigation Drawer Removed
- **Commit**: `caa02db87`
- **Change**: `@react-navigation/drawer` package removed entirely
- **Impact**: Drawer functionality now uses `react-native-drawer-layout` directly

#### Library Replacements for Bundle Size
- **Commit**: `9743149c2` - `graphemer` → `unicode-segmenter`
- **Commit**: `12dc38c5f` - `lodash.isequal` → `fast-deep-equal`
- **Commit**: `9cfa7d0ba` - RNGH removed from web bundle
- **Impact**: Import paths changed; check for any direct usage of these libraries

### New Features

#### Age Assurance V2
- **Commits**: `c4aef9f66`, `b3f775d1d`, `a1857d62b`, `b47f5f023`
- **Location**: `src/screens/AgeAssurance/`
- **Features**:
  - Complete overhaul of age verification system
  - Telephone country code selector (`src/components/forms/TelephoneCountryCodeSelect/`)
  - Birthday fallback for app passwords
  - Improved minimum age handling
  - Config failure handling
  - Organization-specific blurb support
- **New Storage**: Birthday preferences with failure handling

#### Live Now - Live Streaming
- **Commits**: `2223babc2`, `5922c6622`, `1d4263078`, `edb19dd6c`
- **Features**:
  - Live event streaming in open beta
  - Live event feeds
  - Ability to report livestreams
  - Auto-refetch every 5 minutes
  - Fail-open gate behavior

#### Contacts Friend Discovery
- **Commits**: `ad7d49ef8`, `8cee40bef`, `da87515d0`
- **Location**: `src/screens/Onboarding/StepContacts/`
- **Features**:
  - Contact matching flow for finding friends
  - "Find Friends" links throughout the app
  - Expo Contacts integration with permission handling
  - Country-based NUX gating

#### Cashtag Support
- **Commit**: `26aa7c0d9`
- **Feature**: Stock ticker discussions via $SYMBOL syntax (e.g., $AAPL, $TSLA)

#### Fishing Minigame
- **Commit**: `fbd1138d9`
- **Feature**: Easter egg fishing minigame

#### Screenshot Watermarking
- **Commit**: `ab85b509c`
- **Feature**: Posts include watermarks when screenshotted

#### Profile Banner Lightbox
- **Commit**: `0deb9511a`
- **Feature**: Profile banners can be tapped to view in lightbox

#### Composer Prompt on Home
- **Commits**: `bc39b553f`, `8637ba756`, `aa1395542`, `296d22d62`
- **Features**:
  - Prompt on home screen inviting users to post
  - Image upload button integration
  - Gallery auto-open option
  - Styled to not look like a search field

### Dependency Updates

#### Core Dependencies
| Package | Previous | Current |
|---------|----------|---------|
| `@atproto/api` | `^0.18.0` | `^0.18.15` |
| `@bsky.app/alf` | `^0.1.5` | `^0.1.6` |
| `@react-navigation/bottom-tabs` | `^7.3.13` | `^7.9.0` |
| `@react-navigation/native` | `^7.1.9` | `^7.1.26` |
| `@react-navigation/native-stack` | `^7.3.13` | `^7.9.0` |
| `@tanstack/react-query` | `^5.8.1` | `5.25.0` |
| `@react-native-async-storage/async-storage` | pinned to `2.2.0` | `2.2.0` |

#### New Dependencies
- `@bsky.app/expo-image-crop-tool`: `^0.5.0` (replaces `expo-image-crop-tool`)
- `@formatjs/intl-displaynames`: `^6.8.13`
- `expo-contacts`: Added with permission configuration
- `fast-deep-equal`: Replaces `lodash.isequal`
- `unicode-segmenter`: Replaces `graphemer`

#### Removed Dependencies
- `@react-navigation/drawer`
- `lodash.isequal`
- `@types/lodash.isequal`
- `graphemer`

#### Dev Dependencies
| Package | Previous | Current |
|---------|----------|---------|
| `@atproto/dev-env` | `^0.3.181` | `^0.3.204` |
| ESLint | `8.x` | `9.x` |
| `jest-expo` | `~54.0.13` | `~54.0.14` |

### Patch Changes

#### New Patches
- `expo-image+3.0.10.patch` - Adds `loading="lazy"` prop support for web
- `expo-modules-core+3.0.28.patch` - Lifecycle fix
- `react-native+0.81.5+004+IdleCallbackRunnable-crash-fix.patch` - Crash fix
- `react-native-pager-view+6.8.0.patch` - Pager view improvements
- `@haileyok+bluesky-video+0.3.2.patch` - Enhanced with extensive debug logging

#### Removed Patches
- `react-native-screens+4.16.0.patch` (326 lines) - iOS 26 fixes now in upstream

#### Updated Patches
- `expo-modules-core` - Version bump with lifecycle fixes
- `expo-updates` - Version bump
- `react-native-drawer-layout` - Updated for new version

### React Native New Architecture Status

**Current Status**: `newArchEnabled: false`

**Progress Toward New Architecture**:

These changes include several preparatory improvements:

1. **react-native-screens Patch Removed** (`caa02db87`)
   - 326-line patch for iOS 26 compatibility removed
   - Fixes now merged upstream in react-native-screens
   - The patch included `RCT_NEW_ARCH_ENABLED` conditional code paths

2. **Bundle Size Optimizations**
   - `graphemer` → `unicode-segmenter` (smaller, modern)
   - `lodash.isequal` → `fast-deep-equal` (smaller, faster)
   - RNGH removed from web bundle
   - These reduce bundle complexity, easing future migrations

3. **React Navigation Updates**
   - Major version bumps to 7.9.x series
   - Drawer package removed (was a bridge-heavy dependency)
   - Uses react-native-drawer-layout directly

4. **Dependency Consolidation**
   - AsyncStorage pinned to 2.2.0 (new arch compatible version)
   - React Query pinned to specific version for stability

**What's Still Blocking**:
- `newArchEnabled` explicitly set to `false` in app.config.js
- No Fabric-specific component migrations visible
- Bridge-dependent libraries still in use

**Assessment**: These changes are **incremental preparation** rather than direct enablement. The removal of the react-native-screens patch and dependency modernization reduce technical debt, but the app is not yet ready for new architecture. Upstream Bluesky appears to be cleaning up prerequisites before attempting the switch.

### UI/UX Improvements

#### Sidebar & Navigation
- Cleaner sidebar layout (`c4fd9980c`)
- Highlight first feed in sidebar if none selected (`29f8bc998`)
- Scope last-selected-feed to per-account (`d9e37a222`)
- Fix drawer layout on mobile web (`0aaf11aa7`)

#### User Suggestions
- Dismiss button for user suggestions (`e80e2f66c`)
- "See more" opens suggested users modal (`d3878bf0a`)

#### Visual Polish
- Prevent flash of wrong theme on web startup (`10aa46aa9`)
- Button turns blue when tooltip visible (`8f82ad66d`)
- Pressed state for quote embeds (`8136258cd`)
- Clamp GIF sizes (`afe5db8b5`)
- Remove drop shadow on "see more" card on Android (`f390167f6`)

### Bug Fixes

- Fix nested link in repost reason (`cf6d8b060`)
- Fix full-screen back gesture over PagerView on iOS 26 (`d82592f07`)
- Fix long handles in account switch (`7d5de6040`)
- Unregister push token on signout (`65faee4f0`)
- Do not render links if URI is invalid (`9aa360d32`)
- Dismiss recommendation bottom sheet on profile navigation (`55806f108`)
- Fix prefetching starter pack query (`d91e206e2`)
- Fix suggested follow card size (`4b6fbcc05`)
- Catch crop cancelled errors (`4d1b1bb2f`)
- Fix status bar color in video feed (`f985debec`)
- Fix collapsed button width (`9c563f2a4`)
- Fix QR code logo positioning (`f960d2fbc`)

### Build System & CI

- Android builds from source in production (`d5ff9c438`)
- Proguard rules patched in (`43ebb80a5`)
- Lazy load Storybook (`4f6ff6fc7`)
- Exclude images from bundle size workflow (`80049bbe8`)
- Unify build concurrency to one build per platform (`c2e5a18e4`)
- Update GitHub Actions (`699fdbac0`)
- Golang updated to v1.25 (`6ab3751b5`)

### Developer Experience

- CLAUDE.md development guide added (`af9261086`)
- Conductor.json setup for yarn install and web server (`c540dae4e`)
- TanStack Query DevTools exposed for browser extension (`61c919110`)
- Sitemap handlers for bsky.app (`f7e61056f`)

### Client Events & Analytics

- Improved client events for feed interactions (`e684c2157`)
- Standardize metadata for client events in feeds (`2d8de1ddc`)
- Hook up suggestedUser:seen events (`ef3a79495`)
- Fire post:view in more places (`9a0cb09d3`)
- Add more client events to profile followers/following (`6686ebc23`)

### Performance

- Reduce startup hang by not querying fonts (`226b19ae4`)
- Add `loading="lazy"` to expo-image on web (`1cb362b59`)
- Lazy load Storybook (`4f6ff6fc7`)

### Version Bump
- Previous: 1.111.0
- Current: 1.115.0

### Migration Guide for Forks

1. **ESLint Migration**:
   - Delete `.eslintrc.js` if you have customizations
   - Create `eslint.config.mjs` using flat config format
   - Update lint scripts to use new ESLint 9 syntax
   - Remove `@typescript-eslint/eslint-plugin` and `@typescript-eslint/parser`
   - Add `@eslint/js` and `typescript-eslint`

2. **Library Replacements**:
   - Replace `import isEqual from 'lodash.isequal'` with `import isEqual from 'fast-deep-equal'`
   - Replace `graphemer` usage with `unicode-segmenter`

3. **Navigation Updates**:
   - Remove any `@react-navigation/drawer` imports
   - Use `react-native-drawer-layout` directly if needed

4. **Patch Updates**:
   - Remove `react-native-screens+4.16.0.patch` (no longer needed)
   - Add new patches as listed above
   - Update expo-modules-core patch to new version

5. **New Permissions**:
   - Add expo-contacts permission if using contacts feature:
   ```js
   ['expo-contacts', {
     contactsPermission: 'Allow access to contacts for friend discovery.'
   }]
   ```

6. **Testing**:
   - Test Age Assurance flow with various birthdays
   - Verify ESLint runs without errors
   - Test contacts permission flow on device
   - Verify Live Now feature gate behavior

---
*This changelog documents changes merged from the upstream Bluesky Social App repository.*

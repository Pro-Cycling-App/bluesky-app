# Subtree Changelog

## Update from upstream/main (dd7bb1bc1 → 562bf3b)

### Summary
Update from upstream Bluesky Social App repository with 206 commits. This is a large update spanning versions 1.116.0 → 1.118.1 that includes: the Drafts feature for the composer, on-device translation for mobile, iOS 26 / Liquid Glass support, Android edge-to-edge bottom sheets, Lingui v5 migration (massive cross-cutting import changes), a new DraggableList component for pinned feed reordering, web lightbox rework, startup performance improvements via query persistence, and numerous ALF design system migrations. Several files were renamed/moved and many legacy components deleted.

### High-Conflict Risk Areas for Fork Merges

These changes touch many files broadly and are likely to cause merge conflicts with fork customizations:

#### 1. Lingui v5 Migration (`@lingui/macro` → split imports) — ~475 files changed
- **PR**: #9905
- **Change**: `@lingui/macro` is split into `@lingui/core/macro` (for `msg`, `plural`) and `@lingui/react/macro` (for `Trans`)
- **Before**: `import {msg, Trans} from '@lingui/macro'`
- **After**: `import {msg} from '@lingui/core/macro'` + `import {Trans} from '@lingui/react/macro'`
- **Also**: `lingui.config.js` → `lingui.config.ts`, `babel-plugin-macros` removed, replaced by `@lingui/babel-plugin-lingui-macro`
- **Also**: Compiled message format changed from `.js` to `.ts` (`messages.js` → `messages.ts`)
- **Impact**: ANY file in the fork that uses `@lingui/macro` imports will conflict. This is the single largest cross-cutting change in this update.

#### 2. Composer.tsx — Massive Changes (Drafts + Translation + iOS 26)
- **Files**: `src/view/com/composer/Composer.tsx` (hundreds of lines changed)
- **Changes**: Drafts save/restore logic, draft button, video restoration, `usePalette` removed, `Text` import changed from `#/view/com/util/text/Text` to `#/components/Typography`, `FontAwesomeIcon` removed, iOS 26 Liquid Glass conditionals, analytics metrics
- **Impact**: Any fork customizations to the Composer will almost certainly conflict

#### 3. App.native.tsx — New Providers Added
- **Changes**: `KeyboardControllerProvider` import changed (was custom hook, now direct from `react-native-keyboard-controller`), added `TranslateOnDeviceProvider`, added `AppConfigProvider`, added `prefetchAppConfig()`
- **Impact**: Fork customizations adding providers to the app tree will conflict

#### 4. Navigation.tsx — iOS 26 Blur + Notification Restructure
- **PRs**: #9497 (restructure notification/linking to synchronous APIs), #9935 (iOS 26 blur)
- **Impact**: Fork navigation customizations will likely conflict

#### 5. Live Now Files Renamed/Moved
- **From**: `src/components/live/` → `src/features/liveNow/`
- **All files renamed**: `queries.ts`, `utils.ts`, `EditLiveDialog.tsx`, `GoLiveDialog.tsx`, `LiveIndicator.tsx`, etc.
- **Impact**: Any fork imports from `#/components/live/` will break

#### 6. Language Select Dialog Moved
- **From**: `src/view/com/composer/select-language/PostLanguageSelectDialog.tsx`
- **To**: `src/components/dialogs/LanguageSelectDialog.tsx`
- **Impact**: Import path changes needed

### Breaking Changes

#### Lingui v5 Upgrade
- **PR**: #9905
- **Commits**: `9ec0697`
- **493 files changed**, 1318 insertions, 956 deletions
- **Package changes**:
  - `@lingui/cli`: `^4.14.1` → `^5.9.2`
  - `@lingui/react`: `^4.14.1` → `^5.9.2`
  - Added: `@lingui/core` `^5.9.2`, `@lingui/babel-plugin-lingui-macro` `^5.9.2`
  - Removed: `@lingui/macro` (now split), `babel-plugin-macros`
  - Removed: `patches/@lingui+core+4.14.1.patch`
- **Config**: `lingui.config.js` → `lingui.config.ts`
- **Babel**: `'macros'` plugin → `'@lingui/babel-plugin-lingui-macro'`
- **TSConfig**: Added `"lib": ["dom", "esnext"]`, updated sort-import plugin patterns

#### Removed Files
- `src/lib/hooks/useEnableKeyboardController.tsx` — Replaced by direct `KeyboardProvider` from `react-native-keyboard-controller` (v1.20.7)
- `src/view/com/modals/DeleteAccount.tsx` — Replaced by ALF `src/screens/Settings/components/DeleteAccountDialog.tsx`
- `src/view/com/modals/lang-settings/` (entire directory) — Replaced by ALF `src/components/dialogs/LanguageSelectDialog.tsx`
- `src/view/com/util/forms/ToggleButton.tsx` — Removed
- `src/lib/actor-status.ts` — Removed
- `src/view/com/composer/text-input/web/EmojiPickerData.json` — Now uses `@emoji-mart/data` package
- `__mocks__/@notifee/react-native.ts`, `__mocks__/@react-native-camera-roll/camera-roll.js`, `__mocks__/react-native-background-fetch.ts`, `__mocks__/react-native-fs.js`, `__mocks__/rn-fetch-blob.js`, `__mocks__/zeego/dropdown-menu.js` — Old mocks cleaned up

#### Removed Dependencies
| Package | Notes |
|---------|-------|
| `@zxing/text-encoding` | No longer needed |
| `await-lock` | No longer needed |
| `history` | No longer needed |
| `react-native-get-random-values` | No longer needed |
| `react-native-url-polyfill` | No longer needed |
| `expo-task-manager` | No longer needed |
| `@testing-library/jest-native` | Removed (#9820) |
| `babel-plugin-macros` | Replaced by `@lingui/babel-plugin-lingui-macro` |
| `superjson` | Removed (#9811) |

### New Features

#### Drafts (Composer)
- **PR**: #9691 + followups (#9795, #9790, #9788, #9825, #9833, #9815, #9803, #9832, #9827, #9850, #9864)
- **Commits**: `d13df6c` and many followups
- **New Files**:
  - `src/view/com/composer/drafts/DraftItem.tsx` — Individual draft list item
  - `src/view/com/composer/drafts/DraftsButton.tsx` — Button in composer top bar
  - `src/view/com/composer/drafts/DraftsListDialog.tsx` — Dialog listing all drafts
  - `src/view/com/composer/drafts/state/api.ts` — Draft serialization/deserialization (630 lines)
  - `src/view/com/composer/drafts/state/queries.ts` — TanStack Query hooks for drafts
  - `src/view/com/composer/drafts/state/schema.ts` — Draft data schema with Zod
  - `src/view/com/composer/drafts/state/storage.ts` — Native storage (filesystem)
  - `src/view/com/composer/drafts/state/storage.web.ts` — Web storage (IndexedDB)
  - `src/view/com/composer/drafts/state/logger.ts` — Draft-specific logger
  - `src/components/dialogs/nuxs/DraftsAnnouncement.tsx` — NUX announcement dialog
  - `assets/images/drafts_announcement_nux.webp` — NUX image asset
- **Features**:
  - Save/restore drafts locally (filesystem on native, IndexedDB on web)
  - Supports text, facets, images, labels, threadgate, quote/link embeds, videos
  - Draft previews with thumbnails
  - NUX announcement for new users
  - Drafts button always visible in composer toolbar
  - Replies cannot be saved as drafts
  - Character limit for drafts: `MAX_DRAFT_GRAPHEME_LENGTH` constant added to `src/lib/constants.ts`
- **Impact**: Composer.tsx heavily modified — 2500+ lines of new code across draft files

#### On-Device Translation (Mobile)
- **PR**: #9930 + followups (#9987, #9980, #9972, #10007, #10005, #10013)
- **Commits**: `9bbcb47`, `afbade6`, `f614bf6`, `8e2a5ab`, `20bf2cd`, `290e0f2`
- **New Files**:
  - `src/lib/translation/index.tsx` — Native translation provider using `@bsky.app/expo-translate-text`
  - `src/lib/translation/index.web.tsx` — Web stub (no on-device translation)
  - `src/lib/translation/context.ts` — React context for translation state
  - `src/lib/translation/types.ts` — Translation types
  - `src/lib/translation/utils.ts` — Translation utilities
  - `src/lib/translation/README.md` — Documentation
  - `src/components/Post/Translated/index.tsx` — Translated post component
- **Features**:
  - Uses Apple's on-device translation (iOS 18+) and Android's built-in translation
  - Falls back to Google Translate when on-device translation unavailable
  - `HAS_ON_DEVICE_TRANSLATION` env flag added
  - Language list expanded significantly (`src/locale/languages.ts` — 4000+ lines changed)
  - ALT labels made translatable (#9976)
- **Renamed**: `src/lib/hooks/useTranslate.ts` → `src/lib/hooks/useGoogleTranslate.ts`
- **New dependency**: `@bsky.app/expo-translate-text` `^0.2.7`

#### iOS 26 / Liquid Glass Support
- **PR**: #9047 + followups (#9936, #9938, #9935, #9970)
- **Commits**: `73b096e`, `bc61314`, `db1f45d`, `65c4b78`, `1782a65`
- **Changes**:
  - `IS_LIQUID_GLASS` env constant added (`true` when `iOSMajorVersion >= 26`)
  - Bottom sheet adjustments for iOS 26 floaty sheet presentation (safe area subtraction)
  - Blur effect on home screen header (iOS 26 only)
  - Fluid zoom transition for alt text dialog
  - Dialog/Menu components updated for iOS 26 compatibility
  - Composer UI adjustments for Liquid Glass
- **Native Code**: `modules/bottom-sheet/ios/SheetViewController.swift` — iOS 26 safe area adjustments
- **New prop**: `sourceViewTag` on BottomSheetNativeComponent (for fluid transitions)
- **Impact**: Many UI components have `IS_LIQUID_GLASS` conditionals added

#### Android Edge-to-Edge Bottom Sheets
- **PR**: #8342
- **Commit**: `338016e`
- **Changes**:
  - Complete rework of Android `BottomSheetView.kt` (200+ lines changed)
  - New `EdgeToEdgeBottomSheetDialogTheme` style
  - Keyboard visibility tracking and auto-expansion
  - Status bar / navigation bar appearance preservation
  - Edge-to-edge insets handling
  - Material library bumped `1.12.0` → `1.13.0`
- **Deleted**: `src/lib/hooks/useEnableKeyboardController.tsx` (107 lines, replaced by direct KeyboardProvider usage)
- **`react-native-keyboard-controller`**: `1.18.5` → `^1.20.7`
- **Impact**: All screens that used `useEnableKeyboardController` or `useEnableKeyboardControllerScreen` hooks no longer need them

#### Pinned Feed Drag and Drop
- **PR**: #9893
- **Commit**: `00816b7`
- **New Files**:
  - `src/components/DraggableList/index.tsx` — Native draggable list (489 lines)
  - `src/components/DraggableList/index.web.tsx` — Web draggable list (168 lines)
- **Changes**: `src/screens/SavedFeeds.tsx` heavily reworked (385 lines changed)
- **New icon**: `DotGrid` 2x3 icon for drag handles

#### Auth Flow Header Rework
- **PR**: #9368
- **Commit**: `2e34f96`
- **New Files**:
  - `src/screens/Login/components/AuthLayout/Header/index.tsx`
  - `src/screens/Login/components/AuthLayout/Header/index.web.tsx`
  - `src/screens/Login/components/AuthLayout/context.ts`
  - `src/screens/Login/components/AuthLayout/index.tsx`
- **Impact**: Login/signup flow UI changes

#### Web Lightbox Rework
- **PR**: #9481 + perf improvements (#9759, #9762, #9200)
- **Commits**: `d45a2c8`, `3b2573b`, `277e985`, `220e989`
- **Changes**: `src/view/com/lightbox/Lightbox.web.tsx` — 413 lines rewritten with animations and keyboard a11y
- **Renamed strings**: "lightbox" → "image viewer" in user-facing strings (#9807)

#### Startup Performance — Query Persistence
- **PR**: #9594
- **Commit**: `4f1d482`
- **New Files**:
  - `src/lib/persisted-query-storage.ts` — Storage abstraction for TanStack Query persistence
  - `src/lib/__tests__/persisted-query-storage.test.ts` — Tests
- **Changes**: React Query data now persisted to MMKV (native) / IndexedDB (web) for faster startup
- **Impact**: `src/ageAssurance/data.tsx` now uses new storage instead of `AsyncStorage`

#### Bandcamp Embed Support
- **PR**: #9445
- **Commit**: `c83dddc`
- **Changes**: Added Bandcamp as a supported embed player in `src/lib/strings/embed-player.ts`

#### Chat Data Export
- **PR**: #9900
- **Commit**: `775a041`
- **Changes**: Added chat/DM data export to the Export My Data dialog

#### Convert Starter Pack to List
- **PR**: #9675
- **Commit**: `8c4fc08`
- **New File**: `src/components/dialogs/lists/CreateListFromStarterPackDialog.tsx`

#### Germ DM Button
- **PR**: #9848
- **Commit**: `93d2f3a`
- **New File**: `src/screens/Profile/components/GermButton.tsx`

### React Native New Architecture Signals

While there is no explicit "New Architecture migration" PR in this update, several changes indicate the team is actively preparing for and moving toward it:

#### 1. `findNodeHandle` Usage Restricted (PR #10001)
- `findNodeHandle` is **deprecated in the New Architecture** (Fabric). It's a legacy Paper-era API.
- The fix in `Gallery.tsx` restricts `findNodeHandle` to **iOS only** and only for the iOS 26 fluid transition feature, suggesting they're aware of its deprecation and limiting usage.
- Comment in code: `// for iOS 26 fluid transition`

#### 2. Removal of Legacy Polyfills and Dependencies (PR #9946)
- Removed `react-native-url-polyfill`, `react-native-get-random-values`, `@zxing/text-encoding`
- These polyfills were needed for older React Native versions. The New Architecture includes many of these natively.
- Added `setimmediate` as an explicit dependency (New Architecture uses microtasks differently)

#### 3. `BottomSheetNativeComponent` Uses New Architecture Patterns
- The bottom sheet module uses `expo-modules-core` which supports both old and new architectures
- `sourceViewTag` prop added — this is a **view tag** pattern common in Fabric for identifying native views

#### 4. Synchronous API Restructure (PR #9497)
- `Restructure notification/linking handling to use synchronous APIs` — The New Architecture encourages synchronous native module calls (TurboModules are synchronous by default). Moving away from async-heavy patterns is a common pre-migration step.

#### 5. Removal of `useEnableKeyboardController` Custom Wrapper
- Replaced complex ref-counting `KeyboardProvider` wrapper with direct usage of `react-native-keyboard-controller` `^1.20.7`
- `react-native-keyboard-controller` 1.20+ has full New Architecture support with Fabric

#### 6. `expo-task-manager` Removed
- This Expo module had known incompatibilities with the New Architecture. Its removal clears a blocker.

#### 7. Mock Cleanup — Removed Old Native Module Mocks
- Removed mocks for `@notifee/react-native`, `@react-native-camera-roll/camera-roll`, `react-native-background-fetch`, `react-native-fs`, `rn-fetch-blob`, `zeego/dropdown-menu`
- These are all legacy native modules. Removing their mocks suggests these dependencies were fully removed from the codebase (likely in earlier updates), clearing New Architecture blockers.

#### 8. Build Performance: ccache for iOS (PR #9770)
- Enabling `ccache` for dev builds speeds up native compilation, which matters more in New Architecture where there's more native code to compile.

#### 9. Expo SDK 54 + React Native 0.81 Foundation
- The codebase is on Expo 54 / RN 0.81, which has full New Architecture support
- The `expo-modules-core` used by custom native modules supports both architectures
- The `react-native-edge-to-edge` package is New Architecture compatible

#### Assessment
The team appears to be in a **pre-migration cleanup phase**: removing legacy dependencies, eliminating polyfills that the New Architecture provides natively, restricting deprecated API usage (`findNodeHandle`), and moving to synchronous patterns. They haven't flipped the `newArchEnabled` flag yet, but they're systematically removing blockers. The iOS 26 work with Liquid Glass and fluid transitions also exercises native-layer APIs that will need Fabric support.

### ALF Design System Migrations

Several legacy components were refactored to use ALF:

- **Delete Account Dialog**: `src/view/com/modals/DeleteAccount.tsx` → `src/screens/Settings/components/DeleteAccountDialog.tsx` (#9863)
- **Content Language Dialog**: `src/view/com/modals/lang-settings/` → `src/components/dialogs/LanguageSelectDialog.tsx` (#9471)
- **Feed Error Screen**: New ALF error screen (#9895)
- **Prompts Refresh**: Updated prompt styling (#9781)
- **Prompt.Action**: Added optional props for more flexible prompts (#9887, #9889)

### App Config System

- **PR**: #9871
- **New File**: `src/state/appConfig.tsx`
- **Features**: New centralized app config fetching from `https://app-config.workers.bsky.app/config`
- **New env constants**: `APP_CONFIG_DEV_URL`, `APP_CONFIG_PROD_URL`, `APP_CONFIG_URL` in `src/env/common.ts`
- **Currently used for**: Live Now configuration (moved from inline to remote config)
- **Provider**: `AppConfigProvider` added to `App.native.tsx`

### Dependency Updates

#### New Dependencies
| Package | Version | Notes |
|---------|---------|-------|
| `@bsky.app/expo-translate-text` | `^0.2.7` | On-device translation |
| `@emoji-mart/data` | `^1.2.1` | Emoji data (was inline JSON) |
| `@growthbook/growthbook` | `^1.6.5` | Core GrowthBook (added alongside React wrapper) |
| `@lingui/core` | `^5.9.2` | Lingui v5 core |
| `@lingui/babel-plugin-lingui-macro` | `^5.9.2` | Replaces babel-plugin-macros |
| `expo-video-thumbnails` | `^10.0.8` | Video thumbnail generation (for draft previews) |
| `setimmediate` | `^1.0.5` | setImmediate polyfill |

#### Updated Dependencies
| Package | From | To |
|---------|------|-----|
| `@atproto/api` | `^0.18.15` | `^0.19.3` |
| `@bsky.app/alf` | `^0.1.6` | `^0.1.7` |
| `@growthbook/growthbook-react` | `^1.6.2` | `^1.6.5` |
| `@lingui/cli` | `^4.14.1` | `^5.9.2` |
| `@lingui/react` | `^4.14.1` | `^5.9.2` |
| `react-native-keyboard-controller` | `1.18.5` | `^1.20.7` |
| `emoji-mart` | `^5.5.2` | `^5.6.0` |
| `typescript` | `^5.9.2` | `^5.9.3` |
| `typescript-eslint` | `^8.53.0` | `^8.56.0` |
| `@atproto/dev-env` | `^0.3.204` | `^0.3.213` |
| `com.google.android.material:material` (Android) | `1.12.0` | `1.13.0` |

### GIF Handling Rework
- **PRs**: #9809, #9814, #9824, #9822, #9859
- **Changes**: Animated GIFs now treated differently from static GIFs — animated ones go through video system, static ones treated as images
- **New File**: `src/view/com/composer/videos/isAnimatedGif.ts`
- **Impact**: GIF alt text behavior aligned across types, GIFs exempted from active video system, fix for GIFs breaking mute state

### Image Processing
- **PR**: #9955
- **Change**: Removed JPG hardcoding from image processing — images now processed in their original format when possible

### New ESLint Rule
- **PR**: #9789
- **New File**: `eslint/lingui-msg-rule.js` + `eslint/__tests__/lingui-msg-rule.test.js`
- **Purpose**: Enforces Lingui `msg` usage patterns for proper string extraction

### Splash Screen
- **PR**: #9780 (+ revert #9792, + adjust #9797, + web fix #9860)
- **Changes**: New splash screen design, splash assets moved from `assets/` to `assets/splash/`
- **Renamed**: `assets/splash-dark.png` → `assets/splash/splash-dark.png`, `assets/splash.png` → `assets/splash/splash.png`
- **New assets**: `assets/splash/illustration-mobile.png`, `assets/splash/illustration-mobile-dark.png`, `assets/splash/android-splash-logo-white.png`
- **Deleted**: `assets/splash-android-icon.png`, `assets/splash-android-icon-dark.png`

### Bug Fixes
- Fix header measurement for Android content disappearing on switch (#9964)
- Fix email not updating in session state after email change (#9953)
- Fix progress guide flash and extra gap (#9960)
- Fix infinite loading spinner when changing search terms (#9950)
- Fix pull-to-refresh getting stuck on tab switch (#9951)
- Fix webpack dev server freezing after repeated refreshes (#9865)
- Fix native-only function being called on web (#9904)
- Fix keyboard flicker when selecting a draft in composer (#9864)
- Fix GIFs breaking mute state/pausing phone audio (#9859)
- Fix quote embed clickability in composer on web (#9876)
- Fix missing pixel row in text rendering on iOS (#9882)
- Fix console logging in Firefox and other browsers (#9775)
- Fix Live Now status not showing (#9779, #9773)
- Fix string concat bug with trending topics muteword moderation (#9914)
- Fix i18n string interpolation for device name (#9806)
- Fix 'create account' / 'sign in' buttons stacking (#9830)
- Fix stale session issues via API package update (#9998)
- Ensure app session state refreshes on reactivation (#9954)
- Fix crash with translate link on web (#9972)
- Fix stuck header blur (#9979)
- Fix `findNodeHandle` usage on web (#10001)

### UI/UX Improvements
- 'Log in' → 'Sign in' terminology change (#9875)
- 'subtitles' → 'captions' string alignment (#9941)
- 'lightbox' → 'image viewer' in user-facing strings (#9807)
- Add light haptic feedback to Edit Profile button (#9787)
- Add animation to content hider (#9812)
- Make profile description text selectable (#9884)
- Add Renew menu for expired mute words (#9883)
- Show hashtag symbol for Mute option in menu (#9942)
- Content hider now has animation (#9812)
- Profile hover card enabled on quoted post authors (#9896)
- Dedupe trending topics (#9870)
- Remove trending post counts (#9867)
- Back button dismisses lightbox on web (#9874)
- Starter pack follow attribution (#9834)
- Starter packs dialog cleanup + optimistic updates (#9469)
- Different icon for TestFlight builds (#10000)
- Prevent welcome modal from reappearing (#9786)
- Use scroll position to gate post-created feed refresh (#9881)
- Debounce thread preferences (leading edge, 2s) (#9851)
- Add 2px margin below post text (#9748)

### Build System & CI
- CI: Auto-detect Go runtime version from go.mod (#9766)
- Enable ccache for dev builds on iOS (#9770)
- Typecheck `app.config.js` (#9771)
- Remove Sentry Replay from web bundle (#9534)
- Dynamically load `date-fns` locale data (#9535)
- Add documentation on directory structures and naming conventions (#10002)
- New Expo install exclude list: `react-native-reanimated`, `@sentry/react-native`, `react-native-pager-view`

### Analytics
- Search event analytics (#9949)
- Analytics tweaks and tracking callbacks (#9861)
- Add `recId` to `suggestedUser:*` events (#9764)
- A/A test feature flag (#9917)
- Composer open metrics with logContext
- Translation metrics events
- Pass feed URI to sendInteractions endpoint (#9989)

### Version Bumps
- 1.116.0 → 1.117.0 → 1.118.0 → 1.118.1

### Migration Guide for Forks

1. **Lingui v5 Migration** (CRITICAL — touches almost every file):
   - Replace all `import {msg} from '@lingui/macro'` → `import {msg} from '@lingui/core/macro'`
   - Replace all `import {Trans} from '@lingui/macro'` → `import {Trans} from '@lingui/react/macro'`
   - Split combined imports: `import {msg, Trans} from '@lingui/macro'` → two separate imports
   - Replace `import {plural} from '@lingui/macro'` → `import {plural} from '@lingui/core/macro'`
   - Update `babel.config.js`: replace `'macros'` with `'@lingui/babel-plugin-lingui-macro'`
   - Rename `lingui.config.js` → `lingui.config.ts`
   - Remove `@lingui/macro` and `babel-plugin-macros` from deps, add `@lingui/core`, `@lingui/babel-plugin-lingui-macro`

2. **Keyboard Controller Migration**:
   - Remove imports of `useEnableKeyboardController` / `useEnableKeyboardControllerScreen`
   - Remove `KeyboardControllerProvider` import from `#/lib/hooks/useEnableKeyboardController`
   - Use `KeyboardProvider` directly from `react-native-keyboard-controller`
   - Screens that called `useEnableKeyboardController(true)` no longer need to

3. **Live Now Imports**:
   - Update all imports from `#/components/live/` → `#/features/liveNow/`

4. **Translation Hook**:
   - `useTranslate` → `useGoogleTranslate` (renamed)

5. **Delete Account Modal**:
   - Old modal at `src/view/com/modals/DeleteAccount.tsx` deleted
   - New dialog at `src/screens/Settings/components/DeleteAccountDialog.tsx`

6. **Language Settings**:
   - Old modals at `src/view/com/modals/lang-settings/` deleted
   - New dialog at `src/components/dialogs/LanguageSelectDialog.tsx`

7. **Splash Assets**:
   - Update any references to splash asset paths (`assets/splash-*.png` → `assets/splash/*.png`)

8. **Polyfill Removal**:
   - Remove `react-native-url-polyfill`, `react-native-get-random-values` from deps if present
   - Add `setimmediate` `^1.0.5` if not present

9. **Testing**:
   - Verify Lingui compilation works with v5 (`yarn intl:compile`)
   - Test drafts save/restore on both native and web
   - Test on-device translation on iOS 18+ and Android
   - Test bottom sheets on Android (edge-to-edge changes)
   - Test iOS 26 if available (Liquid Glass, blur effects)
   - Verify startup performance with query persistence

---
*This changelog documents changes merged from the upstream Bluesky Social App repository.*

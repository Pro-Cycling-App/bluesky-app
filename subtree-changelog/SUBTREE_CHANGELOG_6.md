# Subtree Changelog

## Update from upstream/main (dd7bb1bc1 → e0ea778e5)

### Summary
Update from upstream Bluesky Social App repository with 294 commits spanning versions 1.116.0 through 1.120.0. This is a large update with several major features: **Composer Drafts** (save/restore post drafts), **iOS 26 support** (edge-to-edge, blur effects, fluid transitions), **on-device translation** (using expo-translate-text on mobile), **web lightbox rewrite** with animations and keyboard a11y, **pinned feed drag-and-drop** reordering, **Android sheets edge-to-edge**, and **bot/automated account badges**. Cross-cutting changes include **TypeScript 6.0**, **Lingui v5**, **React Query update to 5.95**, **react-native-screens 4.24**, and significant project structure documentation updates with a new `features/` directory pattern.

---

### Breaking Changes

#### TypeScript 6.0 Upgrade
- **Commit**: `cc5580848` (#10113)
- **Change**: TypeScript upgraded from `^5.9.2` to `^6.0.2`
- **tsconfig changes**:
  - Added `"lib": ["dom", "esnext"]`
  - Removed `"baseUrl": "."` (paths now use `./` prefix)
  - Plugin `moveUpPatterns` changed from `@lingui/macro` to `@lingui/react/macro`
- **New file**: `src/global.d.ts` — Module declaration for CSS modules (needed for TS 6.0's stricter module checking)
- **Impact**: Fork must use TypeScript 6.0+. `noUncheckedSideEffectImports` enabled by default. May require updates to custom type definitions.

#### Lingui v5 Upgrade
- **Commit**: `9ec06971a` (#9905)
- **Change**: Complete migration from Lingui v4 to v5
- **Config changes**:
  - Removed: `lingui.config.js` (CommonJS)
  - Added: `lingui.config.ts` (TypeScript with `defineConfig`)
  - Babel plugin changed from `macros` (generic `babel-plugin-macros`) to `@lingui/babel-plugin-lingui-macro`
  - Compiled messages output changed from `.js` to `.ts` (`compileNamespace: 'ts'`)
- **Dependency changes**:
  - Added: `@lingui/core` `^5.9.2`, `@lingui/babel-plugin-lingui-macro` `^5.9.2`
  - Updated: `@lingui/react` `^4.14.1` → `^5.9.2`, `@lingui/cli` `^4.14.1` → `^5.9.2`
  - Removed: `@lingui/macro` (replaced by `@lingui/babel-plugin-lingui-macro`), `babel-plugin-macros`
  - Updated: `eslint-plugin-lingui` `^0.11.0` → `^0.12.0`
- **Import changes**: `@lingui/macro` imports become `@lingui/react/macro`
- **Removed patch**: `@lingui+core+4.14.1.patch` (no longer needed)
- **Impact**: All Lingui imports need updating. Any fork code using `@lingui/macro` must change to `@lingui/react/macro`.

#### React Query (TanStack Query) Update
- **Commit**: `500fc1c93` (#10126)
- **Change**: `@tanstack/react-query` from pinned `5.25.0` to `^5.95.2`
- **Also updated**: `@tanstack/query-async-storage-persister` and `@tanstack/react-query-persist-client` to `^5.95.2`
- **Impact**: Check for any deprecated APIs if using React Query directly.

#### `@atproto/api` Major Version Bump
- **Commit**: `3a9526d55` (#9862), `7fd218c97` (#9998), `fd49349b7` (#9784)
- **Change**: `@atproto/api` from `^0.18.15` to `^0.19.5`
- **Impact**: Check for breaking API changes in atproto SDK.

#### Removed Dependencies
| Package | Reason |
|---------|--------|
| `@mattermost/react-native-paste-input` | Replaced by `expo-paste-input` `^0.1.10` |
| `@zxing/text-encoding` | Removed (unused) |
| `await-lock` | Removed (unused) |
| `history` | Removed (unused) |
| `react-native-get-random-values` | Removed (unused polyfill) |
| `react-native-url-polyfill` | Removed (unused polyfill) |
| `@testing-library/jest-native` | Removed (unused test dep) |
| `@react-native/eslint-config` | Removed |
| `ts-node` | Removed (dev-env moved to subdirectory) |
| `@atproto/dev-env` | Removed from root (moved to `dev-env/` subdirectory) |
| `babel-plugin-macros` | Replaced by `@lingui/babel-plugin-lingui-macro` |
| `superjson` | Removed (`b2c95bc35`) |
| `expo-task-manager` | Removed |

#### Removed Patches
| Patch | Reason |
|-------|--------|
| `@lingui+core+4.14.1.patch` | Lingui v5 upgrade |
| `@mattermost+react-native-paste-input+0.8.1.patch` | Replaced by expo-paste-input |
| `@mattermost+react-native-paste-input+0.8.1.patch.disabled` | Cleaned up |
| `expo-modules-core+3.0.28.patch` | Replaced by `expo-modules-core+3.0.29.patch` |
| `expo-notifications+0.32.14.patch` | Replaced by `expo-notifications+0.32.16.patch` (significantly smaller — 992→170 lines) |

#### New ESLint Rules
- **Commits**: `68a4d73d6` (#10151), `2cee96da8` (#9789), `a697841a2` (#10150)
- **New rules**:
  - `import-x/no-extraneous-dependencies` — errors on imports not in package.json (with whitelist for `@jest/globals`, `expo-modules-core`, `@atproto/common-web`)
  - `import-x/no-nodejs-modules` — errors on Node.js built-in imports
  - `react/hook-use-state` — warns when useState get/set names mismatch (e.g. `const [foo, setBar] = useState()`)
  - `bsky-internal/lingui-msg-rule` — enforces proper Lingui `msg` usage
- **ESLint config changes**:
  - Added `.jscodeshift/**` to ignores
  - Removed `src/platform/polyfills.ts` and `src/third-party/**` from ignores
  - Added `tsconfigRootDir: import.meta.dirname` to parser options
- **Standalone commit**: `e0ea778e5` (#10166) — Warn when useState get/set names mismatch (enabled `react/hook-use-state` as warning)
- **Impact**: Fork code may trigger new lint errors. Run `yarn lint` after merge.

#### Dev-env Moved to Subdirectory
- **Commit**: `e2c54a858` (#10009)
- **Change**: E2E mock server infrastructure moved from root-level scripts to `dev-env/` subdirectory with its own `package.json`
- **Script change**: `e2e:mock-server` now runs `cd dev-env && yarn start`
- **Impact**: `@atproto/dev-env` removed from root dependencies

---

### New Features

#### Composer Drafts
- **Commits**: `d13df6c7e` (#9691), `58f532a49` (#9790), `e2a56b019` (#9795), `748706daa` (#9803), `917f099e2` (#9833), `bb6ce8319` (#9825), `3197cbcf3` (#9815), others
- **New Files**:
  - `src/view/com/composer/drafts/DraftItem.tsx` — Individual draft list item with preview
  - `src/view/com/composer/drafts/DraftsButton.tsx` — Button to open drafts list
  - `src/view/com/composer/drafts/DraftsListDialog.tsx` — Dialog showing all saved drafts
  - `src/view/com/composer/drafts/state/api.ts` — Draft save/load/delete API (641 lines)
  - `src/view/com/composer/drafts/state/queries.ts` — TanStack Query hooks for drafts
  - `src/view/com/composer/drafts/state/schema.ts` — Zod schema for draft data
  - `src/view/com/composer/drafts/state/storage.ts` — Native storage (MMKV-based)
  - `src/view/com/composer/drafts/state/storage.web.ts` — Web storage implementation
  - `src/view/com/composer/drafts/state/logger.ts` — Draft-specific logger
- **Features**:
  - Save current composer state as a draft (text, images, links, GIFs, quote posts, labels, threadgate, language)
  - Restore drafts with full fidelity including re-downloading cached media
  - Draft list with previews showing text, image thumbnails, and metadata
  - Automatic draft saving when leaving composer (prompted)
  - Drafts NUX (new user experience) with explanatory overlay
  - Draft previews showing attached media
  - Replies cannot be saved as drafts (disabled for replies)
- **Cross-cutting**: Modifies the Composer component extensively, adds draft save prompts on close, integrates with the existing media/embed pipeline

#### iOS 26 Support
- **Commits**: `73b096e44` (#9047), `bc61314c4` (#9936), `db1f45de9` (#9938), `65c4b7833` (#9935), `1782a6517` (#9970)
- **Features**:
  - Edge-to-edge display support for iOS 26
  - Blur effect on home screen header (iOS 26)
  - Fluid zoom transition for alt text dialog
  - Updated safe area handling throughout the app
  - Tab bar and navigation bar styling updates for iOS 26
  - Different icon for TestFlight builds (`7cec490ce`)
- **New dependency behavior**: Uses `expo-blur` more extensively
- **Cross-cutting**: Touches many layout components, navigation bars, tab bars, and safe area insets

#### On-Device Translation
- **Commits**: `9bbcb472e` (#9930), `afbade6f6` (#9987), `f8886fbfe` (#10015), `290e0f2b5` (#10013), `21d8b07bf` (#10108), `5a2135733` (#10160)
- **New Files**:
  - `src/lib/translation/index.tsx` — Native translation implementation
  - `src/lib/translation/index.web.tsx` — Web translation (Google Translate fallback)
  - `src/lib/translation/context.ts` — Translation context
  - `src/lib/translation/types.ts` — Translation types
  - `src/lib/translation/utils.ts` — Translation utilities
  - `src/lib/translation/README.md` — Documentation
- **Renamed**: `src/lib/hooks/useTranslate.ts` → `src/lib/hooks/useGoogleTranslate.ts`
- **New dependency**: `@bsky.app/expo-translate-text` `^0.2.9`
- **Features**:
  - Uses iOS/Android native translation APIs when available
  - Falls back to Google Translate on web and when native translation unavailable
  - `isTranslationSupported` check from expo-translate-text
  - Prevents over-eager translate button
  - Disables animations for translation events on Android (perf)
  - Logs languages a post is tagged with after translating
  - ALT labels made translatable
  - Crash fix for translate link on web
- **Cross-cutting**: Replaces the old Google-only translation approach, touches post rendering components

#### Pinned Feed Drag-and-Drop
- **Commit**: `00816b70d` (#9893)
- **Features**:
  - Users can now reorder their pinned feeds via drag-and-drop
  - Replaces the previous list-based reordering UI
- **Cross-cutting**: Touches feed preferences state and the feed management screens

#### Web Lightbox Rewrite
- **Commits**: `d45a2c832` (#9481), `220e989f3` (#9200), `3b2573bc5` (#9759), `277e985e1` (#9762)
- **Features**:
  - Complete rewrite of the web lightbox/image viewer
  - Added open/close/swipe animations
  - Keyboard accessibility (arrow keys to navigate, Escape to close)
  - Performance improvements: fixed memoization of List component, deferred re-rendering when opening lightbox
  - Reduced initial jitter
  - Improved smoothness
- **Renamed**: User-facing strings changed from "lightbox" to "image viewer" (`d9ef02737`)
- **Cross-cutting**: Back button now dismisses lightbox on web (`6be775176`)

#### Android Sheets Edge-to-Edge
- **Commit**: `338016ed5` (#8342)
- **Features**:
  - Bottom sheets now render edge-to-edge on Android
  - Proper handling of system bars and insets
- **Related**: "The Great Unjanking of the Sheets" (`aa897f55a` #9973) — major cleanup of sheet behavior across platforms
- **Cross-cutting**: Affects all bottom sheet dialogs throughout the app

#### Bot/Automated Account Badge & Self-Labeling
- **Commit**: `2bd981165` (#10008)
- **New Files**:
  - `src/components/icons/Bot.tsx` — Bot icon
  - `src/screens/Settings/AutomationLabelSettings.tsx` — Settings screen for self-labeling as bot/automated
- **Features**:
  - Badge displayed on bot/automated account profiles
  - Users can self-label their account as automated/bot in settings
  - Profile badge icons scale with font size (`b9f3d04d6`)
- **Cross-cutting**: Adds new navigation route for AutomationLabelSettings

#### LRU Screen Cache for Web
- **Commit**: `530afe87c` (#10063)
- **Features**:
  - Bounded memory screen cache using LRU eviction on web
  - Prevents unbounded memory growth in long browser sessions
- **Impact**: Web performance improvement for long-running sessions

#### Groups Chat Creation Flow
- **Commit**: `f4e14626a` (#10066)
- **New/Modified Files**:
  - `src/components/dms/ChatProfileTabs.tsx` — Selected user tabs with remove buttons (169 LOC)
  - `src/components/dms/InitiateChatFlow.tsx` — Multi-screen flow for group chat creation (1007 LOC)
  - `src/components/dms/dialogs/NewChatDialog.tsx` — Integrated new group flow
- **Features**:
  - "Create group chat" button on Messages screen (feature flag gated, "clip clop" is internal codename)
  - Multi-step flow: search users → select multiple → name group → create
  - Profile search with autocomplete, moderation checks, messaging eligibility validation
  - Animated horizontal scroll tabs showing selected users
- **Cross-cutting**: Leverages existing ProfileCard, Dialog, and session/permissions systems

#### Persisted Query Storage (Startup Performance)
- **Commit**: `4f1d4821c` (#9594)
- **New Files**:
  - `src/lib/persisted-query-storage.ts` — Custom query persister
  - `src/lib/__tests__/persisted-query-storage.test.ts` — Tests
- **Features**:
  - Persists select TanStack Query caches to disk
  - Speeds up app startup by avoiding re-fetching data
  - Custom storage adapter for MMKV/AsyncStorage
- **Cross-cutting**: Integrates with the TanStack Query persistence layer

#### Chat Data Export
- **Commit**: `775a041b7` (#9900)
- **Features**:
  - Added chat/DM data to the "Export My Data" dialog
  - Users can now export their conversation history

#### Content Hider Animation
- **Commit**: `27d946263` (#9812)
- **Features**:
  - Added smooth animation when content hider is toggled (show/hide sensitive content)

#### Download Image on Web
- **Commit**: `80429ec90` (#10025)
- **Features**:
  - Added "Download image" option in web context
  - Simplified download handling across platforms

#### Starter Pack to List Conversion
- **Commit**: `8c4fc087f` (#9675)
- **Features**:
  - Users can convert an existing starter pack into a list

#### Bandcamp Embed
- **Commit**: `c83dddce5` (#9445)
- **Features**:
  - Added embed support for Bandcamp links

#### Mute Word Renewal
- **Commit**: `4ebeb94b4` (#9883)
- **Features**:
  - Added "Renew" menu option for expired mute words

#### Keep Screen Awake During Video
- **Commit**: `69164c640` (#10155)
- **Features**:
  - Screen stays awake while videos are playing
  - Uses `expo-keep-awake`

---

### Refactors & Architectural Changes

#### Live Now Feature Reorganization
- **Commit**: `cc2255cc4` (#9871)
- **Change**: Moved from `src/components/live/` to `src/features/liveNow/`
- **Moved files**: EditLiveDialog, GoLiveDialog, GoLiveDisabledDialog, LinkPreview, LiveIndicator, LiveStatusDialog, queries → index.tsx, utils
- **Impact**: Import paths changed. This establishes the `src/features/` pattern.

#### Delete Account Dialog — ALF Migration
- **Commit**: `49253dbd8` (#9863)
- **Removed**: `src/view/com/modals/DeleteAccount.tsx` (342 lines)
- **Added**: `src/screens/Settings/components/DeleteAccountDialog.tsx`
- **Change**: Migrated from legacy modal system to ALF Dialog pattern

#### Content Language Settings — ALF Migration
- **Commit**: `512ce3d5d` (#9471)
- **Removed files**:
  - `src/view/com/modals/lang-settings/ConfirmLanguagesButton.tsx`
  - `src/view/com/modals/lang-settings/ContentLanguagesSettings.tsx`
  - `src/view/com/modals/lang-settings/LanguageToggle.tsx`
- **Change**: Replaced legacy modal with ALF dialog component

#### Notification/Linking Restructure
- **Commit**: `bdf143ce7` (#9497)
- **Change**: Restructured notification and deep linking handling to use synchronous React hooks
- **Before**: Async deduplication with date tracking (`Linking.getInitialURL()`, `Notifications.getLastNotificationResponseAsync()`)
- **After**: Synchronous hooks (`Linking.useLinkingURL()`, `Notifications.useLastNotificationResponse()`) with explicit cleanup via `Notifications.clearLastNotificationResponse()`
- **Impact**: Eliminates race conditions at startup. Deep links take explicit precedence over notification responses. If you've customized `Navigation.tsx` notification handling, this needs updating.

#### Auth Layout Header
- **Commit**: `2e34f965e` (#9368)
- **New Files**:
  - `src/screens/Login/components/AuthLayout/Header/index.tsx`
  - `src/screens/Login/components/AuthLayout/Header/index.web.tsx`
  - `src/screens/Login/components/AuthLayout/context.ts`
  - `src/screens/Login/components/AuthLayout/index.tsx`
- **Change**: Added structured header component to login flow with platform-specific implementations

#### Prompts Refresh
- **Commit**: `72b7516d9` (#9781)
- **Change**: Refreshed the Prompt component system
- **Added**: Optional props to Prompt.Action (`29bc0f137`)

#### App Config State (Feature-Level Configuration)
- **New File**: `src/state/appConfig.tsx` (87 lines)
- **Purpose**: Dedicated config management for feature flags (initially liveNow)
- **Pattern**: Fetches from `APP_CONFIG_URL`, cached with `Infinity` staleTime, prefetchable at startup via `prefetchAppConfig()`
- **Contains**: `liveNow.allow` (DIDs), `liveNow.exceptions` (per-account overrides)
- **Impact**: Splits feature config out of monolithic `service-config.tsx`, which now focuses on trending context + email confirmation. Establishes pattern for future feature-level configs.

#### ALF Palette Expansion
- **Commit**: `3fae42c07` (#9911)
- **Change**: Removed hardcoded yellow/warning colors, added proper `yellow` and `pink` to ALF palette
- **Related**: `5bcb90908` — Removed all remaining hardcoded yellow color values

#### ALF Feed Error Screen
- **Commit**: `a1cda126e` (#9895)
- **Change**: New standardized error screen for feed loading failures

#### Image Processing Improvements
- **Commits**: `34d8c6fe5` (#10073), `3e37696d8` (#10117), `c2fd87bd8` (#9955)
- **Changes**:
  - Increased image upload resolution (higher quality uploads)
  - Added resolution downsizing to image compression pipeline
  - Removed hardcoded JPG — supports original image format
- **New dependency**: `expo-video-thumbnails` `^10.0.8`

#### GIF Handling Improvements
- **Commits**: `da5c356fc` (#9814), `b3cbb3440` (#9809), `3abc36180` (#9824), `4eca13941` (#9859)
- **New File**: `src/view/com/composer/videos/isAnimatedGif.ts`
- **Changes**:
  - Only animated GIFs treated as videos; static GIFs remain images
  - GIFs now visually display as GIFs (proper presentation)
  - GIFs exempted from active video system
  - Fix for GIFs breaking mute state / pausing phone audio

#### Suggested Follows Refactor
- **Commits**: `97fdd7c59` (#10048), `0b6ff8000` (#9988), `ac939dc84` (#9872)
- **New Files**:
  - `src/screens/Search/util/useSuggestedOnboardingUsers.ts`
  - `src/state/queries/trending/useGetSuggestedOnboardingUsersQuery.ts`
- **Changes**:
  - New `useSuggestedFollowsByActorWithDismiss` hook
  - Removed getSuggestedFollowsByActor fallbacks, uses `recIdStr`
  - Updated onboarding suggestions with new endpoint

#### Profile Screen Refactor
- **Commits**: `8a088e871`, `830f305d2`
- **Changes**:
  - Absolutely positioned header with set header height (improves scroll behavior)
  - Simplified conditionals
  - Uses TanStack Query status over isPending
  - Progress guide flash and extra gap fix (`576761a64`)

#### Metro Config Changes
- **Commit**: `b2c95bc35` (#9811) — Removed superjson resolver overrides (for `copy-anything`, `is-what`)
- **Added**: `unstable_enablePackageExports: false` (needed for Lingui v5 + others compatibility)

#### Toast v2 Codemod
- **Commit**: `35cb2bcf9` (#10045)
- **Change**: Created jscodeshift codemod to migrate Toast calls to v2 API

#### ESLint Warnings Codemod
- **Commit**: `1db01a09a` (#10032)
- **Change**: Created jscodeshift codemod to fix common ESLint warnings

#### Emoji Picker Fork
- **Commit**: `f3ddc074a` (#9763)
- **Change**: Forked MCEmojiPicker for customization
- **Removed**: `src/view/com/composer/text-input/web/EmojiPickerData.json`
- **New**: `@emoji-mart/data` `^1.2.1` added as dependency

#### date-fns Dynamic Locale Loading
- **Commit**: `41b374236` (#9535)
- **Change**: Dynamically loads `date-fns` locale data instead of bundling all locales
- **Impact**: Reduces initial bundle size

#### Sentry Replay Removed from Web
- **Commit**: `adc079e09` (#9534)
- **Change**: Removed Sentry Replay from web bundle
- **Impact**: Reduces web bundle size

---

### Dependency Updates

#### New Dependencies
| Package | Version | Purpose |
|---------|---------|---------|
| `@bsky.app/expo-translate-text` | `^0.2.9` | On-device translation |
| `@emoji-mart/data` | `^1.2.1` | Emoji picker data |
| `@growthbook/growthbook` | `^1.6.5` | Feature flags (core, added alongside react wrapper) |
| `@lingui/core` | `^5.9.2` | Lingui v5 core |
| `@lingui/babel-plugin-lingui-macro` | `^5.9.2` | Lingui v5 babel plugin |
| `expo-paste-input` | `^0.1.10` | Replaces react-native-paste-input |
| `expo-video-thumbnails` | `^10.0.8` | Video thumbnail generation |
| `setimmediate` | `^1.0.5` | Polyfill |
| `@crowdin/cli` | `^4.14.1` | Crowdin CLI (dev) |

#### Updated Dependencies (Major/Notable)
| Package | From | To |
|---------|------|-----|
| `typescript` | `^5.9.2` | `^6.0.2` |
| `@lingui/react` | `^4.14.1` | `^5.9.2` |
| `@lingui/cli` | `^4.14.1` | `^5.9.2` |
| `@atproto/api` | `^0.18.15` | `^0.19.5` |
| `@tanstack/react-query` | `5.25.0` | `^5.95.2` |
| `react-native-screens` | `^4.19.0` | `^4.24.0` |
| `react-native-keyboard-controller` | `1.18.5` | `^1.21.0` |
| `@react-navigation/bottom-tabs` | `^7.9.0` | `^7.15.5` |
| `@react-navigation/native` | `^7.1.26` | `^7.1.33` |
| `@react-navigation/native-stack` | `^7.9.0` | `^7.14.4` |
| `@growthbook/growthbook-react` | `^1.6.2` | `^1.6.5` |
| `expo` | `^54.0.27` | `^54.0.33` |
| `typescript-eslint` | `^8.53.0` | `^8.58.0` |
| `eslint` | (same major) | `^9.39.2` |
| `emoji-mart` | `^5.5.2` | `^5.6.0` |
| `react-native-drawer-layout` | `^4.2.1` | `^4.2.2` |
| `babel-preset-expo` | `~54.0.0` | `~54.0.10` |
| `jest-expo` | `~54.0.14` | `~54.0.17` |
| `@expo/config-plugins` | `~54.0.1` | `~54.0.4` |
| `@bsky.app/alf` | `^0.1.6` | `^0.1.7` |

#### Updated Expo Sub-packages
| Package | From | To |
|---------|------|-----|
| `expo-file-system` | `~19.0.20` | `~19.0.21` |
| `expo-font` | `~14.0.10` | `~14.0.11` |
| `expo-image-picker` | `~17.0.9` | `~17.0.10` |
| `expo-linking` | `~8.0.10` | `~8.0.11` |
| `expo-notifications` | `~0.32.14` | `~0.32.16` |
| `expo-splash-screen` | `~31.0.12` | `~31.0.13` |
| `expo-updates` | `~29.0.14` | `~29.0.16` |
| `expo-video` | `~3.0.15` | `~3.0.16` |

#### New Resolutions
| Package | Version | Notes |
|---------|---------|-------|
| `metro` | `0.83.3` | Pin metro bundler versions |
| `metro-core` | `0.83.3` | Pin metro bundler versions |
| `metro-config` | `0.83.3` | Pin metro bundler versions |
| `metro-runtime` | `0.83.3` | Pin metro bundler versions |
| `metro-source-map` | `0.83.3` | Pin metro bundler versions |
| `**/@expo/image-utils` | `0.8.12` | Updated from `0.8.7` |

#### Removed Resolutions
| Package | Notes |
|---------|-------|
| `**/@react-native-async-storage/async-storage` | No longer needed |
| `**/expo-constants` | No longer needed |
| `**/expo-device` | No longer needed |

#### Updated Patches
| Patch | Change |
|-------|--------|
| `expo-font` | `14.0.10` → `14.0.11` (renamed, same content) |
| `expo-image` | `3.0.10` → `3.0.11` (renamed, same content) |
| `expo-modules-core` | `3.0.28` → `3.0.29` (rewritten, smaller) |
| `expo-notifications` | `0.32.14` → `0.32.16` (significantly smaller: 992→170 lines) |
| `expo-updates` | `29.0.15` → `29.0.16` (renamed) |
| `react-native-drawer-layout` | `4.2.1` → `4.2.2` (renamed) |

#### New Patches
| Patch | Purpose |
|-------|---------|
| `react-native+0.81.5+005+fmt-compat-fix.patch` | Fix for fmt library compatibility |

---

### Bug Fixes

#### Session & Auth
- **Fix current session refresh bug** (`60a0edbbe` #10019) — Fixed a bug where session would not properly refresh
- **Fix email not updating in session state after email change** (`9c29b2867` #9953)
- **Ensure app session state refreshes on reactivation** (`398df1a4b` #9954)
- **Prevent welcome modal from reappearing** (`b9c92d39e` #9786)
- **Fix `findNodeHandle` usage on web** (`1ae77aa99` #10001)

#### Sheets & Dialogs
- **Fix bottom sheet crash on API <30** (`288fad67f` #10084) — Crash fix for older Android devices
- **The Great Unjanking of the Sheets** (`aa897f55a` #9973) — Major sheet behavior cleanup
- **Fix iOS sheet keyboard handling using native scrollview features** (`d4357b2cb` #9959)
- **Fix keyboard flicker when selecting a draft in composer** (`c46219bc4` #9864)
- **Fix report dialog flashing error state on open** (`f08bf5fef` #10099)
- **ContextMenu — return item to right location if keyboard hides** (`7c9f05a2a` #9963)

#### Feed & Content
- **Fix 'Lists' & 'Feeds' not displaying contents on muted profiles** (`ce000ada5` #10101)
- **Ensure mute state stays in sync after mutation** (`876e20166` #10067)
- **Fix `useResolveUriQuery`** (`eb566c5fc` #10163)
- **Fix infinite loading spinner when changing search terms** (`8a1f8997f` #9950)
- **Fix pull-to-refresh getting stuck on tab switch** (`9b24d75c9` #9951)
- **Fix draft restore to calculate shortened grapheme length for links** (`fa84c451d` #10103)
- **Add bookmarks to thread placeholder candidates** (`e28f6d2f3` #10080)
- **Dedupe trending topics** (`9cdfb40ce` #9870)
- **Remove trending post counts until accuracy improves** (`82fea6da7` #9867)
- **Fix string concat bug with trending topics muteword moderation** (`35f9ed82b` #9914)

#### UI & Rendering
- **Fix header measurement for Android content disappearing on switch** (`a95b877ea` #9964)
- **Fix clipped text when using `numberOfLines={1}` on web** (`854ae60e7` #10105)
- **Fix text overflow in post meta on web** (`2c828b575` #10122)
- **Fix Safari layout shift issue with SubtleHover** (`9b20023c8` #10135)
- **Fix Wordle score rendering — prevent Inter from overriding emoji** (`3e7e859c9` #10104)
- **Fix missing pixel row in text rendering on iOS** (`79e27e303` #9882)
- **Fix issue with site loading on older versions of Safari** (`3e7d7ce5f` #10077)
- **Fix web splash flicker and add animation** (`8c4820b1f` #9860)
- **Fix Live Now reporting container collapse on Android** (carried from previous, refined)
- **Force relayout to fix stuck header blur** (`5ee667f30` #9979)
- **Fix bottom bar badge text padding** (`9fe808f8a` #10162)

#### GIFs & Media
- **Fix GIFs breaking mute state/pausing phone audio** (`4eca13941` #9859)
- **Fix clip issue** (`bc7b6f1e1` #10131)
- **Clear link and gif caches to avoid pointing at missing files** (`197c06941` #9898)
- **Align alt text behaviour between gif types** (`e6c4a539c` #9822)
- **Hide alt text note after alt text is added** (`f30acadc7` #10132)

#### Composer & Drafts
- **Handle errors on saving a draft over char limit** (`2f156bb90` #9850)
- **Fix too many re-renders when toggling experimental feed sampling** (`894e2b89d` #10114)
- **Fix quote embed clickability in composer on web** (`122830630` #9876)
- **Detect facets when composer is opened with mention** (`1eb09b721` #9625)

#### Other
- **Fix console logging in Firefox and other browsers** (`26605ee73` #9775)
- **Fix native-only function being called on web** (`98d17fd95` #9904)
- **Fix i18n string interpolation for device name** (`864af57fd` #9806)
- **Debounce thread preferences on leading edge, reduce to 2s** (`a25bc8f24` #9851)
- **Set displayName and description to undefined if no values provided during profile edit** (`be0d00de1` #10016)

---

### UI/UX Improvements

- **Large type accessibility** (`8b67a3ec2` #10037) — Added `maxFontSizeMultiplier` throughout app for better large text support
- **'Log in' → 'Sign in'** (`06fcea988` #9875) — Consistent terminology
- **'subtitles' → 'captions'** (`e656fe688` #9941) — Aligned string terminology
- **'lightbox' → 'image viewer'** (`d9ef02737` #9807) — User-facing string rename
- **Border width reduction** (`498d321bb` #10156) — Changed borderWidths from 2 to 1 for subtler UI
- **2px margin below post text** (`9689a6daa` #9748) — Spacing refinement
- **Profile description text selectable** (`b6b364700` #9884)
- **Light haptic feedback on Edit Profile button** (`26c84399d` #9787)
- **Germ DM button** (`93d2f3a8d` #9848) — New DM button style
- **See more link for suggested users in profile header** (`226204079` #10134)
- **Show hashtag symbol for Mute option in menu** (`85ffc7798` #9942)
- **Mute word dialog return key type set to "done"** (`612a77836` #9967)
- **Add back starter packs button in ProfileMenu** (`bfd2d2019` #10069)
- **Update splash screen colors for Android** (`965b17ff8` #9821)
- **Link to labelers by DID instead of handle** (`f1c50baa4` #9777)
- **VTT cue line adjustment** (`5e8ef6aa9` #10017) — Avoid occlusion by video controls
- **Emoji regex update** (`8ac63d780` #10138) — Better detection of emoji variations and sequences
- **Profile hover card on quoted post authors** (`b12d70335` #9896)
- **Don't show chat notification if viewer is age gated** (`9c9970f68` #10042)
- **Provide age-restricted viewers option to delete account** (`7b5d5a4f7` #10041)
- **Update age assurance copy** (`8ee9709fb` #9885)
- **Wrap interaction counts with `Trans`** (`82f42e734` #9036) — Translation flexibility
- **Add plural formatting for several strings** (`3b9f0b0a6` #9576)
- **Add `Trans` to additional strings** (`d247a6568` #9981)
- **Remove duplicate punctuation** (`1a487d094` #10004)

---

### Performance

- **PostThread perf: reduce skeleton rendering time** (`c06312f09` #10092)
- **Increase parent batch size to reduce onStartReached calls** (`74d8ca8fa` #10086)
- **Lightbox memoization fix for List component** (`3b2573bc5` #9759) — Defer re-rendering when opening lightbox
- **Improve lightbox smoothness** (`220e989f3` #9200)
- **Disable KeyboardProvider preload for faster initial load** (`e49ce42a1` #9778)
- **Speed up startup by persisting some queries** (`4f1d4821c` #9594)
- **Dynamically load date-fns locale data** (`41b374236` #9535) — Smaller bundle
- **Remove Sentry Replay from web bundle** (`adc079e09` #9534) — Smaller web bundle
- **LRU screen cache for web** (`530afe87c` #10063) — Bounded memory

---

### Analytics & Metrics

- **New metrics for post menu interactions** (`850e6e6f5` #10022)
- **Search event analytics** (`7dc3b4fa8` #9949)
- **Analytics tweaks and tracking callbacks** (`4e3c3e990` #9861)
- **Pass correct log context to suggested user metrics** (`d44060a34` #10074)
- **Include recId with log event for suggested profiles** (`0e1e790c3` #10075)
- **Add `recId` to `suggestedUser:*` events** (`017120b3e` #9764)
- **Pass feed URI to sendInteractions endpoint** (`8446efb73` #9989)
- **Log languages a post is tagged with after translating** (`1386a559b` #10014)
- **Add metrics to redirect service** (`5a6942025` #10112)
- **Cache profile view before opening AfterReportDialog** (`37a82761f` #9962) — For analytics context
- **Log onboarding errors for suggested accounts** (`c188fd4f0` #9892)
- **Add A/A test** (`9b8c46cdb` #9917)

---

### Build System & CI

- **Webpack config**: Added fallback configuration in `webpack.config.js`
- **Metro resolutions**: Pinned metro at `0.83.3` across all metro packages
- **CI auto-detect go runtime version** (`a18f1b68e` #9766) — From go.mod
- **Webpack dev server freeze fix** (`c906fea77` #9865) — Fix repeated refresh freezing
- **Typecheck app.config.js** (`2953b3a09` #9771)
- **Enable ccache for dev builds on iOS** (`69a765cc0` #9770)
- **New intl:release script** — `yarn intl:pull && yarn intl:extract:all`
- **RNGH excluded from web bundle** via `expo-doctor` install exclude config
- **New `.jscodeshift/` directory**: Codemods for Toast v2 migration and ESLint warning fixes

---

### Documentation

- **Project structure and naming conventions** (`8b8acb7bd` #10002) — Major update to CLAUDE.md with:
  - New `features/` directory pattern documented
  - Guidelines for Components vs Screens vs Features
  - Legacy directory guidance (avoid writing into `/view`)
  - File and directory naming conventions
  - Platform-specific file organization preferences
  - Documentation and test co-location guidelines
- **Translation library README** (`fe8e8ce7d` #10005) — `src/lib/translation/README.md`
- **Improve error handling docs on Explore page** (`74330500d` #9890)

---

### Version Progression
| Version | Commit |
|---------|--------|
| 1.117.0 | `5642bc40b` |
| 1.118.0 | `75242b9b1` → `1281eb01e` (prep) |
| 1.118.1 | `8c705864a` (patch) |
| 1.119.0 | `041e34858` → `317ce2b3d` (prep) |
| 1.119.1 | `cc4a436e4` |
| 1.120.0 | `7fd6f8f04` |

---

### Migration Guide for Forks

1. **TypeScript 6.0**: Update `typescript` to `^6.0.2`. Review any custom type definitions for compatibility.

2. **Lingui v5 Migration**:
   - Replace all `@lingui/macro` imports with `@lingui/react/macro`
   - Update babel config: `macros` → `@lingui/babel-plugin-lingui-macro`
   - Move from `lingui.config.js` to `lingui.config.ts` with `defineConfig`
   - Compiled messages are now `.ts` files instead of `.js`
   - Remove `babel-plugin-macros` from dependencies

3. **Dependency Cleanup**:
   - Remove: `@mattermost/react-native-paste-input`, `@zxing/text-encoding`, `await-lock`, `history`, `react-native-get-random-values`, `react-native-url-polyfill`, `superjson`, `expo-task-manager`, `ts-node`, `@testing-library/jest-native`, `babel-plugin-macros`
   - Add: `expo-paste-input`, `@bsky.app/expo-translate-text`, `@emoji-mart/data`, `@growthbook/growthbook`, `@lingui/core`, `@lingui/babel-plugin-lingui-macro`, `expo-video-thumbnails`, `setimmediate`, `@crowdin/cli`
   - Update `@atproto/api` to `^0.19.5`

4. **ESLint**: New rules will flag:
   - Imports of packages not in package.json (`no-extraneous-dependencies`)
   - Node.js built-in module imports (`no-nodejs-modules`)
   - useState destructuring mismatches (`hook-use-state`)
   - Run `yarn lint` and fix new warnings/errors

5. **Live Now imports**: Update any imports from `#/components/live/` to `#/features/liveNow/`

6. **Legacy modal removals**: If extending DeleteAccount or ContentLanguageSettings modals, they've been replaced by ALF dialogs in new locations.

7. **Patch maintenance**: Update patch file versions for expo-font, expo-image, expo-modules-core, expo-notifications, expo-updates, react-native-drawer-layout. Remove lingui and paste-input patches.

8. **Testing**: After merge, run:
   ```bash
   yarn install
   yarn lint
   yarn typecheck
   yarn test
   ```

---
*This changelog documents changes merged from the upstream Bluesky Social App repository.*

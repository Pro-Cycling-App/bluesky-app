# Subtree Changelog

## Update from upstream/main (e4485da4f → d89687df7)

### Summary
Update from upstream Bluesky Social App repository with 151 commits. This update includes significant UI/UX improvements, a complete reporting system overhaul, new branding colors and icons, performance optimizations, and substantial component additions including the new SegmentedControl and Toggle Panel components.

### Breaking Changes

#### Brand Color Update - New Blue
- **Commits**: `54af17044`, `d49d43f6e`, `c6215065b`, `5efae600d`
- **Change**: Primary blue color changed from `#1185FE` to `#006AFF`
- **Affected Areas**:
  - App icons (iOS and Android) completely redesigned
  - FAB (Floating Action Button) now uses theme-aware primary color
  - Bottom bar active indicator uses new brand color
  - Logo component now uses `t.palette.primary_500` instead of hardcoded `colors.blue3`
  - Android adaptive icon background color changed
- **Impact**: Any hardcoded references to the old blue color should be updated

#### React Native Patches Restructured
- **Commit**: `8b00b8f94`
- **Change**: Single monolithic `react-native+0.81.5.patch` split into multiple targeted patches:
  - `react-native+0.81.5+001+initial.patch` - Base patches
  - `react-native+0.81.5+002+ScrollForwarder.patch` - Scroll forwarding functionality
  - `react-native+0.81.5+003+ScrollView-disable-recycling.patch` - ScrollView recycling fix
- **Impact**: `patch-package` updated from `^6.5.1` to `^8.0.1` to support multi-patch format. Forks with custom RN patches may need restructuring.

#### Reporting Dialog Complete Overhaul
- **Commit**: `d6ca9e6b4`
- **Change**: The entire reporting system has been restructured:
  - Old files removed: `src/components/ReportDialog/*`, `src/components/dms/ReportDialog.tsx`, `src/lib/moderation/useReportOptions.ts`
  - New location: `src/components/moderation/ReportDialog/*`
  - New architecture with categories and sub-options using Ozone report definitions
  - Added new report categories: child safety, violence/physical harm, sexual/adult content, harassment/hate, misleading, rule breaking, self harm, other
  - Uses `ToolsOzoneReportDefs` from `@atproto/api` for standardized reason types
- **Impact**: Any custom reporting integrations need complete refactoring

#### Removed Components
- **FollowButton.tsx** (`5208aa4a6`): Removed `src/view/com/profile/FollowButton.tsx` - use ProfileCard components instead
- **EmptyStateWithButton.tsx** (`ddf82dcc4`): Removed `src/view/com/util/EmptyStateWithButton.tsx`
- **Old useReportOptions.ts** (`171162649`): Removed `src/lib/moderation/useReportOptions.ts` - use new moderation dialog system

#### API Changes
- **@atproto/api**: Upgraded from `^0.17.0` to `^0.18.0`
- **@atproto/dev-env**: Upgraded from `^0.3.172` to `^0.3.181`

### New Components

#### SegmentedControl Component
- **Commit**: `92926a241`
- **Location**: `src/components/forms/SegmentedControl.tsx`
- **Features**:
  - Compound component pattern with `Root`, `Item`, and `ItemText`
  - Supports `tabs` and `radio` accessibility roles
  - Animated slider with platform-specific shadows
  - Size variants: `small` (32px) and `large` (40px)
  - Haptic feedback on selection
- **Usage**:
```tsx
<SegmentedControl.Root value={value} onChange={setValue} label="Options" type="radio">
  <SegmentedControl.Item value="one">
    <SegmentedControl.ItemText>One</SegmentedControl.ItemText>
  </SegmentedControl.Item>
  <SegmentedControl.Item value="two">
    <SegmentedControl.ItemText>Two</SegmentedControl.ItemText>
  </SegmentedControl.Item>
</SegmentedControl.Root>
```

#### Toggle Panel Component
- **Commit**: `b422765ae`
- **Location**: `src/components/forms/Toggle/Panel.tsx`
- **Features**:
  - Container for toggle items with visual grouping
  - `Panel`, `PanelText`, `PanelIcon`, and `PanelGroup` exports
  - Supports `adjacent` prop for connected panel styling
  - Active state changes background to primary color
- **Usage**: See Threadgate dialog for reference implementation

#### Debug Field Display
- **Commit**: `339f3320c`
- **Location**: `src/components/DebugFieldDisplay.tsx`
- **Purpose**: Internal debugging tool for field inspection

#### iOS-Specific Image Saving
- **Commit**: `c9033dbc3`
- **Location**: `src/lib/media/save-image.ios.ts`
- **Change**: Platform-split implementation avoids requesting photo library read access on iOS when saving images

### Major Feature Updates

#### Onboarding Starter Packs Step
- **Commit**: `1127e5024`
- **Location**: `src/screens/Onboarding/StepSuggestedStarterpacks/`
- **Features**:
  - New onboarding step showing suggested starter packs
  - Includes `StarterPackCard` component
  - Dedicated query hook: `useOnboardingSuggestedStarterPacksQuery`
  - Limited to English speakers (statsig gate)
  - Shows up to 6 starter packs

#### Threadgate Dialog Redesign
- **Commit**: `b422765ae`
- **Features**:
  - Completely refreshed UI using new Panel components
  - Persistent settings toggle (saves preferences for future posts)
  - Tooltip hints for first-time users
  - New tiny chevron icon for compact buttons
  - Storage hook: `src/storage/hooks/threadgate-nudged.ts`

#### Profile Minimal Header
- **Commit**: `13c4ef83f`
- **Change**: Profiles now show a minimal header when scrolling, displaying:
  - Profile name and avatar
  - Follow/action buttons
  - Proper moderation handling
- **Architecture**: Uses `ProfileCard` components for consistent rendering

#### Follow Back Button in Notifications
- **Commit**: `2f678a3cc`
- **Location**: `src/view/com/notifications/NotificationFeedItem.tsx`
- **Feature**: Follow notifications now include a "Follow back" button with shadow cache integration

#### Select Account Redesign
- **Commit**: `ad91029f1`
- **Changes**:
  - Updated visual design for account switching
  - Added "logged out" indicator for expired sessions
  - New JWT utility: `src/lib/jwt.ts` for token inspection

#### Interests Moved Client-Side
- **Commit**: `be53a168b`
- **Location**: `src/lib/interests.ts`
- **Changes**:
  - Interests list now defined client-side instead of server-fetched
  - Added `finance` as a new interest category
  - Exports `interests`, `popularInterests`, and `useInterestsDisplayNames`

### Dependency Updates

#### Core Dependencies
| Package | Previous | Current |
|---------|----------|---------|
| `expo` | `^54.0.9` | `^54.0.22` |
| `react-native` | `0.81.4` | `0.81.5` |
| `@atproto/api` | `^0.17.0` | `^0.18.0` |
| `patch-package` | `^6.5.1` | `^8.0.1` |
| `expo-image-crop-tool` | `^0.1.8` | `^0.4.0` |

#### Expo Packages Updated
- `expo-camera`: `~17.0.8` → `~17.0.9`
- `expo-dev-client`: `~6.0.12` → `~6.0.16`
- `expo-device`: `~8.0.8` → `~8.0.9`
- `expo-file-system`: `~19.0.14` → `~19.0.17`
- `expo-font`: `~14.0.8` → `~14.0.9`
- `expo-image`: `~3.0.8` → `~3.0.10`
- `expo-notifications`: `~0.32.11` → `~0.32.12`
- `expo-system-ui`: `~6.0.7` → `~6.0.8`
- `expo-task-manager`: `~14.0.7` → `~14.0.8`
- `expo-updates`: `~29.0.11` → `~29.0.12`
- `expo-video`: `~3.0.11` → `~3.0.13`
- `expo-web-browser`: `~15.0.7` → `~15.0.9`

#### Dev Dependencies Updated
- `@react-native/babel-preset`: `0.81.4` → `0.81.5`
- `@react-native/eslint-config`: `^0.81.4` → `^0.81.5`
- `@react-native/typescript-config`: `^0.81.4` → `^0.81.5`
- `@types/jest`: `^29.4.0` → `29.5.14`
- `jest-expo`: `~54.0.12` → `~54.0.13`

#### New Dev Dependency
- `ts-plugin-sort-import-suggestions`: `^1.0.4` - TypeScript plugin for sorting import suggestions

### New Patches

#### react-native-reanimated+3.19.1.patch
- **Commit**: `e28aa7d1a`
- **Purpose**: Patches out `PerformanceMonitor` component that was causing issues

#### react-native-uitextview+1.4.0.patch
- **Commit**: `03adb4260`
- **Purpose**: Fixes fractional font size rendering issue

### Client Events & Analytics

New client events added for better analytics:
- `post:view` - Tracks post views in feeds and on post pages (`002a63b12`)
- `profileCard:seen` - Tracks when profile cards are viewed (`084f60c88`)
- `profile:follow` - Enhanced with position tracking (`084f60c88`)
- `threadgate:*` - Composer threadgate metrics (`bc62b3da7`)
- `feed:click` - Desktop feed clicks (`fb058afa5`)
- `feed:error` - Empty feed error tracking (`5a4e012fd`)
- `suggestedUser:seeMore` - See more link clicks (`6086bc5d5`)

### Haptics

New haptic feedback added:
- SegmentedControl selection (`f53d5b6f2`)
- Profile action buttons (`fe5a623b4`)

### UI/UX Improvements

#### Button Changes
- Buttons are round again by default (`bcd0e4019`)
- Added `rectangular` shape option (`e663e824b`)
- Size must be set for rounded styling (`cb2af3cd1`)

#### Icon Updates
- Slightly more rounded repost icon (`dfe9f9fa0`)
- New tiny chevron icon (`b422765ae`)
- Dark/tinted icon variants for iOS (`5efae600d`)
- Updated Android adaptive icon with monochrome support (`c6215065b`)

#### Visual Polish
- Settings divider color tweak (`68a4d8445`)
- Web dim background color update (`de6d2f191`)
- Feed title line height fix (`14604f735`)
- Android ripple effect on feed images (`b1f1b2050`)
- 0.5px media border on high DPI web screens (`bd3594cee`)

### iOS Specific

#### Privacy Manifests Updated
- Added `NSPrivacyCollectedDataTypes` declarations for:
  - Crash data
  - Performance data
  - Diagnostic data
- All marked as not linked to user identity and not used for tracking

#### iOS 26/Xcode 26 Compatibility
- Builds now use `macos-26-xlarge` runner (`31fac4a62`)
- Xcode version updated (`25d44d893`)

#### Image Saving
- No longer requests photo library read access when saving (`c9033dbc3`)
- Increased download timeout for image saving (`24c96ac88`)

### Android Specific

#### Icon Updates
- New adaptive icon with monochrome variant
- Background color changed to `#006AFF`
- Fixed icon file naming (`2d626f7bb`)

#### Fixes
- 2FA input now properly paste-able (`9823bc98e`)
- Dialog crash fix with try/catch (`ec2c580aa`)
- Lightbox performance improvement (`feab782ba`)
- Image gallery fallbacks when saving (`e3f596740`)
- Wake-from-background OTA disabled (`55e9298ee`)

### Build System

#### E2E Scripts Renamed
- `e2e:metro` → `e2e:build`
- `e2e:metro-android` → `e2e:build-android`
- Added `e2e:start` for starting Metro bundler

#### CI/CD Updates
- macOS 26 runner for builds (`31fac4a62`)
- Android CI cache clearing on second build (`ea00663e7`)
- Android TestFlight CI fixes (`12e57378e`)
- iOS debug files now sent to Sentry (`d7e842975`)

### TypeScript Configuration

- Added `ts-plugin-sort-import-suggestions` plugin
- Configured to prioritize `#/` and `@lingui/macro` imports
- Deprioritizes `react-native-reanimated/lib` imports

### Pattern Changes

#### StyleSheet.flatten → ALF flatten
- Multiple components migrated from RN's `StyleSheet.flatten` to ALF's `flatten` utility
- Fixes for undefined return values (`c92ac4abc`)
- Note: Some usages restored where flatten is specifically needed (`531ff6c43`, `a275e8999`)

#### niceDate Function Enhancement
- `src/lib/strings/time.ts` now accepts optional `dateStyle` parameter
- Defaults to `'long'` for backwards compatibility

### Bug Fixes

- Video letterboxing issue (`1caac024a`)
- Password autofill on iOS (`ef97a20c2`)
- React Query error in feeds (`bb365856c`)
- Right nav overflow on web (`e5f313164`)
- Apple Music song embeds (`faa49feec`)
- Profile suggestions animation when no data (`bd5d10282`)
- Hover card line height (`bb908a553`)
- Thread computation for `isLastSibling`, `isLastChild`, `replyIndex` (`27926f093`)
- Share intents on app first launch (`f5a5eee32`)

### Version Bump
- Previous: 1.109.0
- Current: 1.111.0

### Migration Guide for Forks

After pulling in these upstream changes:

1. **Update Dependencies**:
   - Run `yarn install` to update all packages
   - Note: `patch-package` version bump may require reinstalling

2. **Check Hardcoded Colors**:
   - Search for `#1185FE` or `#1083fe` and update to `#006AFF`
   - Update any hardcoded `colors.blue3` references to use theme tokens

3. **Update React Native Patches**:
   - If you have custom RN patches, migrate to the new multi-file format
   - Files should be named `react-native+VERSION+XXX+description.patch`

4. **Reporting System Migration**:
   - Any custom reporting UI needs complete refactoring
   - Use new `src/components/moderation/ReportDialog/` structure
   - Implement `useReportOptions` hook for category-based reporting

5. **Remove Deleted Components**:
   - Replace `FollowButton` usage with `ProfileCard` components
   - Remove `EmptyStateWithButton` imports
   - Update any old `useReportOptions` imports

6. **Test New Features**:
   - Verify SegmentedControl styling matches your theme
   - Test threadgate dialog with persistence toggle
   - Confirm minimal profile header works correctly

7. **iOS Builds**:
   - Ensure Xcode 26 compatibility
   - Verify privacy manifest declarations are acceptable

8. **Android Builds**:
   - Test new adaptive icon rendering
   - Verify 2FA paste functionality
   - Test image saving to gallery

---
*This changelog documents changes merged from the upstream Bluesky Social App repository.*

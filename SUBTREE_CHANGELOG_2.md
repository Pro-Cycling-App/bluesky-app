# Subtree Changelog

## Update from upstream/main (45e64757d → e4485da4f)

### Summary
Major update from upstream Bluesky Social App repository with 590 commits. This update includes a significant Expo SDK upgrade, major dependency updates, and substantial architectural changes that may require careful migration planning for forks.

### Breaking Changes 🚨

#### Expo SDK and React Native Upgrade
- **Expo SDK**: Upgraded from SDK 53 to SDK 54
- **React Native**: Upgraded from 0.79.3 to 0.81.4
- **React/React-DOM**: Upgraded from 19.0.0 to 19.1.0
- **Node.js Requirement**: Bumped from 18.18.2 to 20.19.4
- **Impact**: All Expo modules need to be updated to compatible versions

#### Environment Configuration
- **Complete Restructuring**: The `.env.example` file has been completely rewritten
- **New Required Variables**:
  - `EXPO_PUBLIC_ENV`
  - `EXPO_PUBLIC_RELEASE_VERSION`
  - `EXPO_PUBLIC_BLUESKY_PROXY_DID`
  - `EXPO_PUBLIC_CHAT_PROXY_DID`
  - `EXPO_PUBLIC_SENTRY_DSN`
  - `BAPP_CONFIG_DEV_URL`
- **Impact**: Forks must update their environment configuration

#### Core Component Redesigns
- **Button Component**: Complete redesign - any custom button implementations need migration
- **Dialog System**: Major refactoring of the entire dialog system
- **ALF Design System**: Migrated from internal to external package `@bsky.app/alf`
- **Toast Notifications**: Replaced with Sonner-based system
- **Impact**: Custom components extending these will need updates

#### Removed Features/APIs
- **Post Thread Query**: Removed `src/state/queries/post-thread.ts`
- **SubtleWebHover**: Removed in favor of unified `SubtleHover`
- **Impact**: Any code depending on these APIs needs refactoring

### Major New Features

#### User Features
- **Bookmarks System**: Full bookmark functionality for posts
- **Age Assurance**: New age verification system with UI components
- **Policy Update Overlay**: Automatic policy update notifications
- **Interest Tabs**: Content discovery based on interests
- **Location/GPS Features**: Added location-based functionality
- **Welcome Modal**: New experience for logged-out users

#### Developer Features
- **Device Attestation**: Security feature for signup flow
- **MMKV Storage**: Added `@bsky.app/react-native-mmkv` for efficient storage
- **Enhanced Metrics**: New telemetry and metrics integration
- **Thread Gates**: New implementation for thread visibility control

### Dependency Updates

#### Major Package Updates
- `@atproto/api`: 0.15.15 → 0.17.0
- `@sentry/react-native`: 6.14.0 → 6.20.0
- `expo-*` packages: Updated for SDK 54 compatibility

#### New Dependencies
- `@bsky.app/alf`: 0.1.5
- `@bsky.app/react-native-mmkv`: 2.12.5
- `react-native-device-attest`: 0.1.6
- `expo-keep-awake`: 15.0.7
- `expo-location`: 19.0.7
- `sonner-native`: Toast notification library

### Infrastructure Updates

#### Build System
- New GitHub Actions workflow for PR comments
- Updated Android and iOS build processes
- Enhanced EAS (Expo Application Services) configurations
- Docker configuration improvements

#### Native Modules
- Updated GIF view module for Android
- Enhanced emoji picker module
- Bottom sheet module improvements
- Background notification handler updates
- Scroll forwarder updates for iOS

### UI/UX Updates

#### Navigation and Screens
- Updated navigation structure and screen transitions
- New hashtag screens with authentication prompts
- Improved explore and search modules
- Trending videos interstitial redesign

#### Content Moderation
- Enhanced moderation UI components
- Improved report dialog
- Updated content hiding mechanisms
- New verification check system

### Migration Guide for Forks

1. **Update Node.js** to version 20.19.4 or higher
2. **Review Environment Variables**: Update `.env` files according to new structure
3. **Test Expo SDK Upgrade**: Run comprehensive tests after updating
4. **Update Custom Components**: Migrate any custom buttons, dialogs, or toast notifications
5. **Replace Deprecated APIs**: Update any usage of removed post-thread queries
6. **Test Native Features**: Verify all native modules work correctly
7. **Review Build Scripts**: Update any custom build processes for new requirements

### Version Bump
- Previous: 1.104.0
- Current: 1.109.0

---
*This changelog documents changes merged from the upstream Bluesky Social App repository.*
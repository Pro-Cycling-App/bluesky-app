# Subtree Changelog

## Update from upstream/main (75ffb3d24 → b3be5a26e)

### Summary
Updated repository from upstream Bluesky Social App repository. Fast-forward merge with 162 files changed, significant refactoring of post controls and addition of new features.

### Major Changes

#### Component Refactoring
- **Post Controls Reorganization**: Moved post controls from `src/view/com/util/post-ctrls/` to `src/components/PostControls/`
  - New modular post control system with dedicated components
  - Improved repost button implementation
  - Better separation of concerns

#### New Features
- **Enhanced Share Menu**: New share functionality with recent chats integration
  - `src/components/PostControls/ShareMenu/RecentChats.tsx`
  - `src/components/PostControls/ShareMenu/ShareMenuItems.tsx`
  - Platform-specific implementations for web and native

- **Live Streaming Improvements**: 
  - Added link preview functionality (`src/components/live/LinkPreview.tsx`)
  - Enhanced live dialog components
  - Removed temporary files

- **Media Handling**: New save image functionality (`src/lib/media/save-image.ts`)

- **Video Improvements**: Added bandwidth estimation for video embeds

#### Localization
- **New Language Support**: Added Portuguese (Portugal) locale
  - Complete translation file: `src/locale/locales/pt-PT/messages.po`
- **Updated Translations**: Refreshed translation files for all supported languages

#### New Icons
- `arrowOutOfBoxModified_stroke2_corner2_rounded.svg`
- `arrowShareRight_stroke2_corner2_rounded.svg`
- `arrowTop_stroke2_corner0_rounded.svg`
- `chainLink_stroke2_corner0_rounded.svg`

#### Infrastructure Updates
- **New Patch**: Added react-native-view-shot patch files
  - `patches/react-native-view-shot+4.0.3.patch`
  - `patches/react-native-view-shot+4.0.3.md`
- **Dependency Updates**: Various package.json and yarn.lock updates
- **Build System**: Updates to Docker, GitHub workflows, and build configurations

#### Developer Experience
- **Debugging**: New discover debug component for post controls
- **State Management**: Renamed trending-config to service-config with enhanced functionality
- **Metrics**: Improved logging and metrics collection

### Notable File Changes
- **Deleted**: `src/view/com/util/post-ctrls/PostCtrls.tsx` (replaced by new modular system)
- **Moved**: Post dropdown components relocated and refactored
- **Enhanced**: Profile menu, notification handling, and feed management

### Configuration Updates
- Updated app.config.js
- Enhanced crowdin.yml for localization
- Modified lingui.config.js for translation management

---
*This changelog documents changes merged from the upstream Bluesky Social App repository.*
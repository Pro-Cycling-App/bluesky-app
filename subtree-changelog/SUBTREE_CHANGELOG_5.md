# Subtree Changelog

## Update from upstream/main (5989bf7fa → dd7bb1bc1)

### Summary
Update from upstream Bluesky Social App repository with 19 commits. This update includes a major analytics migration from Statsig to a new in-house analytics system with GrowthBook for feature flags, new archival storage using idb-keyval, Live Events improvements, and various cleanup and fixes.

### Breaking Changes

#### Analytics Migration - Statsig Removed
- **Commits**: `25a1ed120`, `df89b79c2`, `bd9e8244f`
- **Change**: Complete removal of Statsig SDK, replaced with new analytics architecture
- **Removed Files**:
  - `src/lib/statsig/statsig.tsx`
  - `src/lib/statsig/gates.ts`
  - `src/logger/bitdrift/` (entire directory)
  - `src/logger/transports/bitdrift.ts`
  - `eslint/use-typed-gates.js` (custom ESLint rule for Statsig gates)
- **Added Files**:
  - `src/analytics/` (new analytics system)
    - `index.tsx` - Main analytics provider and hooks
    - `features/index.ts` - Feature flag system
    - `features/types.ts` - Feature flag types
    - `identifiers/` - Device and session ID management
    - `metrics/` - Metrics client and types
    - `metadata.ts` - Analytics metadata handling
    - `utils.ts` - Analytics utilities
    - `misc/refParams.ts` - Referral parameter handling
    - `PassiveAnalytics.tsx` - Passive analytics component
- **Impact**: Any code using Statsig gates (`useGate`, `Gate` component) needs migration to new feature flag system. The new system uses GrowthBook under the hood.

#### Analytics System Details

**Backends & Endpoints:**

| Service | Default Endpoint | Env Variable |
|---------|------------------|--------------|
| Metrics | `https://events.bsky.app/t` | `EXPO_PUBLIC_METRICS_API_HOST` |
| GrowthBook | `https://events.bsky.app/gb` | `EXPO_PUBLIC_GROWTHBOOK_API_HOST` |
| GrowthBook Key | `sdk-7gkUkGy9wguUjyFe` | `EXPO_PUBLIC_GROWTHBOOK_CLIENT_KEY` |

The `MetricsClient` batches events and sends them every 10 seconds (or when batch hits 100 events). On web it uses `navigator.sendBeacon`, on native it uses `fetch`.

**Event Identification:**

Events are fully identified with user DIDs. Metadata attached to every event:

| Field | Source | Identifies User? |
|-------|--------|------------------|
| `metadata.session.did` | Logged-in user's DID | Yes |
| `metadata.base.deviceId` | Persisted UUID | Pseudonymous |
| `metadata.base.sessionId` | Per-app-launch UUID | Ties events in session |
| `metadata.geolocation` | Country/region | No |
| `metadata.session.isBskyPds` | Whether on bsky.social | No |

Many event payloads also include DIDs for the target of the action:
- `authorDid` - who wrote the post (on post interactions)
- `followeeDid` - who was followed (on follow events)
- `suggestedDid` - who was suggested (on suggestion events)

This means `session.did` = who performed the action, and payload DIDs = who the action was about.

**For Forks:**

To use your own metrics backend, set environment variables:
```bash
EXPO_PUBLIC_METRICS_API_HOST=https://your-metrics-server.com
```

Your server needs to accept POST to `/t` with JSON body:
```json
{
  "events": [{
    "time": 1706000000000,
    "event": "post:like",
    "payload": { "uri": "...", "authorDid": "..." },
    "metadata": { "deviceId": "...", "sessionId": "...", "session": { "did": "..." }, ... }
  }]
}
```

#### Removed Utilities
- **Commit**: `5989bf7fa`
- **Removed Files**:
  - `src/lib/hooks/useAppState.ts` - Replaced by `src/lib/appState.ts`
  - `src/lib/hooks/useCallOnce.ts` - Replaced by `src/lib/once.ts`
- **Impact**: Update imports if using these hooks directly

### New Features

#### Archival Storage
- **Commit**: `f26ac583e`
- **Location**: `src/storage/archive/`
- **Features**:
  - New `Archive` class for persistent archival storage
  - Uses `idb-keyval` for IndexedDB-backed storage on web
  - Scope-based key management with type safety
  - Separate implementations for native and web platforms
- **Files**:
  - `src/storage/archive/index.ts` - Main Archive class
  - `src/storage/archive/schema.ts` - Storage schema definitions
  - `src/storage/archive/db/index.ts` - Native DB implementation (MMKV)
  - `src/storage/archive/db/index.web.ts` - Web DB implementation (idb-keyval)
  - `src/storage/archive/db/types.ts` - DB interface types
- **Current Status**: Infrastructure only - schema is empty and nothing uses it yet. Likely preparation for future account data export/archival features (an `upstream/archive` branch exists).

#### Live Events Improvements
- **Commits**: `3b7806963`, `f82bc98d8`, `97083bb55`
- **Features**:
  - Dismissible sidebar Live Event Feeds banner
  - More frequent Live Event refreshing
  - Bluecast enabled for Live Now feature

### Dependency Updates

#### New Dependencies
| Package | Version |
|---------|---------|
| `@growthbook/growthbook-react` | `^1.6.2` |
| `idb-keyval` | `^6.2.2` |

#### Removed Dependencies
| Package | Notes |
|---------|-------|
| `statsig-react-native-expo` | Replaced by GrowthBook |

#### Resolutions Added
| Package | Version | Notes |
|---------|---------|-------|
| `@types/estree` | `1.0.6` | Resolve version conflicts |

### Bug Fixes

- **fix IS_DEV in app config** (`dd7bb1bc1`) - Fixed development environment detection
- **Fix Live Now reporting container collapse on Android** (`b4f57b94a`) - UI fix for Android
- **Refactor GrowthHack to always render on iOS with enabled prop** (`6534630d1`) - iOS rendering fix

### UI/UX Improvements

- **Update suggested accounts heading** (`d31bc7f33`) - Improved heading text
- **Allow users to dismiss sidebar LEF banner** (`f82bc98d8`) - User can now hide Live Events banner

### Build System & CI

- **Copyright year bump to 2026** (`8614e5e10`) - Updated LICENSE copyright
- **Resolve same version of @types/estree** (`4f01d4cc8`) - Fixed dependency resolution
- **E2E script updates** - Added cache clear flag, simplified test path

### Version Bump
- Previous: 1.115.0
- Current: 1.116.0

### Migration Guide for Forks

1. **Analytics Migration**:
   - Remove any imports from `#/lib/statsig/statsig`
   - Replace `useGate('gate_name')` with the new feature flag system from `#/analytics`
   - Remove `statsig-react-native-expo` from dependencies
   - Add `@growthbook/growthbook-react` and `idb-keyval` to dependencies

2. **Hook Replacements**:
   - Replace `import {useAppState} from '#/lib/hooks/useAppState'` with `import {useAppState} from '#/lib/appState'`
   - Replace `import {useCallOnce} from '#/lib/hooks/useCallOnce'` with utilities from `#/lib/once`

3. **Testing**:
   - Verify feature flags work correctly with GrowthBook
   - Test archival storage on both web and native
   - Verify Live Events dismissal persists

---
*This changelog documents changes merged from the upstream Bluesky Social App repository.*

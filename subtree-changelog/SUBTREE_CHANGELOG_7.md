# Subtree Changelog

## Update from upstream/main (e0ea778e5 → 7c5a9ab40)

### Summary
Update from upstream Bluesky Social App repository with **265 commits** spanning versions **1.120.0 through 1.122.0**. This is the most integration-impactful update in the series so far, dominated by two things:

1. **A full toolchain migration**: **Yarn 1 → pnpm** (with a new `pnpm-workspace.yaml`), **patch-package → pnpm `patchedDependencies`** (every patch renamed `+`→`@` and the React Native patches consolidated), **`tsc` → `tsgo`** (the Go-based TypeScript native preview compiler), **Node 20 → 24**, and **husky 8 → 9**.
2. **Group Chats** ("group clip clops"/"clops"), a very large multi-surface DM feature touching dozens of commits.

Other notable features: a **new rich-text composer + autocomplete**, a new **`EmojiPicker` component**, **GIF backend migrated from Tenor to KLIPY**, **image grid → carousel** in embeds, **on-device language detection** (replacing `lande`), **web split-view DMs**, **DM message reactions**, a **global keyboard shortcut handler**, and **user-submitted error reports**. Cross-cutting dependency changes include **`@atproto/api` 0.19 → 0.20 (now ESM)**, **`multiformats` 9 → 13**, the new **`@bsky.app/video`** (replacing `@haileyok/bluesky-video`), and new ESLint rules (`no-explicit-any`, enforced named `react` imports).

---

### ⚠️ Breaking Changes (read these first for monorepo integration)

#### 1. Package manager: Yarn 1 → pnpm
- **Commit**: `ff731849b` "Migrate from Yarn 1 to pnpm" (#10465), plus a long tail of CI fixups (`5b7bc56a1`, `5b622079b`, `79872b8be`, `52342f1cd`, `af6041d7b`, `490881625`, `3d4ba946b`, `683ae7610`).
- **`package.json` changes**:
  - `packageManager`: `yarn@1.22.22` → `pnpm@11.1.3+sha512…`
  - `prepare`: `is-ci || husky install` → `is-ci || husky` (husky 9 syntax)
  - `postinstall`: `patch-package && yarn intl:compile-if-needed` → `pnpm intl:compile-if-needed` (no more patch-package)
  - All `yarn …` script invocations rewritten to `pnpm …`
  - **`resolutions` block removed** from `package.json` (Yarn-only). Replaced by `overrides:` in `pnpm-workspace.yaml`.
  - **`lockfile-lint` dependency + config block removed** (Yarn-lock specific).
- **New file `pnpm-workspace.yaml`** (the new center of dependency config):
  ```yaml
  nodeLinker: 'hoisted'          # keeps a flat node_modules like Yarn (no .pnpm virtual store) — eases RN/Metro compat
  autoInstallPeers: true
  strictPeerDependencies: false
  overrides: { … }               # replaces package.json "resolutions"
  allowBuilds: { … }             # explicit allowlist of packages permitted to run install/build scripts
  patchedDependencies: { … }     # replaces patch-package (see below)
  minimumReleaseAgeExclude: ['@atproto/*', '@bsky.app/*']  # supply-chain delay, with bsky packages exempt
  ```
- **Lockfiles**: `yarn.lock` (root + `bskyembed`) removed; `pnpm-lock.yaml` added at root and in `bskyembed`. Sub-projects (`bskyogcard`, `dev-env`) still have their own `yarn.lock` at the time of this update.
- **Impact on the fork**: This is the single biggest change for our monorepo subtree. Our local patch-management WIP (the stale `+`→`@` rename, stashed during this update) was anticipating exactly this — upstream has now done it properly. The subtree integration must adopt pnpm's `patchedDependencies`/`overrides` model.

#### 2. Patch system: patch-package → pnpm `patchedDependencies`
- **Removed deps**: `patch-package`, `postinstall-postinstall`.
- **Every patch file renamed** from patch-package's `name+version.patch` to pnpm's `name@version.patch`, and each must now be **explicitly registered** in `pnpm-workspace.yaml › patchedDependencies` (pnpm does not auto-discover the `patches/` directory the way patch-package did).
- **React Native patches consolidated**: the five split patches (`react-native+0.81.5+001+initial` … `+005+fmt-compat-fix`) are removed and replaced by a single `react-native@0.81.5.patch` (~168 lines).
- **`react-native-compressor` patch shrank dramatically**: 516 → 59 lines.
- **New patches**: `expo-glass-effect@55.0.8` (iOS 26), `expo-image-picker@17.0.11`, `react-native-keyboard-controller@1.21.7`.
- **Version-bumped patches**: `expo-modules-core@3.0.30`, `expo-notifications@0.32.17`, `expo-updates@29.0.17`, `react-native-drawer-layout@4.2.3`, plus `@`-renames for `expo-image`, `expo-haptics`, `expo-media-library`, `react-native-date-picker`, `react-native-pager-view`, `react-native-reanimated`, `react-native-svg`, `react-native-uitextview`, `react-native-view-shot`, `sonner-native`.
- **Patches still present** (`@`-named): `@discord__bottom-sheet@4.6.1`, `@sentry__react-native@6.20.0`, and all the above.

#### 3. TypeScript compiler: `tsc` → `tsgo` (native preview)
- **Commits**: `179f3c82e` "Use `tsgo` for typechecking" (#9474), `6fd81fe5d` "ESM atproto packages" (#10543).
- **Change**: `typecheck` script is now `tsgo --project ./tsconfig.check.json` instead of `tsc …`.
- **New devDep**: `@typescript/native-preview` `^7.0.0-dev.20260428.1` (the Go-based TypeScript compiler, ~10× faster typechecking).
- **Impact**: `pnpm typecheck` requires `tsgo` (provided by `@typescript/native-preview`). Editors/CI that call `tsc` directly should be reviewed. `tsgo` is a preview — occasional behavioral differences from `tsc` are possible.

#### 4. Node 20 → 24
- **Commits**: `6f4ae14f3` "Update to Node.js v24" (#10419), `b4054bf2a` "Update GitHub Actions to Node.js v24-compatible versions" (#10539), `683ae7610` "[pnpm] Install node version via devEngines.runtime".
- **`package.json` `engines`**: `node: ">=20"` → `node: ">=24.15.0"`, plus a new `engines.runtime` block (`{ name: node, version: ^24.15.0, onFail: download }`) so pnpm can auto-provision Node 24.
- **`@types/node`**: `^20.14.3` → `^24.12.2`.
- **`.nvmrc`**: now `24.15.0`.
- **Impact**: Build environments and CI must use Node 24.15+.

#### 5. ESLint: `no-explicit-any` + enforced named `react` imports
- **Commits**: `28327459e` "Enable @typescript-eslint/no-explicit-any" (#10383), `b9561f78e` "Enforce using named imports from 'react'" (#10259), `75c9e2c18` upgrade `eslint-plugin-simple-import-sort` to v13 (#10260).
- **Impact**: `any` is now an error (matches our global rule of never using `any`). `import React from 'react'` default-import style is disallowed — use named imports (`import {useState} from 'react'`). Fork code may need fixes; run `pnpm lint`.

#### 6. `@atproto/api` 0.19 → 0.20 and ESM atproto packages
- **Commits**: `2f4049e6e` "Bump atproto to latest" (#10586), `6fd81fe5d` "ESM atproto packages" (#10543).
- **Change**: `@atproto/api` `^0.19.5` → `0.20.4` (pinned); added `@atproto/syntax` `0.6.1`. atproto packages are now ESM, which required Jest `transformIgnorePatterns`/`moduleNameMapper` additions (see Dependency Updates).
- **Impact**: Check for atproto SDK API changes; ESM resolution may affect any custom bundler/jest config in the monorepo.

#### 7. `multiformats` 9 → 13
- **Change**: `multiformats` `9.9.0` → `^13.4.2` (major jump); resolution pin removed. New Jest `moduleNameMapper` entries point `multiformats`, `multiformats/cid`, `multiformats/bases/base32`, `multiformats/hashes/*`, and `uint8arrays/*` at explicit `dist` paths (pnpm + ESM compatibility).

---

### New Features

#### Group Chats ("group clip clops" / "clops")
The headline feature of 1.120–1.122 — a large, multi-surface DM capability built across a long sequence of commits and two feature-branch merges (`d3f509381` #10181, `cdb8d4bfb` #10360).
- **Creation & membership**: multi-step create flow, add/remove people (`6e3c9c3a9` #10255, `f1764f1d6` #10386), follow from chat settings (`a24d30ab4`).
- **Settings & admin**: edit group name (`adca192f3`), invite links + management (`482fb77e3` #10407, `e34080c4a`, `c81174e4f`), gate invites on `allowGroupInvites` (`09253457f` #10546), ownership-safety fixes (`a7ed3ee3c` #10373).
- **Messaging surfaces**: system messages with grouping/toggle (`2ab1e2c9e` #10312, `75d8fb3db` #10388), chat requests UI (`a889a1b08`, `c8e481583`), locked/ended chat states (`c91eae372`, `7b6a81ac2`), sender name in last message (`dcc06a90a`).
- **Web**: **split-view (two-column) layout for DMs on web** (`0eb382da0` #10258, `366cf7e04`, `e40c9bcf9`, `b1ce51936`).
- **Reactions**: message reactions dialog, viewer-reaction differentiation, a11y/localization (`ac68cfe98`, `c21138ba6`, `26ca0a671`, `632ce5cc3`).
- **Refactors**: chat list reimplemented (`888eca73b` #10059), UI driven by convo *view* instead of convo *state* (`935347c73`, `bc3672cee`), message-list memoization preserved (`5640aed7e`).
- **Note**: gated behind a feature flag, toggled on/off several times during development (`6d53459e9`, `7eec65ac7`, `dc9b9a887`, `e83279136`).

#### New Rich-Text Composer + Autocomplete
- **Commit**: `0a5ae1773` "New rich text composer + autocomplete" (#10159).
- Reworks the composer's text input and mention/autocomplete handling. Related: stop threading `_` through composer functions (`ecc78efb1`).

#### `EmojiPicker` Component
- **Commit**: `a97b15b20` "✨ `EmojiPicker` component" (#10249).
- A new shared emoji picker component (builds on the earlier emoji-mart fork).

#### GIF Backend: Tenor → KLIPY
- **Commits**: `a77b6e352` "Migrate from Tenor to KLIPY" (#10240), `48ea9e0b9` "Rebuild GIF Dialog" (#10261), `445aef8fe` update KLIPY locale, `b41dbec48` "Remove KLIPY from external media settings" (#10496).
- GIF search/provider moved from Tenor to KLIPY, with a rebuilt GIF dialog. GIF rendering hardened and routed through `ConstrainedImage` (`a03cbd8b1` #10453, `8b9361016` #10443).

#### Image Grid → Carousel
- **Commit**: `e80454680` "replace image grid layout with carousel" (#10157).
- In-feed/embed multi-image galleries now use a swipeable carousel instead of a static grid. Embed gallery polish followed (`9778a3da1`, `92fdb6630`).

#### On-Device Language Detection
- **Commit**: `1c38665d4` "Replace lande with native language detection" (#9974).
- Replaces the JS `lande` library with native detection via the new `@bsky.app/expo-guess-language` (`^0.2.8`). Also: faster content-language toggling (`abe0ca521`), dismissible suggested-language prompt fix (`7c5a9ab40` #10581).

#### Global Keyboard Shortcuts
- **Commits**: `cca3326b2` "Create global keyboard shortcut handler" (#10145) + nits/crash fixes (`165dd5a77`, `3685439ff`, `3ff40e960`, `dddc02274`).
- New dependency `react-hotkeys-hook` `5.2.4`. Adds app-wide keyboard shortcuts; `/` shortcut works across keyboard layouts.

#### User-Submitted Error Reports
- **Commit**: `8e06c9fbb` "Add ability for users to send error reports" (#10427).

#### Bsky Video
- **Commits**: `444c5787c` "add bsky video" (#10281), `5ad3597fd` "Add simple video player to embed" (#10205).
- New `@bsky.app/video` (`0.3.4`) replaces `@haileyok/bluesky-video`. Embeds get a simple video player.

#### Other Features
- **Labels on actor statuses** (`8c9a11dc3` #9716).
- **Improved birthdate handling** (`b41d0ad97` #10477).
- **Export-data dialog added to chat settings** (`9e55bb71e` #10478).
- **Keep screen awake for YouTube video** (`34b3c5272` #10479).
- **Double the framerate** (`02b849996` #8446).
- **bskyembed**: simple video player, font smoothing, iOS 26 Safari dark-mode fixes (`5ad3597fd`, `a0d4af5c5`, `19e2dc939`, `13b453e50`).

---

### Refactors & Architectural Changes

- **Shell state**: minimal shell mode converted to refcounting (`3153ea430` #10319); `headerMode` moved out of global shell state (`0df1d6f53`→`0dc062294` #10527).
- **`pal` cleanup**: deleted "non-standard" styles from the legacy `pal` palette (`014ffac90` #10279) — continues the ALF migration.
- **Type-safety**: addressed `any` types in `List` component (`5b7bc56a1` #10504) ahead of enabling `no-explicit-any`.
- **Removed `@expo/config-plugins` direct dependency** (`19df58f1e` #10152) and an unneeded config plugin (`d2519a4f6`).
- **Codemods**: per-file codemod for migrating `useLingui` to v5 (`5da73c2d6` #10505).
- **Module READMEs** added across native modules (`52b8201d2` #10306).
- **CLAUDE.md** updated with pnpm commands, React 19.1 / TS 6 / tsgo, comment-style guidance, and more (`b97cd413e` #10385).

---

### Dependency Updates

#### New Dependencies
| Package | Version | Purpose |
|---------|---------|---------|
| `@atproto/syntax` | `0.6.1` | atproto syntax helpers |
| `@bsky.app/video` | `0.3.4` | Video playback (replaces `@haileyok/bluesky-video`) |
| `@bsky.app/expo-guess-language` | `^0.2.8` | Native language detection (replaces `lande`) |
| `@bsky.app/expo-scroll-edge-effect` | `^0.1.4` | Scroll edge effect (iOS 26) |
| `@bsky.app/sift` | `^0.3.4` | bsky internal lib |
| `@bsky.app/tapper` | `^0.5.3` | bsky internal lib |
| `expo-glass-effect` | `55.0.8` | iOS 26 liquid-glass effects |
| `react-hotkeys-hook` | `5.2.4` | Keyboard shortcuts |
| `fuse.js` | `^7.1.0` | Fuzzy search |
| `slugify` | `^1.6.9` | Slug generation |
| `@typescript/native-preview` | `^7.0.0-dev…` | `tsgo` typechecker (dev) |
| `eas-cli` / `eas-cli-local-build-plugin` | `^18.13.0` / `^18.12.3` | EAS build (dev) |

#### Removed Dependencies
| Package | Reason |
|---------|--------|
| `@haileyok/bluesky-video` | Replaced by `@bsky.app/video` |
| `patch-package` | Replaced by pnpm `patchedDependencies` |
| `postinstall-postinstall` | No longer needed (no patch-package) |
| `lockfile-lint` | Yarn-lock specific |
| `@expo/config-plugins` (devDep) | No longer needed directly |

#### Updated Dependencies (Major/Notable)
| Package | From | To |
|---------|------|-----|
| `@atproto/api` | `^0.19.5` | `0.20.4` (ESM) |
| `multiformats` | `9.9.0` | `^13.4.2` |
| `@ipld/dag-cbor` | `^9.2.0` | `^9.2.7` |
| `zod` | `^3.20.2` | `^3.25.76` |
| `@tanstack/react-query` (+persisters) | `^5.95.2` | `^5.96.2` |
| `@bsky.app/alf` | `^0.1.7` | `^0.1.9` |
| `react-compiler-runtime` | `^19.1.0-rc.1` | `19.1.0-rc.3` |
| `react-native-keyboard-controller` | `^1.21.0` | `^1.21.7` |
| `react-native-webview` | `^13.13.5` | `^13.15.0` |
| `react-native-drawer-layout` | `^4.2.2` | `^4.2.3` |
| `husky` | `^8.0.3` | `^9.1.7` |
| `prettier` | `^3.6.0` | `^3.8.3` |
| `eslint-plugin-simple-import-sort` | `^12.1.1` | `^13.0.0` |
| `eslint-plugin-react-hooks` | `^7.0.1` | `^7.1.1` |
| `@types/node` | `^20.14.3` | `^24.12.2` |
| `@types/react` | `^19.1.12` | `^19.1.17` |

#### Updated Expo Sub-packages
| Package | From | To |
|---------|------|-----|
| `expo` | `^54.0.33` | `54.0.34` |
| `expo-notifications` | `~0.32.16` | `~0.32.17` |
| `expo-updates` | `~29.0.16` | `~29.0.17` |
| `expo-paste-input` | `^0.1.10` | `^0.2.1` |

#### Dependency Pinning
Several previously caret-ranged deps are now pinned exactly (and/or moved into `pnpm-workspace.yaml › overrides`): `react-native-compressor` `1.13.0`, `react-native-reanimated` `3.19.1`, `react-native-screens` `4.24.0`, `psl` `1.9.0`, `@types/psl` `1.1.1`, `unicode-segmenter` `0.14.5`, `@react-native/babel-preset` & `@react-native/normalize-colors` `0.81.5`, `@expo/image-utils` `0.8.12`, `@types/estree` `1.0.6`.

#### Jest Config Changes (pnpm + ESM compatibility)
- New `moduleNameMapper` entries for `multiformats*`, `uint8arrays/*`, `unicode-segmenter/grapheme`, `await-lock`.
- `transformIgnorePatterns` extended with `@atproto/*`, `multiformats`, `uint8arrays`, `@ipld/*`, `cborg`, `await-lock`.

---

### Bug Fixes (selected)
- **OOM crash** — disabled tanstack extension integration (`be56066ee` #10169).
- **Hotkeys crash on native startup** (`165dd5a77` #10180).
- **ReportDialog Android hang** — narrowed dependency arrays (`c0d3010f3` #10171).
- **Feeds not refreshing** (`011d8d2f7` #10167).
- **GrowthBook cache breaking on cold start** (`226a321a2` #10282).
- **Crash deleting account from NoAccessScreen** (`bc9ad2c2d` #10283).
- **Reply failures from fetching posts from PDS** (`9d98890fd` #9925).
- **Layout shift when liking a post on Android** (`a169bd862` #10190).
- **Stale `searchText` in submit handler / autocomplete lag** (`59a2d19c2` #10251, `8bfce0962` #10381).
- **Stale display name for renamed pinned lists** (`a7439eb45` #10466).
- **Image Options not opening on web** (`0858010a5` #10544); **stuck tooltip on Safari** (`7da3f385a`); **word-wrap in image menu on Android** (`f653faab9`).
- **Unnecessary scrollbars on web** (`cc7d8296f` #10502).
- **Picking offloaded videos on iOS 26** (`0b6ae1cc2` #10428).

---

### UI/UX Improvements
- **Refreshed lightbox designs** + animated border-radius transition (`a7fabd9e0` #10330, `e58eeaeb0`, `3358e1947` lazy thumbnail measurement).
- **Settings menus / context-menu presentation** refreshed across the app (`eddbf6ba9`, `9a2cabf64`, `a8debddd0`).
- **Moderation**: renamed settings entry to "Moderation and content filters" (`2c717dc1e` #10192); bumped mod SDK + richer mod-info dialog (`cc5b21586` #10420).
- **Reactions/messages** spacing, rounding, drop-shadow, emoji-only message corners (`587dc8dfe`, `923d06229`, `945d8b229`, `34ae819aa`).
- **Age assurance**: region notice on card (`9b5be84c5`), gate chat settings behind age assurance (`d58ff8944`).
- **Don't show Follow link if user is blocked** (`0b63ee40c` #10491).
- **Use `i18n.date` for date dividers** (`a07e364a8`); translator context + string tweaks (`2798e98c9`).

---

### Performance
- **Removed 90ms tap latency on Android** (`2c5d7f20b` #10392).
- **Disabled keyboard preload on iOS** to prevent keyboard flash (`ff68e4036` #10214).
- **Slow re-renders when toggling content languages** fixed (`abe0ca521` #10128).
- **Hotkeys perf regression** fixed (`dddc02274` #10302).
- **Message-list memoization** preserved (`5640aed7e` #10434).

---

### Build System & CI
- **pnpm CI migration**: a long series of `[pnpm] Fix CI` commits (#10531, #10533, #10529, #10538, #10565, #10567, #10503), lockfile-integrity fixes, and `verify-yarn-lock.yml` → `verify-pnpm-lock.yml`.
- **Node 24 in GitHub Actions** (`b4054bf2a` #10539); EAS invocation fixes for Actions (`70a1a665e`).
- **`bskyembed` migrated to pnpm** (`8cea36f3c` #10482); `Dockerfile.embedr` updated for pnpm (`2f80d983d`).
- **Prometheus metrics added to bskyweb** (`28428a575` #10370); **bskyweb Go 1.26** (`a7b6cd450`).
- **bskyogcard**: avatar-bubbles endpoint (`f527b3524`), CI image builds on push to main (`f0fec7a9d`, `ddcf7d74e`).
- **Prettier config** consolidated + reapplied repo-wide (`5088f00c3` #10542, `46a4f6cbe`).
- **Husky 9** migration (`2cee865f4` #10549).
- **Claude Code workflow** updated to 4.7 (`18052f08e`); push-notification script + docs (`9bf06a80a`).

---

### Version Progression
| Version | Commit |
|---------|--------|
| 1.120.0 | `3d0fbc503` / `e4b4e48ac` (prep) |
| 1.121.0 | `d01362a01` (bump), `c7e9efbf9` (prep) |
| 1.122.0 | `18143d051` (bump) |

---

### Migration Guide for Forks

This update changes how the project is installed, patched, and typechecked. For our subtree-in-a-monorepo, these are the load-bearing items:

1. **Adopt pnpm.** Install with `pnpm install` (Node ≥ 24.15, pnpm 11.1.3 via `corepack`/`packageManager`). Replace any `yarn …` invocations in monorepo glue/CI with `pnpm …`.

2. **Migrate patches to pnpm `patchedDependencies`.**
   - There is no more `patch-package`/`postinstall` step. Patches must be listed in `pnpm-workspace.yaml › patchedDependencies` and named `name@version.patch`.
   - Any fork-specific patches we carry must be re-expressed in this format and registered there.
   - Note the RN patches are now a single `react-native@0.81.5.patch` (not the old 5 split files).

3. **Move `resolutions` → `pnpm-workspace.yaml › overrides`.** The Yarn `resolutions` block is gone. Any fork overrides go under `overrides:`. Build-script allowances go under `allowBuilds:`.

4. **Typecheck with `tsgo`.** `pnpm typecheck` now runs `tsgo` (`@typescript/native-preview`). Ensure CI provides it; review any tooling that shells out to `tsc` directly.

5. **Node 24.** Update build images / CI runners to Node 24.15+. `engines.runtime` lets pnpm download Node automatically (`onFail: download`).

6. **Lint.** `no-explicit-any` is now an error and default `react` imports are disallowed (use named imports). Run `pnpm lint` and fix.

7. **atproto 0.20 + ESM / multiformats 13.** Verify the monorepo's bundler and Jest config handle ESM atproto packages and `multiformats@13`; mirror the new Jest `moduleNameMapper`/`transformIgnorePatterns` if we run tests outside this package.

8. **Known local-install note (this machine):** `pnpm install` aborts on `dtrace-provider`'s `node-gyp rebuild` (exit 126), a transitive native module gated by `allowBuilds`. This is an environment/native-toolchain issue, not an upstream regression — resolve locally (Xcode CLT / node-gyp) or exclude that build before relying on `pnpm typecheck`/`pnpm test`.

9. **Verify after merge:**
   ```bash
   pnpm install
   pnpm lint
   pnpm typecheck
   pnpm test
   ```

---
*This changelog documents changes merged from the upstream Bluesky Social App repository.*

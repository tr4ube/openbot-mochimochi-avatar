# openbot-mochimochi-avatar

[![verify](https://github.com/tr4ube/openbot-mochimochi-avatar/actions/workflows/verify.yml/badge.svg)](https://github.com/tr4ube/openbot-mochimochi-avatar/actions/workflows/verify.yml)

A patch that adds **layered character-portrait avatars** ("character art") to
[OpenBot](https://github.com/nightly-labs/openbot).

With this patch, an agent whose avatar seed is `character:<id>` shows a layered
character illustration instead of the generated avatar: a body layer, eye
layers that blink and look around, closed "happy" eyes, and a breathing
animation. Characters live as plain JSON files in
`<userData>/characters/<id>.json`, so adding a new character needs no rebuild.
The patch covers the desktop renderer (SolidJS) and the mobile app
(`apps/mobile`, plush avatar and thumbnails), including the joined-server path
over the Team API.

## Apply

Target version: **OpenBot v0.77.0** (`git tag v0.77.0`, commit `1dbc27f`).

Requires `git`, `bun` (the repository pins `bun@1.4.2` via `packageManager`),
and Node >= 24. Clone both repositories side by side so the patch path below
works verbatim:

```sh
git clone https://github.com/tr4ube/openbot-mochimochi-avatar.git
git clone --depth 1 --branch v0.77.0 https://github.com/nightly-labs/openbot.git openbot
cd openbot
git apply --check ../openbot-mochimochi-avatar/patches/character-art-v0.77.0.patch
git apply        ../openbot-mochimochi-avatar/patches/character-art-v0.77.0.patch
bun install --frozen-lockfile   # postinstall fetches an Electron dist; needs network
bun run typecheck:node && bun run typecheck:renderer   # quick first check
bun run typecheck:mobile                               # apps/mobile (runs codegen first)
bun run build    # or the dev workflow described in the repository
```

On any other upstream version the context lines may drift; `git apply --check`
tells you first. See "Version policy" below.

## Verify

```sh
bun run test:desktop -- src/main/ipc-channel-coverage.test.ts \
  src/renderer/src/features/sidebar/MariAvatar.test.tsx
bun run test:desktop -- apps/mobile/src/features/agents/model/mari-motion.test.ts
```

The mobile test runs under the root vitest project; `apps/mobile` has no own
`test` script.

## Version policy

- `patches/character-art-v<version>.patch` — one patch per upstream release.
- `v0.77.0` is the current pinned target; newer OpenBot releases get a ported
  patch file alongside, and the version table below tracks verification.

| Upstream version | Patch file | Verified |
|---|---|---|
| v0.77.0 | `patches/character-art-v0.77.0.patch` | CI on ubuntu/windows/macos ([latest run](https://github.com/tr4ube/openbot-mochimochi-avatar/actions/workflows/verify.yml)): clone → `git apply` → `bun install` → `typecheck:node`/`renderer`/`contracts`/`i18n`/`mobile` + patch tests all PASS. Runtime (launching the app, avatar rendering) is not verified. |

## Adding a character

Drop a JSON file into the app's user-data `characters/` directory —
`~/.config/OpenBot/characters/<id>.json` on Linux,
`~/Library/Application Support/OpenBot/characters/<id>.json` on macOS, or
`%APPDATA%\OpenBot\characters\<id>.json` on Windows. The file id must match
`^[a-z0-9-]{1,100}$`. Set the agent's avatar seed to `character:<id>`. The
folder is re-read when a window gains focus; no restart needed.

Schema (validated by `isCharacterArt` in the patch):

```jsonc
{
  "name": "Display name",
  "source": "data:image/webp;base64,...", // original layered image
  "base":   "data:image/webp;base64,...", // body layer (png/webp/jpeg data URL)
  "eyes": [                               // blink/look layers (may be empty)
    { "name": "left", "cx": 0, "cy": 0, "angle": 0,
      "bounds": [x1, y1, x2, y2], "darkBounds": [x1, y1, x2, y2],
      "pixelCount": 0, "image": "data:image/png;base64,..." }
  ],
  "happyShapes": [[[x, y], ...]]          // happy-eye outlines, one polyline set per eye
}
```

A working sample is in `characters/`: `mari-v1.json` (`character:mari-v1`).
Copy it into the `characters/` directory above to try the feature.

## License

OpenBot is distributed under the
[PolyForm Noncommercial License 1.0.0](https://polyformproject.org/licenses/noncommercial/1.0.0).
This patch adds files and hunks to that codebase, and its context lines embed
small excerpts of upstream source, so this repository ships under the same
PolyForm Noncommercial 1.0.0 terms — see `LICENSE`. Noncommercial use only.

# ayugram-release

Unofficial Linux x86_64 builds of [AyuGram Desktop](https://github.com/AyuGram/AyuGramDesktop)
for [FlatPark](https://flatpark.org).

Upstream publishes Windows and macOS binaries but no Linux one. This repository
builds the Linux binary on GitHub Actions and attaches it to a GitHub release.
FlatPark's `registry/com.ayugram.desktop/` pulls it in as **extra-data**, so the
binary is downloaded straight from these releases and installed and updated
through Flatpak.

## How it is built

The recipe is upstream's own documented Linux build
([`docs/building-linux.md`](https://github.com/AyuGram/AyuGramDesktop/blob/dev/docs/building-linux.md)),
run unattended in [`.github/workflows/build.yml`](.github/workflows/build.yml):

1. **check**: every six hours, resolve upstream's latest release tag. If this
   repository already has a release for it, stop.
2. **image**: generate the Rocky Linux 8 build environment from upstream's
   `Telegram/build/docker/centos_env` at that tag (`DEBUG= LTO=`, as for the
   public tdesktop image) and push it to GHCR, tagged with the git tree hash of
   that directory. The next release reuses the image unless upstream has changed
   the environment, because a cold build takes hours.
3. **build**: unpack upstream's `AyuGramDesktop-<ver>-full.tar.gz` source release (what distros build; it can carry submodule fixes made after tagging), run upstream's `centos_env/build.sh` in that image with upstream's
   documented API credentials and `DESKTOP_APP_DISABLE_AUTOUPDATE=ON` (Flatpak
   handles updates), then `cmake --install` and strip.

Nothing is patched. The binary is the stock upstream source built with upstream's
toolchain.

## Artifacts

| File | Contents |
|---|---|
| `ayugram-<ver>-x86_64.tar.zst` | the `cmake --install` tree under `usr/`: `bin/AyuGram` (self-contained, statically linked Qt/FFmpeg/…), desktop file, icons, metainfo, D-Bus service |
| `ayugram-<ver>-x86_64.tar.zst.sha256` | checksum |

Each release's notes record the upstream commit, the build-environment image
digest and the workflow run.

## Rebuilds

A release is never overwritten, because FlatPark pins each artifact's sha256. To
rebuild a version, run the workflow manually with `revision` set (e.g. `2`). That
publishes `v<ver>-2` next to the original release.

## Licence

AyuGram Desktop is GPL-3.0 with the OpenSSL exception (see upstream `LICENSE`).
The corresponding source for every binary is upstream's tagged source tree, which
each release links, plus this build recipe.

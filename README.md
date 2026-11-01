<p align="center">
  <img src="docs/media/banner.webp" alt="Horizon: Minecraft rebuilt in Rust" width="100%">
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Minecraft-Java%2026.3-5D9C3F" alt="Minecraft Java 26.3">
  <img src="https://img.shields.io/badge/launcher-offline--first-6A5ACD" alt="Offline-first launcher">
  <img src="https://img.shields.io/badge/TLauncher-compatible-3B82F6" alt="TLauncher compatible">
  <img src="https://img.shields.io/badge/Rust-stable-B7410E?logo=rust&logoColor=white" alt="Rust">
  <img src="https://img.shields.io/badge/platform-Linux%20%7C%20macOS%20%7C%20Windows-2A2470" alt="Linux, macOS and Windows">
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-3B2F7E" alt="MIT license"></a>
</p>

# Minecraft Free Rust Horizon

A customized build of [Horizon](https://github.com/horizon-rust/horizon), the open-source Rust rewrite of Minecraft: Java Edition. This fork is configured for an **offline player profile**, can prepare game resources from a local **TLauncher or Minecraft Launcher** installation, and includes a fix that preserves the single-player owner's inventory and position when saving a world.

> [!IMPORTANT]
> This is an unofficial community fork. It is not an official Minecraft product and is not approved by or associated with Mojang, Microsoft, TLauncher, or the upstream Horizon project.
>
> The repository contains no Minecraft code, assets, credentials, or prepared game data. Use your own legally obtained Minecraft files and follow the Minecraft EULA and the rules that apply in your country.

## How this fork differs from upstream

### Offline-first launcher

- Play uses a local offline player name and does not require Microsoft sign-in.
- Microsoft sign-in and ownership prompts are removed from the customized launcher interface.
- New Horizon servers accept offline player profiles.
- Existing Horizon servers are switched to offline mode when opened by this launcher.
- Online-mode servers that require Microsoft authentication will not accept an offline profile.

### TLauncher-compatible preparation

- The launcher checks the standard Minecraft/TLauncher game directory. On Linux, this is normally `~/.minecraft`.
- A different directory can be selected under **Settings → Minecraft folder → Choose…**.
- TLauncher profiles appear in `horizon-launcher instance others` and can be copied into a Horizon Java instance with `horizon-launcher instance copy-from`.
- Install and launch the matching Minecraft version in TLauncher first so its local files are available.
- Horizon imports local files; it does not launch TLauncher or connect to TLauncher's account service.

### Single-player save fix

This fork fixes a regression where the integrated single-player server saved the world but omitted the owner's player record. The missing record caused the player to return with an old inventory and position after leaving and reopening a world.

The corrected save path now writes the owner's:

- inventory and equipped items;
- position and dimension;
- health, status, and other player-specific state.

A true dedicated server still saves connected players through its dedicated-player path. The fix is committed as [`1abdd0f52`](https://github.com/pasovivan121-lgtm/minecraft-free-rust-horizon/commit/1abdd0f52).

## Update behavior

There are two separate kinds of updates:

| Update | Behavior in this fork |
| --- | --- |
| **Horizon game versions** | Checked and installed automatically by default. The launcher can install a future Horizon release that supports a newer Minecraft version. |
| **The launcher executable** | Automatic self-replacement is disabled so an upstream launcher update cannot overwrite the offline/TLauncher customization. Rebuild or replace this customized launcher manually to update it. |

A Mojang release such as Minecraft 26.4 does **not** make the current Horizon client support 26.4 automatically. The upstream Horizon project must first publish a compatible Horizon build.

Instances can also be pinned to a specific Horizon build. A new game version may be downloaded while a pinned instance continues using its selected build. Select the newer build for that instance when you are ready to migrate. Back up important worlds before opening them with a newer game version.

Automatic game-version installation can be changed under **Settings → Install new versions automatically**.

## Using the customized launcher

1. Install and launch the matching Minecraft version once in TLauncher or another supported launcher.
2. Start the customized Horizon launcher.
3. Open **Settings** and confirm the Minecraft folder. Choose the correct folder manually if it was not detected.
4. Add or select an offline player name.
5. Create or select a Horizon instance, prepare its resources, and press **Play**.
6. To verify the save fix, enter a world, move and change an inventory slot, choose **Save and Quit to Title**, then reopen the world.

The launcher keeps worlds and settings in its instance data folders. Do not commit or share those folders: they may contain personal worlds, local profiles, logs, and Minecraft-owned assets.

## Repository layout

This repository contains the full Horizon source history plus this fork's changes:

- [`crates/horizon-client/src/game.rs`](crates/horizon-client/src/game.rs) — integrated-world player save fix.
- [`offline-tlauncher.patch`](offline-tlauncher.patch) — patch used to produce the offline/TLauncher launcher customization.
- [`offline-tlauncher-build.md`](offline-tlauncher-build.md) — build provenance and validation notes for the customized launcher.
- [`launcher/`](launcher/) — upstream Horizon launcher workspace.
- [`crates/`](crates/) — Horizon game and supporting Rust crates.
- [`docs/`](docs/) — upstream technical and user documentation.

The offline launcher modifications are retained as a patch so the customization remains auditable alongside the complete upstream history. The patch was produced for the source revision named in the build notes and may need conflict resolution when ported to a newer upstream revision.

## Build from source

Horizon requires Rust, Python, a JDK, platform development libraries, and prepared resources from your own matching Minecraft installation. Start with the upstream developer documentation in [`AGENTS.md`](AGENTS.md), [`CONTRIBUTING.md`](CONTRIBUTING.md), and [`docs/native-client.md`](docs/native-client.md).

For the game client at the revision currently stored here:

```sh
git clone git@github.com:pasovivan121-lgtm/minecraft-free-rust-horizon.git
cd minecraft-free-rust-horizon

# Prepare resources from your own Minecraft installation.
cargo run --release -p horizon-prepare -- --prepare --server-jar /path/to/server.jar
cargo run --release -p horizon-prepare -- sounds

# Build the game containing the player-save fix.
cargo build --profile fast -p horizon-client
```

On Linux, the build normally needs `libasound2-dev` and `libudev-dev`. Prepared files are written under the gitignored `.local/` directory and must never be committed or redistributed.

The offline/TLauncher launcher was originally built with Rust 1.99.1 and `HORIZON_LAUNCHER_VERSION=0.2.30`. See [`offline-tlauncher-build.md`](offline-tlauncher-build.md) before rebuilding it. Because [`offline-tlauncher.patch`](offline-tlauncher.patch) targets its recorded source revision, review the patch rather than applying it blindly to a newer Horizon checkout.

## Validation recorded for this fork

### Player-save correction

- Focused `cargo check -p horizon-client` passed when the corrected client was built.
- An optimized `fast` client build passed.
- The resulting Linux executable was checksum-verified and installed into the active Horizon instance.
- The source public-tree check and Git object integrity check passed before upload.

### Offline/TLauncher launcher

- The release launcher build completed successfully.
- CLI version and help output were checked.
- Account commands exposed offline player names only.
- Other-launcher help included TLauncher.
- Full tests were not run for that historical launcher build.
- `cargo fmt --check` could not run because that build toolchain did not include `rustfmt`.

## Known limitations

- This repository does not distribute Minecraft assets, prepared resources, or account credentials.
- Offline profiles cannot join servers that enforce Microsoft online authentication.
- Updating to a new Horizon game release can replace game behavior with the behavior in that release; verify that important local fixes are included before switching a valuable world.
- The offline/TLauncher patch may require manual updates when rebasing onto newer upstream launcher code.
- Player state that was already lost before installing the save fix cannot be recovered if it was never written to disk.

## Upstream project

Horizon is primarily developed by [Ludovic Marie](https://lu4ovic.com) and the [Horizon contributors](https://github.com/horizon-rust/horizon/graphs/contributors). For upstream documentation, release notes, architecture, and contribution guidance, visit:

- [horizon-rust/horizon](https://github.com/horizon-rust/horizon)
- [playhorizon.dev](https://playhorizon.dev)
- [`docs/README.md`](docs/README.md)
- [`ROADMAP.md`](ROADMAP.md)

Please report issues specific to this fork in this repository rather than presenting them as upstream Horizon issues until they are reproduced on an unmodified upstream build.

## License

The project's original work is available under the [MIT License](LICENSE). Dependencies retain their own licenses; see [`THIRD_PARTY_NOTICES.md`](THIRD_PARTY_NOTICES.md). Minecraft is a trademark of Mojang Synergies AB. Minecraft code and assets belong to Mojang and Microsoft and are governed by the [Minecraft EULA](https://www.minecraft.net/en-us/eula).

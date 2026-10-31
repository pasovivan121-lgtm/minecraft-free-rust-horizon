# Offline TLauncher build

This launcher was built from Horizon source commit `530a125` (2026-10-31), with the changes in [offline-tlauncher.patch](offline-tlauncher.patch).

## What changed

- Play uses an offline player name and never prompts for Microsoft sign-in.
- Removed obsolete Microsoft sign-in and ownership prompts from the bundled language tables.
- TLauncher’s standard Minecraft folder is discovered with the Minecraft Launcher folder. On Linux this is `~/.minecraft`; use Settings → Minecraft folder → Choose… for a custom TLauncher game directory.
- TLauncher profiles are included in `horizon-launcher instance others` and can be copied into a Horizon Java instance with `horizon-launcher instance copy-from`.
- Preparation imports local Minecraft files. Install and launch the matching Minecraft version in TLauncher first; Horizon does not launch TLauncher or use its account service.
- New servers accept offline player names. Existing servers are switched to offline mode when opened in Horizon.
- Automatic launcher replacement is disabled so an upstream update cannot restore Microsoft sign-in.

The original executable is preserved as `horizon-launcher.original-0.2.30`.

## Build and checks

Built the release launcher with Rust 1.99.1 and `HORIZON_LAUNCHER_VERSION=0.2.30`. The release build succeeded. The CLI version and help were checked; account commands list offline names only, and the other-launcher help includes TLauncher. Tests were not run. `cargo fmt --check` could not run because the installed toolchain lacks `rustfmt`.

The source build reports unused-code warnings for legacy sign-in/download helpers, which are no longer reachable from the launcher UI or CLI.

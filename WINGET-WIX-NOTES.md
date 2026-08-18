# winget WiX/.msi installer: notes (fork-only)

Goal: a real per-machine `.msi` beside the portable zip, as Starship and
PowerShell ship, for parity across the four shell-integrated tools (Atuin,
fzf, Yazi, zoxide).

## The problem

winget's portable install puts a symlink in `%LOCALAPPDATA%\Microsoft\WinGet\Links`,
created by whoever runs winget, normally a non-elevated terminal. Windows will
not let an elevated process follow a link that a less-privileged process
created (error 448, "The path cannot be traversed because it contains an
untrusted mount point"). An administrator's SSH session is elevated, so
`yazi`, `ya` and the shell wrapper fail there; a standard user is unaffected.

## The change (commit "Add a WiX .msi Windows installer alongside the portable zip")

- `yazi-build/src/dest.rs`: `cargo xtask dist` also builds
  `yazi-<target>.msi` for x86_64/aarch64 Windows, as it builds the .deb on
  Linux: `cargo wix -p yazi-packing` with the release-windows profile. It runs
  before the archive is staged: cargo-wix reads the workspace, whose
  `yazi-*` members glob would match the staging directory.
- `yazi-packing/wix/main.wxs`: WiX v3 template; both `yazi.exe` and `ya.exe`
  to `Program Files\yazi\bin`, machine PATH entry, LICENSE.
- `draft.yml` uploads the .msi with the .zip and includes it in the draft and
  nightly releases; `publish.yml`'s winget installers-regex matches it.
- CHANGELOG entry (its PR number is a placeholder).

## Found while verifying

- The first WIP ran `cargo wix` as a workflow step after `cargo xtask dist`,
  marked continue-on-error: it failed every time (the staging directory clash
  above, then `--` inside an XML comment, which WiX rejects), silently.

## Verified

- Fork smoke run (branch `winget-wix-smoke`):
  https://github.com/meop/yazi/actions/runs/37309773805. `cargo xtask dist`
  builds both MSIs on windows-latest; on windows-latest and windows-11-arm
  each installs `yazi.exe` and `ya.exe`, adds the machine PATH entry, runs
  them locally and over SSH, uninstalls cleanly.
- Real machine (glass, Windows 11 26H2 x64), zoxide fork's
  `windows-msi-test/repro-ssh.ps1`: winget portable install from the desktop
  session fails from an elevated SSH session (error 448); the MSI works.
  2026-10-05.

## Before upstreaming: Yazi AI policy

https://github.com/yazi-rs/.github/blob/main/AI_POLICY.md requires: disclosure
of the model and extent of AI use; issue, PR, discussion **and commit**
descriptions written by a human; a design discussion in an issue with
maintainer approval before any PR; human review and simplification of the
code; and a statement that the AI product asserts no copyright over the work.
So: open a feature request in your own words first, and rewrite the commit
message yourself before any PR.

Open: `Manufacturer='Yazi Contributors'` is a guess (workspace authors list
sxyazi); confirm with the maintainer.

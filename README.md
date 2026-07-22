# dtr

`dtr` is a small Bash wrapper around `pacman`, `yay`, and the Arch User Repository. It provides readable commands for common package-management tasks without hiding the underlying tools.

It does not perform partial upgrades. Package installation uses `pacman -S --needed`; full repository synchronization is performed only by the upgrade commands.

## Installation

```bash
sudo install -Dm755 dtr /usr/local/bin/dtr
```

Run `dtr` as a regular user. Commands that modify the system invoke `sudo` for `pacman`; `yay` and `makepkg` remain under the calling user.

Requirements:

- Arch Linux or an Arch-based distribution
- Bash, `pacman`, and `sudo`
- `yay` is optional but recommended for AUR packages
- `python`, `git`, and `base-devel` are used by the direct AUR fallback
- `pacman-contrib` enables more precise cache cleanup through `paccache`

## Usage

```text
dtr install PKG...          Install packages
dtr upgrade                 Upgrade repository and AUR packages
dtr repo-upgrade            Upgrade official repository packages only
dtr remove PKG...           Remove packages and unneeded dependencies

dtr search TEXT...          Search repositories and the AUR
dtr info PKG...             Show package information
dtr provides FILE|COMMAND   Find the repository package providing a file
dtr owns PATH...            Find the installed package owning a file

dtr installed [TEXT...]     List or search installed packages
dtr foreign                 List packages absent from sync repositories
dtr orphans                 List orphaned dependencies

dtr clear --cache           Clean the package cache
dtr clear --orphaned        Remove orphaned dependencies
dtr clear --all             Perform both cleanup operations
```

Examples:

```bash
dtr install firefox visual-studio-code-bin
dtr search wireguard gui
dtr provides dig
dtr provides /usr/bin/convert
dtr owns /usr/bin/ssh
dtr clear --cache
dtr clear --orphaned
```

Use `dtr help` for the complete command list. Raw access remains available through `dtr raw PACMAN_ARGS...` and `dtr aur YAY_ARGS...`.

## Package resolution

`dtr install` resolves every requested package independently:

1. Official repositories: install with `sudo pacman -S --needed`.
2. AUR through `yay`: install with `yay -S --needed --aur`.
3. Direct AUR fallback: query the AUR RPC, clone the package base, show the `PKGBUILD` for review, then run `makepkg -si`.
4. Return an error if the package cannot be found.

The direct AUR fallback never builds as root. In interactive mode it requires confirmation after reviewing the `PKGBUILD`.

## Upgrade and cleanup behavior

`dtr upgrade` uses `yay -Syu` when `yay` is available. Otherwise it runs `sudo pacman -Syu` and reports that AUR packages were not upgraded.

`dtr remove` uses `pacman -Rns`.

`dtr clear --orphaned` obtains orphaned dependencies with `pacman -Qdtq` and removes them with `pacman -Rns`.

`dtr clear --cache` uses `paccache` when available, keeping two cached versions of installed packages and removing cached versions of uninstalled packages. Without `paccache`, it falls back to `pacman -Sc`.

## File lookup

`dtr provides` uses the pacman file databases (`pacman -F`). Missing or older file databases are refreshed automatically; use `dtr provides --refresh TARGET` to force an update.

This lookup covers packages from configured repositories. AUR package file lists are not indexed by pacman.

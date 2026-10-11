# Caelestia dotfiles — personal customizations (czl fork)

Fork of [`caelestia-dots/caelestia`](https://github.com/caelestia-dots/caelestia)
maintained at `github.com/melynx/caelestia`.

- **`main`** mirrors upstream `main` (kept clean, fast-forward only).
- **`czl`** holds the personal customizations below, with upstream merged in.

Deployed on **Gentoo** via the caelestia CLI (see *Deployment* below). Written
2026-07-07.

---

## Customizations on the `czl` branch

Each change is a deliberate, portable preference. Machine-specific config
stays in the host's `~/.config/caelestia/` overlay; see *Not in this repo*.

### 1. KeePassXC as the Secret Service — `hypr/hyprland/execs.lua`
- Dropped the `gnome-keyring-daemon --start --components=secrets` autostart.
- Autostart `keepassxc`, which provides `org.freedesktop.secrets`.
- **Why:** KeePassXC is the preferred freedesktop Secret Service provider. On the
  host this is paired with a D-Bus activation shadow (`~/.local/share/dbus-1/
  services/org.freedesktop.secrets.service`) so on-demand activation launches
  KeePassXC, not gnome-keyring — that part is host config, not in the dots.

### 2. fcitx5 CJK input — `hypr/hyprland/execs.lua` + `hypr/hyprland/env.lua`
- `execs.lua`: autostart `fcitx5 -d --replace`.
- `env.lua`: `QT_IM_MODULE`/`SDL_IM_MODULE = fcitx`, `XMODIFIERS = @im=fcitx`.
  `GTK_IM_MODULE` is left unset so GTK apps use Wayland's native text-input.
- **Why:** Chinese Pinyin + Japanese Mozc.

### 3. Gaps + suspend — `hypr/variables.lua`
- `windowGapsOut` 10 → 5, `singleWindowGapsOut` 20 → 10 (tighter gaps).
- `sleepGestureCmd`: `systemctl suspend-then-hibernate` → `systemctl suspend`.
- **Why suspend:** hibernate is disabled by a forced kernel lockdown (integrity),
  so `suspend-then-hibernate` fails outright; plain s2idle suspend/resume is
  verified working.

### 4. Ghostty + Zsh→Fish shell boundary
- Replaced the Foot component and references with Ghostty, including Hyprland
  rules, the terminal keybind target, and Thunar's "Open Terminal Here" action.
- Added the portable Ghostty font, padding, and transparency configuration.
- The account shell remains Zsh. `zsh/zshrc` replaces only interactive sessions
  with Fish and exports `SHELL=/usr/bin/fish`; non-interactive SSH commands keep
  Zsh. Set `ZSH_NO_FISH=1` to open native interactive Zsh.
- **Why:** Fish remains the interactive environment, while Bourne-style remote
  automation and programs that consult the login shell see Zsh.

---

## Deliberately NOT forked

These showed up in the host `~/.config` diff but are **not** real customizations,
so they were excluded to keep the fork clean:

- **`btop/btop.conf`** — differs only because btop 1.4.7 rewrote the config with
  new *default* keys (`disable_presets`, `disable_mouse`, `apple` GPU vendor, …);
  source was 1.4.6. No values were actually changed. Pure version drift.
- **Resizer and Spotify fixes** — formerly carried here, now upstream
  (caelestia-dots/caelestia#459 and #465). Upstream's later rules refactor
  dropped the `^…$` anchors on the Spotify `initial_title` match; that is
  equivalent, because Hyprland rule regexes use `RE2::FullMatch`.

### Not in this repo at all (host-level, per-machine)
- Update-safe overlay in `~/.config/caelestia/`: `hypr-user.lua`, `hypr-vars.lua`,
  `user-config.fish`, `shell.json`. (`hypr-user.lua` also `dofile`s
  `monitors.lua`/`workspaces.lua`.)
- nwg-displays output: `hypr/monitors.{lua,conf}`, `hypr/workspaces.{lua,conf}`
  (per-machine display/workspace layout).
- Generated: `hypr/scheme/current.lua`, fish `conf.d/`, `completions/`,
  `fish_variables`.
- Outside dots entirely: fontconfig CJK (AR PL UKai CN), howdy face-auth,
  KeePassXC D-Bus shadow.

---

## Deployment (Gentoo)

Packaged in the **`nekochigura`** overlay:
- `gui-apps/caelestia-shell` — the Quickshell shell (carries a Qt 6.11 include
  patch; upstream does not build against qtbase-6.11 without it).
- `app-misc/caelestia-cli` — the CLI; carries a patch adding
  `caelestia install --no-packages` (skips the Arch-only AUR-helper step).
- `app-misc/caelestia` — metapackage (shell + cli).

Point the CLI at this fork/branch via `~/.config/caelestia/cli.json` (native
config keys — no CLI patch needed for this):

```json
{
  "dots": {
    "url": "https://github.com/melynx/caelestia.git",
    "branch": "czl"
  }
}
```

- **Deploy configs:** `caelestia install --no-packages`
- **Update configs:** `caelestia update`

Note: the deploy writes into `~/.config` and preserves the `~/.config/caelestia/`
overlay (those files aren't in this repo).

---

## Maintainer workflow — pulling upstream updates

```sh
git fetch upstream
git checkout main && git merge --ff-only upstream/main && git push origin main
git checkout czl  && git merge main            # resolve conflicts in the customized files
git push origin czl
# then on the host:
caelestia update
```

Likely conflicts are limited to the customized Hyprland files, manifest,
Ghostty/Zsh components, Thunar action, and their documentation.

---

## Status / continue-from-here

- [x] Fork `czl` branch created with the 3 customizations, pushed.
- [ ] Create `~/.config/caelestia/cli.json` (above) to point the CLI at the fork.
- [ ] `emerge app-misc/caelestia` from `nekochigura` (currently only sandbox-built,
      committed + pushed to the overlay). Then retire the old `uv` CLI install +
      `/usr/local/bin/caelestia` symlink and the manual `~/.config/quickshell/
      caelestia` cmake build.
- [ ] Optional: reconcile the shell (4 commits behind upstream; rebuild needs the
      Qt 6.11 patch already in the ebuild).

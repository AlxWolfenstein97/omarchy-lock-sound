# Lock Sound

**Stock Omarchy lock, plus one cue:** wrong password plays the current theme’s
`sounds/denied.ogg` through `omarchy-sound`. No theme → no file → silence.
Built for [HEV Suit](https://github.com/AlxWolfenstein97/omarchy-hev-suit-theme);
any theme that ships `denied.ogg` gets it for free.

## Why this exists

Omarchy’s stock lock (`omarchy.lock`) has **no hook** on password failure. The
only supported way to customize it is `omarchy plugin clone omarchy.lock`, which
leaves a private copy under `~/.config/omarchy/plugins/` — fine for one machine,
useless for “install this theme and hear biohazard on a bad PIN.”

This repo **is that clone, published**: same lock UI / PAM / fingerprint flows,
one extra line in `handlePasswordFailure()`, and `clonedFrom: omarchy.lock` so
Omarchy disables the stock lock while this one is enabled (you only run one
locker).

## How it works

1. You fail a password attempt on the lock screen.
2. `Service.qml` → `handlePasswordFailure()` calls
   `Quickshell.execDetached(["omarchy-sound", "denied"])`.
3. `omarchy-sound` (tiny dispatcher you install once — see HEV Suit README)
   plays `~/.local/state/omarchy/current/theme/sounds/denied.ogg` if present.
4. Switch themes: cue follows the theme folder. Switch away from a theme with
   no `denied.ogg`: quiet. Remove this plugin: stock lock comes back, no cue.

Upstream lock updates are **not** merged automatically. This is a deliberate
fork of the lock service; if Omarchy changes lock behaviour a lot, re-check
this repo or re-clone stock and re-add the one-liner.

## Install

Needs the `omarchy-sound` dispatcher (and, for HEV Suit, that theme’s
`sounds/` + hooks). Full sound wiring lives in the
[HEV Suit README](https://github.com/AlxWolfenstein97/omarchy-hev-suit-theme#how-hev-system-sounds-work).

```bash
omarchy plugin add https://github.com/AlxWolfenstein97/omarchy-lock-sound.git --enable
omarchy-restart-shell
```

That enables **Lock Sound** and disables stock `omarchy.lock`. If you already
have a personal clone (e.g. `alex.lock` from `omarchy plugin clone`), remove it
first so only one locker is enabled:

```bash
omarchy plugin remove alex.lock --yes   # or whatever your clone id was
omarchy plugin add https://github.com/AlxWolfenstein97/omarchy-lock-sound.git --enable
omarchy-restart-shell
```

Smoke-test: lock the session, enter a wrong password, hear `denied` (with HEV
Suit active and `omarchy-sound` on your `PATH`).

## Remove

```bash
omarchy plugin remove io.github.alxwolfenstein97.lock-sound --yes
omarchy-restart-shell
```

Removing a `clonedFrom: omarchy.lock` plugin restores the stock lock service.

## Similar plugins

Same workshop as the other Omarchy extenders — thin, one job, theme-aware:

| Plugin | Job |
|--------|-----|
| **[HEV Suit](https://github.com/AlxWolfenstein97/omarchy-hev-suit-theme)** | Theme pack that ships `sounds/denied.ogg` (and login / battery / update) |
| **[OmaHud](https://github.com/AlxWolfenstein97/omahud)** / **[OmaOBS](https://github.com/AlxWolfenstein97/omaobs)** / … | Colour extenders (different surface, same idea) |

Lock Sound is the lock-shaped piece of that puzzle: the theme cannot patch
stock lock by itself.

## Credits / license

Forked from Omarchy’s first-party `omarchy.lock` (DHH / Omarchy). The only
behavioural addition is the `omarchy-sound denied` call. MIT for this packaging;
Omarchy itself remains under its own terms. HEV VO credit lives on the theme
repo, not here — this plugin never ships audio.

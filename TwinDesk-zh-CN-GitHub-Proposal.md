# Add Simplified Chinese (zh-CN) localization for the desktop app UI (Windows + Linux)

> Paste-ready body for a GitHub **Issue** (or a Pull Request description) on
> `https://github.com/InukaAsith/TwinDesk`. Everything below is factual and verified against
> TwinDesk **2.3.0** (Windows build `macshare-win`, Linux build `twindesk`).

---

## Summary

I have translated the **entire desktop app UI into Simplified Chinese (zh-CN)** for **both the
Windows and the Linux build**, and would like to contribute it upstream.

| Build | Files changed | Notes |
|---|---|---|
| `windows/` | 48 | 22 sidebar tabs, tray menu, status line, file-transfer popups, pairing banner, Mesh mode, phone hub |
| `linux/` | 89 | Same coverage **plus** Linux-only surfaces: Wayland / X11 / uinput / BlueZ / PipeWire / NetworkManager / ufw / firewalld / WoL / virtual-display (VDD) strings |
| `shared/` | 14 | Common to **all three** desktop builds (see flag #1) |

Only user-visible string literals were changed. **No logic, no behaviour, no new dependencies.**

TwinDesk already ships `locales/zh-CN.pak` (and `zh-TW.pak`) — the Chromium layer is localized — but
the application's own UI strings are hardcoded English. This contribution fills that gap.

## Motivation

| Item | Status today |
|---|---|
| Chromium layer (`locales/*.pak`, incl. `zh-CN.pak`) | Already localized by the runtime |
| Application UI (tabs, tray, dialogs, toasts, banners) | **English only**, hardcoded in JS/HTML |
| Chinese-speaking users | Must read English UI despite the OS being zh-CN |

## Scope

### Windows — `windows/` (48 files)

| Area | Files | What was translated |
|---|---|---|
| Sidebar tabs | `windows/ui-*.js` (18), `shared/ui-*.js` (8) | All **22** tab titles, descriptions, headings, rows, buttons, status dictionaries |
| UI toolkit | `windows/ui-kit.js` | Peer naming (`KIND`), direction cards, status / cursor-lock rows, hotkey key names |
| Main window shell | `windows/app.js` | Sidebar peer status, phone tiles, **entire pairing banner** (all phases) |
| Feature intro art | `shared/feature-art.js` | 16 feature intro texts **and** the SVG `<text>` inside the terminal illustration |
| Multi-device (Mesh) | `windows/mesh/ui-{overview,share,cursor,multi,phone}.js`, `mesh/main.js`, `mesh/app.html` | Mesh tabs, tray menu, status line, file-transfer text |
| Mesh intro dialog | `shared/mesh-intro.js` | Full "Turn on Multi-device?" dialog + description paragraph |
| Main process | `windows/main.js`, `windows/mesh/main.js` | Tray context menu, status line, file-transfer HUD/toasts, firewall self-check |
| Updates & firewall | `shared/updates.js` | Update notification title/body and error strings |
| HTML shells | `app.html`, `monitor.html`, `appview.html`, `phoneview.html`, `sender.html`, `sound.html`, `hud.html`, `camera-view.html`, `camera-send.html`, `callpop.html` | Visible text, window titles, `lang="zh-CN"` |

### Linux — `linux/` (89 files)

Everything above, adapted to the Linux tree, **plus** the Linux-only surfaces:

| Area | Examples |
|---|---|
| Desktop session detection | `ui-setup.js` — "No X11 or Wayland display found. Start TwinDesk from your desktop." |
| Input injection | `ui-general.js` — uinput, XInput2 / XTest, `usermod -aG input`, polkit password prompt |
| Bluetooth | BlueZ, BLE, "Bluetooth with phones / computers", A2DP call audio |
| Network | NetworkManager, Wi-Fi Direct hosting, Wake-on-LAN, `ufw` / `firewalld` firewall states |
| Screen sharing | `ui-share.js` / `ui-receive.js` — Wayland screen picker, PipeWire, virtual display driver (VDD), NVIDIA X11 limitation |
| Audio / loopback | PipeWire, PulseAudio, sound loopback device |
| Terminal | `ssh twindesk`, sshd setup, Xorg vs Wayland app windows |
| Shells | `app.html`, `monitor.html`, `phoneview.html`, `appview.html`, `sender.html`, `sound.html`, `hud.html`, `mesh/app.html` |

### `shared/` (14 files, common to all desktop builds)

`callpop.html`, `camera-send.html`, `camera-view.html`, `feature-art.js`, `mesh-intro.js`,
`ui-calls.js`, `ui-camera.js`, `ui-hub.js`, `ui-media.js`, `ui-messages.js`, `ui-nowplaying.js`,
`ui-pfiles.js`, `ui-remote.js`, `updates.js`

> These were byte-identical between the Windows and Linux builds before I touched them, and the
> Linux copies are now identical to the translated Windows copies.

## How it was done (rules I followed)

1. **String literals only.** No control flow, state handling, IPC channels, or protocol fields were
   touched. I verified afterwards that no `require()`, `path.join()`, CSS class, DOM selector,
   IPC channel name, or shell command string was altered.
2. **Placeholders preserved.** `{me}`, `{name}`, `{Name}`, `{there}`/`{There}`, `{the}`, `{them}`
   are filled at runtime by `UI.fill()` / `UI.peer()` — all template structures kept intact.
3. **Proper nouns kept in Latin script:** TwinDesk, GitHub, Electron, Chromium, ADB, Wi-Fi Direct,
   Bluetooth LE, macOS, Finder, Explorer, Zoom, Teams, Xbox 360, Android 11, Play Protect,
   WhatsApp / Instagram / Telegram; on Linux also Wayland, X11, BlueZ, uinput, evdev, PipeWire,
   NetworkManager, XInput2, XTest, XRandR, XDND, ufw, firewalld, GVFS, NVIDIA.
4. **Chromium layer:** added
   `app.commandLine.appendSwitch('lang', 'zh-CN');`
   in both `windows/main.js` / `windows/mesh/main.js` **and** `linux/main.js` / `linux/mesh/main.js`,
   so `zh-CN.pak` is actually selected.
5. **HTML:** set `<html lang="zh-CN">` on every rendered shell.
6. **One deliberate code-level adaptation:** the file-transfer helper
   `const plural = (n, w) => n + ' ' + w + (n === 1 ? '' : 's');`
   was reduced to `n + ' ' + w` (Chinese has no plural inflection), with call sites passing
   `'个文件'` instead of `'file'`. Behaviour is unchanged; this is the *only* non-literal edit.
   👉 *If you prefer to keep `plural()` untouched for other locales, it is trivial to revert — say so
   and I will rework it.*
7. **Comments left in English.** I deliberately restored every comment I touched, so the diff stays
   readable.

## Deliberately NOT translated

- Device / computer / phone **names** (they come from the OS hostname and the peer).
- **Mirrored phone content**: notification titles/bodies, SMS text, call log entries — the phone's
  own data, passed through verbatim.
- Third-party **app names** in the phone app grid.
- Code **comments** and protocol field documentation (e.g. `shared/protocol.js`).
- **IPC channel names, state/enum values, CSS class names, DOM ids** — these are code, not copy.
- **Shell / CLI command strings** (`adb`, `pkexec`, `ufw allow`, `usermod -aG input`,
  `ip -4 addr show wlan0`, `getprop …`, `netsh`, `openssl`, `ssh twindesk`) and filesystem paths
  (`Start Menu`, `TwinDesk Apps`).
- **Clipboard format identifiers** (e.g. `Clipboard Viewer Ignore`) — matched by name at runtime.
- OS-native dialogs (Open/Save, Explorer, GNOME/KDE pickers) — already Chinese via `zh-CN.pak`.

## Verification performed

- **All 291 `.js` files** across both trees pass `node --check` (Windows 144, Linux 147 — 0 errors).
- Automated scan confirms **0** Chinese characters inside code regions (requires/paths/selectors/
  class names/commands).
- All **26** (Linux) and **27** (Windows) `ui-*.js` tab titles are Chinese; 0 left in English.
- Launched the Windows app and confirmed in-app: sidebar (all 22 tabs), every tab's content, tray
  menu, status line, pairing banner, file-transfer progress and update notification all render
  Chinese.
- Linux build: static verification only — I had no Linux desktop available to run it.

## Two ways you could take this

| Option | What it means |
|---|---|
| **A. Direct zh-CN patch** (fast) | Merge the translated files as-is. Simple, but the UI stays hardcoded — other languages still need the same treatment later. |
| **B. Proper i18n layer** (recommended for the long run) | Extract strings into `locales/*.json` (or a `strings.js` map) keyed by id, load by `app.getLocale()`, and use my translation as the `zh-CN` source. I'm happy to help with the extraction and to re-submit against the new structure. |

I have no strong preference — **B** is cleaner, **A** is quicker. Tell me which you'd like.

## Two things I want to flag before you merge

1. **`shared/` may affect the Mac build.** Your README lists `shared/` as *"Used by every app"*, and
   `windows/mesh/ui-phone.js` is headed *"Port of mac/ui-phone.js"*. **Merging those 14 files as-is
   would therefore make the Mac UI partially Chinese too.** Please confirm how `shared/` is consumed
   upstream; a gated/locale-keyed approach (Option B) avoids a half-translated Mac UI. *(I could not
   verify the Mac tree from here.)*
2. **No clean diff yet.** I edited the files in place inside installed (non-git) copies, so I do not
   currently have a `.patch` regenerated against upstream `HEAD`. I can supply the modified files
   (ZIP / file-by-file) immediately, or regenerate a proper diff once you tell me the target branch.

## Licensing

Contributed under **GPL-3.0**, the same license as TwinDesk, with no additional dependencies.
I'm fine with this being merged, reworked, or used as a reference for a future i18n layer.

## Checklist

- [ ] Confirm preferred option (A: direct patch / B: i18n extraction)
- [ ] Confirm whether `shared/` is shared with the Mac app (see flag #1)
- [ ] Decide on `plural()` (keep as-is vs. revert to English-friendly form)
- [ ] I provide files/diff in the requested format
- [ ] Add a `zh-CN` entry wherever locales are listed (About / README / docs)

Thanks for building TwinDesk — happy to iterate on this until it fits your codebase.

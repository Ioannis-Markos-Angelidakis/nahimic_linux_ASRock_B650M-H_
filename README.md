# nahimic-linux for ASRock B650M-H/M.2+ (CachyOS / Arch)

This is a **complete, tested port** of [wearzdk/nahimic-linux](https://github.com/wearzdk/nahimic-linux) v0.3.3
for your ASRock B650M-H/M.2+ (Realtek **ALC897**, `alsa.components = HDA:10ec0897,18491897,00100402`).

It runs the **real ASRock Nahimic engine** — the same APO4 runtime and the same
`Generic_ExternalSpeakers` factory settings that the official
[Nahimic(v1.10.11_APO4)](https://download.asrock.com/Utility/Others/Nahimic(v1.10.11_APO4).zip)
driver installs on Windows — through Wine + PipeWire, exactly the way upstream
nahimic-linux does for supported laptops and the MSI MEG X570 UNIFY.

**New in 0.3.3-3: the Windows Equalizer presets.** The EQ page now shows the
same preset chips as the Windows Nahimic app — per profile:

| Profile | Presets |
| --- | --- |
| Music | Dynamic（增加动态）, Heavy Bass（强化低音）, Brilliant（增强明亮）, Vocals（人声增强）, Custom |
| Movie | Dialogues（对白增强）, Immersion（更加沉浸）, Custom |
| Gaming | Fast-paced（节奏紧凑）, Atmospheric（氛围强化）, FPS Competition（第一人称射击）, Racing（极限竞速）, Custom |
| Communication | Balanced（平衡语音）, Clear Chat（通话清晰）, Livestream（实况转播）, Custom |

These are the **real vendor curves**, extracted from the Nahimic3 app bundle
inside the same ASRock installer (`Settings/v1/EQPresets/*/OriginalSettings/*.json`,
e.g. Heavy Bass = 31 Hz +2, 62 Hz +7, 125 Hz +5, 250 Hz 0, 500 Hz −1, 1 k 0, 2 k 0,
4 k 0, 8 k +2, 16 k +1). Clicking a chip writes all ten band gains in one store
update; dragging a slider switches the selection back to Custom — the same
behaviour as the Windows panel. Display names are localized in all 11 interface
languages using the Windows app's own translations.

**New in 0.3.3-4: the system tray icon.** Like the Windows taskbar icon, the panel
now lives in the KDE Plasma system tray:

- the Nahimic icon appears in the panel's tray area — drag it out of the popup
  (or set it *Always shown* in *System Tray Settings → Entries*) to keep it
  permanently visible on the dock, just like on Windows;
- left-click toggles the panel window, right-click opens the menu
  (open / hide / quit);
- closing the window hides it to the tray — the audio service and your settings
  keep running; quitting is explicit, from the tray menu;
- at login the app starts hidden in the tray (the existing autostart entry now
  launches `nahimic --hidden`);
- on a **Plasma Wayland** session the panel restarts itself once under XWayland
  (Qt's tray uses the XEmbed protocol, and Plasma bridges XEmbed icons via
  `xembed-sni-proxy`); on X11 sessions it attaches directly;
- `NAHIMIC_TRAY=0 nahimic` restores the plain no-tray behaviour.

## What was verified during this port

- Your subsystem ID `18491897` is the **first hardware ID** listed in the ASRock
  extension driver (`AVoluteNH3Ext.inf` → `HDAUDIO\FUNC_01&VEN_10EC&DEV_0897&SUBSYS_18491897`).
- ASRock's settings cabinet contains `Devices/Generic_ExternalSpeakers.nsx`
  (`HWID = subsys_1849`, `FormFactor = Speakers`) — a desktop profile, no laptop tuning.
- All four ASRock audio-profile GUIDs (Music / Movie / Gaming / Communication)
  are identical to the ones the control panel already uses.
- The extraction pipeline was run end-to-end: the 11 vendor DLLs hash-match the
  project's pinned Microsoft Update cabinet, the 7 `factory-asrock` files and the
  17 `eqpresets` curves hash-match the ASRock installer, and all 43 unit tests
  pass (13 upstream + 4 ASRock routing + 8 EQ preset + 7 tray + 11 i18n/UI etc.).
- Both Wine binaries (`apo_probe.exe`, `apo_control.exe`) compile warning-free
  with the project's `-Wall -Wextra -Werror` flags.

## Files

| File | Purpose |
| --- | --- |
| `PKGBUILD` | Local package build (same name/version as AUR, `pkgrel=4`) |
| `0001-Add-ASRock-desktop-Line-Out-support.patch` | The ASRock port itself, applies to upstream tag `v0.3.3` |
| `0002-Add-vendor-EQ-presets.patch` | Vendor EQ presets + preset chips in the equalizer, applies on top of 0001 |
| `0003-Add-system-tray-icon.patch` | Windows-like system tray icon (hide-to-tray, tray menu, autostart in tray, Plasma Wayland support), applies on top of 0002 |
| `nahimic-linux.install` | Upstream install script (unchanged) |

## Build on CachyOS

```bash
# build tools and extraction deps
sudo pacman -S --needed base-devel mingw-w64-gcc cabextract innoextract

cd nahimic-linux-asrock
makepkg -si
```

The build downloads the pinned APO4 runtime from Microsoft Update and the ASRock
driver zip from download.asrock.com, verifies SHA-256 hashes of everything it
installs (including the EQ preset curves), compiles the native host, and
installs the package. First launch:

```bash
nahimic            # opens the Qt control panel and activates the service
nahimic --status   # JSON status; expect ready=true, active=true while playing
```

Select the rear **Line Out** as your output device in system sound settings
before starting. Effects attach to Line Out only; plugging headphones into the
front panel automatically bypasses them (as on the MSI desktop port), and
HDMI / Bluetooth keep their own paths.

> If you previously installed/tried the unpatched AUR `nahimic-linux`, remove
> leftover state before first launch: `rm -rf ~/.local/share/nahimic-linux`.
> The local build upgrades cleanly over the earlier `0.3.3-2`/`0.3.3-3` (and
> the AUR `0.3.3-1`); your saved profile and effect settings are preserved —
> only the package files change. After upgrading to `-4`, run `nahimic` once
> and close the window: from then on the icon stays in the tray. A future
> official release that includes this port will upgrade over it.

## Requirements & checks (if it does not attach)

```bash
pactl list sinks | grep -iE "active port|alsa.components"
```

- `alsa.components` must contain `HDA:10ec0897,18491897,` (yours does).
- The active port must be `analog-output-lineout` (PipeWire path naming) or
  `[Out] Line Out` / `[Out] Speaker` (UCM naming) — all three are accepted.
- WirePlumber ≥ 0.5, PipeWire + pipewire-pulse active (`systemctl --user status pipewire pipewire-pulse wireplumber`).
- Service log: `journalctl --user -u nahimic.service -e`
- App data (Wine prefix, settings, state): `~/.local/share/nahimic-linux/`

## What the patches change

### 0001 — ASRock desktop Line Out

1. `host/desktop_audio.py` — new `ASROCK_LINE_OUT_COMPONENTS` table (all nine HDA
   subsystems from the ASRock INF; `18491897` verified on B650M-H/M.2+), ASRock
   routing to `factory-asrock` + `Generic_ExternalSpeakers.nsx`, Line Out port
   acceptance across path/UCM naming, and a headphone-jack name fallback for
   HDA cards that report no port `type`.
2. `packaging/extract_runtime.py` — extracts the ASRock settings from the official
   installer with `innoextract`; the generic AIstone/MSI restore-tool settings
   become optional at build time (this PKGBUILD passes an empty settings path,
   since the Nahimic support site is unreachable from some networks).
3. `packaging/runtime-sha256.json` — pins the seven `factory-asrock` files.
4. `tests/test_routing.py` — four new ASRock routing tests.
5. READMEs document the new supported boards.

### 0002 — vendor EQ presets

1. `packaging/extract_runtime.py` — also pulls the Nahimic3 appxbundle from the
   installer's `%TEMP` payload and extracts
   `Settings/v1/EQPresets/<Profile>/OriginalSettings/<Preset>.json` into
   `eqpresets/`; `packaging/runtime-sha256.json` pins all 17 curves.
2. `host/apo_control.cpp` — new `--set-settings NAME VALUE [NAME VALUE …]` mode:
   one Wine process applies a whole preset (ten COM writes) instead of ten
   process round-trips.
3. `app/backend.py` — `EQ_BANDS` mapping (vendor `EQ32HzGainDB`… → engine
   `kSet_EQ31HzGainDB`…), `eq_presets()` loader for `/usr/share/nahimic-linux/eqpresets`,
   and `set_settings()` with the same read-back verification as `set_setting()`.
4. `app/main.py` — preset chip row in the equalizer dialog: chips follow the
   active profile (switching Music ↔ Gaming swaps the row), clicking applies
   the curve optimistically and then confirms it, manual slider edits move the
   selection back to Custom, and stale clicks during a profile switch are
   discarded.
5. `app/locales/*.json` — preset display names in all 12 catalogs, taken from
   the Windows app's own language packs (two obvious vendor typos corrected:
   fr *Personalisé* → *Personnalisé*, ja *レーシグ* → *レーシング*; tr, ru-Balanced
   and es-Custom are not present in the vendor packs and use standard wording).
6. `tests/test_eq_presets.py` — eight tests: file→setting mapping, chip
   application, coalescing, Custom fallback, profile chip swap, stale-click
   discard, failed-write recovery.

### 0003 — system tray icon

1. `app/main.py` — a `TrayIcon` (`QSystemTrayIcon` + context menu with
   open/hide and quit), hide-to-tray on window close, tray click toggling the
   panel, a `--hidden` start for autostart, and a one-time XWayland restart on
   Plasma Wayland sessions so `xembed-sni-proxy` can host the XEmbed icon.
2. `host/entry.py` — `nahimic --autostart` now launches the panel hidden in the
   tray instead of only activating the service.
3. `app/locales/*.json` — tray strings in all 12 languages.
4. `tests/test_tray.py` — seven tests: menu labels, hide-to-tray, click
   toggling, quit, no-tray close, language retranslation and the Wayland retry
   decision.

## After it works: upstream it

The project explicitly invites new device support (read `AGENTS.md` in the repo).
Test the four upstream acceptance points on your machine, then open a pull
request with both patches (plus your verification notes) against
https://github.com/wearzdk/nahimic-linux:

1. effect switching (enable/disable, Music/Movie/Gaming/Communication)
2. parameter persistence across a cold restart
3. service restart (`systemctl --user restart nahimic.service`)
4. 30+ minutes of continuous playback without dropouts

## Notes

- The vendor runtime stays the project-pinned APO4 **SWC 4.14** from Microsoft
  Update; the ASRock installer ships SWC 5.0.6. The project already mixes the
  pinned runtime with per-vendor settings (that is exactly how the two verified
  laptops work), and the ASRock settings XMLs are byte-identical to the pinned
  ones for every shared file — only `Generic_ExternalSpeakers.nsx` is new.
- EQ presets are data files under `/usr/share/nahimic-linux/eqpresets/`. The
  APO's own `SetCurrentEqPreset` API stays untouched — presets are applied as
  plain band-gain writes, which is one Wine round-trip and needs no extra
  APO-side state. If a preset is ever missing, the chip row simply hides
  (builds without the ASRock zip are unaffected).
- If ASRock releases a newer Nahimic zip, the pinned hash will fail the build;
  re-download, re-extract and update both hashes (see `runtime-sha256.json`).
- This is an independent community port; Nahimic, A-Volute and ASRock
  trademarks and runtime components belong to their respective owners.

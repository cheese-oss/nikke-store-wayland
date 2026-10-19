**Title:** NIKKE: in-game store stays white and closing it ends the game on the Wayland driver

**Template:** Compatibility Report

# Compatibility Report
- Name of the game with compatibility issues: GODDESS OF VICTORY: NIKKE
- Steam AppID of the game: None, not a Steam game (started with `proton run`)

## System Information
- GPU: AMD Radeon RX 9070 XT
- Video driver version: Mesa 26.2.4 (radeonsi, RADV)
- Kernel version: 7.2.10-1-default (openSUSE Tumbleweed 20261014)
- Link to full system information report as [Gist][system-info] (inxi output; the game is not run through Steam)
- Proton version: dwproton-11.0-15

## I confirm:
- [x] that I haven't found an existing compatibility report for this game.
- [x] that I have checked whether there are updates for my system available.

The other NIKKE reports (#72, #55, #29) are about different problems.

<!-- Please add `PROTON_LOG=1 %command%` to the game's launch options and
attach the generated $HOME/steam-$APPID.log to this issue report as a file.
(Proton logs compress well if needed.)-->

## Symptoms <!-- What's the problem? -->

With the game on Wine's Wayland driver (`WINE_GRAPHICS_DRIVER=wayland`,
with `DISPLAY` also set so the launcher can use X11), on a GNOME 50.5
Wayland session:

1. The in-game store opens as an empty white view.
2. Closing the store ends the game. The log on Gist shows, at lines
   6332-6334:

   ```
   xdg_wm_base#13: error 2: destroyed popup not top most popup
   61320.942:06ac:0724:err:waylanddrv:waylanddrv_unix_read_events Protocol error on xdg_wm_base#13 with code 2
   61320.942:06ac:0724:err:waylanddrv:wayland_read_events_thread Failed to read events from the compositor, terminating process
   ```

The [log on Gist][proton-log] was captured with Proton's
[usual Wine debug channels][proton-channels], set through
`WINEDEBUG`, with unwind tracing disabled. Exception trace (`trace:seh`)
lines were removed to reduce the file size. Two checkout URLs were redacted.

## Reproduction

1. Start NIKKE with dwproton-11.0-15 and `WINE_GRAPHICS_DRIVER=wayland` on
   a GNOME Wayland session.
2. Open the store from the game. It stays white.
3. Close the store. The game exits.

This issue was written with the help of generative AI.

[proton-channels]: https://dawn.wine/dawn-winery/dwproton/src/commit/b8c56daa4cd1f8ce0ce9c4135bf587e063e4b02f/proton#L1981-L1982

[system-info]: https://gist.github.com/cheese-oss/2b7af4bf2970d160e033aa0fe4ce637b
[proton-log]: https://gist.github.com/cheese-oss/b2a8b517e6e772515fd74784990e3ca9

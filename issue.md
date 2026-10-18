**Title:** NIKKE: in-game store stays white and closing it ends the game on the Wayland driver

**Template:** Compatibility Report

# Compatibility Report
- Name of the game with compatibility issues: GODDESS OF VICTORY: NIKKE
- Steam AppID of the game: None, not a Steam game (started with `proton run`)

## System Information
- GPU: AMD Radeon RX 9070 XT
- Video driver version: Mesa 26.2.4 (radeonsi, RADV)
- Kernel version: 7.2.10-1-default (openSUSE Tumbleweed 20261014)
- Link to full system information report as [Gist](https://gist.github.com/): attached as `nikke-sysinfo.txt` (inxi output; the game is not run through Steam)
- Proton version: dwproton-11.0-14

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
2. Closing the store ends the game. The attached log shows, at lines
   85735-85737:

   ```
   xdg_wm_base#13: error 2: destroyed popup not top most popup
   5100.302:07fc:0864:err:waylanddrv:waylanddrv_unix_read_events Protocol error on xdg_wm_base#13 with code 2
   5100.302:07fc:0864:err:waylanddrv:wayland_read_events_thread Failed to read events from the compositor, terminating process
   ```

The attached log is Proton's output with the Wine debug channels that
`PROTON_LOG=1` enables ([proton#L1944-L1945][proton-channels]), set
directly in `WINEDEBUG`. NIKKE is not started from Steam, so `SteamGameId`
is unset. Proton then writes no `steam-$APPID.log`
([proton#L1862-L1863][proton-log]), and with `PROTON_LOG` set it turns
Wine's logging off ([proton#L2336-L2337][proton-log-setup],
[proton#L2383-L2384][proton-log-off]). The `+unwind` channel alone wrote
30.2 million `trace:unwind` lines (2.7 GB); those lines are removed from
the attached log. The only other change is a payment checkout token in a
store URL, replaced with `REDACTED`.

## Reproduction

1. Start NIKKE with dwproton-11.0-14 and `WINE_GRAPHICS_DRIVER=wayland` on
   a GNOME Wayland session.
2. Open the store from the game. It stays white.
3. Close the store. The game exits.

This issue was written with the help of generative AI.

[proton-channels]: https://dawn.wine/dawn-winery/dwproton/src/commit/8b178c8870a68f5012ee4a1d82078fa5a7b3a814/proton#L1944-L1945
[proton-log]: https://dawn.wine/dawn-winery/dwproton/src/commit/8b178c8870a68f5012ee4a1d82078fa5a7b3a814/proton#L1862-L1863
[proton-log-setup]: https://dawn.wine/dawn-winery/dwproton/src/commit/8b178c8870a68f5012ee4a1d82078fa5a7b3a814/proton#L2336-L2337
[proton-log-off]: https://dawn.wine/dawn-winery/dwproton/src/commit/8b178c8870a68f5012ee4a1d82078fa5a7b3a814/proton#L2383-L2384

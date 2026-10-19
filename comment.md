**First comment on the issue: tests and patches**

With the whole game on X11, neither symptom happens. In Bottles with
`WINE_GRAPHICS_DRIVER=wayland`, both happen.

**White store.** Each `INTLWebViewHelper.exe` process in the attached log
loads `winex11.drv` (lines 80264, 81137 and 82089), because user32 lists
it for X11 ([user_main.c#L185-L189][user32-list]). Without that entry the
store stays white: Chromium's software output copies each frame into its
window with `GetDC`/`BitBlt`/`ReleaseDC`
([software_output_device_win.cc#68][chromium-sod], current Chromium), the
window's top-level belongs to the game process, and such a DC gets no
window surface ([dce.c#L905-L911][dce-surface]), so the null driver drops
the bits ([driver.c#L411-L416][nulldrv-putimage]).

**Exit when the store closes.** Hiding a popup destroys its `xdg_popup`
([wayland_surface.c#L714-L720][popup-destroy]) even while popups it owns
are still mapped. xdg-shell requires the topmost popup to be destroyed
first ([xdg-shell.xml#L1281-L1286][xdg-destroy]). GE-Proton fixed the
same order independently in [b492349d][ge-fix] (from GE-Proton11-10),
for its own Wayland driver; that patch does not apply to DWProton's.

**Dropdowns, once the store renders.** The store's dropdowns close as soon
as they open. Each is an unowned window of the game process with
`WS_THICKFRAME` and `WS_EX_NOACTIVATE|WS_EX_TOOLWINDOW`. The thick frame
makes it a managed window ([window.c#L436-L437][managed-thickframe]), so it
becomes an `xdg_toplevel`, and a `+waylanddrv,+keyboard` trace shows it
taking keyboard focus from the game window.

**Patches** against the `dwproton-11.0-15` release (Wine
`b75a4125963073b01d2b3bfce6060d697d02066b`):
<https://github.com/cheese-oss/nikke-store-wayland/tree/main/patches>

1. winewayland.drv: Retire nested popup roles before their parent.
2. win32u: Forward bitmap copies into windows of another process.
3. user32: Let INTLWebViewHelper.exe use the Wayland driver.
4. winewayland.drv: Keep non-activating tool popups unmanaged.

Small Win32 test programs that reproduce each problem, run through
`proton run` with the 11.0-15 runner and with a copy of it using the
patched modules:

| Check | 11.0-15 | Patched |
| --- | --- | --- |
| Cross-process software bitmap, Weston | Wrong pixels | Correct pixels |
| Disjoint bitmap clipping, Weston | — | Passes |
| Helper-owned child under a foreign top-level, Weston | — | Passes |
| OpenGL output from the helper, Weston | — | Passes |
| Nested popup teardown, Mutter | Protocol error | Survives |
| Popup teardown from two threads, Mutter | Protocol error | Survives |
| Chromium-style dropdown, Mutter | Loses focus | Stays open; click, wheel and key delivered |
| Dropdown hide and reopen, Mutter | — | Passes |

In NIKKE with the series applied to 11.0-15, as with 11.0-14 before, the
store renders, closing it keeps the game running, and its dropdowns stay
open and work with the mouse, the scrollbar and the keyboard. The mouse
wheel does not scroll a dropdown.

If you want them as a pull request against `wine-dwproton` (dwproton's
`wine` submodule), I can open one.

This comment was written with the help of generative AI.

[user32-list]: https://dawn.wine/dawn-winery/wine-dwproton/src/commit/b75a4125963073b01d2b3bfce6060d697d02066b/dlls/user32/user_main.c#L185-L189
[chromium-sod]: https://chromium.googlesource.com/chromium/src/+/ce7b69d7c8d5bdd1b4d2814777fb5818865bb710/components/viz/service/display_embedder/software_output_device_win.cc#68
[dce-surface]: https://dawn.wine/dawn-winery/wine-dwproton/src/commit/b75a4125963073b01d2b3bfce6060d697d02066b/dlls/win32u/dce.c#L905-L911
[nulldrv-putimage]: https://dawn.wine/dawn-winery/wine-dwproton/src/commit/b75a4125963073b01d2b3bfce6060d697d02066b/dlls/win32u/driver.c#L411-L416
[popup-destroy]: https://dawn.wine/dawn-winery/wine-dwproton/src/commit/b75a4125963073b01d2b3bfce6060d697d02066b/dlls/winewayland.drv/wayland_surface.c#L714-L720
[xdg-destroy]: https://gitlab.freedesktop.org/wayland/wayland-protocols/-/blob/1.49/stable/xdg-shell/xdg-shell.xml#L1281-1286
[managed-thickframe]: https://dawn.wine/dawn-winery/wine-dwproton/src/commit/b75a4125963073b01d2b3bfce6060d697d02066b/dlls/winewayland.drv/window.c#L436-L437
[ge-fix]: https://github.com/GloriousEggroll/proton-ge-custom/commit/b492349d0233dcff26c5a585ddd3f2a4c8c3095c

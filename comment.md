**First comment on the issue: tests and the code involved**

Tests in other setups: with the whole game on X11, neither symptom
happens. In Bottles with `WINE_GRAPHICS_DRIVER=wayland`, both happen.

## White store

In the attached log, each `INTLWebViewHelper.exe` process loads
`winex11.drv` (lines 80264, 81137 and 82089).

With `INTLWebViewHelper.exe` removed from user32's X11 list (below), the
store was still white. A process snapshot then showed the helper on the
Wayland driver, its GPU process running with
`--use-angle=swiftshader-webgl` and its renderers with
`--disable-gpu-compositing`.

The code involved:

- user32 loads winex11 instead of winewayland for `INTLWebViewHelper.exe`:
  [user_main.c#L185-L189][user32-list],
  [user_main.c#L213-L215][user32-load].
- Chromium's Windows software output device copies each frame into its
  window with `GetDC`, `skia::CopyHDC` and `ReleaseDC`
  ([software_output_device_win.cc#68][chromium-sod]); `CopyHDC` uses
  `BitBlt` ([skia_utils_win.cc#179][chromium-copyhdc]). These links are
  to current Chromium; I have not checked the CEF version NIKKE ships.
- A DC whose top-level window belongs to another process gets no window
  surface: [dce.c#L905-L911][dce-surface]. winewayland's driver table has
  no GDI functions ([waylanddrv_main.c#L40-L70][wayland-funcs]), and the
  null driver's `PutImage` returns without drawing:
  [driver.c#L411-L416][nulldrv-putimage].

In a test program, one process copies a bitmap with
`GetDC`/`BitBlt`/`ReleaseDC` into a child of another process's top-level
window. On unpatched 11.0-15 under Weston, the copied pixels do not appear
in that window.

## Exit when the store closes

The code involved:

- A window that is hidden loses its Wayland role
  ([window.c#L202-L204][role-choice], [window.c#L243-L245][role-clear]).
  For a popup, that destroys its `xdg_popup`
  ([wayland_surface.c#L714-L720][popup-destroy]), whether or not popups it
  owns are still mapped.
- xdg-shell requires popups to be destroyed topmost first; otherwise the
  compositor sends `not_the_topmost_popup`, error 2
  ([xdg-shell.xml#L1281-L1286][xdg-destroy],
  [xdg-shell.xml#L45-L46][xdg-error]).

A test program that hides an owned popup while a popup it owns is still
visible gets the same protocol error and exit: with dwproton 11.0-14 and
GE-Proton 11-9 under GNOME and Weston, and with 11.0-15 under Mutter.

## Dropdowns, once the store renders

With the store rendering (patches 2 and 3 below), its dropdowns, such as
the billing city, closed as soon as they opened. Wine traces showed each
dropdown as an unowned `Chrome_WidgetWin_0` created by the game process,
with `WS_POPUP|WS_CLIPSIBLINGS|WS_CLIPCHILDREN|WS_THICKFRAME` and
`WS_EX_NOACTIVATE|WS_EX_TOOLWINDOW`, shown without activation. Each time,
a `+waylanddrv,+keyboard` trace showed a keyboard leave on the game window
and an enter on the dropdown.

The code involved:

- `is_window_managed` returns TRUE for any window with `WS_THICKFRAME`:
  [window.c#L436-L437][managed-thickframe]. A managed window does not take
  the owner hint ([window.c#L497][managed-owner]), and a window without an
  owner surface becomes a toplevel ([window.c#L202-L204][role-choice]).
- Chromium's views toolkit gives menu widgets `WS_POPUP`, `WS_THICKFRAME`
  and `WS_EX_TOOLWINDOW` ([widget_hwnd_utils.cc#120-125][chromium-menu]),
  and `WS_EX_NOACTIVATE` when they cannot be activated
  ([widget_hwnd_utils.cc#56-57][chromium-noactivate]).

A test program that opens a window with those styles beside a focused
child window, in the same process, loses the child's focus under Mutter.

## Patches

I have four patches for these against `dwproton/11.0-15` (Wine
`f078dded59f559884b58fbf0769847e9c9fbb3ac`):
<https://github.com/cheese-oss/nikke-store-wayland/tree/main/patches>.

1. winewayland.drv: Retire nested popup roles before their parent.
2. win32u: Forward bitmap copies into windows of another process.
3. user32: Let INTLWebViewHelper.exe use the Wayland driver.
4. winewayland.drv: Keep non-activating tool popups unmanaged.

Results of the test programs above on a standalone 11.0-15 Wine build
(Wayland only, without X11, audio or Schannel):

| Check | Unpatched | Patched |
| --- | --- | --- |
| Cross-process software bitmap, Weston | Wrong pixels | Correct pixels |
| Disjoint bitmap clipping, Weston | — | Passes |
| Helper-owned child under a foreign top-level, Weston | — | Passes |
| OpenGL rendering, Weston | — | Passes |
| Nested popup teardown, Mutter | Protocol error | Survives |
| Chromium-style dropdown, Mutter | Loses focus | Stays open; click, wheel and key delivered |
| Dropdown hide and reopen, Mutter | — | Passes |

In NIKKE with the series applied to 11.0-14, the store renders, closing it
keeps the game running, and its dropdowns stay open and work with the
mouse, the scrollbar and the keyboard. The mouse wheel does not scroll a
dropdown. I have not tested the 11.0-15 build in NIKKE.

If you want them as a pull request against `wine-dwproton`, I can open
one.

This comment was written with the help of generative AI.

[user32-list]: https://dawn.wine/dawn-winery/wine-dwproton/src/commit/f078dded59f559884b58fbf0769847e9c9fbb3ac/dlls/user32/user_main.c#L185-L189
[user32-load]: https://dawn.wine/dawn-winery/wine-dwproton/src/commit/f078dded59f559884b58fbf0769847e9c9fbb3ac/dlls/user32/user_main.c#L213-L215
[chromium-sod]: https://chromium.googlesource.com/chromium/src/+/ce7b69d7c8d5bdd1b4d2814777fb5818865bb710/components/viz/service/display_embedder/software_output_device_win.cc#68
[chromium-copyhdc]: https://chromium.googlesource.com/chromium/src/+/ce7b69d7c8d5bdd1b4d2814777fb5818865bb710/skia/ext/skia_utils_win.cc#179
[dce-surface]: https://dawn.wine/dawn-winery/wine-dwproton/src/commit/f078dded59f559884b58fbf0769847e9c9fbb3ac/dlls/win32u/dce.c#L905-L911
[wayland-funcs]: https://dawn.wine/dawn-winery/wine-dwproton/src/commit/f078dded59f559884b58fbf0769847e9c9fbb3ac/dlls/winewayland.drv/waylanddrv_main.c#L40-L70
[nulldrv-putimage]: https://dawn.wine/dawn-winery/wine-dwproton/src/commit/f078dded59f559884b58fbf0769847e9c9fbb3ac/dlls/win32u/driver.c#L411-L416
[role-choice]: https://dawn.wine/dawn-winery/wine-dwproton/src/commit/f078dded59f559884b58fbf0769847e9c9fbb3ac/dlls/winewayland.drv/window.c#L202-L204
[role-clear]: https://dawn.wine/dawn-winery/wine-dwproton/src/commit/f078dded59f559884b58fbf0769847e9c9fbb3ac/dlls/winewayland.drv/window.c#L243-L245
[popup-destroy]: https://dawn.wine/dawn-winery/wine-dwproton/src/commit/f078dded59f559884b58fbf0769847e9c9fbb3ac/dlls/winewayland.drv/wayland_surface.c#L714-L720
[xdg-destroy]: https://gitlab.freedesktop.org/wayland/wayland-protocols/-/blob/1.49/stable/xdg-shell/xdg-shell.xml#L1281-1286
[xdg-error]: https://gitlab.freedesktop.org/wayland/wayland-protocols/-/blob/1.49/stable/xdg-shell/xdg-shell.xml#L45-46
[managed-thickframe]: https://dawn.wine/dawn-winery/wine-dwproton/src/commit/f078dded59f559884b58fbf0769847e9c9fbb3ac/dlls/winewayland.drv/window.c#L436-L437
[managed-owner]: https://dawn.wine/dawn-winery/wine-dwproton/src/commit/f078dded59f559884b58fbf0769847e9c9fbb3ac/dlls/winewayland.drv/window.c#L497
[chromium-menu]: https://chromium.googlesource.com/chromium/src/+/ce7b69d7c8d5bdd1b4d2814777fb5818865bb710/ui/views/widget/widget_hwnd_utils.cc#120
[chromium-noactivate]: https://chromium.googlesource.com/chromium/src/+/ce7b69d7c8d5bdd1b4d2814777fb5818865bb710/ui/views/widget/widget_hwnd_utils.cc#56

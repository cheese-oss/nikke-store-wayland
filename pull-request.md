**Title:** winewayland/win32u: Fix NIKKE's in-game store on Wayland

Fixes #ISSUE: the white store, the game exit when the store closes, and
its dropdowns closing at once. The Proton log is attached to #ISSUE.

Four commits against Wine `b75a4125963073b01d2b3bfce6060d697d02066b`
(the dwproton-11.0-15 release), in dependency order:

1. [Retire nested popup roles before their parent][patch1]. Clear child
   roles first on teardown or compositor dismissal, and destroy window
   surfaces under the window data lock. Stands alone.
2. [Forward bitmap copies into another process's windows][patch2]. Queue
   clipped 32-bit `SRCCOPY` images until `ReleaseDC`, then map them
   read-only into the receiving process for painting. Validate copied
   headers and regions; leave unrelated mappings alone.
3. [Let INTLWebViewHelper.exe use Wayland][patch3]. Remove its forced-X11
   entry; depends on patch 2. The launcher and `tbs_browser.exe` retain
   their X11 fallback.
4. [Keep non-activating tool popups unmanaged][patch4]. Use ungrabbed
   popups even with a thick frame; for an unowned popup with a parent
   hint in another process, use the foreground window instead.

GE-Proton also has a patch for the popup protocol error, included from
GE-Proton11-10: [b492349d][ge-fix].

Each commit builds on 11.0-15. [Small Win32 reproducers][tests] were run
through `proton run` with the published runner and a copy using the
patched modules, under headless Weston and Mutter:

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

Weston accepts the unpatched popup teardown, so those checks use Mutter.
The two-thread probe passed 25 of 25 runs with the patch (300 iterations
each); without its window data lock change, 9 of 25 runs failed. A
further probe confirms that posting the bitmap message with an unrelated
mapping's address leaves that mapping intact.

In NIKKE on patched 11.0-15, as on 11.0-14 before, the store renders,
closing it keeps the game running, and dropdowns work with the mouse,
scrollbar and keyboard. The mouse wheel still does not scroll a store
dropdown, despite event delivery in the probe.

Bitmap forwarding covers clipped 32-bit `SRCCOPY`, not hardware output.
Updates flush on `ReleaseDC`; a posted view can leak if its window is
destroyed before handling the message.

This pull request and its patches were prepared with the help of generative AI.

[patch1]: https://github.com/cheese-oss/nikke-store-wayland/blob/2e6480c92e29deee3de579a6bc0782673c487289/patches/0001-winewayland.drv-Retire-nested-popup-roles-before-the.patch
[patch2]: https://github.com/cheese-oss/nikke-store-wayland/blob/2e6480c92e29deee3de579a6bc0782673c487289/patches/0002-win32u-Forward-bitmap-copies-into-windows-of-another.patch
[patch3]: https://github.com/cheese-oss/nikke-store-wayland/blob/2e6480c92e29deee3de579a6bc0782673c487289/patches/0003-user32-Let-INTLWebViewHelper.exe-use-the-Wayland-dri.patch
[patch4]: https://github.com/cheese-oss/nikke-store-wayland/blob/2e6480c92e29deee3de579a6bc0782673c487289/patches/0004-winewayland.drv-Keep-non-activating-tool-popups-unma.patch
[tests]: https://github.com/cheese-oss/dwproton-nikke-split/tree/7ab01306283af86c8ae50b4a96bec499adee7963/tests
[ge-fix]: https://github.com/GloriousEggroll/proton-ge-custom/commit/b492349d0233dcff26c5a585ddd3f2a4c8c3095c

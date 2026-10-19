**This comment, the patches, and the test programs were written with the help of generative AI.**

The store's dropdowns also close as soon as they open; this becomes
visible once the store renders.

[Small Win32 test programs][tests], run through `proton run` with the
published 11.0-15 runner and with a copy using the patched modules, under
headless Weston and Mutter:

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
A further probe posts the bitmap message with an unrelated mapping's
address; the patched receiver leaves the mapping alone.

In NIKKE with the series applied to 11.0-15, the store renders, closing it
keeps the game running, and its dropdowns work with the mouse, scrollbar
and keyboard. The mouse wheel still does not scroll a store dropdown.

**Patches**, against Wine `b75a4125963073b01d2b3bfce6060d697d02066b`
(the dwproton-11.0-15 release):

1. [Retire nested popup roles before their parent][patch1], with window
   surface destruction under the window data lock. Stands alone.
2. [Forward clipped 32-bit bitmap copies into another process's windows][patch2]
   on `ReleaseDC`, using validated read-only mappings.
3. [Remove INTLWebViewHelper.exe from the forced-X11 list][patch3].
   Depends on patch 2.
4. [Keep non-activating tool popups unmanaged][patch4], using the
   foreground window when an unowned popup's parent hint is in another
   process.

GE-Proton also has a patch for the popup protocol error, included from
GE-Proton11-10: [b492349d][ge-fix].

If you want a pull request, I can open one.

[patch1]: https://github.com/cheese-oss/nikke-store-wayland/blob/2e6480c92e29deee3de579a6bc0782673c487289/patches/0001-winewayland.drv-Retire-nested-popup-roles-before-the.patch
[patch2]: https://github.com/cheese-oss/nikke-store-wayland/blob/2e6480c92e29deee3de579a6bc0782673c487289/patches/0002-win32u-Forward-bitmap-copies-into-windows-of-another.patch
[patch3]: https://github.com/cheese-oss/nikke-store-wayland/blob/2e6480c92e29deee3de579a6bc0782673c487289/patches/0003-user32-Let-INTLWebViewHelper.exe-use-the-Wayland-dri.patch
[patch4]: https://github.com/cheese-oss/nikke-store-wayland/blob/2e6480c92e29deee3de579a6bc0782673c487289/patches/0004-winewayland.drv-Keep-non-activating-tool-popups-unma.patch
[tests]: https://github.com/cheese-oss/dwproton-nikke-split/tree/7ab01306283af86c8ae50b4a96bec499adee7963/tests
[ge-fix]: https://github.com/GloriousEggroll/proton-ge-custom/commit/b492349d0233dcff26c5a585ddd3f2a4c8c3095c

This comment and the patch were written with the help of generative AI.

**First comment on the issue: tests and a patch**

Two small Win32 programs, run through `proton run` under headless Mutter
with the published dwproton-11.0-15 runner and with a copy whose modules
include the patch below:

| Test program | 11.0-15 | Patched |
| --- | --- | --- |
| Hides an owner window before its nested popup ([window-probe.c][window-probe]) | Same protocol error | Exits normally |
| A thread exits with a nested popup pair still shown while another thread hides their owner ([popup-race-probe.c][race-probe]) | Same protocol error | 25 of 25 runs exit normally, 300 iterations each |

Under headless Weston, the first program does not get the protocol error
with the unpatched runner.

In NIKKE, with the patch applied together with three other patches for the
store on 11.0-15, closing the store keeps the game running.

**Patch**, against Wine `b75a4125963073b01d2b3bfce6060d697d02066b` (the
dwproton-11.0-15 release):
[0001-winewayland.drv-Retire-nested-popup-roles-before-the.patch][patch].
xdg-shell requires the topmost popup to be destroyed first
([xdg-shell.xml#L1281-L1286][xdg-destroy]). The patch keeps a list of
live child popups for each surface and clears their roles before the
parent's, then asks their window threads to update them. It also destroys
a window's surface with the window data lock held; the current code
releases the lock first ([window.c#L101-L104][destroy-unlocked]). Without
that lock, the second program failed 9 of 25 runs.

GE-Proton has a patch for the same protocol error, included from
GE-Proton11-10: [b492349d][ge-fix] ([issue 834][ge-issue]).

If you want a pull request, I can open one.

[window-probe]: https://github.com/cheese-oss/dwproton-nikke-split/blob/7ab01306283af86c8ae50b4a96bec499adee7963/tests/window-probe.c
[race-probe]: https://github.com/cheese-oss/dwproton-nikke-split/blob/7ab01306283af86c8ae50b4a96bec499adee7963/tests/popup-race-probe.c
[patch]: https://github.com/cheese-oss/nikke-store-wayland/blob/2e6480c92e29deee3de579a6bc0782673c487289/patches/0001-winewayland.drv-Retire-nested-popup-roles-before-the.patch
[xdg-destroy]: https://gitlab.freedesktop.org/wayland/wayland-protocols/-/blob/1.49/stable/xdg-shell/xdg-shell.xml#L1281-1286
[destroy-unlocked]: https://dawn.wine/dawn-winery/wine-dwproton/src/commit/b75a4125963073b01d2b3bfce6060d697d02066b/dlls/winewayland.drv/window.c#L101-L104
[ge-fix]: https://github.com/GloriousEggroll/proton-ge-custom/commit/b492349d0233dcff26c5a585ddd3f2a4c8c3095c
[ge-issue]: https://github.com/GloriousEggroll/proton-ge-custom/issues/834

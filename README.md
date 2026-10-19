# NIKKE's in-game store on winewayland

Four Wine patches for DWProton that fix NIKKE's in-game store when the
game runs on the Wayland driver: the store stays white, closing it ends
the game, and its dropdowns close as soon as they open.

- [issue.md](issue.md): the issue, in DWProton's Compatibility Report
  template. It describes only the symptoms and how to reproduce them.
- [comment.md](comment.md): the first comment on the issue, with the
  tests, the code involved and the patches.
- [pull-request.md](pull-request.md): a pull request description, if the
  developers want one.
- [patches/](patches/): the series, in dependency order.

The first patch stands alone and fixes only the game exit when the store
closes; GE-Proton has a patch for the same protocol error. It has its own
drafts in [popup-order/](popup-order/):

- [popup-order/issue.md](popup-order/issue.md): an issue about the exit
  alone, in the same template, with the same log and system information.
- [popup-order/comment.md](popup-order/comment.md): its first comment,
  with the tests and the patch.

If that issue is posted first, the store drafts should refer to it for
the first patch instead of describing the exit again.

The patches apply to Wine `b75a4125963073b01d2b3bfce6060d697d02066b`, the
source of the `dwproton-11.0-15` release, on branch `dwproton/11.0-15` of
<https://dawn.wine/dawn-winery/wine-dwproton.git>:

```sh
git clone --branch dwproton/11.0-15 \
  https://dawn.wine/dawn-winery/wine-dwproton.git
cd wine-dwproton
git checkout b75a4125963073b01d2b3bfce6060d697d02066b
git am /path/to/patches/*.patch
```

## Before posting

Both issue drafts link to the same checked reports:

- [System information](https://gist.github.com/cheese-oss/2b7af4bf2970d160e033aa0fe4ce637b): the full inxi report.
- [DWProton 11.0-15 log](https://gist.github.com/cheese-oss/b2a8b517e6e772515fd74784990e3ca9): the redacted log with `trace:seh`
  lines removed (789 KB). Warnings, errors, DLL loads and game debug
  output are retained; the full log remains in the local `attachments/`
  directory.

Replace `#ISSUE` in `pull-request.md` with the issue number before posting.

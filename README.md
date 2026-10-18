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

The patches apply to Wine `f078dded59f559884b58fbf0769847e9c9fbb3ac` on
branch `dwproton/11.0-15` of
<https://dawn.wine/dawn-winery/wine-dwproton.git>:

```sh
git clone --branch dwproton/11.0-15 \
  https://dawn.wine/dawn-winery/wine-dwproton.git
cd wine-dwproton
git checkout f078dded59f559884b58fbf0769847e9c9fbb3ac
git am /path/to/patches/*.patch
```

## Before posting

DWProton's [AGENTS.md](https://dawn.wine/dawn-winery/dwproton/src/branch/main/AGENTS.md)
and issue templates ask for:

- a full system information report, attached to the issue as
  `nikke-sysinfo.txt`;
- a Proton log that shows the white store and the exit, attached to the
  issue as `nikke-proton-11.0-14.log.gz` (Forgejo does not accept `.xz`);
- `#ISSUE` in `pull-request.md` replaced with the issue number.

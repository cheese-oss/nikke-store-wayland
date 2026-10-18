# NIKKE's in-game store on winewayland

Four Wine patches for DWProton that fix NIKKE's in-game store when the
game runs on the Wayland driver: the store stays white, closing it ends
the game, and its dropdowns close as soon as they open.

- [issue.md](issue.md): the failures and how they arise.
- [pull-request.md](pull-request.md): the patches, their testing and
  known limits.
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

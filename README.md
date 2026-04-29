# Beat Saber Multiplayer Chat - Quest (QMOD)

Standalone **Quest** port of **[Multiplayer Chat](https://github.com/buttheadicus/BeatSaber-Multiplayer-Chat)** on PC (BSIPA / Zenject). Players install Quest mods as **`.qmod`** files. The usual install flow is **[ModsBeforeFriday (MBF)](https://mbf.bsquest.xyz/)** in a Chromium browser (see [BSMG wiki](https://bsmg.wiki/quest/modding-with-mbf.html)). **BMBF** is deprecated for current Quest modding. Matching PC multiplayer on the wire is a **goal**, not guaranteed initially.

## Status

Bootstrap only. See **[`docs/NEXT_STEPS.md`](docs/NEXT_STEPS.md)** for Git workflow, MBF vs BMBF, and how to bring in **[quest-mod-template](https://github.com/Lauriethefish/quest-mod-template)** when you add a real build.

The PC mod is **C#**; Beat Saber Quest mods are typically **C++** + codegen + `.qmod` packaging. Plan architecture before copying PC sources blindly.

## Repo

Keep the checkout path **without spaces**, e.g. `C:\Users\goria\BeatSaber-Multiplayer-Chat-Quest`.

## License

See `LICENSE`.

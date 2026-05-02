# Beat Saber Multiplayer Chat - Quest (QMOD)

Standalone **Quest** port of **[Multiplayer Chat](https://github.com/buttheadicus/BeatSaber-Multiplayer-Chat)** on PC (BSIPA / Zenject). Players install Quest mods as **`.qmod`** files. The usual install flow is **[ModsBeforeFriday (MBF)](https://mbf.bsquest.xyz/)** in a Chromium browser (see [BSMG wiki](https://bsmg.wiki/quest/modding-with-mbf.html)). **BMBF** is deprecated for current Quest modding. Matching PC multiplayer on the wire is a **goal**, not guaranteed initially.

## Documentation

Wire format, crypto, packet field order (LiteNetLib), and port checklist:

- **[docs/PORTING_DRAFT.md](docs/PORTING_DRAFT.md)** (canonical wire spec and port checklist)
- **[docs/NEXT_STEPS.md](docs/NEXT_STEPS.md)** (Git workflow, MBF, tool bootstrap)

## Status

Skeleton mod from **[quest-mod-template](https://github.com/Lauriethefish/quest-mod-template)** (`CMakeLists.txt`, `src/main.cpp`, `qpm.json`, `scripts/`). You still need **qpm**, **Android NDK**, and codegen matching your BS Quest build.

## Build

1. Install [qpm](https://github.com/QuestPackageManager/QPM).

2. Install **CMake** and **Ninja** (required by **`scripts/build.ps1`**):

   ```powershell
   winget install Kitware.CMake Ninja-build.Ninja
   ```

   Close and reopen the terminal so **`PATH`** includes them, then **`cmake --version`** and **`ninja --version`** should work.

3. **Android NDK:** either:

   - Set **`ANDROID_NDK_HOME`** in Environment Variables to your NDK root (folder containing `ndk-build.cmd` and **`toolchains`**), **or**

   - Create **`ndkpath.txt`** in this repo root: **one line**, same path, forward slashes only, e.g. `C:/Users/goria/AppData/Local/Android/Sdk/ndk/25.2.9519653`.

   You can install NDK via **Android Studio** (SDK Manager), or try **`qpm ndk`** (see **`qpm ndk --help`**).

4. From the repo root:

```powershell
qpm restore
qpm doctor
qpm s build
```

5. Produce **`MultiplayerChatQuest.qmod`**:

```powershell
qpm s qmod
```

6. Bump **`packageVersion`** in **`mod.json`** to match Beat Saber on your headset before distributing.

**Optional extras** (from **`qpm doctor`):** `qpm download adb` for deployment to device; CMake/Ninja can also use **`qpm download cmake`** / **`qpm download ninja`** if you prefer copies managed by qpm instead of **`winget`**.

## Repo path

Keep the checkout path **without spaces**, e.g. `C:\Users\goria\BeatSaber-Multiplayer-Chat-Quest`.

## License

See `LICENSE`.

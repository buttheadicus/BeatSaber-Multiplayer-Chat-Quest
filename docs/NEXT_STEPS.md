# Next steps (Quest port)

Use this as the working checklist for **developers** of this repo. End users who only install released `.qmod` files do not need most of this.

---

## A. Git and GitHub (you already published)

**Day to day**

```powershell
cd C:\Users\goria\BeatSaber-Multiplayer-Chat-Quest
git status
git add .
git commit -m "Your message"
git push
```

**`git fetch origin` is normal.** It only updates your view of what is on GitHub (`origin/main`, etc.). It does not merge into your branch or overwrite your files. If you did not run `git pull` or `git merge`, you do not need to undo anything.

**`git pull`** = `fetch` + merge (or rebase) into your current branch. That can change your local commits. If you only wanted to see remote changes without merging, use `fetch` and compare: `git log main..origin/main`.

**First-time setup (for anyone cloning fresh)**

```powershell
git clone https://github.com/YOUR_LOGIN/BeatSaber-Multiplayer-Chat-Quest.git
cd BeatSaber-Multiplayer-Chat-Quest
```

Use the HTTPS or SSH URL from the green **Code** button on GitHub. If you get **HTTP 400** or auth errors: paste the URL exactly, try **SSH** if HTTPS fails, and clear stale **github.com** credentials in Windows Credential Manager.

**If `origin` is missing**

```powershell
git remote add origin https://github.com/YOUR_LOGIN/BeatSaber-Multiplayer-Chat-Quest.git
git branch -M main
git push -u origin main
```

---

## B. How players install Quest mods (context for releases)

Today, **ModsBeforeFriday (MBF)** is the usual way to patch Beat Saber on Quest 2 / 3 / Pro in a browser. Official entry: [https://mbf.bsquest.xyz/](https://mbf.bsquest.xyz/). Guide: [BSMG wiki: ModsBeforeFriday](https://bsmg.wiki/quest/modding-with-mbf.html).

**BMBF** is deprecated for current Quest modding. Do not document BMBF as the install path for new users.

When this mod is ready, you will ship a **`.qmod`**; players add it through MBF (upload / mod browser) or whatever workflow the community uses at release time.

---

## C. Bootstrap the native mod project (developers)

This repo is **docs + planning** until you add the real build. The standard starting point is **[quest-mod-template](https://github.com/Lauriethefish/quest-mod-template)**:

1. Use the template (or copy its layout) into this repo or a subfolder, as you prefer.
2. Install **qpm** and run **`qpm restore`** when `qpm.json` changes.
3. Set **Android NDK** path in the project (use **forward slashes** in config where the template asks).
4. Run **`build.ps1`** to build, **`buildQMOD.ps1`** to produce a **`.qmod`** for testing.

**Extra reading**

- [Beat Saber Quest modding guide (danrouse)](https://github.com/danrouse/beatsaber-quest-modding-guide)
- [Porting / hooks / codegen context (Fernthedev)](https://github.com/Fernthedev/beatsaber-quest-porting-guide)

The PC Multiplayer Chat mod is **C# (BSIPA)**. Quest mods here are usually **C++**, **codegen**, and **`.qmod`**. Plan what you port (crypto, packets, UI) instead of copying the PC tree wholesale.

---

## D. Parity with PC Multiplayer Chat

High-level goals and migration order live in the **PC** repository:

`BeatSaber-Multiplayer-Chat/BeatSaber-Multiplayer-Chat-Quest.md`

Update that file when you lock wire format, networking, or UI decisions for Quest.

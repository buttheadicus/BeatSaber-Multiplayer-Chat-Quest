# What to do next

Do these in order unless you already satisfy a step.

## 1) Confirm Git sees this folder

Open PowerShell in `C:\Users\goria\BeatSaber-Multiplayer-Chat-Quest`:

```powershell
git status
git remote -v
```

## 2) Create the GitHub repo (if not done)

On GitHub: **New repository** → name matches this folder → create **without** initializing README (you already have a local `.git`).

Then attach the remote:

```powershell
git remote add origin https://github.com/YOUR_LOGIN/BeatSaber-Multiplayer-Chat-Quest.git
git branch -M main
git add .
git commit -m "Docs and bootstrap for Quest port"
git push -u origin main
```

Use your real URL (HTTPS above, **or** `git@github.com:YOUR_LOGIN/BeatSaber-Multiplayer-Chat-Quest.git` for SSH).

**If clone/push fails with HTTP 400:** usually bad URL pasted, SSO/org rules, or stale credentials — copy the HTTPS URL exactly from GitHub → Code → HTTPS; clear old `github.com` entries in Windows Credential Manager if needed.

## 3) Bootstrap the Quest mod project

Follow **[quest-mod-template](https://github.com/Lauriethefish/quest-mod-template)** (extract / **qpm restore**, **NDK** path with **forward slashes**, then **`build.ps1`** / **`buildQMOD.ps1`**).

Pointers:

- [Beat Saber Quest modding guide](https://github.com/danrouse/beatsaber-quest-modding-guide)

- Porting context: [beatsaber-quest-porting-guide](https://github.com/Fernthedev/beatsaber-quest-porting-guide)

## 4) Parity checklist (high level)

Tracked in the **PC repo** under `BeatSaber-Multiplayer-Chat/BeatSaber-Multiplayer-Chat-Quest.md`.

# Next Steps and Project Memory: Shinobi workspace

Last updated: 2026-11-25
Branch: `claude/keen-ride-nf44vn` (latest commit `15f073d`)
Live demo (private artifact): https://claude.ai/artifact/JBqaLW8gzYFJwkox6x5f4E

## 1. Status snapshot

| Item | State |
|---|---|
| `index.html` (repo root) | Legacy Phaser 3 "Shinobi" web game from the earlier session. Not changed in this session. |
| `ember-veil/index.html` | New original game, single file, plain Canvas 2D, no dependencies. Two missions, both playable end to end. |
| DOS files (`*.GRF`, `*.CTL`, `Go.bat`, `TT.DAT`) | Third-party assets of the original arcade port. Not used by any web game. Do not redistribute or copy from them. |
| Drive project folder | `Shinobi` (id `1nQKrgNeMKAb3iDN7bmYrj3EX5kPpyzVs`). Unity-era tree plus `shinobi-web/index.html` (a copy of the legacy game). |

## 2. Decisions and constraints (read before continuing)

1. The reference video (`WhatsApp Video 2026-05-05 at 06.40.42.mp4`, Drive folder `1E21UVAAHwdz0tg16TOd_c8V2AlhGpGbz`, 28.6 s, 1024x576) is a phone recording of a laptop playing Sega's Shinobi.
2. The user asked for the same visuals, actors and everything. A recreation of that specific game was declined. The user chose "Original ninja game": same genre and mechanics, new name, hero, enemies, bosses and art. Keep it that way. Never copy Sega characters, sprites, music or levels.
3. Drive's text reader returns nothing for the video. What worked: `download_file_content` saves the mp4 to a local file, then `ffmpeg -vf "fps=1/3,scale=640:-1,tile=4x3"` makes a contact sheet.
4. What the video shows: a "MISSION 1" title card, a boss dossier card (file number, portrait, city map), "PLAYER 1 READY", a HUD with `1P`, score, `HI`, lives icons, `SAVE` hostage counter and a 3:00 timer, a daytime city street with balconies as the upper floor, crates, enemies on both floors and a hostage on a balcony.
5. User style preferences: answer first, concise, no filler, end with a conclusion or recommendation, state assumptions, never claim something was verified when it was not.
6. Artifact publishing needs the page without `<!DOCTYPE>/<html>/<head>/<body>` wrapper tags. The deployed copy is produced with `grep -v -E '^<!DOCTYPE|^<html|^</html|^<head>|^</head>|^<body>|^</body>|^<meta ' ember-veil/index.html`.

## 3. Ember Veil: what exists

Controls: arrows or WASD move (Down = crouch), X / Space / Up jump (Down + jump drops through a balcony), Z attack, C Ember Bloom (once per mission), Enter to start or continue. Touch buttons appear on touch devices (`?touch=1` forces them).

Mechanics: attack is automatic by distance (slash within 54 px, otherwise a kunai, max 3 in flight). One hit kills the hero. 3 lives. 3:00 timer per mission. HI score is stored in `localStorage` under `ev_hi`.

| | Mission 1: The Ember Docks | Mission 2: The Lantern Market |
|---|---|---|
| Scene | Daytime city street, balconies | Same layout at night, moon, lantern strings |
| Apprentices | 5 | 6 |
| Enemies | Thug, crossbow scout, shield monk, leaper | Same types, denser |
| Boss | Iron Crane (26 HP): walk, chain hook (crouch), stomp with shockwaves (jump) | Twin Masks, 14 HP each |

Twin Masks: one twin attacks (Sun: telegraphed dash plus ground wave; Moon: chain at head height) while the other guards and reflects kunai, so it must be slashed. Every 4 to 5.5 s they swap with a crossing dash. When one dies the survivor enrages (1.4x speed, both attacks, no guard).

Flow: title, mission card, boss dossier, PLAYER 1 READY, play, mission clear (time bonus x10), next mission, final "ALL MISSIONS CLEAR". Game over restarts from Mission 1.

URL options: `?auto=1` autopilot (implies invulnerability), `?speed=N` (up to 8), `?god=1`.

## 4. Code map (`ember-veil/index.html`)

| Where | What |
|---|---|
| `STAGES`, `PAL_DAY`, `PAL_NIGHT`, `applyStage()` | All per-mission data: platforms, hostages, spawns, checkpoints, palette, boss kind and name. Add a mission by adding an entry. |
| `stageState()`, `newGame()`, `loadStage()` | Run state versus per-mission state. |
| `updatePlayer()`, `meleeHit()`, `dmgEnemy()`, `dmgBoss()` | Player and combat. |
| `updateEnemy()` | AI for thug, monk, scout, leaper. |
| `spawnCrane/updateCrane`, `spawnTwins/updateTwins`, `updateBosses()` | Bosses. `S.bosses` is an array; the mission ends when every boss is dead for 2.4 s. |
| `drawBackground/drawFacade/drawLanternString/drawGround/drawPlatforms` | Palette-driven scene. |
| `drawHero/drawEnemy/drawBoss/drawTwinBody` | Sprites drawn from rectangles. |
| `drawHUD/drawIntro/drawEnd/drawTitle` | HUD and screens. |
| `autopilot()` | Test bot. Not a player model. |

## 5. Verification record

Verified: `node --check` on the script; 8 headless Chromium autopilot runs through both missions to the final victory screen, all with zero console errors; screenshots reviewed for the Mission 1 street, Mission 2 street, the Twin Masks guard pose, dash afterimages, boss bars and both dossiers.

Not verified: difficulty and dodge timing by a human player (dodge geometry was only checked by calculation); the Mission 2 title card, the mission-clear screen and the final victory screen were not looked at; touch controls were not tested on a real device; audio was not listened to.

Known limits: the autopilot runs invulnerable and does not dodge; all art and audio are placeholders generated in code.

## 6. Backlog (recommended order)

1. Human playtest of both missions, then tune enemy speeds, boss HP and spawn density.
2. Check the three unreviewed screens and touch controls on a phone.
3. Mission 3 (one more `STAGES` entry plus a new boss kind).
4. Real music and sound effects; sprite sheets instead of rectangles.
5. Title screen options, pause, loading screen.
6. Deployment: host as an itch.io HTML game; optional Capacitor wrapper for mobile.

Legacy Phaser game backlog (from the 2026-05-05 notes) is still valid for the root `index.html`: sprite animations, music and SFX, ranged attack, power-ups, visual polish, web deployment and mobile packaging.

## 7. Quick commands

```bash
# run locally
python3 -m http.server 8080          # then open /ember-veil/index.html
# syntax check the script block
sed -n '/<script>/,/<\/script>/p' ember-veil/index.html | sed '1d;$d' > /tmp/check.js && node --check /tmp/check.js
# full automated playthrough (Playwright is preinstalled in the cloud container)
#   open /ember-veil/index.html?auto=1&speed=8 and poll window.__g.mode until 'victory'
```

## 8. Drive housekeeping

- The connector's `update_file` changes metadata only and cannot overwrite file contents, and the original `next_steps.md` is not editable by it. This file therefore lives in the repo and is uploaded to the Shinobi folder as `next_steps_2026-11-25.md`. The old `next_steps.md` is left as historical.
- Unity-era files are still outdated: the `PROJECT_SUMMARY_*.md` files, `QA_CHECKLIST.md`, `COPILOT_INSTRUCTIONS.md`, the `Assets/` folder and the Unity import paths in `Docs/PLACEHOLDER_ASSETS.md`.

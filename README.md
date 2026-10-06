# FeedJack

Lightweight, cartoon style feedback game for team retrospectives. One host screen (projector), up to 8 players on their phones, 5 anonymous questions. German UI.

## Files

| File | Purpose |
|---|---|
| `src/template.html` | App template (markup, CSS, JS, Firebase config block) |
| `src/data.json` | Questions, limits, avatars, bot names. Build time only, baked into the app |
| `src/version.json` | Version tag. Bump on every delivery |
| `database.rules.json` | Explicit Firebase Realtime Database rules. Paste into the console |
| `tools/build.js` | Builds `dist/index.html`, fails loudly on guard violations |
| `tools/verify.js` | Verification battery incl. `node --check` of every inline script |
| `tools/test/` | Headless end to end test with a mock backend (test only, never shipped) |
| `dist/index.html` | The deliverable. This single file goes to GitHub Pages |

## Build and verify

Run each as its own command:

```
node tools/build.js
node tools/verify.js
python tools/test/run_tests.py
```

## How a round works

1. Host opens the app on the projector laptop, clicks **Ich bin Host**. A 4 letter room code and join link appear.
2. Players open the link (or type the code), enter a name (max 16 characters).
3. Host clicks **Los geht's!**. The player list is frozen into the roster at this moment (late joiners are turned away with E08).
4. For each question: players vote, host sees who has answered (not what), host clicks **Auflösen**, results show on the projector only.
5. After the last question: **Siegerehrung** summary.

Only the host writes the game state. Players only write their own join entry and their own answers. The host computes all results.

## Test bots (host only)

In the host lobby, with no input field focused:

* `b` fills the room with bots up to capacity (8).
* `Shift+B` adds 2 more than capacity (margin test, shows "Platz voll", roster is capped at 8).

Bots answer with random delay and sometimes skip a question. Never use bots in a real session.

## Error codes

| Code | Meaning |
|---|---|
| E01 | Firebase SDK did not load (network, CDN blocked) |
| E02 | Firebase config missing, placeholder, or databaseURL does not match region |
| E03 | No connection to Firebase within 9 s |
| E04 | Room could not be created |
| E05 | Join failed |
| E06 | Room code does not exist |
| E07 | Room full |
| E08 | Round already running |
| E09 | Answer could not be saved |
| E10 | Host could not save game state |
| E11 | Name missing |
| E12 | Resume host room failed |
| E13 | Internal data error (details in console) |
| E14 | Test bot write failed |
| E15 | Host could not read answers |
| E16 | Live listener cancelled (usually rules or permissions) |
| E17 | Player could not load question |

## Firebase setup (manual, we do this together)

1. https://console.firebase.google.com : **Create a project** : name `feedjack` : Google Analytics off : **Create project**.
2. Left menu **Build** : **Realtime Database** : **Create Database** : location **Belgium (europe-west1)** : **Start in locked mode** : **Enable**.
3. Tab **Rules** : select all : paste the full content of `database.rules.json` : **Publish**. Never use test mode.
4. Gear icon next to **Project Overview** : **Project settings** : tab **General** : section **Your apps** : web icon `</>` : nickname `feedjack-web` : do not tick Firebase Hosting : **Register app** : copy the `firebaseConfig` object.
5. Check that `databaseURL` is in the copied config. If it is missing, copy the URL shown at the top of **Realtime Database** : tab **Data**. For Belgium it looks like `https://<project>-default-rtdb.europe-west1.firebasedatabase.app`.
6. Send the config to Copilot. It goes into the marked block in `src/template.html`, then build and verify run again.

The apiKey in a web config is not a secret. Access is controlled by the rules.

## GitHub Pages setup (manual)

1. https://github.com : **New repository** : name `feedjack` : **Public** : **Create repository**.
2. **Add file** : **Upload files** : drop `dist/index.html` : **Commit changes**.
3. **Settings** : **Pages** : Source **Deploy from a branch** : Branch `main`, folder `/ (root)` : **Save**. After 1 to 2 minutes the app is at `https://<user>.github.io/feedjack/`.

Updating later: open `index.html` in the repo : pencil icon : select all : paste new file : **Commit changes**.

## Live vs. staging

Keep a second file `next.html` in the same repo for new versions. Test there with bots, copy to `index.html` only after a full test run, and never on the day of use.

## Cleanup

Rooms stay in the database. Delete them in **Realtime Database** : tab **Data** : hover `rooms` : trash icon.

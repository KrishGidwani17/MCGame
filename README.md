# ⛏ Minecraft Challenge Night

A two-player Minecraft challenge scoreboard with a casino-style challenge roulette. Play "first one to do X wins the point" games with a friend, with live shared scores on two devices.

**Live site:** https://krishgidwani17.github.io/MCGame/

## Features

- Slot-machine roulette that picks a random challenge each round (no repeats until the pool is used up)
- First-to-N scoring with a per-round timer
- Live sync between two devices using shared room codes
- Game history and all-time win tally
- Skip/respin, undo last point, and end-game-early controls
- Add your own custom challenges
- Minecraft-style UI (dirt background, grass header, blocky panels)

## How to play

1. Open the site and click **Create New Room** (or type your own room code and click **Join Room**).
2. Send the invite link to your partner. They join automatically.
3. Enter both names, choose the target score, and click **Start New Game**.
4. The roulette spins and lands on a challenge. Race to do it in Minecraft.
5. Whoever finishes first taps their **got it** button. First to the target score wins.

## Tech stack

- Vanilla HTML, CSS and JavaScript in a single `index.html` (no framework, no build step)
- Hosted on GitHub Pages
- Firebase Realtime Database (compat SDK v10.12.2, loaded from `gstatic.com`) for live sync
- Google Fonts "Press Start 2P" as a fallback pixel font, with an optional local `fonts/Minecraft.ttf`
- Browser `localStorage` for single-device mode

## Project structure

```
index.html        the whole app (markup, styles, script)
fonts/
  Minecraft.ttf   optional, used automatically if present
README.md
```

## How it works

**State.** All shared state is one JSON object:

```
rooms/<CODE> = {
  p1, p2, target,
  game: {
    id, p1, p2, target, s1, s2, round,
    used[], rounds[{ t, winner, secs }],
    startedAt, finished, winner,
    current: { t, d, spin },
    roundStart
  },
  history[],
  custom[]
}
```

**Sync.** Each client subscribes to `rooms/<CODE>` with `.on('value')`. Every change is made through `roomRef.transaction()` (compare-and-swap), so simultaneous actions can't double-score. Each award checks the round number and spin ID, and is dropped if the round has already moved on.

**Roulette.** The challenge is chosen when the round starts and stored with a unique `spin` ID and a `roundStart` timestamp set `SPIN_MS` (4.6s) in the future. Each client animates the reel when it sees a new spin ID, and the "got it" buttons stay disabled until `roundStart` passes.

**Local mode.** With no Firebase config, the same mutation code runs against `localStorage`.

**Firebase quirks handled in code.** Empty arrays and nulls are dropped by Realtime Database, so `normalize()` rebuilds them on read. `undefined` values throw on write, so `clean()` does a JSON round trip before saving.

## Setup and deployment

### GitHub Pages
1. Create a public repo and upload `index.html` (and the `fonts` folder if used).
2. Go to **Settings → Pages**, choose **Deploy from a branch**, select `main` and `/ (root)`, then save.
3. The site appears at `https://<username>.github.io/<repo>/`.

### Firebase sync (optional)
1. Create a project at console.firebase.google.com.
2. Create a **Realtime Database** (not Firestore) and start in test mode.
3. Register a web app under **Project settings → Your apps** and copy the config.
4. Paste the values into the `FIREBASE_CONFIG` block at the top of the script in `index.html`. `apiKey` and `databaseURL` are required.
5. Replace the test-mode rules (they expire) under **Realtime Database → Rules**:

```json
{
  "rules": {
    "rooms": {
      "$room": {
        ".read": true,
        ".write": true
      }
    }
  }
}
```

### Font
Download a Minecraft-style font such as Minecraftia, rename it `Minecraft.ttf`, and put it at `fonts/Minecraft.ttf`. Check the font's license before publishing.

## Customizing

- **Player colors:** edit `--p1` and `--p2` in the `:root` CSS block.
- **Challenges:** edit the `BUILT_IN` array in the script, or add challenges in the UI.
- **Spin length:** change `SPIN_MS`.

## Security notes

- The Firebase web API key is public by design. It identifies the project and is not a secret. Access is controlled by the database rules.
- The rules above are open per room. Anyone who knows a room code can read and write that room. Use a hard-to-guess code (type your own, up to 8 characters) instead of a short generated one.
- There are no user accounts or personal data. Use nicknames.
- User-entered text is rendered with `textContent`, not `innerHTML`.
- Data from the database is not schema-validated, so a malicious writer with a room code could corrupt that room.
- Optional hardening: restrict the API key to your site's domain in Google Cloud Console (APIs & Services → Credentials), or add Firebase Anonymous Auth with stricter rules.
- Stay on the free Spark plan so usage can't be billed.

## Limitations

- Scores are visible to anyone with the room code.
- Free Firebase limits apply. This app uses a tiny fraction of them.
- Simultaneous edits are last-write-wins, except scoring, which uses transactions.

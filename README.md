# ⛏ Minecraft Challenge Night

A Minecraft challenge game with a casino-style challenge roulette. Play solo, or race a friend to finish challenges first ("bake a cake", "catch a pufferfish", and more). Multiplayer is direct browser-to-browser, so there is no server, no account, no API key, and nothing is stored.

**Live site:** https://krishgidwani17.github.io/MCGame/

## Features

- Slot-machine roulette that picks a random challenge each round (no repeats until the pool is used up)
- **Single Player:** complete N challenges as fast as you can, with a per-round timer and a total time
- **Multiplayer:** host a room, share a 6-digit code or invite link, and race to a target score
- Skip/respin, undo last point, end game early, and Play Again
- Add your own challenges (host or solo only, session only)
- Minecraft-style UI (dirt background, grass header, blocky panels)
- No storage: names, scores and history exist only while the page is open

## How to play

**Single player:** enter your name and a goal, then click **Single Player**.

**Multiplayer:**
1. The host enters a name and clicks **Host Multiplayer**. A 6-digit code appears.
2. The host sends the code or the invite link to a friend.
3. The friend opens the link (or types the code), enters a name, and clicks **Join Game**.
4. The host sets the goal ("first to") and clicks **Start Game**.
5. The roulette spins for both players. When you finish the challenge in Minecraft, press **I got it!**. The first to reach the target wins.
6. After a win, either player can click **Play Again**.

## Tech stack

- Vanilla HTML, CSS and JavaScript in a single `index.html` (no framework, no build step)
- Hosted on GitHub Pages
- [PeerJS](https://peerjs.com) 1.5.4 (loaded from jsDelivr) for WebRTC peer-to-peer connections
- Google Fonts "Press Start 2P" as a fallback pixel font, with an optional local `Minecraft.woff2` / `Minecraft.ttf`

## Project structure

```
index.html          the whole app (markup, styles, script)
Minecraft.ttf       optional font, used automatically if present (or Minecraft.woff2)
README.md
```

## How it works

**Peer-to-peer.** The two browsers connect directly using WebRTC. The free public PeerJS broker only introduces them (signaling); game data does not pass through it. The room ID is `mcgame-chal-<6 digits>`. The broker rejects an ID that is already in use, which keeps live room codes unique.

**Host-authoritative.** The host's browser holds the game state and runs all the logic. The guest never modifies state directly. It sends requests and the host validates them:

- guest to host: `hello` (name), `ping`, and `act` with one of `claim`, `respin`, `undo`, `end`, `new`
- host to guest: `state` (a full snapshot after every change), `pong`, `full`

Each `claim` includes the round number and spin ID, so stale or duplicate clicks (for example both players pressing at once) are ignored. `new` is only accepted once a game is over.

**Roulette.** When a round starts, the host picks the challenge and stores it with a unique `spin` ID and a `roundStart` time `SPIN_MS` (4.6s) in the future. Both browsers animate the reel when they see a new spin ID, and the buttons stay disabled until it lands. The guest shifts the host's timestamps onto its own clock to correct for clock differences.

**Connection handling.** Each side sends a heartbeat every 4 seconds. The host frees the guest's seat after 12 seconds of silence (so a refreshed guest can rejoin), and the guest leaves the room after 15 seconds without hearing from the host.

## Setup and deployment

1. Create a public GitHub repo and upload `index.html` (and the font file, if you have one).
2. Go to **Settings → Pages**, choose **Deploy from a branch**, select `main` and `/ (root)`, and save.
3. Your site appears at `https://<username>.github.io/<repo>/` after a minute or two.

There are no keys or configuration to set up.

### Font (optional)
Download a Minecraft-style font such as Minecraftia, rename it `Minecraft.ttf`, and put it next to `index.html`. Check the font's license before publishing.

## Customizing

- **Player colors:** edit `--p1` and `--p2` in the `:root` CSS block.
- **Challenges:** edit the `BUILT_IN` array in the script, or add challenges in the UI.
- **Spin length:** change `SPIN_MS`.

## Security and privacy

- **No secrets and no storage.** There is no API key, database, cookie, or `localStorage`. Everything lives in memory and disappears when the page closes.
- **Room codes** are exactly 6 digits, generated with `crypto.getRandomValues`. A room exists only while the host's tab is open and accepts one guest at a time. The host sees who joined and can remove them.
- **Input handling:** the invite link is accepted only if it is exactly 6 digits. Names and challenge text are stripped of control characters and `<>` and length-limited. All text from other people is inserted with `textContent`, never `innerHTML`.
- **Untrusted network data:** messages are checked against a whitelist and size/rate limits. The guest can only send five kinds of requests, and the host rebuilds and validates every one. The guest also rebuilds each state snapshot from known fields with strict types and limits.
- **Content Security Policy:** a meta tag restricts scripts to the page itself and jsDelivr, and network connections to the PeerJS broker.
- **Known limits:**
  - A 6-digit code has 1,000,000 possibilities. Share it privately and remove any unexpected player.
  - WebRTC reveals each player's IP address to the other player, and the PeerJS broker sees IPs and the room code. Only play with people you trust.
  - The PeerJS broker is a free community service with no uptime guarantee (single player still works).
  - PeerJS loads from a CDN at a pinned version. For maximum control, download `peerjs.min.js` into the repo and reference it locally (and update the CSP accordingly).
  - If the host loses connection or closes the tab, the game ends for both players.

## Troubleshooting

- **"No open room with that code":** the host must keep their tab open, and the code must be typed exactly.
- **"That room already has two players":** the host can click Remove Player, or the guest can wait up to about 12 seconds after a disconnect and rejoin.
- **Can't connect on some networks:** strict corporate or carrier networks can block WebRTC. Try another network or a phone hotspot.
- **Changes don't show after editing:** hard refresh (Ctrl/Cmd + Shift + R) and wait 1-2 minutes for GitHub Pages.
- **Something stopped working after adding a library or service:** check the Content Security Policy meta tag in `<head>`.

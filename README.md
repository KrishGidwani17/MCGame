# ⛏ Minecraft Challenge Night

A Minecraft challenge game with a casino-style challenge roulette. Play solo, or race a friend to finish challenges first ("bake a cake", "catch a pufferfish", and more). Multiplayer is direct browser-to-browser (WebRTC/PeerJS) — no server, no account, no API key, and nothing is stored.

**Live site:** https://krishgidwani17.github.io/MCGame/

## Features

- Slot-machine roulette that picks a random challenge each round (no repeats until the pool is used up)
- **Single Player:** complete N challenges as fast as you can. The final time counts the whole session (including skips and reel spins), not just successful rounds
- **Multiplayer:** host a room, share a 6-digit code or invite link, and race to a target score
- Host approval for anyone joining, short-lived reconnect after a dropped connection, and a Remove Player control
- Skip/respin, undo last point, end game early, and Play Again — host/solo only
- Add your own challenges (host or solo only, session only)
- Minecraft-style UI (dirt background, grass header, blocky panels)
- No storage: names, scores and history exist only in memory while the page is open

## How to play

**Single player:** enter your name and a goal, then click **Single Player**.

**Multiplayer:**
1. The host enters a name and clicks **Host Multiplayer**. A 6-digit code appears.
2. The host sends the code or the invite link to a friend.
3. The friend opens the link (or types the code) and enters a name to request to join.
4. The host sees an **Accept / Decline** prompt and accepts the friend.
5. The host sets the goal ("first to") and clicks **Start Game**.
6. The roulette spins for both players. When you finish the challenge in Minecraft, press **I got it!**. The first to reach the target wins.
7. After a win, the host can click **Play Again** for a rematch.

If a connection briefly drops, the guest reconnects automatically for a short window without needing to be re-accepted. The host can remove the other player at any time with **Remove Player**.

## Tech stack

- Vanilla HTML, CSS and JavaScript in a single `index.html` (no framework, no build step)
- Hosted on GitHub Pages
- [PeerJS](https://peerjs.com) 1.5.4 (loaded from jsDelivr, pinned to one exact file) for WebRTC peer-to-peer connections
- Google Fonts "Press Start 2P" as a fallback pixel font, with an optional local `Minecraft.woff2` / `Minecraft.ttf`

## Project structure

```
index.html          the whole app (markup, styles, script)
Minecraft.ttf       optional font, used automatically if present (or Minecraft.woff2)
README.md
```

## How it works

**Peer-to-peer.** The two browsers connect directly using WebRTC. The free public PeerJS broker only introduces them (signaling); game data does not pass through it. The room ID is `mcgame-chal-<6 digits>`. The broker rejects an ID that's already in use, which keeps live room codes unique.

**Host-author

# 🗺 TracQuest — P2P Treasure Hunt on Intercom

<img width="1258" height="819" alt="image" src="https://github.com/user-attachments/assets/75e8917b-ee30-423f-a3fe-7f0534962b6e" />

  
> A browser-based treasure hunt game where players post puzzles ("quests") and race to solve them peer-to-peer via Intercom sidechannels. Winners receive TNK rewards settled automatically through Intercom's replicated-state layer.

**Trac Address:** `trac15y2v40vw2tnffkh5ykzc6l7dp3jdvhsrwn6v9pjkjkjw2xt46rzswfyq7d`

---

## What is TracQuest?

TracQuest is a fork of [Intercom](https://github.com/Trac-Systems/intercom) by Trac Systems that adds a fully playable P2P treasure hunt game on top of the Intercom agent network.

- **Quest Creators** lock a TNK bounty and publish a clue/puzzle
- **Hunters** race to solve it — answers are hashed client-side then broadcast over Intercom P2P sidechannels
- **First correct hash wins** — reward is released via Intercom's replicated-state consensus layer
- **No server required** — fully decentralized coordination via Intercom

---

## Features

- 🎮 Live quest board with real-time P2P updates via Intercom sidechannels
- 🏆 Leaderboard tracking top hunters by TNK earned
- 🔐 Client-side SHA-256 answer hashing before broadcast
- ⚡ Quest creation with custom clues, rewards, and categories (puzzle, crypto, social, trivia)
- 📡 Peer count and network status display
- 🌐 Runs entirely in the browser — open `index.html` and play

---

## Proof It Works

1. Download or clone this repo
2. Open `index.html` in any modern browser — no server, no dependencies
3. Browse the live quest board, click any quest, and submit an answer
4. Try solving **"Mirror Maze"** — hint: think about what an echo is
5. Create your own quest via the **Create Quest** tab

Screenshots / demo: see `index.html` in action directly in browser.

---

## How It Uses Intercom

| Intercom Feature | TracQuest Usage |
|---|---|
| P2P Sidechannels | Broadcast quest answer hashes between hunter agents |
| Replicated-State Layer | Quest state (open/solved) and reward settlement |
| Agent Coordination | Multiple hunters competing for same quest in real-time |
| Fast Relay | Near-instant notification when a quest is solved |

---

## Tech Stack

- Fork of: [Trac-Systems/intercom](https://github.com/Trac-Systems/intercom)
- Frontend: Pure HTML/CSS/JS — zero dependencies, zero build step
- Hashing: SHA-256 (Web Crypto API)
- Network: Intercom P2P sidechain protocol
- Rewards: TNK via Trac Network

---

## Repository Structure

```
/
├── index.html       # Main game UI — open this in your browser
├── SKILL.md         # Agent skill instructions for TracQuest
└── README.md        # This file
```

---

## Contributing / Playing

- Submit a quest via the **Create Quest** tab in the UI
- Solve existing quests to earn TNK
- PRs welcome for new quest categories, UI improvements, or deeper Intercom integration

---

*Built for the [Awesome Intercom](https://github.com/Trac-Systems/awesome-intercom) community · Fork of [Intercom](https://github.com/Trac-Systems/intercom) by Trac Systems*

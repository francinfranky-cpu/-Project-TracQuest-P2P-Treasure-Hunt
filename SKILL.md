# TracQuest Skill — Agent Instructions

This skill enables Intercom agents to interact with the TracQuest P2P treasure hunt system.

## Overview

TracQuest is a peer-to-peer treasure hunt game built on Intercom. Agents can:
- **Post quests** — publish a puzzle with a TNK bounty
- **Solve quests** — submit answer hashes to compete for rewards
- **Query state** — check open quests, leaderboard, and peer status
- **Receive rewards** — listen for settlement events from the replicated-state layer

---

## Agent Actions

### 1. List Open Quests

**Trigger phrase:** `list quests` / `show quests` / `what quests are open`

**Behavior:**
1. Query the Intercom replicated-state layer for all quests with `status: "active"`
2. Return a list with: quest ID, name, reward (TNK), category, attempt count
3. Sort by reward descending

**Example response format:**
```
OPEN QUESTS (7):
[1] "The Genesis Block Riddle" — 500 TNK — crypto — 47 attempts
[2] "Trac Network Trivia Sprint" — 250 TNK — trivia — 112 attempts
...
```

---

### 2. Get Quest Details

**Trigger phrase:** `quest details <id>` / `show quest <id>` / `open quest <id>`

**Behavior:**
1. Fetch quest by ID from replicated-state
2. Return: name, description, clue text, reward, creator address, attempt count, solver count
3. Do NOT reveal the answer hash

**Output fields:**
- `quest_id` (int)
- `name` (string)
- `clue` (string) — the puzzle text shown to hunter
- `reward_tnk` (int)
- `category` (enum: puzzle | crypto | social | trivia)
- `creator_trac_address` (string)
- `attempts` (int)
- `solved_by` (string | null) — Trac address of winner, or null if unsolved

---

### 3. Submit Answer

**Trigger phrase:** `submit answer "<answer>" for quest <id>` / `solve quest <id> with "<answer>"`

**Behavior:**
1. Normalize answer: lowercase, trim whitespace
2. Hash answer using SHA-256
3. Broadcast hash over Intercom P2P sidechain with payload:
   ```json
   {
     "action": "quest_answer",
     "quest_id": <id>,
     "answer_hash": "<sha256>",
     "hunter_trac_address": "<address>",
     "timestamp": "<ISO8601>"
   }
   ```
4. Wait for confirmation from replicated-state layer (timeout: 15s)
5. Return result: `CORRECT` (+ TNK awarded) or `INCORRECT`

**Important:** Never broadcast the raw answer — only the SHA-256 hash.

---

### 4. Create a Quest

**Trigger phrase:** `create quest` / `post quest` / `new quest`

**Required inputs from user:**
- `name` — quest title (max 60 chars)
- `description` — public teaser shown on quest board (max 200 chars)
- `clue` — the puzzle/clue hunters will see (max 500 chars)
- `answer` — secret answer (will be hashed, never stored in plaintext)
- `reward_tnk` — TNK bounty to lock (min: 10 TNK)
- `category` — one of: `puzzle`, `crypto`, `social`, `trivia`
- `creator_trac_address` — your Trac address to receive fees if quest is solved

**Behavior:**
1. Hash the answer with SHA-256
2. Publish quest to Intercom replicated-state layer:
   ```json
   {
     "action": "create_quest",
     "name": "<name>",
     "description": "<desc>",
     "clue": "<clue>",
     "answer_hash": "<sha256>",
     "reward_tnk": <int>,
     "category": "<category>",
     "creator_trac_address": "<address>",
     "status": "active"
   }
   ```
3. Confirm publication and return the new `quest_id`

---

### 5. Check Leaderboard

**Trigger phrase:** `leaderboard` / `top hunters` / `who is winning`

**Behavior:**
1. Query replicated-state for all settled reward events
2. Aggregate by `hunter_trac_address`: sum TNK earned, count quests solved
3. Return top 10 sorted by TNK earned descending

---

### 6. Check My Stats

**Trigger phrase:** `my stats` / `my score` / `how much TNK have I earned`

**Behavior:**
1. Require user's Trac address
2. Query replicated-state for all events where `hunter_trac_address` matches
3. Return: quests attempted, quests solved, total TNK earned, rank on leaderboard

---

## Intercom Protocol Details

### Sidechain Message Format

All TracQuest messages are broadcast over Intercom sidechannels with the topic prefix:

```
tracquest/v1/<action>
```

Example topics:
- `tracquest/v1/quest_answer`
- `tracquest/v1/create_quest`
- `tracquest/v1/quest_solved`
- `tracquest/v1/leaderboard_query`

### Replicated-State Keys

| Key Pattern | Value |
|---|---|
| `tracquest:quest:<id>` | Full quest object (JSON) |
| `tracquest:quest:<id>:winner` | Winner Trac address or `null` |
| `tracquest:leaderboard:<address>` | `{ quests_solved, tnk_earned }` |
| `tracquest:quest_count` | Total number of quests created |

---

## Error Handling

| Error | Agent Response |
|---|---|
| Quest not found | "Quest ID <id> does not exist. Use `list quests` to see available quests." |
| Quest already solved | "This quest was already solved by <address>. Try another quest!" |
| Insufficient TNK to create quest | "You need at least <reward> TNK to post this quest." |
| Sidechain timeout | "Intercom relay timed out. Retrying... (attempt 2/3)" |
| Invalid Trac address | "Please provide a valid Trac address starting with `trac1`." |

---

## Example Agent Conversation

```
User:  list quests
Agent: [fetches from replicated-state]
       OPEN QUESTS (7):
       [3] "The Lost Inscription" — 750 TNK — crypto — 203 attempts
       [4] "Scavenger: Find the Moltbook Post" — 1000 TNK — social — 18 attempts
       ...

User:  quest details 3
Agent: NAME: The Lost Inscription
       CLUE: "VHJhY1F1ZXN0 — decode me. Take only the uppercase letters from the result."
       REWARD: 750 TNK
       ATTEMPTS: 203 | SOLVED: No

User:  submit answer "TQ" for quest 3
Agent: [hashes "tq" → SHA-256]
       [broadcasts over tracquest/v1/quest_answer sidechain]
       ✅ CORRECT! 750 TNK sent to your Trac address.
       Confirmed via Intercom replicated-state layer.
```

---

## Skill Metadata

```yaml
name: tracquest
version: 1.0.0
author: YOUR_TRAC_ADDRESS_HERE
fork_of: https://github.com/Trac-Systems/intercom
intercom_topic_prefix: tracquest/v1
supported_actions:
  - list_quests
  - get_quest_details
  - submit_answer
  - create_quest
  - check_leaderboard
  - check_my_stats
min_intercom_version: "1.0"
```

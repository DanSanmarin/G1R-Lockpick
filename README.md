# G1R Lockpick

An offline solver for the lockpicking minigame in **Gothic 1 Remake**. One HTML file. Open it, describe the lock, get the shortest sequence of turns that opens it without snapping your pick.

---

## Are you worthy?

So. You met a chest.

And instead of enjoying the *masterfully engineered*, *deeply rewarding* tumbler puzzle that so generously elevates this remake above the crude original — a puzzle that adds, let us be honest, immeasurable value, tension, and meaningful player expression to the experience — you sat there. Clicking. Left, right, left, right. Watching a little copper pin wander between seven holes like it owed you money. Snapping your third lockpick on a chest that contained two apples and a rusty knife.

And now you are here, in a git repository, reading a README, because you could not beat a row of five sliders.

Truly, the gaming press was right: this is *the* standout feature.

Take the tool, you poor thing. Nobody has to know. The pick you did not break is between you and me.

*(If you genuinely like the minigame: good for you, sincerely. This repo is for the rest of us. Stay for the BFS write-up at the bottom.)*

---

## Using it

Open `index.html` in a browser. That is the whole installation. No server, no build step, no dependencies, no network — it works on a laptop in a cave with no signal.

**1. The Lock.** Pick the plate count (3–7) and click the hole each pin currently sits in. Hole 4 — the lit column — is where they all have to end up.

**2. Couplings.** In game, turn each plate one step to the **right** and watch what else moves. Record it in the grid: click a cell to cycle it through `→` (moves the same way), `←` (moves the opposite way), `·` (does not move). Turning a plate back costs nothing, so undo each test before moving on.

> Test **every** plate. Couplings in this game are one-way as often as mutual — plate 2 can drag plate 5 while plate 5 leaves plate 2 alone — so no row can be inferred from another. If a pin sits at hole 1 or 7 and cannot be turned outward, test that plate in the other direction and press `⇄` on its row to flip what you saw.

**3. Solution.** The sequence appears as big tokens: `1L 2R 4L 5R` — plate number, then `R` for right or `L` for left, read left to right. Click any token to see the lock at that point, or step through it with `←` / `→` and the playback controls. **Copy sequence** puts it on the clipboard as plain text.

---

## The Lock Journal

A chest keeps its couplings forever, so a lock you solved once is worth keeping.

**Save to journal** opens a dialog: location (the three camps, the mines, orc territory, Sleeper's Temple, wilderness, other), a name, and optional notes. Saving writes one JSON line:

```json
{"id":"lk...","saved":"2026-09-19T10:00:00.000Z","location":"Old Camp","name":"Chest by Diego","notes":"under the stairs","plates":5,"start":[5,3,2,5,4],"links":[[1,0,-1,0,0],[0,1,1,0,0],[0,0,1,0,0],[0,0,0,1,1],[0,0,0,0,1]],"turns":4,"sequence":"1L 2R 4L 5R"}
```

The row carries the **full coupling map**, not just the answer — so **Load** on a saved entry restores that lock into the solver and you never test a plate in that chest again.

Three ways to keep it, in order of preference:

| | What it does | Where it works |
|---|---|---|
| **In the page** | Entries persist in browser storage across restarts | Always |
| **Link journal file** | Pick or create `g1r-lockpick-journal.jsonl` anywhere you like (next to this file, for instance) and every save rewrites it with the complete journal | Chrome / Edge, via the File System Access API |
| **Download / Import** | Export the whole journal as `.jsonl`, import it back later or on another machine — deduplicated by `id`, malformed lines skipped | Always |

A page opened over `file://` cannot silently write to your disk — that is a browser security boundary, not an oversight. **Link journal file** is the closest thing to it, and it needs your explicit pick of the file. Where that API is missing, the button does not appear and the status line says so.

---

## The rules, as implemented

- A lock has three to seven plates. Each pin rests in one of seven holes; the lock opens when **every pin is in hole 4 at the same time**.
- Turning a plate moves its own pin one hole, and drags each coupled pin one hole — in the same or the opposite direction.
- A coupling may be mutual or **one-way**. Plate 2 dragging plate 5 says nothing about what plate 5 does to plate 2.
- No pin may be pushed past hole 1 or hole 7. That is the move that strains and breaks the pick, so the solver never proposes one.
- Raising the Lockpicking skill adds pick durability and *removes* couplings. It makes a lock simpler, not different — re-read the rows after you train.

## How the solver works

The lock is a state: one hole index per plate, so at most 7⁷ = 823,543 states. Each turn is a deterministic edge — add the coupling row to the current state, or subtract it for a left turn — and a turn that would push any pin outside 1–7 is simply not an edge. Finding the fewest turns is therefore a **breadth-first search** from the entered state to `4 4 4 4 …`, which guarantees the shortest sequence and proves it when no sequence exists at all.

The whole search is a flat typed-array frontier over base-7 encoded states. The worst case — seven plates, all pins at the edges — settles in well under a tenth of a second.

Which is, admittedly, a lot less time than the minigame took from you.

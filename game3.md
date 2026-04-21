# Game 3 — The Great Banana Heist

## Concept

King Kong has stolen the golden bananas and fled into the jungle. Athar's ape must chase him by leaping across jungle trees, counting on to reach the right branch. After 5 successful jumps, Athar collects a banana stash and faces Kong in a final race.

**Core skill: counting on.** Each round gives a starting tree and a hop count. Athar must identify the landing tree. All trees look the same and are numbered — the numbers are the scaffold. Wrong tree = fall and retry. Right tree = leap forward and chase continues.

---

## Levels

| Level | Character | Trees shown | Hop range | Number range |
|---|---|---|---|---|
| 1 | Chimpanzee | 6 | 1–2 | starts 1–8, always fits on screen |
| 2 | Orangutan | 10 | 1–3 | starts 1–12, always fits on screen |
| 3 | Gorilla | none (keypad) | 1–4 | starts 2–15 |

Each level is independent — Athar picks from the start screen. Each level leads to its own boss race after 5 rounds.

---

## Screens

1. **start-screen** — Title, 3 level buttons (Chimp / Orangutan / Gorilla), Play button
2. **game-screen** — HUD (round pips, miss count) + tree row (L1/L2) or keypad (L3) + ape character + prompt
3. **banana-screen** — Banana stash celebration after 5 completed rounds
4. **boss-cutscene** — Kong appears dramatically, short dialogue, tap to race
5. **boss-screen** — Left-to-right race track with banana in center; ape vs Kong
6. **result-screen** — Win or lose; play again / home

---

## Game Screen Layout

```
┌────────────────────────────────────────────┐
│  HUD: Round 3/5  ● ● ○ ○ ○   Misses: 1   │
├────────────────────────────────────────────┤
│                                            │
│   🐒  "King Kong is getting away!          │
│        Count on 3!"                        │
│                                            │
│   [🌳1] [🌳2] [🌳3] [🌳4] [🌳5] [🌳6]  │
│    ↑ APE HERE                              │
│                                            │
└────────────────────────────────────────────┘
```

- Starting tree: highlighted border, ape character sitting on top
- Other trees: clickable, numbered, identical images (4 tree types, varied)
- Prompt shown above trees
- No banana visible anywhere before the answer is confirmed

### Level 3 layout (keypad)

```
┌────────────────────────────────────────────┐
│  HUD: Round 3/5  ● ● ○ ○ ○   Misses: 0   │
├────────────────────────────────────────────┤
│                                            │
│   🦍  "Count on 3 from tree 7!"           │
│                                            │
│   ┌───┬───┬───┐                           │
│   │ 7 │ 8 │ 9 │                           │
│   ├───┼───┼───┤                           │
│   │ 4 │ 5 │ 6 │   [JUMP! →]              │
│   ├───┼───┼───┤                           │
│   │ 1 │ 2 │ 3 │                           │
│   └───┴───┴───┘                           │
│         [0]                               │
└────────────────────────────────────────────┘
```

---

## Correct Answer Flow

1. Whoosh SFX
2. Ape does arc-jump animation from start tree to landing tree
3. sfx-correct plays on landing
4. Landing tree flashes gold
5. Short pause (1s), then new question: startTree = old targetTree, new hopCount generated
6. Trees re-render with new visible range (panning right)
7. After 5 correct jumps: → banana-screen

## Wrong Answer Flow

1. Ouch SFX
2. Ape fall animation (drops below tree, sad pose briefly)
3. Ape bounces back to starting tree (idle pose)
4. totalMisses++
5. Same question repeats (same startTree, same hopCount)
6. Hint bar appears after 2nd miss on same question: "Try counting on your fingers!"

---

## Banana Stash Screen

- Full-screen celebration
- banana-stash.png flies in from top
- yahoo.mp3 plays
- "You found [N] bananas!" text
- sfx-victory plays
- After 2.5s → boss-cutscene

---

## Boss Cutscene

- Kong slides in from right side of screen (idle → attack pose)
- sfx-roar plays
- Dialogue bubble: "ROOOAR! You'll NEVER catch me! Let's race!"
- Tap to continue → boss-screen

---

## Boss Race Screen

```
┌─────────────────────────────────────────────────┐
│  "Count on [N] from [X]!"                       │
├─────────────────────────────────────────────────┤
│                                                 │
│  [APE]  ●──────────[🍌]──────────●  [KONG]    │
│          step 2/5            step 1/5           │
│                                                 │
│  Keypad (all levels in boss round)              │
└─────────────────────────────────────────────────┘
```

- Both characters start at opposite edges
- 5 steps each to reach the central banana stash
- Player answers question → ape advances 1 step (correct only; can retry)
- After each player attempt (right or wrong), Kong rolls dice → may advance
- First to 5 steps wins

### Kong's speed calibration

```
P(Kong advances per player turn) = min(0.90, totalMisses × 0.22)

0 misses → P = 0.00  → Kong never moves   → player always wins
1 miss   → P = 0.22  → Kong rarely moves  → player likely wins
2 misses → P = 0.44  → Kong sometimes moves → roughly 50/50
3 misses → P = 0.66  → Kong often moves   → Kong likely wins
4 misses → P = 0.88  → Kong very fast     → player unlikely to win
5+ misses → P = 0.90 → Kong nearly always advances
```

After each player attempt (hit or miss), Kong's probability is checked once and he advances or not. This means wrong answers cost doubly — they don't advance the player AND give Kong an extra chance to advance.

---

## Result Screen

**Win:** ape victory pose, confetti, yahoo.mp3, "You got the bananas! Kong can't stop you!"  
**Lose:** kong-victory.png, sfx-roar, "Kong got there first! Try again!"  
Buttons: Play Again (same level), Home

---

## State Object

```js
const G = {
  level:        1,    // 1 | 2 | 3
  round:        0,    // 1–5
  startTree:    0,    // current start
  hopCount:     0,    // how many to count on
  targetTree:   0,    // startTree + hopCount (correct answer)
  totalMisses:  0,    // across all 5 rounds; feeds boss calibration
  roundMisses:  0,    // misses on the current round (resets each round)
  firstTry:     true, // was current round answered first try?
  roundResults: [],   // 'clean' | 'miss' per completed round
  interacting:  true,
  // boss
  playerPos:    0,    // steps taken toward center (0–5)
  kongPos:      0,    // steps taken toward center (0–5)
  bossStart:    0,
  bossHop:      0,
  bossTarget:   0,
  bossInput:    '',   // keypad input string
  bossInteracting: true,
  playerWon:    false,
};
```

## Level Config

```js
const LEVEL_CFG = {
  1: { char: 'chimpanzee', label: 'Chimp',     treeCount: 6,  maxHop: 2, startMin: 1, startMax: 8  },
  2: { char: 'orangutan',  label: 'Orangutan', treeCount: 10, maxHop: 3, startMin: 1, startMax: 12 },
  3: { char: 'gorilla',    label: 'Gorilla',   treeCount: 0,  maxHop: 4, startMin: 2, startMax: 15 },
};
```

---

## Assets Used

| Asset | Usage |
|---|---|
| chimpanzee-idle/jump/fall/victory.png | Level 1 hero |
| orangutan-idle/jump/fall/victory.png | Level 2 hero |
| gorilla-idle/jump/fall/victory.png | Level 3 hero |
| king-kong-idle/attack/sad/victory.png | Boss |
| tree-1/2/3/4.png | Tree platforms (cycled by tree number mod 4) |
| banana-stash.png | Stash reward |
| bg-jungle-arena.png | All screens |
| whoosh.wav | Jump SFX |
| ouch.wav | Fall SFX |
| sfc-correct.wav | Correct landing |
| sfx-wrong.wav | Wrong tap |
| sfx-victory.wav | Round/game win |
| sfx-roar.mp3 | Kong entrance |
| yahoo.mp3 | Banana celebration |
| jungle-boss-race.wav | Looping boss race music |

---

## Code Architecture (following game1 patterns)

- Single `game3-jungle-hop.html` file
- Frijole font (Google Fonts)
- `const G = {}` state object, reset on new game
- `showScreen(id)` for screen switching
- `mkAudio()` + `play()` for SFX
- CSS animations via class toggling + `void el.offsetWidth` reflow trick
- `clamp()` throughout for responsive sizing
- `user-select: none`, `-webkit-tap-highlight-color: transparent`
- JS sections delimited by `// ===== SECTION =====` comments

---

## Build Task List

1. HTML skeleton + head (fonts, meta, title)
2. CSS: variables, reset, screen base
3. CSS: start screen
4. CSS: game screen (HUD, tree row, ape, prompt)
5. CSS: tree slots, start-tree highlight, wrong/correct states
6. CSS: jump arc animation + fall animation
7. CSS: Level 3 keypad
8. CSS: banana screen, boss cutscene, boss race track
9. CSS: result screen + confetti
10. JS: state, constants, audio, screen nav
11. JS: start screen — level selection + play button
12. JS: question generation (all levels)
13. JS: tree rendering (L1 + L2)
14. JS: tree click → correct/wrong → animations
15. JS: pan transition to next question
16. JS: round completion → banana screen
17. JS: Level 3 keypad input + submit
18. JS: boss cutscene
19. JS: boss race — player movement + question loop
20. JS: boss race — Kong AI (probability-based advance)
21. JS: boss race — win condition + result screen
22. JS: confetti + result screen
23. Polish: hint bar, mobile touch, edge cases

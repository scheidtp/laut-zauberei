# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Laut-Zauberei** is a German-language speech therapy learning game for children ages 4–8. It trains phonetic distinction of four German sounds: **S**, **SCH**, **CH**, and **Z**.

## Running the App

No build tools or dependencies. Open `index.html` directly in any browser:

```bash
open index.html   # macOS
```

Deploy by pushing to GitHub and enabling GitHub Pages (main branch, root directory).

## Architecture

The entire application lives in a single file: `index.html` (~1,360 lines). It contains inline CSS, inline JavaScript, and all HTML — no external dependencies except Google Fonts.

### Game Modes

1. **Laut erkennen** (`identify`) — Word and lautschrift start blurred; player uses a reveal button then picks the sound (S / SCH / CH / Z)
2. **Wörter sortieren** (`sort`) — Click a word to select it, then click a sound bin to place it; 9 words sorted into 4 bins (S, SCH, CH, Z); score is 5 pts per correct placement
3. **Lücke füllen** (`fill`) — Choose the correct word to fill a sentence blank; questions come from the separate `fillQuestions` array (not the `words` database)
4. **Laut orten** (`position`) — Pick where in the word the target sound appears (vorn / Mitte / Ende); a 3-segment bar lights up the correct position after answering

### State

All state is global JS variables (no persistence between sessions):

```javascript
let mode = 'identify'   // current game mode
let score = 0           // total points (10 per correct, 5 per sort placement)
let round = 0           // current round (0–10)
let correctCount = 0    // correct answers this session
let currentItem = null  // current question word object (with .type added by allWordsList())
let answered = false    // whether current question is answered
let roundLog = []       // history for results screen
const TOTAL_ROUNDS = 10
```

### Word Database

`words` holds ~120+ German words split into `s`, `sch`, `ch`, `z` arrays. Z is defined separately after the main object (`words.z = [...]`). Each entry:

```javascript
{
  word: "Schlange",
  artikel: "die",           // der | die | das
  emoji: "🐍",
  hint: "child-friendly description",
  lautschrift: "[SCH-lan-ge]",
  highlight: "Schl",        // letters to underline — can be multi-char cluster: 'S','Sch','Schl','St','Sp','Zw','tz','z'
  position: "vorn"          // vorn | mitte | ende
}
```

`allWordsList()` flattens all four arrays and adds a `type` field (`'s'|'sch'|'ch'|'z'`) to each item via spread.

### Fill Questions

`fillQuestions` is a separate hardcoded array independent of the `words` database:

```javascript
{
  sentence: 'Die ____ scheint heute besonders hell.',
  options: ['Sonne ☀️', 'Schule 🏫', 'Kirche ⛪'],
  answer: 0,        // index of correct option
  sound: 's',
  extraInfo: '...'  // optional phonetic explanation shown in feedback
}
```

### Tips

`tips` object maps `'s'|'sch'|'ch'|'z'|'general'|'position'` to hint strings shown in the tip box below the game card. `setTip(type)` updates it after each answer.

### Key Functions

- `renderGame()` — main render dispatcher based on `mode`; calls `showResults()` when `round >= TOTAL_ROUNDS`
- `renderIdentify()` / `answerIdentify()` — sound identification mode
- `renderSort()` / `checkSort()` — word sorting mode (click-to-select + click-bin)
- `renderFill()` / `answerFill()` — fill-the-blank mode
- `renderPosition()` / `answerPosition()` — sound position mode with visual 3-segment bar
- `showResults()` / `switchTab()` — results screen with tabbed correct/wrong review
- `highlightWord(word, highlight, colorVar)` — wraps the `highlight` substring in a colored `<span>`
- `artDef(w)` / `artDefCap(w)` — return grammatical article for a word object (lowercase / sentence-start capitalized)
- `allWordsList()` — returns shuffled flat array of all words with `type` injected
- `setMode(m)` — switches mode and resets score/round/correctCount

### CSS Design Tokens

Sound colors defined as CSS variables:
- `--s-color: #FF6B6B` (red)
- `--sch-color: #4ECDC4` (turquoise)
- `--ch-color: #FFE66D` (yellow)
- `--z-color: #C77DFF` (purple)

Dark background (`#1a1a2e`), child-friendly fonts: **Fredoka One** (headings/buttons) and **Nunito** (body).

## Phonetic Rules Implemented

- ST/SP at word start → pronounced as SCH (e.g., *Stein* → [SCHtein], *Spinne* → [SCHpinne]); these words carry `highlight: 'St'` or `highlight: 'Sp'` and appear in the SCH category
- CH has two variants: Ach-laut after a/o/u (*Buch*, *Nacht*) vs. Ich-laut after e/i/ä (*Milch*, *Mädchen*)
- Z sounds like [ts]; also written as `tz` (*Katze*) or `zw` (*Zwerg*)

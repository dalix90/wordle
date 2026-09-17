# Building Wordle, step by step

Eight steps. Each one ends with a commit and something new working on the page.

Everything happens in `index.html`. Open the repo on github.com, press `.`, and
you're in the editor.

After each step: commit, wait ~30 seconds, reload your Pages URL.

If something breaks, open the browser console (right-click → Inspect → Console).
Red text there tells you which line is wrong.

---

## Step 1 — Check that the page works

Nothing to write yet. Just confirm the setup.

Open `https://YOUR-USERNAME.github.io/wordle/`. You should see the word WORDLE
on a dark background.

If you don't, Pages isn't on yet. Go back to the README setup section.

Now change the `<p>` line in `index.html` to say anything you like, commit it,
wait, and reload. Once you've watched your own words show up on a real website,
the rest of this is just more of the same.

**Commit:** `First change`

---

## Step 2 — Draw the board

Wordle is a grid: 6 rows of 5 boxes. You're going to build it with JavaScript
instead of typing 30 `<div>`s by hand.

### The HTML

Replace the two lines inside `<body>` with this:

```html
  <h1>WORDLE</h1>
  <div id="board"></div>
```

### The CSS

Add this inside the `<style>` tag, at the bottom:

```css
  #board {
    display: grid;
    gap: 5px;
    margin-top: 30px;
  }

  .row {
    display: grid;
    grid-template-columns: repeat(5, 1fr);
    gap: 5px;
  }

  .tile {
    width: 62px;
    height: 62px;
    border: 2px solid #3a3a3c;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 32px;
    font-weight: 700;
    text-transform: uppercase;
  }
```

### The JavaScript

Add this just before `</body>`:

```html
  <script>
    const ROWS = 6;
    const COLS = 5;

    const board = document.getElementById("board");

    for (let r = 0; r < ROWS; r++) {
      const row = document.createElement("div");
      row.className = "row";

      for (let c = 0; c < COLS; c++) {
        const tile = document.createElement("div");
        tile.className = "tile";
        row.appendChild(tile);
      }

      board.appendChild(row);
    }
  </script>
```

### What to check

30 empty boxes, 6 rows of 5.

If you see nothing, the `<script>` tag is probably above the `<div id="board">`
instead of below it. The script runs top to bottom, so the board has to exist
before the code looks for it.

**Commit:** `Draw the board`

---

## Step 3 — Type letters into it

Right now typing does nothing. Let's fix that.

Add this inside the `<script>`, below the loop you just wrote:

```js
    let currentRow = 0;
    let currentGuess = "";

    function drawCurrentRow() {
      const tiles = board.children[currentRow].children;

      for (let c = 0; c < COLS; c++) {
        tiles[c].textContent = currentGuess[c] || "";
      }
    }

    function addLetter(letter) {
      if (currentGuess.length >= COLS) return;
      currentGuess = currentGuess + letter;
      drawCurrentRow();
    }

    document.addEventListener("keydown", function (event) {
      if (/^[a-zA-Z]$/.test(event.key)) {
        addLetter(event.key.toLowerCase());
      }
    });
```

### What's going on

`currentGuess` is a string that holds what you've typed so far. Every time it
changes, `drawCurrentRow` copies it into the boxes, one character per box.

`/^[a-zA-Z]$/` is a **regular expression** — a pattern. It means "exactly one
character, and it has to be a letter." That's how the code ignores Shift, arrow
keys, and everything else.

`if (currentGuess.length >= COLS) return;` stops at 5 letters. `return` means
"quit this function now, do nothing."

### What to check

Type. Letters appear. Type a sixth one and nothing happens.

**Commit:** `Type letters`

---

## Step 4 — Backspace and Enter

Two more keys. Backspace removes a letter, Enter moves to the next row.

Add `deleteLetter` and `submitGuess` below `addLetter`:

```js
    function deleteLetter() {
      currentGuess = currentGuess.slice(0, -1);
      drawCurrentRow();
    }

    function submitGuess() {
      if (currentGuess.length < COLS) return;

      currentRow = currentRow + 1;
      currentGuess = "";
    }
```

Then extend the keydown listener so it handles all three cases:

```js
    document.addEventListener("keydown", function (event) {
      if (event.key === "Enter") {
        submitGuess();
      } else if (event.key === "Backspace") {
        deleteLetter();
      } else if (/^[a-zA-Z]$/.test(event.key)) {
        addLetter(event.key.toLowerCase());
      }
    });
```

`slice(0, -1)` means "everything except the last character."

### What to check

Type five letters, press Enter, keep typing. You're on row two.

Backspace deletes. Enter with fewer than 5 letters does nothing.

There's a bug on purpose: after 6 rows, `currentRow` points past the end of the
board and you get an error in the console. Step 6 fixes it.

**Commit:** `Backspace and Enter`

---

## Step 5 — The secret word and the colors

This is the real Wordle logic, and the only genuinely tricky part of the whole
project.

### Load the word list

Add this in the `<head>`, above the `<style>` tag:

```html
<script src="words.js"></script>
```

That file gives you `ANSWERS` (the words the game can pick) and `ALLOWED` (the
words you're permitted to guess).

### Pick a word

Add this near the top of your `<script>`, under `const COLS = 5;`:

```js
    const answer = ANSWERS[Math.floor(Math.random() * ANSWERS.length)];
```

`Math.random()` gives a decimal between 0 and 1. Multiply by the list length and
round down, and you get a random position in the list.

While building, it helps to know the answer. Add this temporarily, and delete it
when you're done:

```js
    console.log("answer:", answer);
```

### The colors

Add to the CSS:

```css
  .tile.correct { background: #538d4e; border-color: #538d4e; }
  .tile.present { background: #b59f3b; border-color: #b59f3b; }
  .tile.absent  { background: #3a3a3c; border-color: #3a3a3c; }
```

### The scoring rule

Here's the naive version that *almost* works:

```js
// Don't use this one.
function scoreGuess(guess, answer) {
  const result = [];
  for (let i = 0; i < COLS; i++) {
    if (guess[i] === answer[i])            result.push("correct");
    else if (answer.includes(guess[i]))    result.push("present");
    else                                   result.push("absent");
  }
  return result;
}
```

Try it in your head. Answer is `eaten`, guess is `eerie`.

- The first `e` is green. Good.
- The second `e` is yellow. Correct — `eaten` has another `e`.
- The last `e` is *also* yellow. Wrong. There are only two `e`s in `eaten`, and
  both are already accounted for.

The fix: keep a copy of the answer's letters and cross each one off as you use
it. Do all the greens first, because a green letter has first claim.

Use this version:

```js
    function scoreGuess(guess, answer) {
      const result = new Array(COLS).fill("absent");
      const remaining = answer.split("");

      // First pass: greens. Cross off each letter we use.
      for (let i = 0; i < COLS; i++) {
        if (guess[i] === remaining[i]) {
          result[i] = "correct";
          remaining[i] = null;
        }
      }

      // Second pass: yellows, but only from letters still left over.
      for (let i = 0; i < COLS; i++) {
        if (result[i] === "correct") continue;

        const found = remaining.indexOf(guess[i]);
        if (found !== -1) {
          result[i] = "present";
          remaining[found] = null;
        }
      }

      return result;
    }
```

`split("")` turns `"eaten"` into `["e","a","t","e","n"]` so you can cross letters
off by setting them to `null`. `continue` means "skip to the next loop turn."

### Paint the tiles

Replace `submitGuess` with this:

```js
    function submitGuess() {
      if (currentGuess.length < COLS) return;
      if (!ALLOWED.includes(currentGuess)) return;

      const result = scoreGuess(currentGuess, answer);
      const tiles = board.children[currentRow].children;

      for (let i = 0; i < COLS; i++) {
        tiles[i].classList.add(result[i]);
      }

      currentRow = currentRow + 1;
      currentGuess = "";
    }
```

### What to check

Guess the word from the console log. All five turn green.

Then test the duplicate-letter case deliberately. Open the console and run:

```js
scoreGuess("eerie", "eaten")
```

You want `["correct", "present", "absent", "absent", "absent"]`. If the last `e`
comes back `present`, you're still on the naive version.

**Commit:** `Color the guesses`

---

## Step 6 — Winning, losing, and messages

The game still doesn't know when it's over, and it silently ignores bad guesses
instead of saying why.

### The message popup

Add to the HTML, above `<div id="board">`:

```html
  <div id="message"></div>
```

Add to the CSS:

```css
  #message {
    position: fixed;
    top: 80px;
    background: #ffffff;
    color: #000000;
    padding: 12px 18px;
    border-radius: 4px;
    font-weight: 700;
    opacity: 0;
    transition: opacity 200ms;
    pointer-events: none;
  }

  #message.show { opacity: 1; }
```

Add to the JavaScript:

```js
    let messageTimer;

    function showMessage(text) {
      const el = document.getElementById("message");
      el.textContent = text;
      el.classList.add("show");

      clearTimeout(messageTimer);
      messageTimer = setTimeout(function () {
        el.classList.remove("show");
      }, 1500);
    }
```

`setTimeout` runs code later. `clearTimeout` cancels a pending one, so two quick
messages don't fight over the popup.

### Wire it up

Add a `gameOver` flag next to your other variables:

```js
    let gameOver = false;
```

Replace `submitGuess` one more time:

```js
    function submitGuess() {
      if (gameOver) return;

      if (currentGuess.length < COLS) {
        showMessage("Not enough letters");
        return;
      }

      if (!ALLOWED.includes(currentGuess)) {
        showMessage("Not in word list");
        return;
      }

      const result = scoreGuess(currentGuess, answer);
      const tiles = board.children[currentRow].children;

      for (let i = 0; i < COLS; i++) {
        tiles[i].classList.add(result[i]);
      }

      if (currentGuess === answer) {
        gameOver = true;
        showMessage("You got it!");
        return;
      }

      currentRow = currentRow + 1;
      currentGuess = "";

      if (currentRow === ROWS) {
        gameOver = true;
        showMessage(answer.toUpperCase());
      }
    }
```

Add the same guard to the other two functions, so typing stops once the game
ends:

```js
    function addLetter(letter) {
      if (gameOver) return;
      if (currentGuess.length >= COLS) return;
      currentGuess = currentGuess + letter;
      drawCurrentRow();
    }

    function deleteLetter() {
      if (gameOver) return;
      currentGuess = currentGuess.slice(0, -1);
      drawCurrentRow();
    }
```

### What to check

- Win. It says so, and typing stops.
- Lose six times. It shows you the word.
- Guess `zzzzz`. It says "Not in word list."

That console error from Step 4 is gone, because `gameOver` stops the code before
it can run off the end of the board.

**Commit:** `Win, lose, and messages`

---

## Step 7 — The on-screen keyboard

Phones don't have a physical keyboard, and the colored keys are genuinely useful
for remembering which letters you've ruled out.

### The HTML

Add below `<div id="board">`:

```html
  <div id="keyboard"></div>
```

### The CSS

```css
  #keyboard {
    display: flex;
    flex-direction: column;
    gap: 8px;
    width: 100%;
    max-width: 500px;
    margin-top: 30px;
  }

  .keyrow {
    display: flex;
    gap: 6px;
    justify-content: center;
  }

  .key {
    flex: 1;
    min-width: 0;
    height: 58px;
    border: 0;
    border-radius: 4px;
    background: #818384;
    color: #ffffff;
    font-size: 13px;
    font-weight: 700;
    text-transform: uppercase;
    cursor: pointer;
  }

  .key.wide { flex: 1.5; }
  .key.correct { background: #538d4e; }
  .key.present { background: #b59f3b; }
  .key.absent  { background: #3a3a3c; }
```

### The JavaScript

```js
    const KEY_ROWS = ["qwertyuiop", "asdfghjkl", "↵zxcvbnm⌫"];
    const keyboard = document.getElementById("keyboard");

    for (const letters of KEY_ROWS) {
      const row = document.createElement("div");
      row.className = "keyrow";

      for (const letter of letters) {
        const key = document.createElement("button");
        key.className = "key";
        key.dataset.key = letter;
        key.textContent = letter === "↵" ? "enter" : letter;

        if (letter === "↵" || letter === "⌫") {
          key.classList.add("wide");
        }

        key.addEventListener("click", function () {
          if (letter === "↵") submitGuess();
          else if (letter === "⌫") deleteLetter();
          else addLetter(letter);
        });

        row.appendChild(key);
      }

      keyboard.appendChild(row);
    }
```

### Coloring the keys

A key should only ever get better news, never worse. If `E` is already green, a
later guess where `E` is yellow must not downgrade it.

```js
    const RANK = { absent: 0, present: 1, correct: 2 };

    function paintKey(letter, state) {
      const key = keyboard.querySelector('[data-key="' + letter + '"]');
      if (!key) return;

      const current = ["absent", "present", "correct"].find(function (s) {
        return key.classList.contains(s);
      });

      if (current && RANK[current] >= RANK[state]) return;

      key.classList.remove("absent", "present", "correct");
      key.classList.add(state);
    }
```

Then call it from `submitGuess`, inside the loop that colors the tiles:

```js
      for (let i = 0; i < COLS; i++) {
        tiles[i].classList.add(result[i]);
        paintKey(currentGuess[i], result[i]);   // <- add this line
      }
```

### What to check

- Clicking keys plays the game.
- Keys pick up colors as you guess.
- A key that went green never goes back to yellow.

**Commit:** `On-screen keyboard`

---

## Step 8 — Make it yours

It's a finished game now. Everything from here is your call.

Easy:

- Change the colors. The greens and yellows are just hex codes in the CSS.
- Add words to `ANSWERS` in `words.js`. Family names, inside jokes, five letters
  each.
- Change the win message based on how many guesses it took.

Medium:

- Add a **Play again** button that reloads the page.
- Make the tiles flip over when a guess is submitted. Look up CSS
  `@keyframes` and `transform: rotateX()`.
- Shake the row when a guess isn't a real word. Also `@keyframes`.
- Remember the score — wins, losses, streak — with `localStorage`.

Harder:

- One word per day for everybody, instead of random. Take today's date, turn it
  into a number, and use that number to pick from the list. Everyone who plays
  on the same day gets the same word.
- A **Share** button that copies a grid of colored squares to the clipboard,
  without spoiling the letters.
- Hard mode: any green or yellow letter you've found must be used in every
  guess after that.

`solution/index.html` has the row shake, a small pop when a tile fills, and the
fancier win messages, if you want to see how those are done.

# ashley
A wordle copycat

**Play it:** https://dalix90.github.io/wordle/

## Files

| File | What it is |
| --- | --- |
| `index.html` | The game. This is the one you edit. |
| `words.js` | The list of five-letter words. |
| `LESSON.md` | Step-by-step instructions for building it. Start here. |
| `solution/index.html` | The finished version. Look only when stuck. |

## Setting up (once)

1. Make a repo on github.com called `wordle`. Public, with a README.
2. Upload these files, or push them from your computer.
3. Turn on GitHub Pages:
   - Settings > Pages
   - Source: **Deploy from a branch**
   - Branch: **main**, folder: **/ (root)**
   - Save
4. Wait about a minute, then open `https://dalix90.github.io/wordle/`.
5. Edit the README above to use your real username.

## How to edit

Go to the repo page on github.com and press the `.` key.

That opens **github.dev**, a full code editor in the browser. Nothing to
install.

To save your work:

1. Click the Source Control icon in the left sidebar (the branching arrows).
2. Type a message describing what you did.
3. Click the check mark (Commit & Push).

Wait roughly 30 seconds, then reload your Pages URL to see the change.

## Why it's slow to see changes

GitHub has to rebuild the site after every commit. That's the 30 seconds.

If you want to see a change *right now*, github.dev can't do it; it has no way
to run the page. Two options:

- Commit and wait. Fine for most steps.
- Download the repo and open `index.html` directly in a browser. Instant, but
  then you're editing files on your computer instead.

Committing and waiting is simpler. Just don't commit after every single
character.

## Rules of the game

- Guess a five-letter word in six tries.
- Green: right letter, right spot.
- Yellow: right letter, wrong spot.
- Gray: that letter isn't in the word at all.

# House Sort

Split your group across houses — without anyone seeing anyone's votes.

One file, `index.html`. No server, no accounts, nothing stored. Share it (WhatsApp file share
works, or host it free on GitHub Pages / Netlify) — the group then runs the whole vote from the
chat.

Works for **any roster of 4–20 people** and **any layout of 2–6 houses** with 2–9 beds each.

## Who does what

**The organiser:**

1. Open the tool → *I'm running the vote*.
2. Type the roster (one name per line), shape the houses (presets, ± steppers, add/remove), and
   pick how many people may share a preference **row** — *No ties* (strict, strongest
   anti-gaming), or shelves of up to 2 / 3 / 4 (tied shelves pull slightly weaker each, so big
   "don't care" groups self-dampen). The rule travels inside the round code; chips are
   validated against it.
3. Choose a passphrase → **Generate key, codes & round code**. You get three things:
   - the **round code** (`H44S1-…`) — post it in the group chat. It's public: it carries the
     roster, the layout and the public key, nothing else.
   - **private invite codes** (one per person) — DM each friend theirs. A code only proves
     whose chip is whose; it encodes no preferences.
   - the **key file** download — the only copy of the private key, AES-sealed with your
     passphrase. It was never stored in the browser. Keep it somewhere a screenshot isn't.
4. When everyone's chips are in the chat: paste the chips + key file + passphrase →
   *Reveal arrangements*.
5. Post the **results code** (`H44R1-…`) in the chat. It contains only the roster, the layout
   and the aggregate pull between each pair — never an individual ranking.

**Each friend:**

1. Open the tool → *I'm voting* → paste the round code from the chat.
2. Pick your own name (it never enters the chip), paste your invite code → **Check in & start
   ranking**.
3. Drag the bubbles into the rows — the top row is who you want most. Empty "drop here"
   rows show where the next slot lands, and you can stack several people on one row only
   when the organiser allows ties; people sharing a row count as tied, and a bigger row
   pulls a little weaker per person, so a large "don't mind" group can't dominate.
   Drop a bubble **in the gap between two rows** (or **above the top row**) to open a new
   row at that exact spot, or **below the last row** to give someone their own new bottom
   row — that's how you split a tied group apart. On a phone, **press and hold** a bubble
   to lift it, and swipe normally to scroll the page. (Keyboard-only: focus a bubble and
   press ↑/↓.)
4. **Create my chip** → copy the encrypted gibberish → post it in the chat. Done.

**Anyone:**

- Open the tool → *See the splits* → paste the results code. No key needed.
- The viewer also has a **live layout playground**: move beds between houses (or tap a preset
  like *3 + 5*) and the best groupings recompute instantly — no revote. Perfect for "what if we
  do 3 + 5 instead of 4 + 4?"

## Why your vote can't be gamed

- **Strict ranking.** The list is a total order — you can't blanket everyone a 10.
- **Mutual weighting.** A friendship only scores when *both* sides rank each other high. Maxing
  three friends means nothing unless they did the same back.
- **Anonymity.** Chips are randomized RSA-OAEP ciphertexts — identical ballots look completely
  different in chat. Only the organiser's key opens them, and decrypted ballots are never
  rendered, stored, or sent: the tool shows arrangements only. The results code carries just
  the pair-pull matrix, so the public can't reconstruct anyone's list.
- **No revisions.** Chips are final once posted; the reveal needs every code. Duplicate chips
  pasted twice are safely ignored.
- **Maximin ordering.** Splits are ranked by the unhappiest household member first, so nobody
  gets parked as collateral — and three good options are shown, so fine-tuned ranks don't buy
  any specific outcome.
- **Tamper-proof codes.** Forged codes, chips claiming the wrong name and mangled result codes
  are rejected outright.

## Technical

- Single `index.html`, vanilla JS + WebCrypto. Serve it over **https** (or localhost) — Web
  Crypto is unavailable on plain http, and the tool refuses to run rather than weaken
  encryption.
- RSA-OAEP (SHA-256, 2048-bit) ballots; the private key is AES-GCM-sealed with the passphrase
  (PBKDF2, 310k iterations) inside the JSON key file.
- Code formats: `H44S1-` round code (roster + layout + public key), `H44C1-` chip (one
  encrypted ballot), `H44R1-` results code (roster + layout + pair-weight matrix only).
- Scoring: exact enumeration of all legal splits when the partition space allows (up to
  60 000), otherwise random sampling; person score = mean mutual weight with housemates;
  maximin ordering with mean tiebreak.
- Layout rules in the builder keep the bed count equal to the roster: sizing a house moves a
  bed to/from the fullest/smallest neighbour; removing a house spreads its people over the
  rest, capped at 9 beds per house.

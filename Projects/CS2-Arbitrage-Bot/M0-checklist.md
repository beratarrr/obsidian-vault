# M0 — First milestone: your own starting point

This gives you the **target** and the **decisions**, not the steps. The route through it is yours to draw. No code on purpose.

## The objective
Get **one venue's prices flowing out as normalized `Quote`s**, printed to your console. Nothing else. Raw venue data in → clean `Quote` objects out. No detection, no storage, no second venue.

## Done when (outcomes, not steps)
- [ ] One command runs and prints `Quote`s for a handful of real CS2 items, pulled live from one venue.
- [ ] Those `Quote`s are in *your* normalized shape — not the venue's raw JSON.
- [ ] Nothing venue-specific leaked past the connector: the thing that prints doesn't know which venue it came from.
- [ ] It's a git repo with a real commit and a README saying what M0 does.

## Decisions you own (answer these yourself — no single right answer)
- Which venue first? Lean toward least friction to read (public endpoint, no login gymnastics). Why that one?
- What's the minimum a `Quote` actually needs to be useful at M2/M3? You sketched a shape in the design doc — do you still trust it, or change it now?
- How do items get into the connector — hard-coded list, config file, CLI arg? What will annoy you least three milestones from now?
- Where does config live, and how do you keep any secret out of git from commit #1?
- Sync or async? Don't over-engineer M0 — but *decide* deliberately, don't drift into it.

## Worth thinking about (nudges, not answers)
- The venue hands you a "price" — is that a bid, an ask, a last trade, or the lowest listing? They are not the same, and getting it wrong quietly poisons everything downstream.
- What happens on a 429, junk, or an empty response? You don't have to handle it *well* in M0 — but notice where it would go.
- If someone added a second venue tomorrow, what would they have to implement, and would your code force them into the same shape? That's your real M0 test.

## Cheap traps to sidestep
- No storage, no detector, no second venue yet. One venue, printed. Resist.
- Don't let raw venue field names spread through your code — convert at the boundary.
- Don't spend the night hunting the "perfect" architecture. Make it run, commit, then judge it.

## The honest bar
You're done with M0 when you can say: *"I run it, real quotes come out in my own shape, and nothing downstream knows which website they came from."* Then commit, jot the microstructure lesson (bid vs ask vs last) in the README, and M1 is next.

---
If you get stuck on a *specific* thing, bring me that specific thing. The map through M0 is yours.

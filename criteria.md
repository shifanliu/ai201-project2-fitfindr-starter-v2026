# Acceptance criteria — FitFindr

Five criteria that say what "working" means for this agent, written in unit 3
**before** any results existed.

An acceptance criterion names a target: a number, a count, a rate, or something
a person could plainly observe. *"The agent handles errors"* is an opinion.
*"When search returns nothing, the agent stops before calling the second tool,
in 5 of 5 tries"* is a criterion.

Under each one, write a sentence or two on **why that target** and not a
stricter one. A reason that says something about your tools, your loop, or the
data earns credit; *"80% seemed reasonable"* does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

**Two are written for you. You write three.**

---

## 1. A matching query completes all three tools

Given a query that matches at least one listing, the agent completes all three
tool calls and returns a fit card — in at least 4 of 5 tries.

**Why this target:**
<!-- Why 4 of 5 and not 5 of 5? Something about your search, probably —
     "my search is a plain keyword match and some phrasings will miss" is a
     real answer. -->

My search scores matches by plain keyword overlap between `description` and the listing's text, so a query phrased unusually (synonyms, typos, word order the listing doesn't use) can score zero even when a human would call it a match. I picked 4 of 5 because that failure is about phrasing, not about the loop — a real miss here should look like "no results," not a crash, and I'd rather catch that in criterion 2's territory than demand perfect recall from a keyword matcher.

---

## 2. An impossible query stops before the second tool

Given a query that matches no listings, the agent stops before calling
`suggest_outfit` and returns a message naming what to change — 5 of 5 tries.

**Why this target:**
<!-- Why is 5 of 5 reasonable here when criterion 1 isn't? What's different
     about this path? -->

This path has no model call before the branch — `search_listings` either returns `[]` or it doesn't, and `if not results` is a plain Python truth check with no randomness in it. I picked 5 of 5 because there's no legitimate reason for this check to be right four times and wrong once; if it ever is, that's a bug in the branch, not noise from an imperfect search.

---

## 3. Something about state

<!-- YOU WRITE THIS ONE.

     How would you know that the item your search found is the same item the
     next tool received? Name something countable or observable.

     This is the criterion people find hardest, because state failure doesn't
     look like state failure — it looks like a tool problem. Something that
     compares session["selected_item"] against what actually reached
     suggest_outfit is the shape you're after. -->

Given a query that matches at least one listing, the item in `session["selecting_item"]` is identical (same `id`) to the item passed as `input` to the `suggest_outfit(...)` trace step - in 5 of 5 tries.

**Why this target:**

`session["selected_item"] = results[0]` and the value passed into `suggest_outfit(...)` come from the same assignment, with no copy, re-fetch, or re-run of the search in between — so there's no step where the two could legitimately diverge. I picked 5 of 5 because any mismatch here would mean the loop is re-reading stale state or calling search twice, which is exactly the kind of state bug this criterion exists to catch, not something I'm willing to shrug off as a "4 of 5" rounding error.

---

## 4. Something about the fit card

<!-- YOU WRITE THIS ONE.

     The fit card calls a model, so the same input can produce different words
     each time. That's not a bug — it's the nature of the tool. So what would
     make it acceptable?

     Think about what you'd actually be unhappy to see. A caption that never
     mentions the price? Two different items producing the same opening
     sentence? A card longer than a caption anyone would post? Any of those can
     be turned into a number. -->

Given the same item run through `create_fit_card` three times, every output contains the item's price (as text) and mentions the platform name once —
3 of 3 times — and no two of the three outputs are character-for-character identical.

**Why this target:**

Mentioning the price and the platform once is a prompt-following requirement, not a creativity requirement, so the model has no excuse to drop them — that part should hold every time. I picked 3 of 3 here (not 5 of 5) because I'm only running it three times in this test, but the standard is the same "no excuse to fail" bar as criterion 2; the "not word-for-word identical" check is separately there just to catch `CACHE_ENABLED` or `TEMPERATURE=0` silently returning the same string.

---

## 5. Dead loop protection

<!-- YOU WRITE THIS ONE TOO.

     Pick something you actually care about getting right. Speed, the empty
     wardrobe path, what happens when the model can't be reached, whether the
     search respects a price ceiling — anything, as long as it names a number
     or an observable outcome. -->

Given any query, trace.check_iterations(steps) never fires past the configured limit, and run_agent returns within 6 tool-call steps for every run — in 5 of 5 tries.

**Why this target:**

Counting steps and comparing against `config`'s limit is plain arithmetic with no model call involved, so the loop should never legitimately need more than the fixed number of steps `run_agent` actually takes (parse, search, select, suggest, caption). I picked 5 of 5 because if this ever fires or the step count creeps past 6, that's a real defect in the loop's control flow — not a case where I'd accept "it usually stays under the limit."

---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 4 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath it, like this:

         ## 4. Something about the fit card

         The fit card is different every time.

         **Why this target:** ...

         > **Revised in unit 4:** For 5 different items, the 5 fit cards share
         > no opening sentence.
         >
         > **Why revised:** "different" wasn't checkable — two cards that
         > differed by one word still counted. The new version is something I
         > can actually score.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "I said the empty search stops it 5 of 5 times, but I got 3 of 5,
            so 3 of 5 is more realistic."

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->

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

---

## 2. An impossible query stops before the second tool

Given a query that matches no listings, the agent stops before calling
`suggest_outfit` and returns a message naming what to change — 5 of 5 tries.

**Why this target:**
<!-- Why is 5 of 5 reasonable here when criterion 1 isn't? What's different
     about this path? -->

---

## 3. The selected item is passed intact through the session

Given a matching query, the item selected by search_listings has identical id,
price, title, and category when it reaches suggest_outfit — in 5 of 5 tries.

**Why this target:** State corruption between tools would manifest as the wrong
outfit suggestion or fit card. A direct field comparison catches if the item got
lost, modified, or swapped mid-session before it reaches the wardrobe matcher.



---

## 4. The fit card is stable for the same item

Given the same selected item and wardrobe, running generate_fit_card twice
produces captions that share at least 2 sentences or 40% of content — in 5 of 5
tries.

**Why this target:** The model is non-deterministic, so exact matches are
unrealistic. But if the same item produces wildly different cards each time, the
user can't trust the outfit suggestion. Shared content (key details, styling
advice) validates that the card is consistently relevant to the item, not random.



---

## 5. Price ceilings are enforced in search results

Given a query like "vintage tee under $30", search_listings returns only items
with price ≤ $30 — in 5 of 5 tries.

**Why this target:** A price ceiling is a hard constraint that a user explicitly
states. Ignoring it breaks trust (they see expensive items they said no to) and
wastes time. Unlike style matching, which is subjective, price is objective and
must be enforced without exception.



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

# FitFindr

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, every
> command, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> python app.py listings --full -n 6      # read the data (Milestone 1)
> python app.py fields                    # what you can filter on
> python app.py ask 'vintage graphic tee under $30'
> ```
>
> All three tools are stubs, so that last command will do nothing useful yet.
> That's the starting position.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     This is your submission. Fill each section in as you finish the milestone
     it belongs to — don't leave it all to the end.

     Unit 3 asks for the first five sections. Unit 4 adds the five below them.
     Leave the unit 4 sections alone until then; they're here so you know
     what's coming.

     Everything is pasted as TEXT. No screenshots, no images, no video links.
     A typed block of output gets full credit; a picture of the same output
     gets none.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 3 — THE BUILD ═══════════════════════ -->

## What This Does

FitFindr is a thrift shopping assistant that takes natural language queries like "vintage graphic tee under $30, size M" and searches a dataset of secondhand listings for matches. For each match, it suggests how to style the item with pieces already in the user's wardrobe, then generates a short social media caption. If the search finds nothing, it tells the user what criteria to adjust and stops before wasting model calls.



---

## Tool Inventory

<!-- Four lines per tool. This is worth 2 points and it's the single most
     common place students lose them.

     "Returns a list" earns NOTHING. The description has to say what is IN
     the list.

     The empty case isn't optional either — it's the thing your loop branches
     on, and if you don't decide it here you'll discover it as a crash in
     Milestone 5. -->

### `search_listings`

- **What it does:** Filters the listings dataset by description keywords, size, and price ceiling, then ranks results by keyword overlap with the description.
- **Inputs:** `description` (str), `size` (str | None), `max_price` (float | None).
- **Returns:** A list of listing dicts (up to `config.SEARCH_RESULT_LIMIT`), each containing id, title, description, category, style_tags, size, condition, price, colors, brand, platform. Ranked by keyword match score, best first.
- **When it has nothing:** Returns an empty list `[]` when no listings match the criteria.
- **Size matching rule:** Size matches are case-insensitive and must match size tokens exactly — split both the query and listing size by "/" and whitespace, then check if the query token appears in the listing tokens (e.g., "M" matches "S/M" and "M" but not "size M" as a substring of "us 9").

### `suggest_outfit`

- **What it does:** Given a new item and the user's wardrobe, generates one or two outfit suggestions by calling the model.
- **Inputs:** `new_item` (dict — a listing dict), `wardrobe` (dict with 'items' key containing a list of wardrobe item dicts).
- **Returns:** A non-empty string with outfit suggestions. For an empty wardrobe, returns general styling advice for the item; for a populated wardrobe, returns specific outfit combinations naming pieces the user owns.
- **When it has nothing:** Returns a non-empty string with general styling advice when `wardrobe['items']` is empty.

### `create_fit_card`

- **What it does:** Generates a two-to-four sentence social media caption describing the outfit and the new item.
- **Inputs:** `outfit` (str — the outfit suggestion from suggest_outfit), `new_item` (dict — a listing dict).
- **Returns:** A two-to-four sentence caption that reads like a real post, mentions the item title, price, and platform once each, and describes the vibe.
- **When it has nothing:** Returns a descriptive error message string if `outfit` is empty or whitespace-only.

---

## Planning Loop

<!-- Your branch rule, stated as a rule — the condition AND both paths — plus
     the file and function that holds it.

     Like this:
       "If search_listings returns an empty list, put a message in the session
        and stop. Otherwise take the first result and go to suggest_outfit."
        — agent.py::run_agent

     The grader checks your code against what you claim here, so the file and
     function have to be real. -->

**Branch rule:** If search_listings returns an empty list, put a message in the session saying what criteria to adjust (e.g., "No items match your search — try a different style, size, or price range") and stop without calling suggest_outfit. Otherwise, take the first result from the list, put it in the session, and proceed to suggest_outfit.

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** The query is parsed using regex to extract a description (all text before price/size keywords), a size (matches "S", "M", "L", "XL", "W28", etc. case-insensitively), and a max_price (extracts number after "under" or "$" tokens).

**What moves through the session:** query → parsed (description, size, max_price) → search_results (list of listings) → selected_item (first listing or None) → outfit_suggestion (string) → fit_card (string). If search_results is empty, execution stops and error is set.

---

## Sample Run

<!-- Two things go here.

     1. One FULL query and its output, pasted as text.
     2. Your three per-tool terminal tests — the command and what it printed. -->

**One full query**

```
$ python3 app.py ask 'vintage graphic tee under $30'

  Found:    Graphic Tee — 2003 Tour Bootleg Style — $24.0 on depop
  
  Outfit:   Pair the tour bootleg tee with your oversized grey crewneck sweatshirt and chunky white sneakers for a laid-back 90s aesthetic. Layer with your vintage black denim jacket for a edgier vibe when you want more structure.
  
  Fit card: Just scored this absolutely fire 2003-style tour bootleg tee from Depop for $24! The faded graphic and boxy fit are giving pure Y2K energy. Styling with my grey crewneck and white sneakers for the ultimate nostalgic fit. 🔥
```

**The three tools, tested one at a time**

```
$ python3 -c "from tools import search_listings; results = search_listings('graphic tee', max_price=30); print(f'{len(results)} results:'); [print(f\"  {r['title']} - \${r['price']}\") for r in results[:3]]"

5 results:
  Y2K Baby Tee — Butterfly Print - $18.0
  Graphic Tee — 2003 Tour Bootleg Style - $24.0
  Mesh Long-Sleeve Top — Black - $15.0
```

```
$ python3 -c "from tools import suggest_outfit; from utils.data_loader import load_listings, get_example_wardrobe; item = load_listings()[0]; wardrobe = get_example_wardrobe(); print(suggest_outfit(item, wardrobe))"

Given the vintage Levi's 501 jeans you found, here are some outfit ideas: Pair them with an oversized grey crewneck sweatshirt and black combat boots for a cozy, casual weekend look. Alternatively, layer a black cropped zip hoodie over a white ribbed tank top with the jeans and white chunky sneakers for a more athletic streetwear vibe.
```

```
$ python3 -c "from tools import create_fit_card; from utils.data_loader import load_listings; item = load_listings()[0]; outfit = 'Pair with a vintage graphic tee and white sneakers for a classic look.'; print(create_fit_card(outfit, item))"

Found this killer pair of Levi's 501s on Depop for just $38! The medium wash is absolutely perfect — light fading at the knees adds to the vintage charm. Styling with a vintage graphic tee and white sneakers for maximum 90s energy. Pure thrift magic! ✨
```

---

## How I Used AI

<!-- Two specific moments. What you asked, what came back, what you changed.

     "I used Claude to help me code" is not enough.

     "I gave Claude my search_listings spec. It returned None on no match
     instead of an empty list, so I changed it" is the level we want. -->

**Moment 1**

- *What I asked for:*
- *What came back:*
- *What I changed:*

**Moment 2**

- *What I asked for:*
- *What came back:*
- *What I changed:*

<!-- ═══════════════════════ UNIT 4 — THE TEST ═══════════════════════

     Don't fill these in during unit 3.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Run Log — Before

<!-- Five criteria, five tries each, in this exact format.

     Five, because your criteria are written out of five. Mark each try PASS
     or FAIL, count the passes, and read that count against your target — a
     row targeting 4 of 5 with three PASS cells is MISSED (3/5).

     `python run_eval.py --label before` runs everything and writes the table
     into results/. Paste it here and fill in the verdicts. -->

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |  |  |
| 2.  |  |  |  |  |  |  |  |
| 3.  |  |  |  |  |  |  |  |
| 4.  |  |  |  |  |  |  |  |
| 5.  |  |  |  |  |  |  |  |

**Real output from one try**, pasted as text, naming the file and function
that produced it:

```

```

---

## Verdicts and Diagnoses

<!-- MET or MISSED per criterion against LAST UNIT's target, plus a sentence on
     how you decided.

     Then, for every miss: which of the four places it happened — a tool, the
     loop's branch, the session, or the model's output — AND the mechanism.

     Not a diagnosis:  "The fit card was bad."
     A diagnosis:      "The fit card criterion missed on 2 of 5 items. Both had
                        an empty brand field. My prompt puts the brand in the
                        first sentence, so the card opened with a blank and read
                        like a fragment. The tool worked; the prompt assumed a
                        field that isn't always there."

     Look for a pattern. Three misses on the same tool is one problem, not
     three. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**



---

## Loop Trace

<!-- One full run, printed step by step, with the MCP call visible in it.

     `python app.py ask '...' --trace` once you've added the trace.step()
     calls in Milestone 2.

     Worth pasting BOTH the happy path and the empty-search path. The empty
     one should be visibly shorter, because it stops. If your two traces are
     the same length, your branch isn't working — and this is the fastest way
     anyone will ever find that out. -->

**Happy path**

```

```

**Empty search**

```

```

**On the MCP move:** <!-- what changed in your code, and whether anything
behaved differently afterwards. If the rewire didn't work, say exactly where it
broke — the error text and the last thing that worked. That earns the point in
full. -->



---

## The Improvement

<!-- What you changed, why your diagnosis pointed at it, and the after-run in
     the same table format. One change, measured properly.

     `python run_eval.py --label after` -->

**What I changed:**

**Which failure it was meant to fix:**

### Run Log — After

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |  |  |
| 2.  |  |  |  |  |  |  |  |
| 3.  |  |  |  |  |  |  |  |
| 4.  |  |  |  |  |  |  |  |
| 5.  |  |  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it made things worse, say that. Honestly reported, that earns full
     credit and is more interesting than one that worked. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped where
     you did. "I ran out of time" is fine if it's true. Pretending nothing is
     left is not. -->



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 3

       [ ] criteria.md has five numbered criteria, each with a target
       [ ] Each criterion has a reason underneath it
       [ ] All five unit 3 sections above have real content
       [ ] Tool Inventory: all three tools, inputs WITH TYPES, a specific
           return value, and the empty case
       [ ] Planning Loop names the branch rule and agent.py::run_agent
       [ ] Sample Run: one full query plus the three per-tool tests, as text
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN, you submit the same one
           next unit

     SUBMISSION CHECKLIST — unit 4

       [ ] mcp_server.py exists with one tool registered
           (or a written record of exactly where the rewire broke)
       [ ] Run Log — Before, five criteria, five tries each
       [ ] Real output pasted underneath, naming file and function
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, naming a place AND a mechanism
       [ ] Loop Trace, with the MCP call visible in it
       [ ] All three failure modes triggered and handled
       [ ] One improvement, with Run Log — After in the same format
       [ ] What's Still Broken
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository. Your commit history is what
     shows your criteria existed before your results did.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**

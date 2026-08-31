# TEMPO Scheduler — Book the Resident, Not the Day

A booking guide for the **front-desk scheduler** of an Internal Medicine residency
continuity clinic.

You do not need to know anything about medicine or computers to use it. You ask it three
questions — *who is in clinic today?*, *when can I book Dr. X?*, *how do I make today work?* —
and it answers in one plain sentence, every time, from the residency block schedule that is
built into it.

Everything runs inside the browser. There is no login, no server and no database.

**No patient information, ever.** The tool works in counts — "6 follow-ups", "3 urgent" —
and never asks for a name, a date of birth or a medical record number. Nothing about a
patient is stored anywhere.

---

## Getting started

Open `index.html` in any browser, or visit the deployed site. Then do this once:

1. Go to **Settings**.
2. Under every `E` and `A` block in the roster table there is a small weekday box. Set it to the
   day that resident actually comes to clinic during that block. Leave it on `—` for any block
   the programme has not decided yet.
3. Check the **firm partners** and **lane labels** match how your clinic actually runs.
4. Press **Save settings**.

That's it. Your settings stay in this browser until you change them.

### The tool never guesses a clinic day

This matters more than anything else in the app, so it is worth being blunt about.

A resident on an `E` or `A` block comes to clinic **one weekday a week**, and the programme
usually decides which weekday only about a month ahead. If a tool invents that day, it will
confidently tell you a resident is free on a day they are on service — or, worse, tell you they
are unavailable on a day they are actually sitting in clinic.

So TEMPO does not invent it. Until somebody types the day into Settings, that resident shows as
**"clinic day not decided yet"**. They are listed separately on Tab 1, they are **not** counted in
the day's totals, they never appear in the heatmap counts, and Tab 2 says *"the clinic day has not
been set"* rather than *"no clinic"*. A blank is always safer than a guess.

The clinic day is stored **per block**, because it changes when the rotation changes. If a
resident has the same day all year, the **Set every E/A block to** column at the end of their row
fills all of them in one go.

---

## The five tabs

| Tab | What it answers |
|---|---|
| **1 · Who's in clinic?** | Pick a date → the block, the weekday, and two columns of the residents available, each tagged FULL WEEK, *elective day (Wed)*, or *away — confirm*. Plus the day's numbers: residents, template slots, target total, supervision cap, and room for advance bookings. |
| **2 · When can I book Dr. X?** | Pick a doctor and roughly how far out ("12 weeks, ± 2"). You get their whole year as a colour strip, a list of their real clinic dates in that window, and one sentence telling you which date to book. If they have no clinic at all in that window, it gives you the nearest dates on either side and what to do about it — it never returns "no result". |
| **3 · Year at a glance** | Every weekday from July to June, coloured by how many residents are in clinic. Hover for the names, click to open that date in Tab 1. Use it to steer batches of bookings toward the quiet days months ahead. |
| **4 · Build today's day** | Type in what is already booked and who is waiting. Press **BUILD THE DAY** and the tool caps, releases, balances and staggers the day, then hands you a real grid, a numbered move list in plain sentences, and warnings. Print it or copy it. |
| **5 · Someone called out** | Pick the absent resident (or "an attending dropped") and the tool shows exactly where each patient goes, in order, and re-draws the day with the changes highlighted. |

Every tab has a **Rule cards** panel at the bottom explaining, in plain English, how the tool
decided what it decided. Every computed number shows its rule beside it — for example
`cap = 3 attendings × 32`. It is a guide, not a black box.

---

## The availability rule (O / N / E / A)

Each resident has one rotation per block, written as a single letter. The letter tells you
the clinic pattern:

| Code | Rotation | Clinic |
|---|---|---|
| **O** | Outpatient block | In clinic **every weekday** of that block |
| **N** | Inpatient, night float, ICU | **No clinic at all** that block |
| **E** | Everything else — elective, SNF, ED Obs, community / ILE | **One fixed weekday** each week — you set it in Settings, per block |
| **A** | Away rotation — BUMCP, Mayo, Gateway, NIH | Treated like **E**, but flagged **"away — confirm"**. Toggleable in Settings |

`availabilityFor(resident, date)` returns one of four values:

| Value | Meaning |
|---|---|
| `FULL` | Outpatient block — in clinic today |
| `ELECTIVE_DAY` | `E`/`A` block and today matches the weekday you entered |
| `ELECTIVE_TBD` | `E`/`A` block, but **nobody has set the weekday yet** — might be in clinic, unknown |
| `NONE` | Not in clinic today |

`residentsOnDate(date)` returns only the residents **known** to be in clinic — `FULL` plus
`ELECTIVE_DAY`. `residentsUndecidedOnDate(date)` returns the `ELECTIVE_TBD` ones so the scheduler
can chase them up. Nothing that is merely *possible* is ever counted as *booked*.

A resident on their elective day gets the **full template** for that weekday — exactly the
same as an outpatient resident.

### Academic year 2026–27 block dates

| | | | |
|---|---|---|---|
| B1 Jul 1 – Jul 28 | B2 Jul 29 – Aug 25 | B3 Aug 26 – Sep 29 | B4 Sep 30 – Nov 3 |
| B5 Nov 4 – Dec 1 | B6 Dec 2 – Dec 29 | B7 Dec 30 – Jan 26 | B8 Jan 27 – Feb 23 |
| B9 Feb 24 – Mar 23 | B10 Mar 24 – Apr 27 | B11 Apr 28 – Jun 1 | B12 Jun 2 – Jun 30 |

Dates outside July 1, 2026 – June 30, 2027 get a friendly "outside this year" message rather
than a wrong answer.

---

## Slot classes

| Class | Colour | Rule |
|---|---|---|
| **Est** | blue | Follow-ups. Bookable any time. |
| **New** | teal | New patients only **until 14 days before** the date. After that it becomes an Est slot. |
| **FROZEN** | red outline | Hidden **until 2 days before**. Then released for urgent, hospital-discharge or waitlist patients only. |
| **HospFU** | amber | Hospital follow-up only, **within 14 days of discharge**. |
| **Any** | purple | PGY-1 slots. Anything, always 60 minutes. |

### Clinic template

Hours 8:00 AM – 4:30 PM last start. **Lunch 12:00–1:00 is never bookable.**

**PGY-2 — 30-minute slots with a class**
- Mon / Tue / Wed (16): 8:00 Est · 8:30 Est · 9:00 New · 9:30 Est · 10:00 FROZEN · 10:30 Est · 11:00 New · 11:30 Est · *lunch* · 1:00 HospFU · 1:30 Est · 2:00 New · 2:30 Est · 3:00 FROZEN · 3:30 Est · 4:00 New · 4:30 Est
- Thursday (8): **morning only** — the AM half of the above
- Friday (14): **starts at 9:00** — as Mon–Wed without 8:00 and 8:30

**PGY-1 — 60-minute "Any" slots**
- Mon / Tue / Wed (8): 8, 9, 10, 11, 1, 2, 3, 4
- Thursday (4): 8, 9, 10, 11
- Friday (7): 9, 10, 11, 1, 2, 3, 4

Templates are editable in Settings, one line per slot (`8:00 Est`).

---

## Capacity formulas

| Number | Formula |
|---|---|
| **Target** per resident per day | PGY-2 **12** Mon–Wed / **6** Thu / **10** Fri · PGY-1 **5** / **3** / **4** |
| **Pre-book ceiling** | `floor(target × 70%)`. Advance bookings may never exceed it. The held-back 30% is Frozen + HospFU + same-day flex. |
| **Supervision cap** | `attendings × 32` (× **16** on Thursday, morning only) |
| **Day bookable total** | `min(Σ targets, supervision cap)` |
| **Hard maximum** per resident | the number of slots in the template. Never exceeded, and slots are never shortened. |

All of these are editable in Settings.

### Continuity levels

- **Must see own doctor** — never moved.
- **Prefer own team** — may move to the firm partner only.
- **Any doctor** — freely movable.

A **firm** is one PGY-2 paired with one PGY-1, sitting in one attending's lane. Pairs and lane
labels are set in Settings (default: by roster order).

### What BUILD THE DAY does, in order

1. **LOCK** — hold "must see own doctor" and "prefer own team" where they are.
2. **CAP** — apply targets and `attendings × 32`. If over, list who to move to their **next
   clinic date**; "any doctor" first, PGY-1 slots protected, "prefer own team" only if still over.
3. **RELEASE FROZEN** — discharges → HospFU/Frozen, urgent → Frozen, then waitlist and new
   patients into open slots, firm by firm.
4. **BALANCE** — shift "any doctor" patients from over-target residents to an under-target
   firm partner, then the same lane. Never pushes anyone above target.
5. **STAGGER** — group into attending lanes and offset start times by 0 / 10 / 20 minutes so
   the attending is never asked for three patients at once.

### When someone calls out

"Must see own doctor" → firm partner, up to **2 over target**; anyone left goes to that
doctor's own next clinic dates. "Prefer own team" → partner, then the same lane.
"Any doctor" → any open slot today. Whoever is left goes on the **nurse reschedule list**.

---

## Updating the block codes when the schedule changes

The block schedule changes. You do not need a programmer to keep up with it.

**The easy way — in the app.** Open **Settings → Roster and block schedule**. Every resident
has twelve boxes, left to right, one per block. Type `O`, `N`, `E` or `A` into any box and
press **Save settings**. The weekday box underneath each block lights up whenever that block is
`E` or `A`; set it when the programme tells you the day, and leave it on `—` until then. You can
also mark a resident **inactive** (they disappear from every tab), and change their firm partner
and lane.

Use **Export settings (JSON)** to save a copy, and **Import settings** to load it into another
browser or share it with a colleague. **Reset settings** puts everything back to the built-in
defaults.

**The permanent way — in the file.** Settings live in one browser only. To change the codes
for everybody, edit the `ROSTER_PGY1` and `ROSTER_PGY2` arrays near the top of the `<script>`
block in `index.html`. Each line is `['Full Name','NENENONENENE']`, where the twelve letters
are Blocks 1 → 12 in order. Block dates are the `BLOCKS` array just above them.

**At the end of June**, when a class graduates: update the roster arrays with the new names,
and give each graduating resident's panel to a named successor before you remove them.

---

## Deploying to Vercel

The app is one static file, so there is nothing to build.

1. Push this repository to GitHub.
2. In Vercel, **Add New → Project → Import** this repository.
3. Framework preset: **Other**. Build command: **none**. Output directory: **`.` (the root)**.
4. **Deploy.**

Any other static host works the same way — GitHub Pages, Netlify, or just opening
`index.html` from a USB stick.

---

## Checking it still works

Nine acceptance tests are built into the page and run every time it loads. Open the browser
console (or add `?selftest=1` to the URL to see them on the page):

1. Block dates map correctly, and dates outside the year give a friendly message.
2. `availabilityFor` follows O / N / E / A for a real resident across four blocks.
3. `residentsOnDate` lists everyone in clinic on a Wednesday in B3.
4. Thursday is morning only; Friday has no 8:00 or 8:30; nothing lands in the lunch hour.
5. A window with no clinic at all still returns the nearest dates either side.
6. One attending and 60 booked → cap 32 → 28 moves, "any doctor" first, PGY-1 untouched.
7. The call-out cascade sends must-see patients to the partner up to 2 over target and lists
   the rest with their next clinic dates.
8. The heatmap draws every weekday of the year with counts and hover names.
9. A resident whose clinic day is undecided is never guessed onto a date, never counted in a
   day's totals, and Tab 2 says "not decided yet" rather than "no clinic" — and setting the day
   adds exactly that one resident.

All nine report `PASS`.

---

## Files

- `index.html` — the whole app: markup, styles, data and logic in one file. No frameworks, no build step, no dependencies.
- `README.md` — this file.

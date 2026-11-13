# Registration Trainer

A practice copy of the University of Richmond BannerWeb registration pages
(Spring 2027), so you can rehearse registration morning: wait for 7:00:00,
open Register for Classes, go to Plans, Add All, Submit — and recover when
something goes wrong.

It is **not connected to BannerWeb**. Nothing you do here registers you, and
no data leaves your browser. The university crest is replaced by a plain
"PRACTICE" badge on purpose.

## Running it

Double-click `index.html`. No install, no server.

- **Start** begins a timed *Clean load* run, 10 seconds before 7:00.
- During a run the page fills the window like the real one. Press **Esc**
  (or double-click the PRACTICE badge) for the timer, Hint, Restart and Exit.
- The dark strip at the bottom is a stand-in for your computer's clock and
  shows the practice time.
- Your last runs are listed on the start screen (saved in this browser only).

Internet is only used for the Open Sans font; offline, headings fall back to
Arial and everything else works.

## Changing things

Everything you would normally want to change is in **`js/config.js`**:

| Setting | What it does |
|---|---|
| `MY_PLAN` | Your plan: subject, number, section, CRN, title, days, times, units. Leave `crn: ''` for a course saved without a section. |
| `PLAN_NAME`, `STUDENT_NAME` | Text shown in the Plans tab and header. |
| `START` | What the Start button runs: `mode` (`timed`/`guided`), `scenario`, `startBeforeOpen`, `rush`. |
| `SCENARIOS` | Names, briefs and target times of the practice scenarios. |
| `SERVER_DELAY_MS` | How long each click "waits for the server". These are guesses, not measurements. |
| `MAX_HOURS`, `TIME_TICKET_*` | Registration limits and the time-ticket message. |

> Only PHIL 120's meeting time is confirmed. The times for CMSC 221,
> CMSC 222 and MATH 300 in `MY_PLAN` are placeholders — fix them for
> realistic calendar and time-conflict behaviour.

### Scenarios

| Key | What goes wrong |
|---|---|
| `clean` | Nothing. Load the plan and submit. |
| `planahead` | Build the plan in Plan Ahead and save it with a name. |
| `hold` | Financial Responsibility Agreement not accepted yet. |
| `closed` | One planned section is full; find another that fits. |
| `nocrn` | One course was saved without a section; Add All skips it. |
| `prereq` | One course fails with a prerequisite error. |
| `waitlist` | One section is full but has waitlist seats. |
| `swap` | Already registered; switch sections with Conditional Add and Drop. |
| `gauntlet` | `hold` + `closed` + `nocrn` together. |
| `random` | One of the above, without telling you which. |

## Project structure

```
index.html              page shell; loads the CSS and scripts in order
css/
  base.css              shared basics and the start screen
  banner.css            the BannerWeb replica (header, tabs, tables, panels, dialogs)
  trainer.css           trainer overlays: Esc bar, hint box, spinner, results, taskbar
js/
  config.js             plan, scenarios, delays, limits  ← edit this one
  util.js               small helpers (escaping, time formatting, overlap checks)
  storage.js            run history in localStorage
  state.js              run state, practice clock, log/mistake/notify, fake catalogue, scenarios
  registration.js       Summary rules, Submit, finishing and grading a run
  hints.js              works out the next correct click
  views/
    shell.js            page frame, breadcrumbs, notifications, window fitting
    screens.js          home tiles, landing, Select a Term, Prepare, info pages
    register.js         Register for Classes (search, CRNs, Plans, Summary, calendar)
    plan-ahead.js       Plan Ahead (Build your plan)
    dialogs.js          Class Details, agreement, Name Your Plan
    trainer.js          start screen, Esc bar, hint box, results window
  actions.js            what each click does (with the fake server delay)
  events.js             DOM listeners: click, change, input, keyboard
  main.js               boot
tests/
  smoke.test.js         plays every scenario by following the hints
```

The scripts are plain `<script>` files rather than ES modules so the page
works when opened straight from disk (browsers block modules on `file://`).
They share top-level names, so the load order in `index.html` matters.

### How it works, briefly

- `state` (in `state.js`) holds the whole run. Every click goes through
  `performAction()` → `handleAction()` (in `actions.js`), which changes
  `state`; the page is then redrawn from `state` by `renderRun()`.
- The replica is laid out at a fixed 1674px "design width" measured from
  real screenshots, then zoomed to fill the window (`fitBanner()`), so the
  proportions match the real page at any window size.
- Clickable elements that hints can point at carry `data-hint="…"`.
  `nextHint()` returns the key of the next correct click; the hint box and
  the test both use it.

## Testing

```
npm install playwright
node tests/smoke.test.js
```

It plays all nine scenarios from start to finish, following the trainer's own
hints, and fails if one gets stuck or the page throws an error.

## Accuracy notes

Built from the Registrar's *Student Registration Guide*, the Registrar's
BannerWeb tutorial videos, and screenshots of the real pages.

- Layout and labels match those sources closely; exact error wording and
  server timing are approximations.
- The *Select A Plan* page isn't shown in any source and is reconstructed.
- The Financial Responsibility Agreement step comes from an older UR guide
  and may no longer apply.

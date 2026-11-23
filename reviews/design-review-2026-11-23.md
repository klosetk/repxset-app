# Rep Counter design review (Nov 23, 2026)

**Scope:** I reviewed the current `rep-counter.html` in headless Chromium at 390x844 (DPR 1), in light and dark mode, and read the source. Flows covered: fresh onboarding (all 5 steps), Train for strength (Upper body), Cardio (Jump rope + Rowing machine) and Stretching (Pigeon pose, both sides), every workout phase (Form check, Get ready, Work, Paused, Rest, Rest with CC, End confirm), Done, Saved, Analytics, Profile, Inbox (test message, unanswered and answered), and the check-in banner on Home. No JS errors in either theme.

**Not verified:** real audio playback, reminder timing on a real device, VoiceOver/TalkBack.

**Not repeated:** items from the Nov 22 review that are now fixed (focus loss, switch names, unit conversion, Profile-to-plan sync, caption overlap, PR timing, Enter on onboarding, and others). A few Nov 22 items are still open. I list them once at the end and don't count them below.

---

## High

### 1. Check-ins only arrive while the app is open, but setup promises "the way a real coach would"
- **Where:** Onboarding step 3 "When do you train?", Profile > Gym schedule, Inbox.
- **Problem:** `checkReminders()` runs only on an interval while the page is open. If the app is closed at 5:30 PM, the user hears nothing. The message appears the next time they open the app, which is usually after the moment has passed. The headline copy ("Your coach will check in before each one") sets an expectation the product can't meet. The caveat sits in 13px fine print under the sticky button. On onboarding step 3 that line is cut off behind the Next bar at 390x844.
- **Fix:**
  - Add a reliable channel. The simplest is **"Add to my calendar"**: an .ics file with a weekly repeat and a 30-minute alert per gym day, which works today with no push setup.
  - If web push is possible in the host, ask for notification permission *on this step*, right after the first day is picked.
  - Rewrite the sub-headline to be honest: "Your coach checks in 30 min before each session when you open the app. Add it to your calendar so you never miss one."

### 2. Equipment step is meaningless for Cardio and Stretching
- **Where:** Plan > Step 3 "Choose equipment" for timed categories.
- **Problem:** Jump rope offers Weights / Cable machine / Resistance band / Kettlebell / Machine / None. Pigeon pose offers the same six chips. Equipment doesn't change anything for timed moves (no weight field, no PR logic). It's an extra screen of 6 irrelevant chips per move, and the step counter still says "Step 3 of 4".
- **Fix:**
  - When every item is timed, skip the equipment step and show "Step 3 of 3".
  - For mixed plans, show equipment cards only for rep-based moves.
  - Move the form tip that lives on this screen to the "Sets and time" card as a collapsible "How to do it".

### 3. Two-sided stretches count down one combined clock, so the side switch is easy to miss
- **Where:** Workout, Stretch phase (e.g. Pigeon pose, "1 min (both sides)").
- **Problem:** The hero clock runs 1:00 to 0:00 across both sides. The only change at the halfway point is the small label "First side" becoming "Second side", one rep tone and an optional voice line. With voice off, or with the phone on the floor during a stretch, the switch is easy to miss. The plan copy "1 min (both sides)" is also ambiguous: 1 minute each, or in total?
- **Fix:**
  - Count each side separately: 0:30 per side, with the clock resetting at the switch.
  - Show a large "LEFT" / "RIGHT" side label at 30px+ in place of "First side".
  - Give the switch its own visual moment: a full-width color flash or a 3-2-1 countdown, matching the end-of-set countdown.
  - In the plan, label it "30 sec each side".

### 4. Rep count is still secondary to elapsed time during a set (carried over, still the biggest readability gap)
- **Where:** Workout, Work phase (strength).
- **Problem:** At arm's length the 170px clock shows elapsed time ("0:01"), which the user doesn't act on. The number they need (reps done or left) is 56px. The progress dots are 14px with a 1.2:1 outline on the light background, so they're effectively invisible from a bench.
- **Fix:**
  - Swap the hero: show the rep number at ~170px ("3", with "of 10" underneath at 30px), and move elapsed set time into the meta line or drop it.
  - Make the dots 20px, and give the unfilled ones a 3:1 border (e.g. `--muted` at 60%), or remove them, since the beat bar already shows the tempo.

---

## Medium

### 5. Control outlines are nearly invisible (WCAG 1.4.11)
- **Where:** All `.ghost` buttons and unselected `.chip`s in both themes: Finish set, End workout, Skip rest, No / I'm going tomorrow, Hear this coach, the onboarding chips, and the stepper buttons.
- **Problem:** The border `--line` is 1.21:1 on the light background and 1.35:1 on the dark one. WCAG asks for 3:1 for control boundaries. On the workout screen, the two secondary buttons read as floating text, not tappable buttons.
- **Fix:**
  - Give ghost buttons a filled `--surface` background plus a 3:1 border (light ~`#9AA3B2`, dark ~`#4A5366`).
  - Keep `--line` for card dividers only.

### 6. "Voice on" / "CC" / "Sound on" header chips are confusing and visually loud
- **Where:** Workout header.
- **Problem:**
  - Two filled-blue pills compete with the phase label and exercise name for attention.
  - "Sound" mutes everything, including the voice. Yet "Voice on" stays filled when Sound is off, so the chips contradict each other.
  - "CC" is the only abbreviation.
  - The chips are 40px tall (below 44px).
- **Fix:**
  - Collapse the three chips into one 44px speaker icon button showing the current state. Tapping it opens a small sheet with three rows: Coach voice, Subtitles, Timer sounds.
  - If they stay inline, use outline style for "on" (not solid accent), and show Voice as disabled when Sound is off.

### 7. "Form check" phase has no time cue and the wrong primary action
- **Where:** Workout, first screen after Start ("FORM CHECK").
- **Problem:** The clock is hidden and there's no progress indicator, so the user doesn't know whether to wait or tap. The full-width primary is "Pause", while the thing they likely want ("I'm ready") is the small secondary "Skip". The bottom 40% of the screen is empty.
- **Fix:**
  - Show a thin progress bar tied to the clip length, labeled "Starting in ~12s".
  - Make the primary button "I'm ready" and demote Pause.
  - Put the exercise's set/rep target in large type in the empty space, so the brief doubles as a preview.

### 8. Rest has no "+15s" option and repeats the next-set info
- **Where:** Workout, Rest phase.
- **Problem:** Users can only skip rest, not extend it, which is the common need on heavy sets. The meta line under the exercise name already reads "Set 2 of 3", and the "Up next: set 2 of 3" line below the clock repeats it.
- **Fix:**
  - Replace the "Up next" line with a "+15s" chip next to the clock.
  - Keep the next-set info only in the meta line, prefixed "Next:".

### 9. Long cardio sessions can't be adjusted mid-session, and the controls use strength wording
- **Where:** Workout, Cardio "GO" phase (e.g. Rowing machine 10 min, Outdoor run 20 min).
- **Problem:**
  - There's no way to add time or extend once started.
  - "Finish set" and "Set 1 of 1" are odd for a single 20-minute run.
  - The meta line reads "None · Set 1 of 3 · 1 min", which puts the equipment "None" first.
- **Fix:**
  - For timed moves, rename the buttons "Done" / "+1 min".
  - Hide "Set 1 of 1" when sets = 1.
  - Omit equipment when it's "None". For rep moves, say "Bodyweight".
  - The same "None" label appears on the Sets and time cards; drop it there too.

### 10. "Yes" in the check-in leads nowhere
- **Where:** Inbox, after answering Yes.
- **Problem:** The coach replies, and then the conversation dead-ends. The user said they're going to the gym, but the inbox offers nothing to start a workout.
- **Fix:** Under the coach's reply, add a primary button: "Repeat last workout" if one exists, otherwise "Plan today's workout". Add a secondary "Open saved" when saved workouts exist.

### 11. "I'm going tomorrow" gives no confirmation, and the header contradicts it
- **Where:** Inbox, after answering Tomorrow.
- **Problem:** The code books a one-off check-in for tomorrow, but the UI never says so. The sub-header still reads "Next session: today at 6:00 PM" right after the user said they aren't going today.
- **Fix:**
  - Add a small system line under the reply: "Check-in set for Tue 6:00 PM".
  - Update the sub-header to "Next session: tomorrow at 6:00 PM".
  - Offer a one-tap "Change time".

### 12. "No" replies lean on guilt
- **Where:** Inbox coach replies (CHECKIN.no), e.g. Shimmer: "that stings a little"; Onyx: "I'm a little disappointed".
- **Problem:** Guilt from a coach character can push people to disengage with reminders, or turn them off entirely. That risk is highest on the "Only when I slip" path, which only reaches people who are already struggling. There's also no way to give a reason.
- **Fix:**
  - After "No", show quick reasons: Sick or sore / Too busy / Need a rest day.
  - Reply supportively per reason ("Rest is training too. See you Wednesday.").
  - Keep the playful disappointment for the "Hype" coach only, or soften it across all coaches.

### 13. Onboarding asks for too much before the first workout
- **Where:** Onboarding steps 1–5.
- **Problem:**
  - Five required screens.
  - Step 2 adds a 4-part "How goals work" explainer (~130 words) about features the user hasn't seen. It pushes the "Pick at least one goal to continue" hint below the fold, so the disabled Next button looks broken.
  - Step 3 pre-selects "Every session" reminders before any day is chosen, and the button says "Next" even though the step is optional.
- **Fix:**
  - Cut the explainer to one line with a "Learn more" disclosure.
  - Put the disabled-state hint directly above the sticky button.
  - On step 3, hide the reminder options until a day is picked, and label the button "Skip for now" when no days are selected.
  - Consider merging Coach + Sound into one step (pick coach, with voice and subtitle toggles below it).

### 14. Analytics ignores timed work and the schedule
- **Where:** Analytics > Totals, Consistency.
- **Problem:**
  - Totals shows Workouts / Time / Sets / Reps, so a cardio or stretching user sees "0 Reps" forever. Their minutes are only counted on the Done screen ("Timed").
  - Consistency shows days trained but doesn't compare them with the gym days the user scheduled.
- **Fix:**
  - Swap Reps for "Active time" when timed work exists, or add it as a fifth stat.
  - On the Consistency week row, outline scheduled days and fill trained ones ("2 of 2 planned sessions").

### 15. Home doesn't use what it knows about the user
- **Where:** Home (Train tab).
- **Problem:**
  - After onboarding, the screen is still mostly brand: a plates graphic, a 72px "Rep Counter" mark and a marketing sentence.
  - The user's next session, any pending check-in and their last workout are missing or tiny. The last workout is the muted unlabeled line "Pigeon pose" under the Repeat button.
  - The bell sits on its own row, so the tabs float mid-screen.
- **Fix:**
  - Pin the tabs to the top, or move them to a bottom nav, and put the bell in the same header row.
  - Replace the logo block with a "Today" card: "Monday · 6:00 PM session" plus "Repeat: Bench press, Bicep curl" with a label ("Last workout").
  - Keep the brand mark for onboarding only.

---

## Low

### 16. Captions shift the workout layout
- **Where:** Workout (Rest, Work) with CC on.
- **Problem:** The caption slot sits between the clock and the rep count or tip, with no reserved height. Each caption pushes the content below it down by ~70px and back, which makes the rep count jump during a set.
- **Fix:** Reserve a fixed min-height for the caption slot (2 lines, ~64px) whenever subtitles are on, or overlay the caption above the control bar.

### 17. Exercise picker: the "Add your own" card takes the top slot
- **Where:** Plan > Step 2 "Pick exercises".
- **Problem:** A rarely used custom-entry form sits above a 40-item list and pushes the first exercise below the halfway point.
- **Fix:** Turn it into a single "+ Add your own exercise" row at the end of the list (and above it when the search or filter returns nothing). Expand it inline on tap.

### 18. Rest chips wrap and the label is too narrow for what it controls
- **Where:** Sets and reps / Sets and time > "Rest between sets"; Profile > Workout defaults.
- **Problem:**
  - Five chips wrap "2:00" onto its own row at 390px.
  - For a cardio plan of 1 x 10 min moves, "between sets" is wrong: the rest actually runs between exercises.
- **Fix:**
  - Use a 5-segment control that fits on one row.
  - Label it "Rest between sets and moves".

### 19. Coach and switch patterns differ between onboarding and Profile
- **Where:** Onboarding step 4 vs Profile > Coach; Sound toggles.
- **Problem:**
  - Onboarding shows coaches as cards with descriptions; Profile shows bare name chips, with the blurb for the selected coach only.
  - The Coach voice switch reads "Voice on/Voice off" while every other switch reads "On/Off".
  - The preview is a separate full-width button instead of a play control on each coach.
- **Fix:**
  - Use the same coach card component in both places, with a 44px play button on each card.
  - Make all switches read On/Off, with the row label carrying the meaning.

### 20. Small accessibility gaps
- **Problems:**
  - The Inbox "waiting" dot is a `<span aria-label>` with no role, so screen readers skip it.
  - The check-in banner has no dismiss control and covers the tab bar for 20s.
  - The "REST" phase label is 16px bold in `--rest` (#0B8F72, 3.67:1 on the light background), below 4.5:1 at that size.
  - The rest clock in the same color passes only because it's very large.
- **Fix:**
  - Give the dot `role="img"`, or add visually hidden "Unanswered" text.
  - Add a close button to the banner and drop its auto-hide to ~8s.
  - Darken the light-mode `--rest` to ~#087A61 (≥4.5:1).

---

## Still open from Nov 22 (not counted above)
- Home tabs float mid-screen (folded into #15).
- Header chips are 40px.
- Analytics stacks seven empty cards on first run.
- A 0-set workout still shows "Done." with 0/0/0.
- Exercises can't be reordered.

## What's working
- **Strong typographic system.** Barlow Condensed numerals with tabular figures give big, steady, non-jittery clocks. The color shift (accent for work, green for rest) reads instantly from across a room.
- **Captions as an in-flow slot** with a speaker label ("ONYX") instead of a floating toast. Rep numbers are filtered out of the captions, so they don't flicker every rep.
- **Inbox chat pattern.** A voice bubble with play/stop, answer buttons that don't move under your finger while audio plays, and replies played in that message's own coach voice. "Send me a test message" in Profile is a great way to preview the feature.
- **Goal tabs with colored dots** on exercises, plus "x of y exercises fit your goals". This makes onboarding goals feel useful right away.
- **Timed-flow basics are right.** Cardio and stretch moves get sensible defaults (Jump rope 3 x 1 min, Rowing 1 x 10 min), the volume screen retitles itself "Sets and time", and timed sets end with a 3-2-1 countdown.
- **Dark mode is genuinely designed,** not inverted: a lighter accent with dark text on primary buttons, and accessible muted text (about 6:1 on the background in both themes).
- **Safe destructive actions:** two-tap End workout, and a Clear analytics confirm with a real Cancel and a plain-language consequence.

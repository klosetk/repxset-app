# Rep Counter product review (Nov 23, 2026)

Based on a read of `rep-counter.html`, `coach-lines.json` and the Nov 22 UX review. I did not run the app on a phone.

## 1. Who it's for and why it matters

Rep Counter is for casual-to-intermediate gym-goers who train alone, don't want to think between sets, and want company more than spreadsheets. The value prop: **"A coach in your ear who reads the form tip, counts the reps, times the rest and notices when you do well, so you just lift."** The voice coaches (with real personality, plus PR and heavy-lift awareness) set it apart. Strong, Hevy and similar loggers beat it on data, and it can't win that fight yet. It should win on *feeling coached*.

## 2. What's strong

- **The coach is the product, and it's well made.** The lines have real tone ("Stopped a few short? That's okay"), there are three different personalities, and the rest lines never repeat back to back. The early-end and short-set lines are kind, with no shame. That matters for retention.
- **Hands-free flow:** tip, then countdown, rep calls, rest, "last set coming up", next exercise. That sequence is the core loop, and it holds together.
- **Accountability loop:** the schedule check-in scripts change with consistency (new / streak / slip / missed). That's thoughtful behavioral design.
- **Breadth for v1:** 6 categories, timed holds with a switch-sides cue, goal tags, saved workouts, wake lock, and an offline localStorage mirror.

## 3. Gaps and risks, ranked by impact on retention and the first week

1. **Reps are counted by a metronome, not by the lifter.** `loop()` calls rep N every `pace` seconds and ends the set at the target. Real sets speed up and slow down (grinding the last reps, pauses at the top). If the coach says "10" while you're on rep 7, trust breaks on day one. Fix: make the count follow the lifter (tap anywhere for each rep, or a "slow the count" mode that holds near failure), and make the logged reps editable after every set. **Biggest risk in the app.**
2. **Reminders don't arrive while the app is closed.** The check-in only fires if `checkReminders()` runs, which means the app is open. Onboarding asks for a schedule and reminder frequency, which promises a nudge. In the artifact setup the nudge will mostly not arrive, and users will feel misled. Either say so plainly in onboarding ("You'll see this when you open the app") or prioritize the PWA with web push.
3. **One weight for all sets, and no per-set edits.** Logging can't record a 135/155/175 pyramid or a drop set. History is "sets × reps @ one weight". That makes PRs and progress charts unreliable for exactly the people who care about them.
4. **No "what should I lift today?" guidance.** Weights start from the last plan, but there's no "last time: 3×10 @ 50 lb, try 55". Progressive overload suggestions are the #1 reason people keep using a log after week two. The data is already there in `logs` and `rc-best`.
5. **A workout holds only one category.** Changing category clears your picks. Real sessions mix them: upper + core, strength + a stretch cooldown. Your partner will hit this in the first session.
6. **Missing editing basics:** you can't reorder exercises, edit or delete a past workout, add a set mid-workout, swap an exercise when the machine is taken, or undo a "Finish set". Every one of these is a support ticket waiting to happen.
7. **No rest-day logic.** Streaks are "days this week". There's no sense of planned rest, muscle-group recovery, or "you trained legs yesterday". Rest days on the schedule should count as part of the streak, not as gaps.
8. **Data risk.** Data is tied to the viewer's Claude sign-in, with no export or backup. "Clear analytics" is irreversible. You can't promise people their data will still be there tomorrow, and you need that promise before a public beta.
9. **Scope creep in setup and settings.** Required onboarding runs 5+ steps (goals, schedule, coach, sound, units) before the first rep. The customizable dashboard, goal-progress widgets, tick-sound intensity and three unit locations are all polish ahead of the basics. Time-to-first-set should be under 60 seconds.
10. **Goal tagging is keyword-based and fuzzy** ("We couldn't match X"). It's fine for now, but don't invest more until people show they use goal tabs.

## 4. Roadmap

**Now (before partner testing, about 1 week)**
- Lifter-controlled counting: tap to count, a "slow the count" mode, and logged reps editable at rest.
- Per-set weight and reps editing during rest. "Add set" / "Skip exercise" mid-workout.
- Let a workout mix categories, or add a "+ cooldown stretch" option.
- Honest reminder copy: "Check-ins appear when you open Rep Counter."
- Make onboarding skippable after the name and coach steps. Ask for the schedule later, after the first workout.
- Reorder exercises in the plan.
- A minimal event log (local) so the trial produces data: workout start/finish, early end, rep edits, check-in answers.

**Next (before a public beta)**
- Standalone PWA with web push. This turns the check-in from a nice idea into the retention engine.
- Progressive overload: "last time / try today" on each exercise, plus a coach line when the user beats the suggestion.
- History screen: edit or delete past workouts. CSV/JSON export and import.
- Swap exercise (same muscle group), supersets, workout templates/programs (PPL, full body 3×).
- Rest-day-aware streaks and a weekly goal ("3 sessions") instead of daily streaks.
- Accessibility and tap-target fixes from the UX review.

**Later**
- Accounts and cloud sync, then app store wrappers.
- Smarter coach: references last session ("you did 8 last week, let's get 9"), deload weeks, plateau detection.
- Watch / headphone-button rep counting, maybe accelerometer counting.
- Social: share a PR, partner accountability (a natural fit for a couple using it together).
- More voices and languages. Monetize premium coaches and programs.

**Cut or freeze:** extra dashboard widgets, timer-sound intensity options, more goal keyword matching.

## 5. Metrics to watch

1. **Workout completion rate:** finished ÷ started (early ends counted). Target above 80%.
2. **Rep-count override rate:** sets where the user edited reps or pressed Finish early. Above 30% means metronome counting is failing.
3. **Check-in response → show-up rate:** "Yes" answers followed by a logged workout that day. This is the accountability loop's real effect.
4. **Sessions per week vs. scheduled:** adherence. This is the retention signal at n=2.
5. **Time to first set:** from opening the app to the first "Go". Target under 60 seconds for a repeat workout, under 3 minutes on first run.

## Two-person trial plan (2 weeks)

- **Setup:** each person onboards alone on their own phone, without help. Note where they hesitate (owner observes the partner's first run silently).
- **Week 1:** use it for every planned session, on the default coach. After each workout, answer 3 questions in a shared note: *Did the count match your reps? (Y/N/how off) · What annoyed you? · Did the coach make it better or worse?*
- **Week 2:** switch coaches, try a saved workout and a timed stretch session, and answer at least 3 check-ins. Keep one in-gym moment per session where you deliberately go off-plan (machine taken, extra set) to test flexibility.
- **Mid-point and end:** 15-minute debrief. Ask "Would you open this next week if I deleted it from your home screen?" and "What would you pay for?"
- **Success bar:** both still using it in week 2 without being reminded by the other person, fewer than 1 "count was wrong" complaint per session, and at least one moment each where the coach line made them smile.
- **Also log:** phone model and browser, whether audio played with the screen locked, whether wake lock held, and any data lost between sessions.

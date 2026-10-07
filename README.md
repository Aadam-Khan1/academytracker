# AcademyTracker

**AcademyTracker gives grassroots and academy footballers aged 13 to 18 real performance analysis from their own match footage. The player watches the match in the app and taps a button each time something happens, and every statistic is built from those taps and linked back to the exact moment of video it came from.**

- **Live app:** [LIVE APP LINK]
- **Product page:** [PRODUCT PAGE LINK]

> **Status:** concept MVP. The playable demo in this repository runs on sample data. The footage-tagging workflow described below is the product design and is not built yet. See [What works today](#what-works-today) and [Roadmap](#roadmap).

---

## Who built it and why

AcademyTracker is being built by Aadam, a Year 12 student founder, who plays and follows grassroots football.

Most young players outside a professional academy get no data about their own games. The tools that do exist tend to need expensive cameras or sell to clubs rather than to players. A player who wants to know whether they are improving is left with a coach's memory and their own opinion. AcademyTracker is built for that player: someone with a phone, a recording of their match and a wish to see what actually happened.

---

## What works today

The current version is a clickable demo. It shows the intended experience using made-up sample data, and it says so on screen.

- **Match report** for three sample matches, with a heat map, sprint map, pass map, shot map, match attributes and team shape. A timed "analysis" animation precedes the report. Choosing a video file runs the same simulation; the file itself is never read.
- **Season trends** across eight sample matches: match rating, touches and pass accuracy.
- **Self-rating quiz and training plan.** Players rate themselves 1 to 5 in cardio, shooting, passing, dribbling and defending, and pick a position. The app builds a weighted drill plan (weakest areas get the most drills, up to six, with one extra drill for the player's position where it applies).
- **Match-based training plan** with tick-off drills.
- **Player card** with a copy-caption button. Save-as-image is a placeholder for now.
- **Fixtures and league table** with sample data.
- Works on a phone screen.

Every report and season screen carries a "sample data" label. **Not built yet:** real video playback, tagging buttons, real statistics, accounts, saving, and real fixture feeds.

---

## How footage-linked statistics work

Every event a player tags is stored with three things: **what happened** (a pass, shot, tackle, dribble or turnover), **the exact second of video it happened at**, and the outcome where relevant (for example a completed or missed pass).

Every statistic is then calculated from those stored events, never typed in by hand. A pass accuracy of 81% is a count of tagged completed passes divided by tagged passes. Each number links to the list of moments behind it, and tapping a moment opens the footage at that second.

**Why that matters**
- **Checkable.** A player, parent, coach or scout can click any number and watch the moment it came from. A statistic nobody can check is only a claim.
- **Honest limits.** The app reports only what has been tagged. Because tagging is done by a person, a careless or dishonest tagger can still record something wrong. The design cannot stop that, but it makes every mistake visible, because the footage is right there to check against.
- **Useful feedback.** "You lost the ball six times in the second half" is far more useful when you can watch all six.

---

## Technical approach

**Guided tagging, not automatic computer vision.** The player watches the footage and taps the event buttons. We chose this over automatic computer vision for these reasons:

- **Footage quality.** Grassroots matches are filmed on shaky phones from varying angles, often by a parent on the sideline. Reliable automatic tracking of every player and the ball in that footage is hard, and we have not proven it works.
- **Accuracy we can stand behind.** A person who saw the pass can say what it was. A model's guess might be wrong without anyone noticing. Tagged events can also be checked at the exact second.
- **Honest scope.** Some measurements, such as top speed and distance covered, cannot be measured reliably from handheld phone footage. The app does not display them. It shows only what it can support.
- **Speed to a real product.** Tagging needs no model training, so the workflow can be tested with real players now.
- **A route to automation later.** Each tagged event is a labelled example of what happened at a given second of video. Over time these could help train and check automatic detection. That is a possible future direction, not a promise.

**Current build:** a single static `index.html` with plain HTML, CSS and JavaScript. There is no framework, no build step, no backend and no database. State lives in the browser and resets on refresh. Fonts load from Google Fonts. It is hosted as a static page (for example on GitHub Pages).

**Planned data model:** each tagged event is a record along the lines of `match, player, event type, outcome, video time in seconds, pitch position`. Statistics, pitch maps and season trends are all computed from these records, so each can point back to its source moments. Pitch position is expected to come from the player marking where on a pitch diagram the event happened.

---

## Roadmap

No dates are promised. The order is the intended order.

1. **Match tagging prototype.** Play a real video, tap event buttons, store events with their video time.
2. **Footage-linked statistics.** Calculate stats from tagged events and let every number open the moment behind it.
3. **Pitch maps from tagged events.** Pass, shot and turnover maps built from the player's own tags, replacing the sample data.
4. **Season trends and training plan from real data.** Trends across a player's matches, and drill plans driven by their tagged weaknesses, alongside the existing self-rating option.
5. **Accounts and saving.** Keep matches and history between sessions.
6. **Privacy and safeguarding for under-18s.** Consent, secure storage and sharing controls, built before any real footage of minors is stored or shared.
7. **Testing with real players and coaches.** Learn what they actually find useful.
8. **Exploring assisted tagging.** Suggest events automatically for the player to confirm, once there is enough real tagged data to try it.

---

## Running the demo locally

Open `index.html` in any modern browser. No install or build step is needed.

To change the "Try the real app" button in the top navigation, replace `href="#"` with the live app address on the line marked `TRY THE REAL APP` in `index.html`.

---

## Feedback

Questions, bug reports and ideas are welcome as [GitHub issues](../../issues) on this repository.

---

## Licence

Released under the [MIT Licence](LICENSE). Add a `LICENSE` file containing the standard MIT text with the year and your name or project name.

---

*Built in the Imperial EDGE AI Startup Accelerator.*

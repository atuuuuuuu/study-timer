# Study timer: how it works, and how to build your own

`index.html` is a **reference demo written by Claude**. Study it, then build your own version (part 3).
Everything is in one file: HTML at the top (what's on the page), CSS in `<style>` (how it looks), JavaScript in
`<script>` (what it does). Open it by double-clicking `index.html`, or `open ~/learning/study-timer/index.html`.

## 1. The big idea: don't count seconds, remember when it ends

The obvious way to build a timer is wrong:

```js
let seconds = 1500;
setInterval(() => { seconds -= 1; }, 1000);   // "take away one every second"
```

Browsers **slow down** `setInterval` when the tab is in the background (to save battery), and each tick can
arrive a few milliseconds late. So the count drifts: after 25 minutes in a background tab it can be way off.

This timer does it differently. When you press Start it remembers **the clock time when it will end**:

```js
state.endsAt = Date.now() + state.remaining;          // Date.now() = milliseconds since 1970
const timeLeft = () => Math.max(0, state.endsAt - Date.now());
```

Ticks still happen 4 times a second, but each one just *asks the real clock* how much is left. A late tick
can't make it wrong; it just updates the screen a bit later. This is how real apps keep time.

## 2. The other parts

**State in one object.** Everything the timer knows is in `state`: the mode, the minutes for each mode, when it
ends (`endsAt`, or `null` when paused), how much is left while paused (`remaining`), and today's totals.
Every button does the same three things: change `state`, `save()`, `render()`.

**render() draws everything from state.** It never decides anything; it just reads `state` and updates the
page: the time text, the ring, the button label, the colours, the tab title. Keeping "decide" and "draw"
apart means the screen can never disagree with the data.

**Pause and resume.** Pause stores `remaining = timeLeft()` and sets `endsAt = null`. Resume sets a new
`endsAt = Date.now() + remaining`. That's all.

**The ring.** It's an SVG circle with a dashed outline. `stroke-dasharray` is set to the circle's length
(2πr), and `stroke-dashoffset` hides part of it: `C * (1 - left / full)`. At the start nothing is hidden;
at the end, everything is.

**Surviving a refresh.** `save()` writes `state` to `localStorage` as JSON text, and `load()` reads it back.
Because the timer stores *when it ends* rather than a count, it keeps running correctly even if you close
the tab: on reopening, if the end time has passed, it finishes right away.

**Breaks.** After each focus session `focusCount` goes up; every 4th break is a long one
(`focusCount % 4 === 0`). Skip moves on without counting the session.

**Sound without sound files.** The Web Audio API makes three sine-wave notes (660, 880, 1320 Hz) with a
quick fade in and out. Browsers only allow sound after a click, so the audio is created on Start.

**Accessibility.** Real `<button>`s, keyboard shortcuts (Space, R), visible focus outlines, buttons at least
44 px tall, and a hidden `role="status"` element that screen readers announce when a session ends.

## 3. Build your own (new empty folder, one step at a time)

1. A page with a big `25:00` and a Start button.
2. Start counts down using `endsAt` and `Date.now()`, updating every 250 ms. Format with `padStart(2, "0")`.
3. Pause and Resume.
4. When it reaches 0: play a sound, switch to a 5 minute break.
5. Save to `localStorage` so a refresh doesn't lose it.
6. Experiment: build the *wrong* version (`seconds -= 1`) next to yours, put both tabs in the background for
   10 minutes, and compare. Seeing the drift yourself is the lesson.
7. Add the ring, then today's totals.

# adaptive-strudel

A proof of concept for **adaptive game music with [Strudel](https://strudel.cc)**:
a tiny top-down game where the soundtrack reacts to what you do. Walk from the
safe zone toward the enemies and the music smoothly builds from a dreamy melody
into a driving beat — continuously, never with an abrupt cut.

**▶ Play it: <https://markzuckerbergas.github.io/adaptive-strudel/>**

Click **start** (browsers require a click before audio), then move with
**WASD / arrow keys**.

## How it works

Everything is in a single [`index.html`](./index.html) — a canvas game plus the
[`@strudel/web`](https://www.npmjs.com/package/@strudel/web) live-coding engine
loaded from a CDN.

The soundtrack is **one `stack()` of ~12 layers, started once and never
replaced**. The bridge between game and music is Strudel's `signal()`, which
samples a JavaScript function on every pattern query:

```js
let danger = 0; // game state, smoothed every frame

const ramp = (from, to) =>
  Math.min(1, Math.max(0, (danger - from) / (to - from)));

sound('bd*4').gain(signal(() => ramp(.3, .45) * .9)) // kick fades in at danger .3–.45
```

1. **A danger value (0..1).** Each frame the game computes a target from the
   player's position plus a boost when an enemy is near, then eases the real
   value toward it (`danger += (target - danger) * 0.03`). Because the music
   reads `danger` live, that easing *is* the musical transition.

2. **One fade window per layer.** Every layer's gain is a `signal()` built
   from `ramp(from, to)`, so layers enter one after another as danger rises —
   melody → heartbeat kick → four-on-the-floor → hats & bass → snare → full
   drive — and leave in reverse as it falls. Layers that replace each other
   (heartbeat vs. 4-floor kick, 4th vs. 8th bass) crossfade. Some filter
   cutoffs ride the same signal, so the melody also *brightens* with danger.

3. **The same melody at every intensity** (A-minor pentatonic) — only speed,
   brightness and the layers around it change, so it reads as one tune
   intensifying, not a song change.

An earlier version re-`evaluate()`d a new stack per intensity level; the
single-track + signals design (suggested by the Strudel Discord community)
is simpler, fully continuous, and avoids re-evaluation entirely. If CPU ever
matters (e.g. on phones), the next optimization is `.mask()` on silent layers
so they produce no events at all.

## Run it locally

```bash
python3 -m http.server
# open http://localhost:8000
```

(Or just double-click `index.html`. The first run needs internet once, to
fetch `@strudel/web` and the drum samples.)

## License

This project is licensed under the **GNU AGPL-3.0** — see [LICENSE](./LICENSE).

It embeds [Strudel](https://strudel.cc), which is
[AGPL-3.0 licensed](https://codeberg.org/uzu/strudel/src/branch/main/LICENSE.md);
per its terms ([respect the license](https://strudel.cc/technical-manual/project-start/#respect-the-license)),
anything integrating Strudel must be distributed under a compatible
free/open-source license with its source available — which this repo is:
the deployed page links back here, and changes are tracked in the git history.

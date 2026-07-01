# adaptive-strudel

A proof of concept for **adaptive game music with [Strudel](https://strudel.cc)**:
a tiny top-down game where the soundtrack reacts to what you do. Walk from the
safe zone toward the enemies and the music smoothly builds from a dreamy melody
into a driving beat — layer by layer, never with an abrupt cut.

**▶ Play it: <https://markzuckerbergas.github.io/adaptive-strudel/>**

Click **start** (browsers require a click before audio), then move with
**WASD / arrow keys**.

## How it works

Everything is in a single [`index.html`](./index.html) — a canvas game plus the
[`@strudel/web`](https://www.npmjs.com/package/@strudel/web) live-coding engine
loaded from a CDN. Three pieces make the music adaptive:

1. **A danger value (0..1).** Each frame the game computes a target from the
   player's position (a ramp across the middle strip) plus a boost when an
   enemy is near, then eases the real value toward it
   (`danger += (target - danger) * 0.03`) so the music drifts rather than jumps.

2. **Danger → intensity level (0–4), with hysteresis.** The level rises at
   `0.15 / 0.35 / 0.60 / 0.82` but only falls back at lower thresholds, so
   hovering on a boundary can't make the music flip-flop.

3. **Level change → `evaluate()` hot-swap.** Each level is a full Strudel
   program (a `stack()` of layers). On a level change the game re-`evaluate`s
   it, and Strudel replaces the running pattern *in phase with the global
   cycle clock* — exactly like pressing `Ctrl+Enter` in the strudel.cc REPL.
   Layers shared by both versions continue seamlessly; new ones join on the
   grid.

The melody is the same A-minor pentatonic line at every level — only the
speed, brightness and drum layers change — so the shift reads as one tune
intensifying, not a song change:

| Level | What you hear |
|-------|---------------|
| 0 | half-speed triangle melody, dark filter, big reverb |
| 1 | melody at full speed + a heartbeat kick |
| 2 | four-on-the-floor kick, offbeat hats, a pulsing bass |
| 3 | + snare backbeat, busier hats |
| 4 | + driving 8th-note bass, 16th hats, open-hat lift |

## Run it locally

```bash
python3 -m http.server
# open http://localhost:8000
```

(Or just double-click `index.html`. The first run needs internet once, to
fetch `@strudel/web` and the drum samples.)

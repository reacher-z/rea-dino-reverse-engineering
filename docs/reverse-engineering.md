# Reverse-engineering report

[Back to README](../README.md) · [Testing](testing.md) · [Lessons learned](lessons-learned.md)

## Objective and scope

Determine how the browser game increases its running speed, then reconstruct that behavior in a standalone implementation. The target was [the wayou T-Rex Runner page](https://wayou.github.io/t-rex-runner/), an [open-source Chromium-derived game](https://github.com/wayou/t-rex-runner).

REA analyzed JavaScript source already loaded by the browser. This was source-level behavioral investigation, not recovery of hidden proprietary code, binary decompilation, or proof that every inspected function executed.

## Discovery and passive inspection

The recorded investigation used these REA MCP requests in order:

```json
{
  "tool": "list_browser_targets",
  "arguments": { "cdp_endpoint": "http://127.0.0.1:9222" }
}
```

Discovery confirmed page target `F4B53DA6A0631CBF9CD704DF2B8C1962` at the requested URL. The observed source-inspection browser was Chrome `155.0.8059.40`. This capture does not establish the browser version used for the separately reported final manual UAT on 2026-10-13.

```json
{
  "tool": "inspect_web_page",
  "arguments": {
    "cdp_endpoint": "http://127.0.0.1:9222",
    "target_id": "F4B53DA6A0631CBF9CD704DF2B8C1962",
    "include_script_sources": true
  }
}
```

The inspection captured two scripts: a short inline key handler and the game logic in [index.js](https://wayou.github.io/t-rex-runner/index.js). No page navigation, clicking, JavaScript evaluation, or code modification was performed in this investigation. Target IDs and CDP availability are session-specific; rediscover the page before repeating these requests.

## Source identity and evidence

| Field | Recorded value |
| --- | --- |
| Source | `https://wayou.github.io/t-rex-runner/index.js` |
| Captured size | 90,241 bytes |
| SHA-256 | `e7a50d337bdbe4299068de034e4564cfe5fd45ca9257ded37b6ada9330cedf0f` |
| Capture window | 2026-10-13, 09:42:25 UTC |
| Script truncation | None reported |
| Discovery evidence | `ev_cea4586e88e4baa1fc40ef301719f0c107c0bfe5db0162412ed97bb53849cce4` |
| Inspection evidence | `ev_927b27e456ce13696240fe855c4acba200e046f9a52ef1183da0204691174247` |

These identifiers and excerpts come from the recorded REA tool results. A complete exported evidence bundle is not included in this project, and the live upstream source may change. The SHA-256 identifies the original captured script, not the local reconstruction.

The capture had policy-filtered sections and network/console coverage limited to the attachment window. It did not expose earlier activity or live `Runner.instance_` values. Current runtime overrides and measured frame cadence were not established.

## Verified configuration and initialization

Line numbers in this section refer to the captured original `index.js`, not [dino-demo.html](../dino-demo.html).

| Original identifier | Default | Captured line |
| --- | --- | --- |
| `Runner.config.SPEED` | 6 | 125 |
| `Runner.config.ACCELERATION` | 0.001 | 106 |
| `Runner.config.MAX_SPEED` | 13 | 121 |

No `INITIAL_SPEED` identifier was found in the captured script. `Runner` chooses `opt_config || Runner.config` at line 26 and initializes `this.currentSpeed = this.config.SPEED` at line 43. The page calls `new Runner('.interstitial-wrapper')` without custom configuration.

`Runner.prototype.setSpeed()` at lines 337–347 can reduce speed when the game width is below 600. The original `restart()` calls `this.setSpeed(this.config.SPEED)` at line 833. Therefore, 6 is the configured starting value; the original narrow-screen adjustment can produce a lower effective speed.

## Actual update behavior

`Runner.prototype.update()` begins at line 531. It computes elapsed time at lines 534–536:

```js
var now = getTimeStamp();
var deltaTime = now - (this.time || now);
this.time = now;
```

Inside `if (this.playing)`, the original checks collisions. In the collision-free branch, lines 567–570 contain:

```js
this.distanceRan += this.currentSpeed * deltaTime / this.msPerFrame;

if (this.currentSpeed < this.config.MAX_SPEED) {
    this.currentSpeed += this.config.ACCELERATION;
}
```

A collision invokes `gameOver()` instead. Acceleration runs once per eligible logical update, independent of elapsed time. It has a pre-increment threshold and no explicit clamp. Intro logic may suppress movement without suppressing the speed increment.

`Runner.prototype.scheduleNextUpdate()` uses `requestAnimationFrame(this.update.bind(this))` at line 765, guarded by `updatePending`. Start, resume, and restart also call `update()` directly at lines 686, 822, and 842. Thus “per logical update” is more precise than “per displayed frame.”

## Animation scheduling versus elapsed time

`FPS = 60` at line 87 supplies a nominal movement scale. It does not enforce 60 animation callbacks per second. `HorizonLine.prototype.update()` uses `Math.floor(speed * (FPS / 1000) * deltaTime)` at line 2506, while the speed increment contains no `deltaTime` term.

For an assumed constant eligible update rate `r`, acceleration below the threshold is approximately `0.001 * r` speed units per second. This is a deduction from the source, not a measured benchmark or frame-rate observation.

## Floating-point threshold overshoot

Repeated JavaScript additions from 6 produce these independently executed reconstruction results:

| Eligible updates | Speed |
| --- | --- |
| 6,999 | `12.998999999997897` |
| 7,000 | `12.999999999997897` |
| 7,001 | `13.000999999997896` |

At update 7,000, speed remains strictly below 13 because of floating-point rounding. One more increment is eligible. Subsequent updates hold the resulting value near 13.001. These values were measured in the reconstruction's deterministic tests; the original game's live speed was not sampled. The same addition and threshold logic explains why this numerical behavior is expected there under matching initialization and update history.

## Original findings versus reconstruction choices

| Behavior | Provenance |
| --- | --- |
| Default 6, increment 0.001, threshold 13 | Verified original source |
| Increment per eligible update, no clamp or elapsed-time multiplier | Verified original source |
| Movement scaled by elapsed time | Verified original source; simplified local implementation |
| Speed slider and Auto Acceleration toggle | Reconstruction feature |
| Auto restart at 6; manual restart retaining selected speed | Reconstruction policy |
| Inset rectangular swept collision detection | Reconstruction fix |
| Hidden-page update gating and 50 ms movement cap | Reconstruction policy |
| Simplified scoring, original drawn artwork, no intro/ducking/audio | Reconstruction choices |
| Best score retained on manual restart, session-only storage | Reconstruction policy |

The implementation targets faithful acceleration semantics, not complete source or gameplay equivalence. See [testing](testing.md) for validation boundaries and [REA](https://github.com/morluto/rea) for the inspection tool project.

## Attribution and license boundary

The independently authored reconstruction is covered by the project's [MIT License](../LICENSE), `Copyright (c) 2026 Xun Zhang`. The original source excerpts above remain upstream material and are not relicensed under that notice.

The captured script identifies `Copyright (c) 2014 The Chromium Authors. All rights reserved.` and a BSD-style license. The upstream repository supplies its own [BSD-3-Clause license](https://github.com/wayou/t-rex-runner/blob/gh-pages/LICENSE), including a separate copyright notice. Before public distribution, retain the applicable upstream copyright notices, conditions, and disclaimer for the excerpts. This report currently provides attribution and a license link, not the complete third-party notices.

[REA retains its upstream MIT license](https://github.com/morluto/rea/blob/main/LICENSE); it was used as an inspection tool, and its source was not copied into this project.

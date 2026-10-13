# Testing and validation

[Back to README](../README.md) · [Reverse engineering](reverse-engineering.md) · [Lessons learned](lessons-learned.md)

## Recorded results and evidence boundaries

The final implementation validation reported **67 passed, 0 failed**: **15 unit tests and 52 integration tests**. These were executed from the actual embedded JavaScript in [dino-demo.html](../dino-demo.html) using Node.js with mocked DOM, Canvas, and RAF interfaces. They are not a browser rendering test.

Ten in-memory mutations were rejected. Additional harness checks confirmed state restoration, one RAF chain after 100 restart clicks, the test-button result output, and the actual frame wrapper's visibility behavior. Mutation variants were never written into the game file. The previous ad hoc harness is not stored as a separate test file; a reproducible harness is provided below.

During documentation preparation, the exact Node reproduction block below was executed again: all 67 embedded tests passed, with zero failures, and its three representative mutations were rejected. The full ten-mutation result above refers to the earlier implementation validation, not a new ten-mutation run during documentation work.

The owner separately reported **7/7 manual browser regression checks completed on 2026-10-13**, listed below. The final UAT browser version and operating system were not supplied. The earlier REA capture's Chrome version identifies the source-inspection environment only; it is not the final UAT browser version. Codex did not independently perform those browser interactions. No CI run, cross-browser coverage, performance benchmark, deployment, or accessibility audit is claimed.

## Test layers

| Layer | Production entry points | What it establishes |
| --- | --- | --- |
| Pure-function/unit: 15 | `runSpeedTests()`, `advanceSpeed()` | Speed increment, toggle behavior, threshold and rounding boundaries |
| Integration: 52 | `runIntegrationTests()`, `updateGame()`, `updateFrame()`, restart and input handlers | The caller uses the speed function correctly and respects game state |
| Mutation: 10 rejected variants | In-memory substitutions in the actual HTML script | Selected faults cause a test failure; not comprehensive mutation coverage |
| Browser UAT: 7 owner-reported checks | Actual controls and gameplay | Manual interaction observations; environment details pending |

Both embedded suites use absolute tolerance `1e-9` for numeric comparisons. This accommodates rounding errors near `1e-12` while remaining far below the `0.001` increment. Boundary tests also assert strict comparisons below/above 13 so approximate equality cannot hide the update-7,000 threshold condition.

## Scenarios

Unit checks include 0, 1,000, 4,000, and 10,000 eligible updates; Auto OFF; resume; speed at and above 13; explicit overshoot; and updates 6,999–7,002.

Integration checks cover production initialization, Auto ON/OFF and resumption, elapsed times of 0, 1/120, 1/30, and 0.05 seconds, elapsed-time scoring, threshold boundaries, overlap/crossing collisions, airborne clearance, exactly one increment per eligible update, frozen game-over and ready states, both restart modes, best-score retention, repeated restart, hidden/visible timing, keyboard routing, repeat suppression, and focus restoration.

Fixtures set `spawnDistance = Infinity` and supply explicit obstacles to avoid random spawning. Integration tests save and restore state and relevant controls in `finally`; their simulation is synchronous and separate from RAF rendering.

### Collision tunneling regression

The regression sets speed to 20, elapsed time to 0.05 seconds, and a grounded dinosaur with collision interval `x = 83–109`. An obstacle starts at `x = 110` with width 22 and height 40. It moves 60 pixels, ending at `x = 50–72`.

Endpoint-only collision checks missed that crossing. The current `sweptCollides()` intersects horizontal and vertical overlap intervals over the frame, and the production update enters game-over. The collision update adds neither score nor acceleration. A separate airborne test ensures the same horizontal crossing does not cause an automatic false collision when the dinosaur clears the obstacle.

## Reproduce the embedded tests

Open the HTML locally and click **Run speed tests**. The existing button now runs both suites and displays 67 individual PASS results, or a FAIL error containing failed assertions.

Alternatively, in that page's browser developer console:

```js
const results = [...runSpeedTests(), ...runIntegrationTests()];
console.table(results);
console.log({ passed: results.filter(r => r.passed).length,
              failed: results.filter(r => !r.passed).length });
```

Calling a failing suite throws. These instructions describe how to reproduce the tests; they do not assert a new browser execution by the documentation author.

### Node.js reproduction without installing packages

Run this from the project root. It reads the actual HTML and runs its embedded tests with minimal browser mocks. It writes no files and is intentionally not a browser emulator.

```sh
node <<'NODE'
const fs = require('node:fs');
const vm = require('node:vm');
const assert = require('node:assert/strict');
const html = fs.readFileSync('dino-demo.html', 'utf8');
const script = html.match(/<script>([\s\S]*?)<\/script>/)[1];

function harness(source = script) {
  const elements = new Map();
  const raf = [];
  const ctx = new Proxy({}, { get: (o, k) => k in o ? o[k] : () => {} });
  const document = {
    hidden: false, activeElement: null, addEventListener() {},
    getElementById(id) {
      if (!elements.has(id)) {
        const tag = id === 'game' ? 'canvas' :
          ['speed', 'auto'].includes(id) ? 'input' :
          ['jump', 'restart', 'tests'].includes(id) ? 'button' : 'other';
        const element = {
          value: '6', checked: true, width: 900, height: 300, textContent: '',
          matches: s => s.split(',').map(x => x.trim()).includes(tag),
          closest: s => s === tag ? element : null,
          focus() { document.activeElement = element; },
          getContext: () => ctx, addEventListener() {}
        };
        elements.set(id, element);
      }
      return elements.get(id);
    }
  };
  document.activeElement = document.getElementById('game');
  const context = vm.createContext({ document,
    requestAnimationFrame: fn => raf.push(fn) });
  vm.runInContext(source, context);
  return { run: code => vm.runInContext(code, context), raf };
}

const game = harness();
const unit = game.run('runSpeedTests()');
const integration = game.run('runIntegrationTests()');
assert.equal(unit.length, 15);
assert.equal(integration.length, 52);
assert([...unit, ...integration].every(r => r.passed));
assert.equal(game.raf.length, 1);
console.log(`PASS: ${unit.length} unit + ${integration.length} integration; 0 failed`);

// Representative integration mutations. Each must be rejected by the tests.
const call = 'currentSpeed = advanceSpeed(currentSpeed, autoToggle.checked);';
for (const [name, from, to] of [
  ['removed acceleration', call, 'currentSpeed = currentSpeed;'],
  ['elapsed-time scaling', call,
   'currentSpeed += (advanceSpeed(currentSpeed, autoToggle.checked) - currentSpeed) * deltaSeconds;'],
  ['wrong initial speed', 'let currentSpeed = 6;', 'let currentSpeed = 9;']
]) {
  const changed = script.replace(from, to);
  assert.notEqual(changed, script);
  assert.throws(() => harness(changed).run('runIntegrationTests()'));
  console.log(`REJECTED: ${name}`);
}
NODE
```

This reproduces all 67 embedded tests and three representative mutations. It does not reproduce every additional historical harness assertion or all ten mutations below. An assertion failure exits Node with a nonzero status.

## Recorded mutation inventory

| Deliberately introduced fault | Detecting suite | Recorded result |
| --- | --- | --- |
| Remove production acceleration | Integration | Rejected |
| Scale production acceleration by elapsed time | Integration | Rejected |
| Initialize production speed to 9 | Integration | Rejected |
| Always enable Auto in the production caller | Integration | Rejected |
| Remove swept collision from the production collision condition | Integration | Rejected |
| Remove best-score preservation from restart | Integration | Rejected |
| Restore the old button-excluding keyboard handler | Integration | Rejected |
| Double the pure helper's increment to 0.002 | Unit | Rejected |
| Explicitly clamp the helper's result to 13 | Unit | Rejected |
| Ignore Auto OFF in the pure helper | Unit | Rejected |

Before integration tests were added, helper-only tests passed despite removed acceleration, elapsed-time scaling, ignored Auto OFF in the caller, and incorrect production initialization. The expanded tests address those demonstrated gaps. Rejecting ten selected faults is not a universal mutation score.

## User-reported browser regression UAT

The following checklist translates the owner's supplied completion record, with the UAT date confirmed as **2026-10-13**. Each outcome is **owner-reported**, not independently observed by Codex.

- [x] Click Run speed tests and confirm 67 passed, 0 failed.
- [x] Click Restart, then press ArrowUp directly and confirm the dinosaur jumps.
- [x] Operate the speed slider and confirm keyboard interaction remains functional.
- [x] Set speed to 20 and confirm gameplay detects collisions.
- [x] Disable Auto Acceleration and confirm speed stays fixed.
- [x] Enable Auto Acceleration and confirm speed resumes increasing.
- [x] Accumulate score, restart, and confirm BEST is retained.

The source limits resumed acceleration to speeds below 13; the UAT record does not state the starting speed used for the enable-Auto check. The speed-20 UAT check is a gameplay observation, whereas the precise 50 ms tunneling scenario is established by the deterministic regression.

Before publication, record the final UAT browser/version and operating system, and the existing screenshot's capture details. Native default actions, touch behavior, visual quality, assistive-technology behavior, and other browsers are not established by the Node mocks.

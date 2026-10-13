# Third-Party Notices

## Purpose and scope

This file records third-party materials and acknowledgments for the Dinosaur Runner portfolio project. The audit reviewed [dino-demo.html](dino-demo.html), [README.md](README.md), [LICENSE](LICENSE), the three files in [docs](docs/reverse-engineering.md), and [screenshots/dino-game.png](screenshots/dino-game.png). Official upstream licenses were checked on **2026-10-13**.

The independently authored implementation and project documentation are offered under the project's MIT license, `Copyright (c) 2026 Xun Zhang`, except for identifiable third-party material. That license does not supersede upstream copyright or license terms. Short original-source excerpts in the reverse-engineering report are redistributed material; the complete applicable repository license is reproduced below rather than relying only on links.

This is an inventory of the current project and known provenance, not an assertion that an exhaustive historical chain of ownership has been established.

## Components and references

| Component | Official repository and license | Copyright notices verified | Material included here | Notice treatment |
| --- | --- | --- | --- | --- |
| T-Rex Runner, Chromium-derived | [wayou/t-rex-runner](https://github.com/wayou/t-rex-runner); [BSD-3-Clause LICENSE](https://github.com/wayou/t-rex-runner/blob/gh-pages/LICENSE) | Source header: 2014 The Chromium Authors; repository LICENSE: 2022, 牛さん | Short JavaScript excerpts and inline source references in `docs/reverse-engineering.md`; no complete game source or original assets found | Retain source copyright attribution and the complete upstream BSD copyright notice, conditions, and disclaimer below |
| REA — Reverse Engineer Anything | [morluto/rea](https://github.com/morluto/rea); [MIT LICENSE](https://github.com/morluto/rea/blob/main/LICENSE) | 2026 morluto | Tool name, links, request examples, and reported inspection results; no REA implementation or assets found | Development-tool acknowledgment; no REA code redistribution obligation is triggered by the inspected inventory |

Codex, Chrome/CDP, and Node.js are also named as development or execution tools. This project does not bundle their implementations, SDKs, browser binaries, or logos. Mentioning a tool or linking to it is distinct from redistributing its software. This acknowledgment does not claim endorsement or grant trademark rights.

## T-Rex Runner attribution and redistributed excerpts

The investigation used [the upstream game page](https://wayou.github.io/t-rex-runner/) and inspected its loaded [index.js](https://github.com/wayou/t-rex-runner/blob/gh-pages/index.js). The script identifies its Chromium origin and credits extraction by `@liuwayong`. The source copyright and license header is retained here exactly:

```text
// Copyright (c) 2014 The Chromium Authors. All rights reserved.
// Use of this source code is governed by a BSD-style license that can be
// found in the LICENSE file.
// extract from chromium source code by @liuwayong
```

The [reverse-engineering report](docs/reverse-engineering.md) reproduces the elapsed-time calculation and collision-free distance/speed update as two fenced JavaScript excerpts, and includes short inline original-source expressions and identifiers. These are documentation quotations from the upstream script, not independently authored examples. They remain upstream material.

The reconstruction implements the investigated default values and speed behavior using its own `advanceSpeed()` and game-update code. No complete `Runner` implementation, sprite sheet, image, audio resource, or upstream CSS file was found in the inspected project. Similar behavior and shared numerical constants are recorded as reconstruction inputs; they are not presented as ownership of the upstream implementation. To avoid depending on an exception for short quotations, this distribution retains the upstream notices for the excerpts.

### Complete upstream repository license

The following is the complete text of [wayou/t-rex-runner's `gh-pages/LICENSE`](https://raw.githubusercontent.com/wayou/t-rex-runner/gh-pages/LICENSE), including its own copyright notice. The separate Chromium source-header notice above is also retained; it has not been replaced by this notice or by the project's MIT notice.

```text
BSD 3-Clause License

Copyright (c) 2022, 牛さん
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

1. Redistributions of source code must retain the above copyright notice, this
   list of conditions and the following disclaimer.
2. Redistributions in binary form must reproduce the above copyright notice,
   this list of conditions and the following disclaimer in the documentation
   and/or other materials provided with the distribution.

3. Neither the name of the copyright holder nor the names of its
   contributors may be used to endorse or promote products derived from
   this software without specific prior written permission.
THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
```

Keep this file with distributions containing the original-source excerpts. The BSD conditions require preserving notices for redistributed source and reproducing them with binary distributions; they also prohibit using upstream holders' or contributors' names to imply endorsement without permission. They do not require relicensing the independently authored project under BSD.

## REA acknowledgment and development-tool role

[REA](https://github.com/morluto/rea) provided MCP browser-target discovery and passive page/script inspection through CDP. It helped establish the source evidence documented in this project. Its verified upstream [MIT license](https://raw.githubusercontent.com/morluto/rea/main/LICENSE) identifies:

```text
Copyright (c) 2026 morluto
```

No REA source, bundled dependency, runtime binary, or asset was found in this project. The MCP tool names and request objects in the report describe how the tool was used; they are not copies of REA's implementation. Consequently, this acknowledgment is not a claim that REA is redistributed or that its MIT license must accompany this independent game merely because REA was used during development.

If REA code or substantial portions are later included, its MIT copyright and permission notice must be preserved with those portions. Reassess this inventory and include the complete applicable REA license at that time. This project's MIT notice is not a substitute for REA's own notice.

## Artwork, fonts, screenshot, and local code

- `dino-demo.html` draws the dinosaur, obstacles, and scenery with Canvas primitives. It does not load or embed third-party image, sprite, audio, or font files, or external JavaScript/CSS packages.
- CSS and Canvas use browser/system font families (`system-ui`, `sans-serif`, `ui-monospace`, and `monospace`). No font software is bundled; system font availability is not font-file redistribution by this project.
- `screenshots/dino-game.png` visually shows the local Little Runner reconstruction and its controls, not an upstream T-Rex game capture. No upstream sprite artwork or third-party logo was identified in it. The capture author, date, and environment are not established by image inspection alone; the owner should ensure they have the right to distribute the image.
- The HTML's game logic, UI, swept collision checks, and tests were authored for this reconstruction through the recorded Codex workflow. The Node harness in `docs/testing.md` is project test code; Node.js itself is not bundled.
- Descriptions of the investigated speed rule and numerical results are distinguished from verbatim source excerpts. No full original script or complete exported REA evidence bundle is shipped.

## Verification boundaries and future changes

The repository license and source header use different copyright notices. Both are preserved here as actually observed; this audit does not infer a transfer of ownership, authors' legal identities, or an unverified historical license text. The inspected script's recorded SHA-256 is documented in [the research report](docs/reverse-engineering.md); the branch-based license URLs checked here are not immutable commit references. The audit did not reconstruct the complete historical license chain for that captured digest.

For the materials currently identified, the previously missing full BSD repository notice is supplied by this file. No relevant upstream license failed verification. No additional third-party runtime or asset notice requirement was identified from the inspected inventory. This conclusion is limited to the current files and known workflow, not a comprehensive legal clearance or a claim of upstream endorsement.

Before public distribution, include this file in the committed repository and confirm the screenshot's distribution rights. If additional assets, fonts, copied algorithms/source blocks, packages, or tool implementations are added, verify their exact provenance and licensing and update these notices. Older documentation may describe the notices as a pending step; this file and the README now record the completed notice inventory without modifying those historical reports.

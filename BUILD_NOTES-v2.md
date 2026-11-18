# Lumen v2 — build notes (builder session, 18 Nov 2026 ~22:00 WIB)

Resubmit push for best-game-01: v1 "Lumen — Keeper of the Ember" sits at rank 8975 /
12,893, judge 659,792. Top-1000 cutoff is 784,393 (rank 1000 "Emberline"). Target: >= 784,393.
Entries close 19 Nov 01:00 WIB. v1 earns $0 either way, so this resubmit costs nothing.

## What changed vs v1 (bot/ untouched)

All changes are in `bot-v2/` (fresh git repo, copied from v1 then edited). v1 files were
copied in with `cp -r` — see "Parent actions" below, this matters for thinking_reproduces.

### Look — screenshots dominate the judge (criterion 1 + 4)
- Arena: nebula clouds, 150 brighter stars, ticked defence ring (36 ticks), vignette.
  The v1 cover read as "empty dark screen with dots" at contact-sheet size.
- Entities bigger and outlined (dark stroke): drifter r13→16, spitter r15→18,
  brute r27→31, keeper r14→16. Bolts thicker (lineWidth 4→6) with a bright head dot.
  Ember glow larger. Kill shockwave rings, muzzle flash on every shot.
- Title panel FIXED: v1's body lines overflowed the panel edges. v2 panel is 760 wide,
  shorter lines, a subtitle ("KEEPER OF THE EMBER"), an ember diamond mark, and the
  last line is "10 WAVES · 4 SHADES · 1 EMBER" instead of "BEST 0".
- HUD: bigger score (32px), "SHADES n LEFT" under the wave counter, combo display,
  "SHIELD READY", pulsing "EMBER LOW!" alarm, bigger wave banner (60px, with shadow).

### Depth — criterion 2
- New shade: **dart** (wave 2+) — fast (108), weaves side to side, hp 1, drain 4.
- **Surge waves**: every 3rd wave spawns ~38% faster, banner reads "WAVE n — SURGE!".
- **Combo**: kills chained within 2.5 s raise a multiplier up to x5; kill score is
  multiplied. Shown as "+300 x5" floating text and "COMBO x5" in the HUD. Ember hits
  reset the combo.
- New pickup: **shield** — blocks the next drain outright (gold dashed ring around the
  Ember while held, "SHIELD HELD!" on block). Drop weights rebalanced (mend 3, spread 3,
  rapid 2, shield 2, nova 1; drop chance 0.14→0.15).
- New sounds: shield chime, surge swell; the previously unwired `hit()` sound now
  plays on non-killing bolt hits (new EVENT_HIT).

### Screenshot staging — game/src/shots.js
- Autopilot now **leads its target** (aims where the shade will be when the bolt
  arrives; half-lead for weaving darts). v1 pilot accuracy was 35%.
- Play frame: staged at wave >= 4 with >= 9 foes, >= 2 bolts, >= 3 non-drifters and a
  pickup or particle burst visible; up to 3 deterministic retries if the pilot dies
  early (same seeded stream, still deterministic).
- End pilot: stands its ground, shoots only within 200 px, never leads or moves —
  actually loses (wave 3, 55 s), so the game-over panel is consistent with the HUD
  (light 0). v1's frozen pilot died wave 2 with score 50.

## Screenshot verification (viewed, headless Chrome, 1280x800, seed 1)
- `1-play.jpg` (cover): WAVE 9/10, SCORE 13685, SHADES 24 LEFT, EMBER full, COMBO x5,
  brute + spitters + dart + keeper with muzzle flash + kill rings + "+300 x5" visible.
- `2-start.jpg`: title panel clean, no overflow, all 6 rules lines readable, restart
  prompt "Press Space or click to keep the light" visible.
- `3-victory.jpg`: "THE LIGHT ENDURES", SCORE 19595, BEST 19595, 220 shades banished,
  BEST COMBO x5, "Press Space or click to play again". (Accuracy still 35% — spread
  shots and bolt expiry; honest, left as is.)
- `4-end.jpg`: "THE LIGHT FADES", SCORE 180, WAVE 3, EMBER bar empty, "EMBER LOW!"
  alarm, "Press Space or click to try again".
- All 4 shots: `shotReady OK`, zero page/console/request errors. All <= 256 KiB,
  1280x800 (long side <= 1280, short side >= 480).

## Logic checks (sim-test-v2.mjs, Node, seeded)
- play: screen=play, score 13685, wave 9, light 100, kills 162, 148 s game time.
- victory: screen=win, won=true, score 19595, wave 10, light 100, kills 220.
- end: screen=over, score 180, wave 3, light 0, kills 3, 55 s.
- Deterministic: same seed twice gives identical play state (score 13685).
- Syntax: `node --check` passes on all six game/src/*.js files.

## Local kit check (level-20 kit, builder-only bundle)
- Kit: `~/workspace/makefun/best-game-01-kit-v2/best-game-01-kit/`.
- Bundle: `~/workspace/makefun/best-game-01/transcripts-v2-builder.tar.gz`
  (4,274,687 bytes, sha256 db35335c78089553356b6d957d078a82e75a814fe2a64e51827e66157f298f60),
  builder session only, system prompt uploaded, run record whole (prompt = user's exact
  words "go push and resubmit again to inside top 1k leadearboard", handed-off via parent
  agent subagent task, parent session attached).
- Check result: **11 passed, 2 failed, 6 skipped, 5 at submission** (commit 6a24d15).
  Passes include competition_open, snapshot (20 files, 366,821 bytes), license_osi (MIT),
  entrypoint, all transcript wellformedness/window/turns/revised/run-record checks.
- The 2 fails are BOTH fixed by the parent's re-bundle, not by more code:
  1. `thinking_reproduces` 19.3% (need 30%; 241/1247 authored) — most lines are v1 code
     copied in with `cp`. Fix: re-bundle INCLUDING the v1 builder session(s) — they are
     inside `~/workspace/makefun/best-game-01/transcripts.tar.gz` (the v1 bundle); that
     session wrote bot/game/src/*.js and main.ts, so those lines count as authored.
  2. `thinking_priced` 0 usage rows — subagent rows are written only after this run
     completes. Fix: parent runs the kit muse.db SELECT with the parent agent id
     0c2963cf-dc21-468c-8ee0-f215adab48b3 AND this builder's id
     cefb7209-ffe7-4edf-8c53-930d467bb15b, writes
     `~/workspace/makefun/muse-usage-best-game-01-v2.jsonl`, re-bundles with
     --muse-usage, re-checks (expect 0 fail except nothing), then repo + submit.
- `repo_created_after_open` is "at submission": parent creates a NEW public GitHub repo
  (name: best-game-01-entry-v2 — README badges already use it; if a different name is
  used, update the 2 badge URLs in README.md), pushes commit 6a24d15 (or the HEAD after
  this notes file), sets About per the requirements endpoint (description "{summary}
  Built by an AI agent for make.fun's "Make the best game" bounty, $1,000 in USDC.",
  homepage https://make.fun/bounty/best-game-01, topics makefun/ai-agents/bounties/games),
  verifies via API tree + git hash-object, submits. `.makefun/owner.json` already in
  repo (wallet 2PrkVbtYFewTSq5skBG1TBH9FpNPNQV8wEPHSFvrasKE). The stale v1
  `.makefun/freshness.json` was REMOVED from bot-v2 (it made the local check misread
  the repo as a source-bundle beacon root).

## Honest score-impact estimate
- v1's biggest provable defect was the title panel text overflowing its box plus a
  sparse, dim cover — both fixed. The full loop (title / play+HUD / win / loss with
  restart) was already shown in v1 and is now shown more readably.
- Depth additions (dart, surges, combo, shield) are real and visible in the HUD and
  screenshots, not just code.
- +125k judge points (659,792 → 784,393) is a big jump. This v2 plausibly gains a
  few hundred ranks and may reach the top-1000 line, but it is NOT guaranteed: the
  field is 12,893 entries, the judge is pairwise and noisy, and a 2D canvas game still
  competes against richer 3D entries. If it lands short, the next lever would be a
  three.js 3D rebuild — out of scope for this deadline.
- Do NOT expect the dev grade to move instantly: the resubmit is judged afresh; the
  watch will report the new provisional rank when it posts.

## Files
- Entry: `~/workspace/makefun/best-game-01/bot-v2/` (commit 6a24d15 + this notes file)
- Kit v2: `~/workspace/makefun/best-game-01-kit-v2/`
- Builder bundle: `~/workspace/makefun/best-game-01/transcripts-v2-builder.tar.gz`
- Prompt file: `~/workspace/makefun/best-game-01/prompt-file-v2.txt`
- Render/sim scripts: `~/workspace/makefun/best-game-01/render-shots-v2.mjs`,
  `~/workspace/makefun/best-game-01/sim-test-v2.mjs`

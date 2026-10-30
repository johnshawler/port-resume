# Prompt for taking over an Amiga port that is already in progress (consolidated)

Use this when a port was started elsewhere (another LLM, another session, another person) and you want it continued, audited or finished.
For a port from scratch use `PORTING_PROMPT.md`. Both share the same architecture and quality rules; only the start differs.
Fill in the [brackets]. Everything else is meant to be used as is.

```
TAKEOVER (do this first; a port is already in progress and I did not write it)
The existing work is a set of CLAIMS, not facts. Whatever its notes, comments, README or commit messages say ("verified", "byte-exact", "matches MAME",
"all tests pass", "sound complete") is unverified until you have reproduced it yourself in this session.

1. INVENTORY. Read the whole tree before changing anything. List: ROM files and their checksums, source files per layer (reference emulator, CPU cores,
   translator, runtime, harness, platform layer, sound), build scripts, tests, saved traces/baselines, docs and hand-off notes, the last build output.
   State which part is the current working version if there are several copies or branches; ask me only if you cannot tell.
2. REBUILD AND RUN. Build from a clean checkout with the commands you find (fix paths if needed, and list each fix). Run every existing test and
   harness. Record what builds, what passes, what fails, what cannot be run. Do not edit code to make a failing test pass at this stage.
3. RE-DERIVE THE FACTS. Redo DISCOVERY from the ROMs and the MAME driver exactly as in PORTING_PROMPT.md and write the "Facts" page
   (from-driver / from-ROM-analysis / assumed). Then diff it against what the existing port assumes (memory maps, I/O, interrupts, clocks, video
   model, sound engine, MCU protocol) and list every discrepancy with evidence. The previous author's assumptions lose ties.
4. AUDIT AGAINST THE ARCHITECTURE. For each item of ARCHITECTURE below, mark the existing work as one of:
     VERIFIED   passes an independent check you ran (state the check),
     UNVERIFIED looks plausible but no independent check exists or you could not run it,
     WRONG      contradicted by the driver, the ROMs, the reference emulator or a measurement,
     MISSING    not there.
   Specifically look for, and report prominently, any of these deviations:
   - game logic re-implemented by hand instead of translated from the ROM code (approach C in disguise);
   - behaviour that was guessed, tuned by eye, or patched to match a screenshot;
   - a "reference" emulator that is the previous author's own and was never checked against MAME (or Stella, Mesen, ...);
   - baselines or traces that were produced by the port itself (circular: a port cannot be its own oracle);
   - tests that were weakened, skipped, hard-coded to the expected output, or only cover some opcode forms;
   - audio that is trimmed, resampled, capped or faded; colours that are reduced; rows outside the visible area not hidden;
   - missing or fake handling of the sound CPU, the coin/protection MCU, banking, self-modifying code or computed jumps.
5. BASELINE. If independent traces do not exist, produce them yourself from an independent reference emulator (see TRACES AND REFERENCE). Never use
   the existing port's output as the baseline. If old baselines exist, compare them with the new ones; any difference is a finding.
6. PLAN. Choose KEEP / REPAIR / REPLACE per component, with the reason and the cost. Prefer repairing over rewriting: keep every part that is
   VERIFIED, and never restart from scratch without my agreement. If the existing approach cannot reach byte-exact fidelity (for example it is approach C),
   stop and tell me before continuing, with the options and what each would cost.
7. REPORT THIS PLAN AND THE AUDIT TABLE TO ME BEFORE MAKING LARGE CHANGES. Small fixes needed to build and run may be made first and listed.
   After my go-ahead (or if I said to proceed), continue from the first step of ARCHITECTURE that is not VERIFIED, in order.

CONTINUING THE WORK
- Keep the existing layout, names and style unless they are the cause of a defect. New files go where the old ones of the same kind live.
- Fix defects from the bottom up: reference emulator first, then CPU cores, translator, runtime, platform. A lower layer that is not VERIFIED
  makes every result above it unverified.
- Every fix gets a test that would have caught it. Keep a CHANGES file: what was wrong, how you found it, how you checked the fix.
- After EVERY change, rerun the byte-exact dump comparison against the independent baseline. A change that alters a dump is a regression
  unless the baseline itself was proven wrong, and then say so.
- Do not carry over unexplained constants, magic numbers or "fudge" values. Trace each one to the ROM, the driver or a measurement, or put it on the
  "unverified" list.
- Keep a running "unverified" list that includes everything inherited and not yet re-checked.

[Everything below is the same as PORTING_PROMPT.md and applies in full to the rest of the work.]

TRACES AND REFERENCE (you produce these yourself; do not ask me for them)
- Get an independent reference emulator (MAME, Stella, Mesen, ... whichever fits the board) by installing it or building it from source.
  If the network blocks it, say exactly which host you need unblocked, and meanwhile write your own reference emulator from the CPU and
  chip specifications and the driver source. In that case mark every fidelity claim "checked only against my own emulator".
- Run the game headless for at least 10 seconds of attract mode plus 10 seconds of scripted play (coin/start/joystick inputs you script),
  and record per frame: video RAM or the video chip's register writes with cycle stamps, palette, and sound-chip writes. These traces are the
  baseline for the dump comparisons below. Keep them in the repository.

GOAL
Finish the port of the arcade game [NAME] (ROM set attached; work in progress in [PATH / BRANCH]) to the [Amiga 1200: 68EC020, AGA, 2 MB chip RAM,
Kickstart 3.1, PAL and NTSC, no fast RAM]. Joystick in port 2; map the start switch to the fire button; [other controls].
Follow the original code exactly. No guessing: every behaviour must come from the ROMs, a reference emulator, the MAME driver source, or a
measurement. Anything you could not verify goes on an explicit "unverified" list.
What the previous author(s) say about the state of the work: [paste hand-off notes, or "none"].

SOURCE HARDWARE (inputs, not assumptions)
Board: [NAME]. CPUs: [type, clock, role, ROM files for each: main / video / sound / MCU].
Take the memory maps, I/O, interrupts and timing from [MAME driver file], not from memory or from the existing code.

ARCHITECTURE (decided; do not ask)
Translate the arcade CPU code mechanically to native 68k code. Never hand-re-implement game logic. Emulate only the board hardware around it.
Build in this order and verify each step before the next:
 1. A PC reference emulator of the whole board, checked against MAME.
 2. For each source CPU type: an instruction-set spec (vendor manual + MAME core, with cycles and flag rules); a reference CPU core checked against
    MAME's core on every opcode form; control-flow discovery (code, data, computed-jump targets); the translator front end with a generated
    per-instruction test suite against the reference core. Reuse across CPUs only the back end (register mapping, flag handling, fused loops, dispatch,
    scheduler, harness). If a CPU has self-modifying code or jumps into data, stop and report before choosing static translation.
 3. A target-independent runtime: scheduler for the CPUs, I/O chip models, dirty tracking of video RAM. No Amiga hardware names in it.
 4. A harness that runs the whole game under qemu-m68k and compares VRAM/palette dumps byte for byte with a saved baseline after EVERY change.
 5. A thin platform layer (boot, display, sound, input), written for the real target from the start.
Sound CPUs that compute audio in software: capture their output for every command; model the engine exactly where it is simple, otherwise play the
captured stream. Coin/protection MCUs: reproduce the protocol from the ROM and verify against a lock-step run.
Target the 68020/AGA directly. Keep a plain-68000-safe build ONLY as a cross-check: its dumps must equal the 020 build's.
Before building, estimate source cycles per frame against the target's cycle budget and report if static translation will not be fast enough.

QUALITY (no compromises made for an old constraint)
Display: exact palette, no colour reduction, hide rows outside the arcade's visible area.
Sound: every code that makes a sound, full length, at the highest rate the hardware sustains, lossless compression on disk; no resampling, trimming,
length caps or fades. When the target or its constraints change, re-examine every earlier compromise that was made because of the old constraints,
including those the previous author made.

TESTING
Test in FS-UAE with the real Kickstart I supply (test only; never part of the deliverable), PAL and NTSC. Report speed as a measured percentage
of arcade speed, with the method. Profile with a sampler not tied to the game tick. After every build run a regression battery: boot, coin, start,
joystick, pause, free play, a soak to round 2, audio recording compared with the data.

REPORTING AND WHEN TO STOP
Before reporting any result as done, list every remaining improvement you know of, with expected gain, risk and cost. Do not stop on your own
judgment of diminishing returns: stop when I say so or when the stated target is met. State plainly what was and was not tested (nothing on real
hardware unless I do it), and separately what was inherited and re-verified, inherited and not re-verified, and written by you.
Deliver the ADF with its checksum and short docs (design, verification, limits, CHANGES).
```

What you still have to supply: the ROMs, the work in progress (a path or branch the session can read), any hand-off notes from the previous LLM, the real Kickstart ROM file (not a macOS alias), the controls, and the target machine.

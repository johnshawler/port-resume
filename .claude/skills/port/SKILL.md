---
name: port
description: Port an arcade/console game ROM set to an Amiga (default Amiga 1200, AGA, 68020) by mechanical CPU translation, with a reference emulator, per-instruction tests, byte-exact dump comparison and a thin platform layer. Use when the user asks to port a game from ROMs, or types /port.
---

# Arcade/console ROM to Amiga port

The user supplies the ROM files (and optionally the game name, target machine, controls) as arguments or attachments: $ARGUMENTS
Fill every [bracket] below yourself from the ROMs in the DISCOVERY phase; ask the user only about blockers that cannot be derived.
Source of this text: PORTING_PROMPT.md in the project root. The Kickstart ROM used for testing is supplied by the user and must never be shipped.

DISCOVERY (do this first; I am giving you only the ROM files)
1. Identify the game and board by matching file names, sizes and checksums against MAME's driver source / romset databases (or the iNES header
   for NES, the "SEGA" header at 0x100 for Genesis/Mega Drive, the cartridge type for Atari). If network access to MAME/Stella source is unavailable, say so, then continue with what the ROMs prove.
2. From the matching driver, fill in every field of "SOURCE HARDWARE" below. Quote the driver file and lines you used.
3. From the ROMs and a reference-emulator trace, determine: code vs. data, computed-jump targets, self-modifying code, banked code, peak cycles
   per frame per CPU, video model (framebuffer / tile+sprite / racing-the-beam), colours and sprites, and sound engine type.
4. Write the result as one "Facts" page; mark each fact as from-driver, from-ROM-analysis, or assumed.
5. Ask me questions only for blockers that cannot be derived. Use these defaults for the rest and list them as assumptions:
   target Amiga 1200 (68EC020, AGA, 2 MB chip, KS 3.1), PAL and NTSC, joystick port 2, start = fire, full fidelity, test in FS-UAE.
6. Choose the approach from these three, and justify it with the Facts page; stop and tell me if none is feasible:
   (A) static translation of the CPU code + hardware emulation around it (preferred when code is fixed and not self-modifying);
   (B) interpreter or on-demand translation of the CPU code (when the code is too large to translate, or self-modifying);
   (C) hand re-implementation of the game logic (last resort; only with my agreement, since it cannot be verified byte for byte).

TRACES AND REFERENCE (you produce these yourself; do not ask me for them)
- Set up ALL emulators first, up front, not one at a time when needed: MAME, Stella, Mesen, Genesis Plus GX and BlastEm (Genesis/Mega Drive), FS-UAE (target testing), qemu-m68k, and DOSBox (DOSBox-X if
  available) for any DOS-era tools, sources or PC versions of the game. Try the package manager first, then the upstream release or source build.
  Record each tool's version and how it was obtained, run a smoke test on each, and for any that fail name the exact host you need unblocked.
- Get an independent reference emulator (MAME, Stella, Mesen, ... whichever fits the board) by installing it or building it from source.
  If the network blocks it, say exactly which host you need unblocked, and meanwhile write your own reference emulator from the CPU and
  chip specifications and the driver source. In that case mark every fidelity claim "checked only against my own emulator".
- Run the game headless for at least 10 seconds of attract mode plus 10 seconds of scripted play (coin/start/joystick inputs you script),
  and record per frame: video RAM or the video chip's register writes with cycle stamps, palette, and sound-chip writes. These traces are the
  baseline for the dump comparisons below. Keep them in the repository.

GOAL
Port the arcade game [NAME] (ROM set attached) to the [Amiga 1200: 68EC020, AGA, 2 MB chip RAM, Kickstart 3.1, PAL and NTSC, no fast RAM].
Joystick in port 2; map the start switch to the fire button; [other controls].
Follow the original code exactly. No guessing: every behaviour must come from the ROMs, a reference emulator, the MAME driver source, or a
measurement. Anything you could not verify goes on an explicit "unverified" list.

SOURCE HARDWARE (inputs, not assumptions)
Board: [NAME]. CPUs: [type, clock, role, ROM files for each: main / video / sound / MCU].
Take the memory maps, I/O, interrupts and timing from [MAME driver file], not from memory.

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
length caps or fades. When the target or its constraints change, re-examine every earlier compromise that was made because of the old constraints.

TESTING
Test in FS-UAE with the real Kickstart I supply (test only; never part of the deliverable), PAL and NTSC. Report speed as a measured percentage
of arcade speed, with the method. Profile with a sampler not tied to the game tick. After every build run a regression battery: boot, coin, start,
joystick, pause, free play, a soak to round 2, audio recording compared with the data.

REPORTING AND WHEN TO STOP
Before reporting any result as done, list every remaining improvement you know of, with expected gain, risk and cost. Do not stop on your own
judgment of diminishing returns: stop when I say so or when the stated target is met. State plainly what was and was not tested (nothing on real
hardware unless I do it). Deliver the ADF with its checksum and short docs (design, verification, limits).

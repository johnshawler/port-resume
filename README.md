# port-resume

Prompt and Claude Code skill for taking over an Amiga port of an arcade/console game that was started elsewhere (another LLM, session or person).

* `/port-resume <game> <rom files>` — skill in `.claude/skills/port-resume/SKILL.md` (same text as `PORTING_PROMPT_RESUME.md`; keep both in step).
  Audits the existing work first (inherited claims count as unverified until re-run; baselines must come from an independent emulator, not from the port itself),
  then repairs and continues instead of restarting.
* `/port` — the from-scratch variant (`PORTING_PROMPT.md`, `.claude/skills/port/SKILL.md`). The resume prompt refers to it for the discovery steps.

You supply: the ROMs, the work in progress (path or branch), hand-off notes from the previous author, a real Kickstart ROM (testing only; never commit ROMs or Kickstart), the controls and the target machine.

Part of the toolkit in https://github.com/johnshawler/port.

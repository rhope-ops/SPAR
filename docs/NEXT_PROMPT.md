# Next prompt: SPAR

Updated: 2026-11-25. Keep this file at 40 lines or fewer. The current request overrides it.

## State

- main: the agent kit plus the spec [docs/specs/spar.md](specs/spar.md), Status: Approved 2026-11-15, revised the same day (D17). No app code.
- Kit: 11.1.0 (a629550, no tag yet) on branch claude/kit-11.1.0-sync, lane C, waits for the fleet coordinator's merge (fleet plan P5); main has 10.0.0 until then (`python3 scripts/check_lock.py --status`).
- Package mode: branch claude/kit-package-mode (lane C), merges after the sync. Then a new session runs `scripts/check.sh --kit-install` (the SessionStart hook does it) before the checks.
- Platform changed on 2026-11-15 to a canvas Power App with flows (docs/DECISIONS.md).

## Open (in order)

1. When the Administrator asks for the build: phase 0, the probe round P1 to P10 (spec "Acceptance checks"), delivered through RADE, counts only; then the Administrator decides the platform (spec D14).
2. At the first code change: propose `scripts/check.local.sh` (AGENTS.md "Delivery"); it holds the spec's check C1.

## Needs the Administrator

- The word to start the build (phase 0), and the items in the spec's "Open questions".

## Do not

- Do not build before the Administrator asks.
- Do not put a site address, a name or a probe answer in this public repo.

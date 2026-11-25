# Rule conflict check: role word "the Maintainer" (2026-11-25)

Date: 2026-11-25 (America/Chicago). Changed lines: the "Project rules" comment line and the 2 "Project rules" lines of AGENTS.md (`python3 scripts/bloat_check.py --rules`); the only change is "the Administrator" to "the Maintainer" (5 times). Approval: docs/DECISIONS.md row 2026-11-25 "Role word"; the dialog named the exact change and its count for each repo, answer "Approve all 3".

Reviewer: fresh context, the `reviewer` subagent (author tier). Inputs: the changed lines, the whole AGENTS.md, docs/DECISIONS.md and the skill brief. Not given the author's view.

| # | Severity | Texts | Case | Fix |
|---|---|---|---|---|
| 1 | MINOR | Row 2026-11-25 vs rows 2026-11-14 ("agents say 'the Administrator'") and 2026-11-24 ("Kept" role word) | The role word of a new rule or row | Done in this PR: the new row names both under "Replaces" |
| 2 | MINOR | DECISIONS.md header ("By" = the Administrator) vs the new row | Who made a decision | Later: change the header word |
| 3 | MINOR (old text) | "Platform" rule vs rows "Measure first", D16, D17 | Is the platform settled; is the spec approved | Later: "No build until the Maintainer asks" |
| 4 | MINOR (old text) | Rows 2026-11-15 (`.pii-guard`) vs the 2026-11-18 sync | Which guard file holds | Later: mark the "Replaces" cells |
| 5 | MINOR (answered) | Approval "Approve all 3" vs hard rule "Rule text" | A short answer as approval | The dialog showed the exact change ("the Administrator" to "the Maintainer", 5 times, possessives the same way) |

Findings 2-4 are in older text; they do not change rule text in this pull request.

rule-conflict-check: b18532e9fb7a

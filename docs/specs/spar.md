# Spec: SPAR

Status: Approved 2026-11-15 (D16) · Revised 2026-11-15 (D17) · Owner: the Administrator · Updated: 2026-11-15

## Goal

People who own SharePoint groups or administer a site lose time on access work: they cannot see who owns which group, who can do what where, or change many memberships in one action. SPAR is a canvas Power App with Power Automate flows that shows this, changes it in bulk, routes requests to group owners, and sorts a unit into groups from the Global Address List (GAL) with one Sync.

## Terms

- Site: one site collection, selected by its address. SPAR reads its top web. A subsite is shown as one node marked "subsite, not read".
- Site admin: an account with Full Control on the site or the site collection administrator flag.
- Group owner: an account that is the owner of a SharePoint group, or a member of the group that is the owner.
- Last owner of the site: the last account that is a site admin.
- Unit group: a SharePoint group that Setup (R7) made for one level of the unit chain. Rule group: a group that a rule (R11) defines.
- Part: the rows that the app sends to the dispatcher flow in one run.
- Fixture mode: a test switch, for a site admin on a TEST site only, that replaces the GAL read with a table of invented entries. The app shows "Fixture mode" on each screen while it is on.

## Done

- Each V check and C1 passes on the work tenant, with a second account on a second site.
- P1, P2, P3, P5, P6 and P7 pass.
- P4, P8, P9 and P10 are measurements: each needs a reported result, not a pass.
- `scripts/check.sh` passes.

## Requirements

Access and rights

- R1. At start SPAR reads the rights of the signed-in account on the selected site and shows only the functions those rights permit: owner functions (R2, R3, R5, R6) for a group owner; all functions for a site admin. R7 to R12, R20 and R22 are for a site admin only. The app only hides functions; SharePoint enforces the rights, because each flow runs as the user.

Views

- R2. "My groups" lists each SharePoint group the user owns (directly or through an owner group) and each group the user is in, with the member count and the members.
- R3. The ownership view shows, for a selected group, its owner, the owner of that owner up to the top, and each group it owns, as a tree. A group whose owner is missing or is a deleted account is marked.
- R4. The access view shows site > libraries and lists > folders, with the permission level of each group and each directly granted person on the selected node; a node with unique permissions is marked. Folders load when the user opens a library or a folder. For the selected node, "Who can" lists the people who can read, contribute and edit, each with the path (direct, or through which group). Files are not shown.
- R17. Microsoft 365 group members are shown read-only as "managed in Microsoft 365". A removal preview shows the count of selected people who keep access through a Microsoft 365 group.
- R21. Other access paths: an Entra security group, an "Everyone" claim and a sharing link are each shown as one row marked "not expanded" on each node and group where they occur. "Who can" (R4), R19 and each removal preview show the count of not-expanded paths, so that no list is shown as complete when it is not.
- R19. "Why does this person have access?": for a selected person, SPAR lists each path (direct grant, or through which group) to each level on the site, its libraries and lists, and each folder already scanned (R20).
- R20. Clean-up list: groups with no owner, empty groups, accounts that no longer exist, direct grants to single persons, and nodes with unique permissions, each with a count. The default scan covers the site, its libraries and lists. A folder scan runs for one library when the user asks, shows its progress, and can be stopped; the list says which libraries are scanned.

Changes

- R5. The user can select many people (single, all, filtered by a search) and in one action add them to, move them between, or remove them from groups the account may edit.
- R6. A user can send a request to the owner of a group to add named people. Requests are rows in a request list on the site; SPAR sends no e-mail. The owner sees pending requests in SPAR, approves or rejects each, and an approved request is applied by the owner's own account. The requester sees the state of each request.
- R12. Site admin functions: remove selected people from the site; delete selected groups; remove direct grants and restore inheritance; delete selected lists, libraries, folders and files.
- R13. Each change (R5, R6, R8 to R10, R12) shows a preview with counts first, then pass or fail for each row, and reads the changed object again to confirm the result.
- R23. Bulk: the app splits a change into parts and sends one part in each flow run; each run ends inside the 120 second limit. The app shows progress (rows done of rows total). If the app closes or a part fails, the rows not done are reported as not done and nothing is applied twice. P5 sets the part size and the call method (one HTTP action for each row, or the SharePoint `$batch` endpoint).
- R14. Delete safety: a typed count is needed for a content delete, a group delete, or a change to 25 or more people; content goes to the site recycle bin only; an action that removes the last owner of the site or of a group, or the user's own Full Control, is blocked.
- R24. Restore data: a deleted SharePoint group and a restored inheritance do not go to a recycle bin. Before either action SPAR writes the members and the permission levels of the object to the change record, and the preview says that the action has no recycle bin.
- R15. Each change writes one row to a change record list on the same site: person, group or object, action, by whom, when. No rank, office symbol, unit or other GAL attribute is stored. The change record is a convenience record that a user with write rights can edit; the Microsoft 365 audit log is the authority, and the record screen says so.

Setup, Sync and rules

- R22. First run: a site admin creates the three SPAR lists on the site (change record, rules, requests) with one button, with the permissions that "Open questions" decides. On a site with no SPAR lists, a group owner sees the read-only views and a notice to ask a site admin; no change function is offered.
- R7. Setup reads the user's own GAL entry, derives the unit chain (office symbol, flight or squadron, group, wing) and suggests one SharePoint group for each level. The user accepts or rejects each suggestion; each accept has a level dropdown (None, Read, Contribute, Edit) with None selected. Each accepted level is saved as a rule (R11) for its unit group.
- R8. Sync, leavers: one button checks the GAL and proposes removal from all SharePoint groups and all direct grants on the site for (a) each member of a unit group or a rule group whose GAL entry no longer matches the rule of that group, and (b) each person on the site whose account is gone. A person who was never in a unit group or a rule group is not a leaver.
- R9. Sync, arrivals: the same button lists GAL people who match the rule of a unit group and are not in it, and proposes the adds. Built only if probe row P4 or P9 passes; if not, SPAR offers "add by search" in its place and the Sync screen says so.
- R10. Sync never changes anything before the user presses Apply. The proposal lists each add, move and removal with the rule that matched; the user can untick rows.
- R11. A rule builder defines custom groups: attribute dropdown (rank, office symbol, unit, job title, department), operator, value, and a search for terms, with a live count of matches. Rules are saved in a list on the site. A rule holds attribute values (for example an office symbol) and never a named person; a named person is added to a group with R5.

Data and delivery

- R16. No export: no GAL attribute of a person is written to a file, an e-mail, the change record, the request list, or flow run history. The trigger and every action of each flow have secure inputs and secure outputs on (C1).
- R18. Delivery: SPAR is a RADE application (https://github.com/rhope-ops/RADE/blob/main/docs/make-an-app.md). Each wing imports its own copy through a RADE guided step. The step also shares the app and the run-only use of the flows with the group the wing names, and names a second owner. The app shows its installed version. It uses standard connectors only and needs no install on the user's computer.

## Out of scope

- Changes to Microsoft 365 group membership.
- Expanding the members of an Entra security group, an "Everyone" claim or a sharing link (R21 shows them as not expanded).
- Drag and drop.
- "Copy access from" a named person, and a side-by-side compare of two people or groups (agent proposals, not kept on 2026-11-15).
- Permission edits on single files.
- Any export, report file or e-mail that holds names or GAL attributes; any e-mail or Approvals notice.
- Scheduled or unattended Sync.
- More than one site collection in one action; reading subsites.
- Tenant or environment settings.
- RSAT, Get-User, a PowerShell window, or any directory read other than the GAL through the Office 365 Users connector and the SharePoint people search.

## Edge cases and errors

- E1. The account owns a group but cannot read site permissions -> the access view shows only what the account can read, with a notice; no error.
- E2. A GAL field is blank or does not parse -> the person is listed under "Not sorted" and is never proposed for removal for that reason.
- E3. A GAL read fails or is throttled during Sync -> Sync proposes no removals and says which part was not read.
- E4. The owner of a group is a deleted account, or ownership forms a loop (A owns B, B owns A) -> the ownership view marks it and still draws.
- E5. A request goes to a group with no valid owner -> it goes to the site admins.
- E6. A view has more rows than the app's data row limit -> SPAR pages or says "showing N of M"; it never shows a partial list as complete.
- E7. A part is throttled or passes the 120 second limit -> the app sends the remaining rows in smaller parts; rows not done are reported as not done (R23).
- E8. The group changed after the preview was made (a second person) -> SPAR reads again before Apply and shows the difference.
- E9. An action would remove the last owner or the user's own Full Control -> blocked with the reason (R14).
- E10. A connector that SPAR needs is blocked by the environment's data policy, or the import fails -> the RADE step reports FAILED with the connector name; nothing is half installed.
- E11. The site has no SPAR lists, or the user cannot write to the request list -> SPAR says who can fix it (R22); no change is made without its change record row.
- E12. A folder scan is stopped or fails -> the clean-up list and R19 mark that library "partly scanned" with the count read.

## Constraints

- Platform: canvas Power App and Power Automate cloud flows; standard connectors only, and only two: SharePoint and Office 365 Users. Each write to a group or a permission is a flow that uses "Send an HTTP request to SharePoint" as the user.
- One window, central (D15): the app and its flows are in one solution. The app sends an operation name and a part (a list of rows) to one dispatcher flow, which makes the SharePoint calls and returns pass or fail for each row. A new operation is a new branch of that flow, not a new flow. No script runs on the user's computer. A canvas Power App cannot start PowerShell on a computer without a premium desktop flow, a cloud runbook or an installed handler; none of these is permitted here.
- Tests: a SharePoint group accepts only real directory accounts. Membership checks use real test accounts and report counts only. Checks of GAL logic (Setup, Sync, rules) use fixture mode with invented entries.
- The work tenant's rules in https://github.com/rhope-ops/RADE/blob/main/docs/engine.md apply: no reconnaissance; no write across the DoD365 boundary; a probe says what it reads and reports counts only.
- Published limits (Microsoft, read 2026-11-15 from the documentation source on GitHub; Confidence: high for the text, medium for this tenant):
  - Office 365 Users has 5 functions: DirectReports, Manager, MyProfile, SearchUser, UserProfile. The search term "Applies to: display name, given name, surname, mail, mail nickname, and user principal name". It has no filter by department, office or company. A profile gives CompanyName, Department, DisplayName, JobTitle, OfficeLocation and others. Source: https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/connections/connection-office365-users
  - Flow run retention in storage: 30 days. Inbound and outbound synchronous request: 120 seconds. Apply to each: 5,000 items (Low), concurrency 1 to 50. Source: https://learn.microsoft.com/en-us/power-automate/limits-and-config
  - Non-delegable query in a canvas app: first 500 records by default; the limit can be raised in the app settings. Source: https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/delegation-overview
- Proven on this tenant: a flow with 52 "Send an HTTP request to SharePoint" actions ran as the user (https://github.com/rhope-ops/90-161.Z/blob/main/flow/defs/90161Z-T11-rights-probe-r5.json); solution import by a guided step (https://github.com/rhope-ops/90-161.Z/blob/main/flow/solution/README.md).
- Personal data: examples and fixture entries use invented people. The repo is public: no site address, name or probe answer goes in it.

## Decisions

All by the Administrator, 2026-11-15, scope SPAR.

| ID | Decision | Alternatives rejected, and why |
|---|---|---|
| D1 | SPAR is a canvas Power App with Power Automate flows, so that it can be managed as an app, with no install on the computer. Drag and drop is not required. | WPF window app (rule of 2026-11-14): replaced |
| D2 | One copy for each wing, imported through a RADE guided step. The code and the package live in rhope-ops/SPAR. | One shared app in one environment; SPAR inside the RADE window |
| D3 | Users: anyone who owns a SharePoint group. What a user sees and can do follows the rights of the account. | Site admins only |
| D4 | SharePoint groups only; Microsoft 365 group members are read-only. | Changing Microsoft 365 groups: route not proven |
| D5 | Directory data comes from the GAL only. No RSAT and no AD browse: other units may not have them. It must work at once, with no install. Reading people's attributes is approved for this purpose, provided nothing is ever exported. | Get-User |
| D6 | Ownership view: who owns a group and which groups it owns, drawn as a chain that flows up. | — |
| D7 | Access view: which groups have which level on which site, library and folder, and who can read, contribute or edit there. File level only if it is simple. | — |
| D8 | A user can send a request to a group's owner to add people; the owner approves. | — |
| D9 | Setup reads the user's own entry and suggests unit groups (office symbol -> flight or squadron -> group -> wing); accept or reject each; level asked at each accept. | Read on the whole site by default |
| D10 | Custom groups come from a rule builder with dropdowns and a search for terms. Rank is a main attribute. (D17: no named users in a rule.) | — |
| D11 | Sync proposes, then applies. Offboard = proposed removal from all SharePoint groups and direct grants on the site. | Apply at once; automatic groups only; flag only |
| D12 | The change record is a list on the same site; no file on the computer. | No record; counts only |
| D13 | Bulk delete covers people from groups, people from the site, groups and direct grants, and content, limited to what the account's rights permit. Safety: preview with counts, typed confirm, recycle bin only, never the last owner or yourself. | — |
| D14 | Measure first: the platform (D1) is confirmed or changed after the probe round, from P5, P8 and P9. D15 is newer and wins where the two differ: a PowerShell window is not a candidate, and it would need a new decision that replaces D15. If the flow route fails P5, the spec changes first. | Change the platform now, with no numbers |
| D15 | One window: the Power App fires a modular operation for what is done on screen, through one dispatcher flow in the same solution. Central, nothing for the user to extract or run. No obfuscation. | PowerShell scripts started by the app (not possible with no install); a second PowerShell window beside the app |
| D16 | The spec is approved. "Why does this person have access" (R19) and the clean-up list (R20) are kept. The typed confirm starts at 25 people (R14). | "Copy access from" and compare: not kept |
| D17 | Revision after an agent review, 11 changes: leavers are defined from unit and rule groups, with no stored baseline (R8); real test accounts and fixture mode (Terms, Constraints); D15 wins over D14 and P9 runs from the flow; the app sends parts, and P5 tests `$batch` (R23); rules hold attribute values and no named users, and C1 checks the secure settings (R11, R16); other access paths are shown as not expanded (R21); folder scans on demand (R20, P10); restore data before a group delete or a restored inheritance (R24); the change record is a convenience record (R15); first run and sharing (R22, R18); "site" and "last owner" defined, R7 to R12 for site admins, no Approvals or Outlook. | The spec as approved in D16: the first Sync would propose removal of every person outside the unit, and the fixtures could not exist |

## Acceptance checks

Probe rows (P) run in one RADE round on the work tenant and report pass or fail and counts only, no names. Device checks (V) use a TEST site, real test accounts for membership, fixture mode for GAL entries, and a second account. C1 runs in the repo.

| ID | Covers | Check | Pass when |
|---|---|---|---|
| P1 | R5, R12 | Flow adds, then removes, 1 member of a TEST group; once as a site admin, once as a second account that is only the owner of that group | Each HTTP result 2xx for both accounts; member count +1, then back |
| P2 | R3 | Flow reads the owner of each SharePoint group | A result for each group; count reported |
| P3 | R7 | Flow reads MyProfile; reports which of the 5 unit fields are filled | Office symbol and unit can be derived (yes or no for each level) |
| P4 | R9 | Flow tries to list people by office symbol with SearchUser | Measurement: count returned, and whether it equals the known count of a small test unit |
| P5 | R23, E7 | Flow writes 50, 500 and 2,000 memberships (about 50 test accounts in 40 TEST groups), once with one HTTP action for each row and once with `$batch`; time each | Times and flow action counts reported for both methods; none lost; no run over 120 s at the chosen part size |
| P6 | R16 | Run history of a flow with secure inputs and outputs on the trigger and each action | The person fields show as hidden |
| P7 | R18, E10 | Save and import with SharePoint and Office 365 Users in one environment | Import ends with no data policy error; else the step prints FAILED with the connector name |
| P8 | R23 | The same 50 membership changes on a TEST group by PowerShell with PnP.PowerShell 1.12.0; time it | Measurement: seconds reported beside P5. Reference only (D14, D15) |
| P9 | R9 | Flow lists people by office symbol through the SharePoint people search ("Send an HTTP request to SharePoint"), with the user's own account | Measurement: count returned, and whether it equals the known count of the small test unit of P4 |
| P10 | R20, R19 | Flow reads the unique-permission flag of each folder in the largest TEST library | Measurement: folders read, seconds, and folders for each second |
| C1 | R16 | `scripts/check.local.sh` reads each flow definition in the repo | The trigger and every action have secure inputs and secure outputs on; else FAIL with the action name |
| V1 | R1, E1, R22, E11 | Sign in as: group owner with no other rights on the site; site admin; group owner on a site with no SPAR lists | Owner sees only owner functions and the E1 notice; admin sees all; on the third site only read-only views and the R22 notice |
| V2 | R2, R3, E4 | Test site with 6 groups in a 3-level ownership chain, 1 ownerless group, 1 loop | Tree matches the fixture; the 2 defects are marked |
| V3 | R4, E6 | Fixture: 2 libraries, 3 folders, 2 with unique permissions; 1 list above the row limit | Levels and "Who can" match the fixture; the long view says "showing N of M" |
| V4 | R5, R13, R15, R23 | Select 20 people, add to 2 groups, move 5, remove 5; then close the app during a change of 200 rows | Preview counts = 20, 5, 5; each row PASS; re-read matches; 30 record rows; after the close, rows done + rows not done = 200 and a second run adds none twice |
| V5 | R6, E5 | Second account requests 3 people; owner approves 2, rejects 1; 1 request to an ownerless group | 2 added by the owner's account; requester sees 2 approved, 1 rejected; the ownerless request is with the site admins; no e-mail is sent |
| V6 | R7 | Setup in fixture mode with an invented entry; accept 3, reject 1, levels None, Read, Contribute | 3 groups exist with those levels and 3 rules; the rejected one does not |
| V7 | R8, R10, E2, E3 | Fixture mode: in unit groups 2 leavers and 1 blank office field; 1 deleted account; 5 people on the site who were never in a unit or rule group; then force a GAL read failure | Proposal = 3 removals, 1 "Not sorted", 0 of the 5; nothing changes before Apply; unticked row is not applied; on failure 0 removals |
| V8 | R9 | Only if P4 or P9 passed: 4 new people in the unit (fixture mode) | Proposal = 4 adds; else the screen shows the "add by search" text |
| V9 | R11 | Rule "rank is X and office symbol starts with Y" in fixture mode | Live count = fixture count; the rule row holds the 2 values and no person; the builder offers no way to put a named person in a rule |
| V10 | R12, R14, R24, E9 | Delete 1 group, 1 direct grant, 1 folder; restore inheritance on 1 folder; try to remove the last owner and yourself | Typed count asked; folder is in the recycle bin; the change record holds the members and levels of the group and of the folder; the 2 removals are blocked |
| V11 | R16, R15 | After V4 to V10: search the change record, the request list and the flow run history for the invented rank and office values | 0 hits |
| V12 | R17 | Remove 3 people, 2 of whom are in the Microsoft 365 group | Preview shows "2 keep access through Microsoft 365" |
| V13 | E8 | A second account removes a member between preview and Apply | SPAR shows the difference before it applies |
| V14 | R18 | A second wing account imports through the RADE step on a second site | Import PASS; the app shows its version; a user in the named group can open the app and run a change; V1 passes there |
| V15 | R19 | Fixture person with 1 direct grant and access through 2 groups | 3 paths shown, each with the right level and node |
| V16 | R20, E12 | The V2 and V3 fixtures plus 1 empty group and 1 deleted account; scan 1 library, stop the scan of a second | Each of the 5 lists shows the fixture count; the second library is marked "partly scanned" |
| V17 | R21 | Fixture node with 1 Entra security group, 1 "Everyone" claim and 1 sharing link | 3 rows marked "not expanded"; "Who can" and a removal preview on that node show the count 3 |

## Assumptions

- The office symbol, unit and rank can be read from a GAL entry (display name, job title, department, office or company) · the Administrator's old script did this with directory attributes · medium; P3 decides.
- A group owner who is not a site admin can change the members of the owned group by flow, and can read enough of the site for SPAR to start · SharePoint group setting "who can edit membership = owner" (capability report, function 11) · medium; P1 with the second account decides.
- The wing's environment permits the 2 connectors together · 90-161.Z rule R7 (https://github.com/rhope-ops/90-161.Z/blob/main/docs/setup-and-updates.md) · medium; P7 decides.
- SearchUser matches only the start of each field, so P4 probably fails where the office symbol is not at the start of the display name · agent's reading of the connector, not checked on the tenant · medium; P4 decides.
- "Send an HTTP request to SharePoint" can call the `$batch` endpoint and the people search as the user · community practice, not proven on this tenant · medium; P5 and P9 decide.
- "Site" means one site collection and subsites are not read (Terms) · agent's choice from "Out of scope"; the Administrator approved the need for a definition, not this wording · medium.

## Open questions

- The platform after the probe (D14) · the Administrator, from P5 and P9, with P8 as a reference · every build phase after 0. If it changes, this spec changes first.
- How the change record, rules and request lists are protected when every user must write to them (REC: item-level "read own" on the record and the requests, read-only rules for group owners; proven as capability function 21) · decided after the probe · build phase 2.
- If P4 and P9 both fail: is "add by search" enough for onboarding, or does the wing keep a roster list on the site · the Administrator · build phase 5.

## Risks

- The GAL connector searches by name, not by unit, so Sync may find leavers but not arrivals · REC: R8 and R9 are separate; P4 and P9 decide R9.
- Offboarding is not complete while access comes from a Microsoft 365 group, an Entra security group, an "Everyone" claim, a sharing link or a grant on a single file · REC: R17 and R21 show the counts; moving access to SharePoint groups is a later, separate decision.
- Flow run history keeps data for 30 days · REC: R16, C1, P6 and V11.
- A bad GAL field or a bad rule removes access from many people · REC: R8 limits leavers to unit and rule groups; R10, E2, E3.
- Fixture mode on a real site would propose changes from invented data · REC: site admin only, shown on each screen, and Apply still needs the preview and the typed count.
- One copy for each wing means each update is a new import at each wing · REC: the version in the app and RADE's newer-version notice.
- A flow is permanent after the import, but Microsoft may turn off a flow that is not triggered for 90 days (source: the limits page in Constraints), and a flow whose importing account is removed has no owner · REC: at start the app checks that the dispatcher flow answers and says what to do if not; the RADE step names a second owner (R18).
- Bulk speed depends on flow limits, and a large change can reach the daily request limit of the user's licence · REC: P5 reports the action count and sets the part size and the method; P8 is the reference.
- A folder scan of a large library is slow · REC: R20 scans folders only on demand; P10 gives the rate.

## Build order

Each phase needs its own approval. The spec changes first if a probe result changes a requirement.

| Phase | Content | Blocked by |
|---|---|---|
| 0 | Probe round P1 to P10, then the platform decision (D14) | — |
| 1 | Read-only: R1 to R4, R17, R19, R20, R21 | P2, P10 |
| 2 | First run and changes: R22, R5, R13, R14, R15, R16, R23, C1 | P1, P5, P6 |
| 3 | Requests: R6 | Phase 2 |
| 4 | Setup, Sync leavers, rule builder, fixture mode: R7, R8, R10, R11 | P3 |
| 5 | Sync arrivals: R9 | P4, P9 |
| 6 | Site admin deletes: R12, R24 | Phase 2 |
| 7 | RADE packaging, sharing and second-wing import: R18 | P7, phase 1 |

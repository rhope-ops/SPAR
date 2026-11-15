# Spec: SPAR

Status: Draft · Owner: the Administrator · Updated: 2026-11-15

## Goal

People who own SharePoint groups or administer a site lose time on access work: they cannot see who owns which group, who can do what where, or change many memberships in one action. SPAR is a canvas Power App with Power Automate flows that shows this, changes it in bulk, routes requests to group owners, and sorts a unit into groups from the Global Address List (GAL) with one Sync.

## Done

Each acceptance check below passes: probe rows and device rounds on the work tenant, a second account on a second site, and the no-export search. `scripts/check.sh` passes.

## Requirements

- R1. At start SPAR reads the rights of the signed-in account on the selected site and shows only the functions those rights permit: owner functions (R2, R3, R5, R6) for an account that owns a group; all functions for an account with Full Control or the site collection administrator flag.
- R2. "My groups" lists each SharePoint group the user owns and each group the user is in, with the member count and the members.
- R3. The ownership view shows, for a selected group, its owner, the owner of that owner up to the top, and each group it owns, as a tree. A group whose owner is missing or is a deleted account is marked.
- R4. The access view shows site > libraries and lists > folders, with the permission level of each group and each directly granted person on the selected node; a node with unique permissions is marked. For the selected node, "Who can" lists the people who can read, contribute and edit, each with the path (direct, or through which group). Files are not shown.
- R5. The user can select many people (single, all, filtered by a search) and in one action add them to, move them between, or remove them from groups the account may edit.
- R6. A user can send a request to the owner of a group to add named people. The owner sees pending requests in SPAR, approves or rejects each, and an approved request is applied by the owner's own account. The requester sees the state of each request.
- R7. Setup reads the user's own GAL entry, derives the unit chain (office symbol, flight or squadron, group, wing) and suggests one SharePoint group for each level. The user accepts or rejects each suggestion; each accept has a level dropdown (None, Read, Contribute, Edit) with None selected.
- R8. Sync, leavers: one button checks each person on the site against the GAL and proposes removal from all SharePoint groups and all direct grants on the site for each person who is no longer in an accepted unit or whose account is gone.
- R9. Sync, arrivals: the same button lists GAL people in the accepted units who are not in the matching group and proposes the adds. Built only if probe row P4 passes; if not, SPAR offers "add by search" in its place and the Sync screen says so.
- R10. Sync never changes anything before the user presses Apply. The proposal lists each add, move and removal with the rule that matched; the user can untick rows.
- R11. A rule builder defines custom groups: attribute dropdown (rank, office symbol, unit, job title, department), operator, value, and a search for named users and terms, with a live count of matches. Rules are saved in a list on the site and hold no person data.
- R12. Site admin functions: remove selected people from the site; delete selected groups; remove direct grants and restore inheritance; delete selected lists, libraries, folders and files.
- R13. Each change (R5, R6, R8 to R10, R12) shows a preview with counts first, then pass or fail for each row, and reads the changed object again to confirm the result.
- R14. Delete safety: a typed count is needed for a content delete, a group delete, or a change to 25 or more people (REC number, see Open questions); content goes to the site recycle bin only; an action that removes the last owner of the site or of a group, or the user's own Full Control, is blocked.
- R15. Each change writes one row to a change record list on the same site: person, group or object, action, by whom, when. No rank, office symbol, unit or other GAL attribute is stored.
- R16. No export: no GAL attribute is written to a file, an e-mail, the change record, the rules list, or flow run history. Each flow action that carries person data has secure inputs and secure outputs on.
- R17. Microsoft 365 group members are shown read-only as "managed in Microsoft 365". A removal preview shows the count of selected people who keep access through a Microsoft 365 group.
- R18. Delivery: SPAR is a RADE application (https://github.com/rhope-ops/RADE/blob/main/docs/make-an-app.md). Each wing imports its own copy through a RADE guided step. The app shows its installed version. It uses standard connectors only and needs no install on the user's computer.

REC, optional requirements (agent proposals, not decisions; each stays out of the build until the Administrator keeps it):

- O1. "Why does this person have access?": select a person, see each path to each level on each node.
- O2. Clean-up list: groups with no owner, empty groups, accounts that no longer exist, direct grants to single persons, nodes with unique permissions.
- O3. "Copy access from": give a new person the same groups as a named person.
- O4. Compare two people or two groups side by side.

## Out of scope

- Changes to Microsoft 365 group membership.
- Drag and drop.
- Permission edits on single files.
- Any export, report file or e-mail that holds names or GAL attributes.
- Scheduled or unattended Sync.
- More than one site collection in one action.
- Tenant or environment settings.
- RSAT, Get-User, or any directory read other than the GAL through the Office 365 Users connector.

## Edge cases and errors

- E1. The account owns a group but cannot read site permissions -> the access view shows only what the account can read, with a notice; no error.
- E2. A GAL field is blank or does not parse -> the person is listed under "Not sorted" and is never proposed for removal for that reason.
- E3. A GAL read fails or is throttled during Sync -> Sync proposes no removals and says which part was not read.
- E4. The owner of a group is a deleted account, or ownership forms a loop (A owns B, B owns A) -> the ownership view marks it and still draws.
- E5. A request goes to a group with no valid owner -> it goes to the site owners.
- E6. A view has more rows than the app's data row limit -> SPAR pages or says "showing N of M"; it never shows a partial list as complete.
- E7. A flow is throttled or a batch passes the 120 second limit -> the batch continues in parts; rows not done are reported as not done.
- E8. The group changed after the preview was made (a second person) -> SPAR reads again before Apply and shows the difference.
- E9. An action would remove the last owner or the user's own Full Control -> blocked with the reason (R14).
- E10. A connector that SPAR needs is blocked by the environment's data policy, or the import fails -> the RADE step reports FAILED with the connector name; nothing is half installed.

## Constraints

- Platform: canvas Power App and Power Automate cloud flows; standard connectors only (SharePoint, Office 365 Users, Approvals or Outlook if R6 needs a notice). Each write to a group or a permission is a flow that uses "Send an HTTP request to SharePoint" as the user.
- One window, central (D15): the app and its flows are in one solution. The app sends an operation name and a list of rows to one dispatcher flow, which makes the SharePoint calls and returns pass or fail for each row. A new operation is a new branch of that flow, not a new flow. No script runs on the user's computer. A canvas Power App cannot start PowerShell on a computer without a premium desktop flow, a cloud runbook or an installed handler; none of these is permitted here.
- The work tenant's rules in https://github.com/rhope-ops/RADE/blob/main/docs/engine.md apply: no reconnaissance; no write across the DoD365 boundary; a probe says what it reads and reports counts only.
- Published limits (Microsoft, read 2026-11-15 from the documentation source on GitHub; Confidence: high for the text, medium for this tenant):
  - Office 365 Users has 5 functions: DirectReports, Manager, MyProfile, SearchUser, UserProfile. The search term "Applies to: display name, given name, surname, mail, mail nickname, and user principal name". It has no filter by department, office or company. A profile gives CompanyName, Department, DisplayName, JobTitle, OfficeLocation and others. Source: https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/connections/connection-office365-users
  - Flow run retention in storage: 30 days. Inbound and outbound synchronous request: 120 seconds. Apply to each: 5,000 items (Low), concurrency 1 to 50. Source: https://learn.microsoft.com/en-us/power-automate/limits-and-config
  - Non-delegable query in a canvas app: first 500 records by default; the limit can be raised in the app settings. Source: https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/delegation-overview
- Proven on this tenant: a flow with 52 "Send an HTTP request to SharePoint" actions ran as the user (https://github.com/rhope-ops/90-161.Z/blob/main/flow/defs/90161Z-T11-rights-probe-r5.json); solution import by a guided step (https://github.com/rhope-ops/90-161.Z/blob/main/flow/solution/README.md).
- Personal data: tests and examples use invented people. The repo is public: no site address, name or probe answer goes in it.

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
| D10 | Custom groups come from a rule builder with dropdowns and a search for users and terms. Rank is a main attribute. | — |
| D11 | Sync proposes, then applies. Offboard = proposed removal from all SharePoint groups and direct grants on the site. | Apply at once; automatic groups only; flag only |
| D12 | The change record is a list on the same site; no file on the computer. | No record; counts only |
| D14 | Measure first: the platform (D1) is confirmed or changed after the probe round, from P5, P8 and P9. Candidates: Power App only; Power App for owners plus a PowerShell and PnP window for the admin's bulk work and Sync; window app only. RSAT is optional in each. | Change the platform now, with no numbers |
| D15 | One window: the Power App fires a modular operation for what is done on screen, through one dispatcher flow in the same solution. Central, nothing for the user to extract or run. No obfuscation. | PowerShell scripts started by the app (not possible with no install); a second PowerShell window beside the app |
| D13 | Bulk delete covers people from groups, people from the site, groups and direct grants, and content, limited to what the account's rights permit. Safety: preview with counts, typed confirm, recycle bin only, never the last owner or yourself. | — |

## Acceptance checks

Probe rows (P) run in one RADE round on the work tenant and report pass or fail and counts only, no names. Device checks (V) use a TEST site with invented people and a second account.

| ID | Covers | Check | Pass when |
|---|---|---|---|
| P1 | R5, R12 | Flow adds, then removes, 1 member of a TEST group | Both HTTP results 2xx; member count +1, then back |
| P2 | R3 | Flow reads the owner of each SharePoint group | A result for each group; count reported |
| P3 | R7 | Flow reads MyProfile; reports which of the 5 unit fields are filled | Office symbol and unit can be derived (yes or no for each level) |
| P4 | R9 | Flow tries to list people by office symbol with SearchUser | Count returned equals the known count of a small test unit |
| P5 | R5, E7 | Flow writes 50, 500 and 2,000 memberships; time each | Times reported; none lost; batches over 120 s continue in parts |
| P6 | R16 | Run history of a flow with secure inputs and outputs on | The person fields show as hidden |
| P8 | R5, E7 | The same 50 membership changes on a TEST group, by flow (P5) and by PowerShell with PnP.PowerShell 1.12.0; time each | Both times reported in seconds; the Administrator decides the platform from them (D14) |
| P9 | R9 | PowerShell with PnP, no RSAT: list people by office symbol through the SharePoint people search, with the user's own account | Count returned equals the known count of the small test unit of P4 |
| P7 | R18, E10 | Save and import with SharePoint, Office 365 Users and Approvals in one environment | Import ends with no data policy error; else the step prints FAILED with the connector name |
| V1 | R1, E1 | Sign in as: group owner with no site rights; site admin | Owner sees only owner functions and the E1 notice; admin sees all |
| V2 | R2, R3, E4 | Test site with 6 groups in a 3-level ownership chain, 1 ownerless group, 1 loop | Tree matches the fixture; the 2 defects are marked |
| V3 | R4, E6 | Fixture: 2 libraries, 3 folders, 2 with unique permissions; 1 list above the row limit | Levels and "Who can" match the fixture; the long view says "showing N of M" |
| V4 | R5, R13, R15 | Select 20 people, add to 2 groups, move 5, remove 5 | Preview counts = 20, 5, 5; each row PASS; re-read matches; 30 record rows |
| V5 | R6, E5 | Second account requests 3 people; owner approves 2, rejects 1; 1 request to an ownerless group | 2 added by the owner's account; requester sees 2 approved, 1 rejected; the ownerless request is with the site owners |
| V6 | R7 | Setup with an invented entry; accept 3, reject 1, levels None, Read, Contribute | 3 groups exist with those levels; the rejected one does not |
| V7 | R8, R10, E2, E3 | Fixture: 2 leavers, 1 deleted account, 1 blank office field; then force a GAL read failure | Proposal = 3 removals, 1 "Not sorted"; nothing changes before Apply; unticked row is not applied; on failure 0 removals |
| V8 | R9 | Only if P4 passed: 4 new people in the unit | Proposal = 4 adds; else the screen shows the "add by search" text |
| V9 | R11 | Rule "rank is X and office symbol starts with Y" on the fixture | Live count = fixture count; the rules list holds no person data |
| V10 | R12, R14, E9 | Delete 1 group, 1 direct grant, 1 folder; try to remove the last owner and yourself | Typed count asked; folder is in the recycle bin; the 2 removals are blocked |
| V11 | R16, R15 | After V4 to V10: search the site lists, flow run history and the user's computer for the invented rank and office values | 0 hits |
| V12 | R17 | Remove 3 people, 2 of whom are in the Microsoft 365 group | Preview shows "2 keep access through Microsoft 365" |
| V13 | E8 | A second account removes a member between preview and Apply | SPAR shows the difference before it applies |
| V14 | R18 | A second wing account imports through the RADE step on a second site | Import PASS; the app shows its version; V1 passes there |

## Assumptions

- The office symbol, unit and rank can be read from a GAL entry (display name, job title, department, office or company) · the Administrator's old script did this with directory attributes · medium; P3 decides.
- A group owner who is not a site admin can change the members of the owned group by flow · SharePoint group setting "who can edit membership = owner" (capability report, function 11) · medium; P1 with the second account decides.
- The wing's environment permits the 3 connectors together · 90-161.Z rule R7 (https://github.com/rhope-ops/90-161.Z/blob/main/docs/setup-and-updates.md) · medium; P7 decides.

## Open questions

- The platform after the probe (D14) · the Administrator, from P5, P8, P9 · every build phase after 0. If it changes, this spec changes first.
- Which of O1 to O4 are kept · the Administrator · the size of build phase 1.
- The number for a "large" change in R14 (REC: 25) · the Administrator · build phase 2.
- How the change record and rules lists are protected when every user must write to them (REC: item-level "read own" on the record, proven as capability function 21) · decided after the probe · build phase 2.
- If P4 fails: is "add by search" enough for onboarding, or does the wing keep a roster list on the site · the Administrator · build phase 5.

## Risks

- The GAL connector searches by name, not by unit, so Sync may find leavers but not arrivals · REC: R8 and R9 are separate; P4 decides R9. A display name that holds the office symbol may make P4 pass.
- Offboarding is not complete while access comes from a Microsoft 365 group · REC: R17 shows the count; moving access to SharePoint groups is a later, separate decision.
- Flow run history keeps data for 30 days · REC: R16, P6 and V11.
- A bad GAL field removes access from many people · REC: R10, E2, E3.
- One copy for each wing means each update is a new import at each wing · REC: the version in the app and RADE's newer-version notice.
- A flow is permanent after the import, but Microsoft may turn off a flow that is not triggered for 90 days (source: the limits page in Constraints), and a flow whose importing account is removed has no owner · REC: at start the app checks that the dispatcher flow answers and says what to do if not; each wing names a second owner at import.
- Bulk speed depends on flow limits: each action in a Power App is a flow run · REC: P5 sets the batch size; P8 compares it with PnP before the build (D14).

## Build order

Each phase needs its own approval. The spec changes first if a probe result changes a requirement.

| Phase | Content | Blocked by |
|---|---|---|
| 0 | Probe round P1 to P9, then the platform decision (D14) | — |
| 1 | Read-only: R1 to R4, R17 (and O1, O2 if kept) | P2 |
| 2 | Changes: R5, R13, R14, R15, R16 | P1, P5, P6 |
| 3 | Requests: R6 | Phase 2 |
| 4 | Setup, Sync leavers, rule builder: R7, R8, R10, R11 | P3 |
| 5 | Sync arrivals: R9 | P4 |
| 6 | Site admin deletes: R12 | Phase 2 |
| 7 | RADE packaging and second-wing import: R18 | P7, phase 1 |

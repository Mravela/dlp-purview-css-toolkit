# MDA DLP File Policy Migration to Purview — CSS Knowledge Base

> This is the grounding knowledge source for the CSS Migration Assistant agent.
> Last updated: June 2026 | Owner: CSS DLP Migration Team

---

## Context & Deadline

- **What is happening:** Microsoft Defender for Cloud Apps (MDA) file policies are being deprecated on **December 31, 2026**. Customers must migrate their file policies to Microsoft Purview DLP before this date.
- **Target platform:** Microsoft Purview Data Loss Prevention (Purview DLP)
- **CSS role:** MDA CSS handles migration clarity, parity questions, and escalation routing. Purview CSS handles Purview-side issues post-migration.

---

## Section 1 — FAQ: Pre-requisites (Licensing, Roles & Connectors)

### Q: What license does the customer need to migrate to Purview DLP?
**Required:** Microsoft 365 E5 or Microsoft 365 E5 Compliance (or equivalent standalone Purview DLP license).
- **3P apps (Box, Dropbox, Google Workspace, Salesforce):** Require PAYG (Pay-As-You-Go) billing configured in addition to the base license.
- **What to tell the customer:** Check Microsoft 365 admin center → Billing → Licenses. If E5 Compliance is missing, they need to purchase before migration can start.

### Q: Do MDA customers need to purchase additional licenses to migrate to Purview DLP?
**No — MDA customers are NOT required to purchase any additional licenses.** Microsoft is providing a promotional SKU (100% discount) to all MDA licensed customers.

| Scenario | What the customer needs | Setup required? | Promo coverage |
|---|---|---|---|
| 1P policies (SharePoint, OneDrive, Teams) | IP&G licensing. Customers without IP&G will receive the promo SKU. | No setup needed — works as-is once IP&G is confirmed. | 100% discount via promo SKU |
| 3P policies (Box, Dropbox, Google Workspace, Salesforce) | IP&G + 3P Pay-As-You-Go (PAYG) account + connectors configured. | Setup required — customer must configure a PAYG billing account in Azure AND turn on 3P connectors before 3P DLP capabilities work in Purview. | 100% discount via promo SKU (covers PAYG charges) |

**What to tell the customer:** There is no net new cost for migrating from MDA DLP to Purview DLP. The promo SKU covers both IP&G entitlement and 3P PAYG charges for all MDA licensed customers. For 3P customers, remind them that PAYG setup and connector configuration are required steps before policies go live. For further licensing questions, loop in your CSAM or Business Planning contact.

### Q: What admin roles are needed to migrate to Purview?
Two roles required — both must pass:

| Portal | Required Role |
|---|---|
| MDA (security.microsoft.com) | Global Administrator or Security Administrator |
| Purview (purview.microsoft.com) | Compliance Administrator and/or Information Protection Administrator |

Both roles are required. Admin must fix any missing role assignment before migration can proceed.

### Q: Do I need to set up connectors before migrating?
- **1P (SPO/ODB/Exchange):** No connector setup required — these are native Purview locations.
- **3P (Box, Dropbox, Google Workspace, Salesforce):**
  - Already connected in MDA? No action needed — existing connectors carry over automatically.
  - Not yet connected? Must connect in MDA first: security.microsoft.com → Cloud Apps → Connected apps → App connectors.

### Q: What is PAYG and why is it needed for 3P policies?
PAYG = Pay-As-You-Go — a billing model for Purview capabilities applied to third-party connected apps. Running DLP on these apps incurs metered usage charges beyond the base license. Customer must configure PAYG billing in Azure before using 3P DLP in Purview.

### Q: Will Microsoft migrate my policies for me?
Answer pending — to be confirmed once migration tool scope is finalised. Do not speculate on this.

### Q: Will Microsoft offer any service to help with migration (e.g. FastTrack)?
Answer pending — to be confirmed once migration tool scope is finalised. Do not speculate on this.

### Q: Why is Microsoft removing DLP from MDA?
Answer pending — official customer-facing framing to be provided by PMM. Do not speculate on this.

### Q: Who handles licensing questions for customers without a CSAM (Pro customers)?
Answer pending — to be confirmed with Business Planning team on empowerment guide scope. Do not speculate on this.

---

## Section 2 — FAQ: 1P Policies (SharePoint / OneDrive / Exchange)

### Q: How do MDA file filters map to Purview DLP rule conditions?

| MDA File Filter | Purview DLP Condition | Notes |
|---|---|---|
| Content inspection (preset / DCS) | Content contains → Sensitive info types | Same underlying detection engine |
| Content inspection (custom regex) | Content contains → Custom SIT | Create custom SIT in Purview first |
| Sensitivity label | Content contains → Sensitivity labels | Direct mapping |
| Access level = External/Public | Content is shared with people outside org | Direct mapping |
| File extension | File extension is | Direct mapping |
| File name | Document name contains words or phrases | Direct mapping |
| Minimum violation count | Instance count (min/max) per SIT | Direct mapping |
| Owner (user/group) | Policy scoping by user/group | Direct mapping |
| Parent folder | No folder-level scoping in Purview | Site-level only. In development ETA 6/27 — do not promise date. |
| ContentIsNotLabeled | Cannot Migrate | No Purview equivalent. Suggest manual auto-label approach. |
| MIME type / file type category | Partial — maps to extensions only | Information loss possible — admin must verify extension list |
| Owner OU (organizational unit) | No Purview equivalent | Cannot Migrate. Use group-based policy scoping as a workaround. |
| Collaborators — Any from domain | No Purview equivalent | Cannot Migrate. Purview DLP does not support per-domain collaborator filtering. |
| Collaborators — Entire organization | No Purview equivalent | Cannot Migrate. Use sharing scope conditions as the closest approximation. |
| Collaborators — Groups / Users | Policy scoping by user/group | Partial — achievable via policy-level user/group scoping, not as a per-rule condition. |

### Q: How do MDA governance actions map to Purview DLP actions?

| MDA Governance Action | Purview DLP Action | Status |
|---|---|---|
| Notify file owner | User notifications → Notify who last modified | Full |
| Notify specific users | User notifications → Notify specific people | Full |
| Send alert | Incident reports → Send alert to admins | Full |
| Remove public access | Restrict access → Block everyone except owner | Full |
| Remove external users | Restrict access → Block people outside org | Full |
| Make private | Restrict access → Block everyone except owner | Full |
| Admin quarantine | Admin quarantine | Full |
| Apply / Remove sensitivity label | Separate auto-labeling policy required | Separate policy needed |
| User quarantine | Restrict access + Power Automate flow | Workaround required |
| Trash / delete file | DLP restrict + Power Automate flow | No direct equivalent |
| Remove specific collaborator | SharePoint admin or Power Automate | No direct equivalent |
| Expire shared link | SharePoint sharing policies | No direct equivalent |

### Q: Where do customers see DLP matches and alerts now?

| What you need | Where in Purview |
|---|---|
| Policy matches & alerts | purview.microsoft.com → Data loss prevention → Alerts |
| Full activity history | purview.microsoft.com → Data loss prevention → Activity explorer |
| Simulation mode results | purview.microsoft.com → DLP → Policies → [Policy] → Simulation results |
| Auto-labeling matches | purview.microsoft.com → Information Protection → Auto-labeling → [Policy] → Items to review |

### Q: Should I enforce the Purview policy immediately?
No — always run in Simulation mode first.

| Policy type | Recommended first step | When to enforce |
|---|---|---|
| 1P DLP policy (SPO/ODB/Exchange) | Set to Simulation mode | After 1–2 weeks of validated coverage |
| Auto-labeling policy | Run simulation — review which items would be labeled | After confirming correct items match |
| 3P DLP policy (Box, Dropbox etc.) | Created as Disabled — review before turning on | After confirming connector is healthy and PAYG is configured |

---

## Section 3 — FAQ: 3P Policies (Box, Dropbox, Google Workspace, Salesforce)

### Q: Does Purview support Box / Dropbox / Google Workspace / Salesforce?
Yes — these four apps are supported in Purview DLP via the "Instances" location. AWS, ServiceNow, and Cisco Webex are **not supported** (no Purview equivalent).

Pre-req: MDA app connector must be active. If already connected in MDA, no re-setup needed — connector carries over automatically.

| App | Purview Location | Notes |
|---|---|---|
| Box | Instances | Connector required |
| Dropbox | Instances | Connector required |
| Google Workspace | Instances | Connector required |
| Salesforce | Instances | Connector required |
| AWS / ServiceNow / Webex | — | Not supported |

### Q: What 3P-specific limitations should I know?

| Capability | 3P (Instances) Status |
|---|---|
| Folder-level scoping | Not available — policy applies across the entire connected app |
| Auto-labeling | Roadmap — not yet available for 3P apps |
| TrashFile action | Legacy 3P split — creates policy in Disabled mode |
| MakePrivate / RemoveExternalUsers / RemoveSharedLink | Legacy 3P split — created in Disabled mode |
| BlockAccess | 3P Locations split — created in Disabled mode |

**Policy split note:** If a single MDA policy has both TrashFile AND ContentExtensionMatchesWords, Purview requires two separate policies to cover both.

### Q: Can Microsoft share the roadmap for 3P app support in Purview?
Roadmap details to be confirmed with PG. **Do not promise a date to customers.**

---

## Section 4 — FAQ: Auto-labeling

### Q: Why do I need a separate Purview policy for label application?
In MDA, "Apply sensitivity label" was a governance action inside the file policy. In Purview, label application and DLP enforcement are separate policy types:
- **Purview DLP policy** → detects content and restricts access / alerts
- **Purview auto-labeling policy** (Information Protection) → applies the sensitivity label

When migrating an MDA policy that applied a label, **two Purview policies are needed** — both created in Simulation mode. Navigation: purview.microsoft.com → Information Protection → Auto-labeling

### Q: Does auto-labeling work for 3P apps (Box, Dropbox)?
Not yet. Auto-labeling in Purview Information Protection currently supports first-party workloads only (SharePoint, OneDrive, Exchange). 3P app auto-labeling is on the roadmap but not yet available. **Do not promise a date.**

---

## Section 5 — FAQ: Already in Purview (Post-migration)

### Q: Purview DLP policy is enabled but it's not blocking anything. What's wrong?
This is a Purview CSS question — hand off the case. Before handing off, verify:
- Policy is not still in Simulation mode
- For 3P: MDA connector is healthy (security.microsoft.com → Cloud Apps → Connected apps)
- Policy is scoped to the correct location
If connector is healthy and policy is on but still not enforcing → route to Purview CSS.

### Q: Labels are not being applied by my auto-labeling policy. What should I check?
Check first (MDA CSS scope):
- Is the auto-labeling policy still in Simulation mode? If yes — labels won't apply until turned on.
- Is the sensitivity label published to the relevant users?
- Is the policy scoped to the correct SharePoint site / OneDrive accounts?
If all checks pass and labels still not applying → route to Purview CSS.

### Q: Can MDA and Purview policies run at the same time?
**No — they cannot co-exist.** Running equivalent policies in both MDA and Purview simultaneously creates enforcement conflicts.

Correct sequence:
1. Run Purview policy in simulation for 1–2 weeks — validate coverage
2. Disable the corresponding MDA policy (security.microsoft.com → Cloud Apps → Policies → Edit → Disable)
3. Enable the Purview policy for enforcement
4. Monitor for 1–2 weeks, then delete the MDA policy

**Never enable a Purview policy while the equivalent MDA policy is still active.**

---

## Section 6 — Escalation Routing Guide

### Stage 1 — Pre-migration Assessment

| Scenario | Owner | What to capture |
|---|---|---|
| Purview DLP license not valid | Billing / Tenant admin | Confirm tenant license SKU. Point to admin portal or billing team. |
| Admin missing Purview Compliance or IP Admin role | Customer's IT Admin | Identify missing role. Share role assignment steps. |
| 3P connector not connecting | MDA CSS owns | Capture: connector name, auth error, connector health screenshot. |

### Stage 2 — Setup & Policy Creation
MDA CSS advises only on Purview policy setup — guides on what maps to what, but does not own Purview policy configuration. Once guidance is given, hand off to Purview CSS for hands-on setup support.

| Scenario | Owner | What to capture |
|---|---|---|
| How do I recreate my MDA condition in Purview? | MDA CSS advises → Purview CSS | Use Parity table to identify equivalent. Hand off to Purview CSS for setup. |
| MDA action has no Purview equivalent | MDA CSS advises → Purview CSS | Confirm gap using Parity table. Share documented workaround. Set honest expectation — known gap. |
| Customer wants to migrate label-apply action | MDA CSS advises → Purview CSS | Advise: label apply/remove moves to Information Protection → Auto-labeling policy. |
| Policy created in Purview but not matching expected content | Purview CSS | Hand off immediately. Capture: policy name, SIT/label condition, workload scoped, test file, Activity Explorer output. |

### Stage 3 — Parity Gaps

| Scenario | Owner | What to capture |
|---|---|---|
| Customer identifies a capability with no Purview equivalent | Purview CSS | Confirm against Parity table. Acknowledge gap honestly. Route to Purview CSS for product feedback intake. **Do not promise a delivery date or roadmap timeline.** |
| Customer blocked on a gap and cannot complete migration | Purview CSS | Do not unblock with workarounds that change security posture. Escalate urgency if Dec 31, 2026 deadline is at risk. |

### Stage 4 — Post-Migration & Enforcement

| Scenario | Owner | What to capture |
|---|---|---|
| Purview DLP policy enabled but not enforcing | Purview CSS | Capture: policy name and state, workload, test file, Activity Explorer output, whether MDA policy is still active. |
| DLP alerts not appearing in Purview | Purview CSS | Capture: alert configuration, Activity Explorer output, policy rule conditions. |
| Customer sees duplicate matches or double-enforcement | MDA CSS first-check | Likely co-existence — confirm MDA policy is disabled first. If still persists → route to Purview CSS. |

### Always Capture Before Routing
- Tenant ID + org name
- Migration stage (Assessing / Building / Testing / Rolling out / Decommissioning MDA)
- Issue description — exact capability or behavior, confirmed/reproduced
- Gap vs. break confirmed — checked against Parity table
- MDA policy status — is the MDA policy still active? (co-existence risk)
- Business impact — what protection is at risk
- Steps already taken and results
- Urgency signal — churn risk, exec visibility, Dec 31 2026 deadline pressure

---

## Section 7 — Confirmed Parity Gaps (Cannot Migrate)

The following MDA capabilities have **no current Purview equivalent**:
- AWS / ServiceNow / Cisco Webex app support
- Folder-level scoping (in development, ETA 6/27 — do not promise date)
- Remove specific collaborator
- User quarantine (native)
- Expire shared link
- Auto-labeling for 3P apps
- Owner OU (organizational unit) filtering
- Collaborators — Any from domain filtering
- Collaborators — Entire organization filtering
- ContentIsNotLabeled condition

---

## Section 8 — Glossary

| Term | Definition |
|---|---|
| MDA | Microsoft Defender for Cloud Apps. File policies deprecated Dec 31, 2026. |
| Purview DLP | Microsoft Purview Data Loss Prevention — the target platform for migrated file policies. |
| File Policy | MDA construct that detects files matching filters and applies governance actions. Being replaced by Purview DLP policies. |
| DLP Rule | A rule inside a Purview DLP policy. One policy can contain multiple rules. |
| Sensitive Info Type (SIT) | A classifier in Purview detecting specific sensitive content patterns. Equivalent to MDA preset expressions / DCS. |
| Sensitivity Label | A classification tag applied to files or emails. In Purview, applied via auto-labeling policy — not inside a DLP policy. |
| Auto-Labeling Policy | A separate Purview Information Protection policy that applies sensitivity labels. Required when migrating MDA policies that had "apply label" as an action. |
| Simulation Mode | Purview DLP policy state where policies log matches without enforcing. Run for 1–2 weeks before enforcement. Not available in MDA. |
| Admin Quarantine | A DLP action moving a file to an admin-controlled quarantine folder. Supported in both MDA and Purview. |
| User Quarantine | An MDA action with no direct Purview equivalent — workaround: Restrict access + Power Automate. |
| Instances | A Purview DLP location type for third-party connected apps (Box, Dropbox, Google Workspace, Salesforce). Requires active MDA connector. |
| Parity Gap | A capability present in MDA that has no current equivalent in Purview. |
| CSS | Customer Success / Support — the team this toolkit is built for. |
| Power Automate | Microsoft's workflow automation tool. Used as a workaround for MDA governance actions with no Purview equivalent. |
| Activity Explorer | A Purview tool showing DLP events across workloads. Equivalent to MDA file policy match monitoring. |
| Deprecation deadline | December 31, 2026 — MDA file policies are turned off. Customers must complete migration before this date. |
| Cohort | A customer tier grouping used in migration wave planning. Cohort 1 = highest priority / most complex. |
| S500 | Strategic 500 — Microsoft's top strategic accounts. Require escalated handling — do not handle solo. |

---

## Section 9 — CSS Handshake Protocol

**Purpose:** Define the handshake between MDA CSS and Purview CSS for migration-related customer cases.

### SPOCs
⚠️ SPOCs to be confirmed — names will be added once alignment is complete.

| Team | SPOC / Team Lead | Scope | Contact |
|---|---|---|---|
| MDA CSS | [TBD] | MDA DLP — migration triage, escalation intake | [TBD] |
| Purview CSS | [TBD] | Purview DLP — post-migration support, parity gaps | [TBD] |

### Warm Handoff Steps

| Step | Action | Owner |
|---|---|---|
| 1 | Confirm customer is actively migrating or has completed migration | MDA CSS engineer |
| 2 | Check parity gap — if Cannot Migrate, route to product feedback intake (not Purview CSS) | MDA CSS engineer |
| 3 | If issue is Purview-side (config, policy behaviour, connector), initiate warm handoff to Purview CSS SPOC | MDA CSS engineer |
| 4 | Purview CSS SPOC acknowledges and picks up the case | Purview CSS SPOC |
| 5 | MDA CSS engineer remains CC'd until customer confirms resolution | MDA CSS engineer |

### What NOT to hand off
- Parity gap cases (Cannot Migrate) — route to product feedback intake instead
- Cases still in pre-migration assessment — keep with MDA CSS
- Licensing questions — handle via CSAM or Business Planning contact, not Purview CSS

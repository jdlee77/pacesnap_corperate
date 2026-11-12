# Design notes: the business continuity rework

These are working notes so we can iterate as a team. They cover what changed from the public-safety PACE Snap, why, and the open questions to settle next.

## The lens

A CEO's question is not "which radio do we use?" but **"when something breaks, can the company shift and keep operating?"** That means:

1. The **frontline** can still reach a manager, report that they are safe, and get what they need to keep working.
2. **Leadership** can convene and make decisions even when the company's own systems are down or compromised.
3. The company can reach **employees, customers, suppliers, regulators and insurers** within its RTOs.
4. Companies in a **sector** can coordinate, and **sectors** can reach each other, because no company recovers alone.

## Level mapping

| Public safety | Business continuity | Notes |
| --- | --- | --- |
| Personal | Frontline | Adds "I check in safe" and "I ask for resources". The household stays in, because people do not come to work if they cannot reach their families. |
| Unit | Site | Store, plant, branch, crew. Adds accountability, life-safety and the supplier path. |
| Agency | Company | Priority + RTO per function, activation authority, executive sign-off gate, executive brief. |
| State | Sector | Sector picker (16 CISA sectors) with a suggested function for each. Reach-out point outside the impact area. Sector council review gate. |
| Region | Crosswalk | Participating sectors, liaisons, dependency grid, hub and missing-sector findings, crosswalk-complete gate. |

## Engine changes

- **New paths:** company login (SSO / email tenant), corporate network, conference-bridge provider, mass-notification provider, call-center provider, separate status host, government portal, building systems, landline network.
- **`via` dependency (any-of):** a hosted service that needs *some* phone network to deliver. It fails only when all of those networks are down. It is used for mass notification, the hotline, the bridge and the contact center.
- **Condition `site` (building access):** desk landlines and the PA system fail when the building is closed.
- **Trigger `cyber`:** a triggered rung counts as listening in the ransomware drill if it switches on when a cyber incident is declared.
- **Storage keys** are `pace-biz-v1-*`, so this app does not collide with the public-safety app on the same `jdlee77.github.io` origin. Save files are `app: pace-snap-biz`.

## How the two apps work together

PACE Snap (public safety) and PACE Snap for Business Continuity use the same method and engine but face each other:

- **Public safety app:** "how counties, partners, responders and leadership reach *the agency*." Its shared plan PDF is what an agency hands to partners, including companies.
- **Business app:** "how employees, customers, partners and *responders* reach *the company*." Its public safety contact sheet is what a company hands to agencies.

The handoff today is by file: a PDF, a .vcf and a `pace-snap-responder-1` CSV going one way, and the agency's shared plan PDF going the other. The design choice behind the sheet: it is built from a real PACE ladder in the plan, so it inherits distinct paths, the drills and the monitoring, rather than being a separate list that drifts out of date. On top of that it adds checks that only matter for responders: staffed 24/7, not also published to customers, caller verification, and what the person who answers can authorize.

### Next steps for the bridge

1. **Import in the public safety app.** Done in `jdlee77/pacesnap`: Agency, State and Region plans have a Private-sector partners section that imports `pace-snap-responder-1` CSVs (one file, many files, or a combined directory), flags draft and overdue sheets, and exports a partner directory as PDF or CSV. Partner numbers are kept out of the agency's shared plan.
2. **Reverse export.** A matching export from the public safety app (for example `pace-snap-agency-1`), so a company's Sector or Crosswalk plan can import the county or state EOC and private-sector liaison contacts.
3. **Directory roll-up.** Partly done: the partner directory CSV re-exports in the same schema. Still to do: sort and filter by sector and location, and a sector field on the company sheet.
4. **Verification.** A shared, offline-friendly verification practice (for example, a callback through the published dispatch line) that both apps print on their sheets.

## Open questions for the team

1. **Names.** Is "Crosswalk" the right word for the cross-sector level, or would "Cross-sector" or "Lifeline" read better to industry? Should the product be called "PACE Snap Business" or something else?
2. **Activation.** Is a single "who can activate" field enough, or do CEOs want explicit activation thresholds (for example, "any site down more than 4 hours" or "any confirmed ransomware")?
3. **BIA import.** Should the Company level import functions, priorities and RTOs from a BIA spreadsheet (CSV), so the plan inherits them instead of being typed twice?
4. **Roll-up.** Should a Company plan roll up the CSVs from many Site plans and show which sites are not ready? That is likely the biggest adoption feature for multi-site companies.
5. **Sector packs.** One suggested function per sector is a start. Should each sector get its own asset library and drills (for example, Healthcare: patient-transfer coordination and a tribal/county health officer path; Finance: payment-system status)?
6. **Crosswalk data.** The starting interdependencies are generic. Should we source them from a published reference and let a region save its own as a template?
7. **Standards.** Which mappings matter most to your buyers? We cite topic-level clauses only. Each one needs review by someone who audits against it.
8. **Vendor neutrality.** The asset names stay generic ("out-of-band crisis platform", "mass notification") on purpose. Do we want optional vendor presets?
9. **Offline / PWA.** Bundle jsPDF and the fonts, and make it installable, so the plan opens with no network. This matters more for business users, whose laptops may be the thing that is down.

## Not done yet

- No automated test suite. Checks so far: a headless smoke test that loads the sample at every level, confirms every gate passes, and exports both PDFs and the CSV with no script errors; plus a check that a typical company kit is lost in the hurricane drill and recovers when satellite text and a runner are added.
- The guided setup still covers only the Frontline level.
- The public safety contact sheet is tested by script (readiness, the privacy and 24/7 checks, exports on every level, and adding the ladder to older saved plans), not yet by a 911 center or EOC importing the files.

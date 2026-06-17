# Volunteer User Stories — Gap Analysis Notes

**Org:** `volunteer-npc` (NPC scratch org)  
**Analysis date:** June 16, 2026  
**Source stories:** `Volunteer User Stories.csv`  
**Current setup:** Nonprofit Cloud Volunteer Management + VMQS Quick Start package + Experience Cloud guest signup site

---

## Executive summary

Twelve user stories were evaluated against the current scratch org configuration. **None are fully covered out of the box.** Seven are **partially** supported, four are **missing**, and several overlap or assume a different data model than what is installed.

The most important finding is a **data model mismatch**: several stories (especially US-312) assume **Campaign / Campaign Member** for event volunteer tracking, while the org is configured for **NPC Volunteer Management** (Volunteer Initiative → Job Position → Job Position Shift → Job Position Assignment).

---

## Scorecard

| Status | Count | Stories |
|--------|-------|---------|
| Covered | 0 | — |
| Partial | 8 | US-312, US-320, US-321, US-328, US-340, US-341, US-68, US-69 |
| Missing | 4 | US-313, US-329, US-330, US-331 |

**Overlapping stories:**

- **US-321 ↔ US-341** — both address volunteer skills and availability (program manager vs data analyst lens)
- **US-312 ↔ US-328** — both involve event completion and volunteer participation updates

---

## Data model mismatch

| User stories assume | Current org uses |
|---------------------|------------------|
| **Campaign** = event | **Volunteer Initiative** = program/event grouping |
| **Campaign Member** = volunteer participation | **Job Position Assignment** = volunteer participation |
| Contact fields for Role, Hours, Skills, Availability | Person Account + **PersonCompetency**, **PersonLocationAvailability**, assignment fields |

### Decision required

Before significant build work, choose one path:

1. **Adapt stories to NPC Volunteer Management** (recommended if standardizing on NPC)
2. **Keep Campaign language** and build integration between Campaigns and Volunteer Management
3. **Use Campaigns only** and deprioritize VMQS (would underuse installed package)

---

## What is in place today

### Platform and licenses

- NPC scratch org with Volunteer Management, Fundraising, Case Management, Program Management
- Person Accounts, Communities, OmniStudio
- Admin user assigned: `Volunteer_Management_User`, `Fundraising_User`, `Social_Program_Management_Caseworker`

### VMQS Quick Start (installed)

- Data model: Volunteer Initiative → Job Position → Job Position Shift → Job Position Assignment
- Guest signup flows: `VMQS_Sign_Up_Guest_User`, `VMQS_Sign_Up_Guest_User_Volunteer_Handling`
- Attendance quick actions: Mark Complete, Mark Absent
- Sample rollup flows, location/address flow, competency/examination flows
- Reports: shift capacity, upcoming volunteers, assignments by initiative/position/month, etc.
- Custom fields permission set: `VMQS_Custom_Fields`

### Experience Cloud signup site

- Site name: **Volunteer Sign Up** (status: **Live**)
- URL: `https://fun-business-2052.scratch.my.site.com/volunteersignup/s/`
- Guest sharing rules for Volunteer Initiative, Job Position, Job Position Shift
- `VMQS_Volunteer_Sign_Up_Access` permission set assigned to guest user
- Home page: welcome text + signup flow with URL parameters (`inputId`, `email`, `fName`, `lName`, `volunteerId`)

### Known limitations already identified

1. **No test volunteer data** — no initiatives, positions, or shifts to demo end-to-end
2. **Guest PSL missing** — scratch org lacks "Volunteer Management in Experience Cloud Guest Access" license; guest user has create-only (not edit) on assignments
3. **No volunteer application / interest capture** — VMQS explicitly excludes this
4. **No background check integration** — examinations exist but are manual
5. **Fundraising and Case Management** — licensed but not configured beyond permission sets
6. **Group Membership / Interest Tags** — may not be fully enabled in Setup

---

## Story-by-story summary

### US-312 — Track volunteer participation at events

**Status:** Partial / model mismatch

VMQS supports participation tracking through Job Position Assignments with status, hours, and reporting. However, the story context specifies Campaign/Campaign Member, which is not the configured model. Reframing to Volunteer Initiative and Job Position Assignment is required.

### US-313 — Non-Solicitable contact flag

**Status:** Missing

No Non-Solicitable field, exclusion logic, or compliance reporting exists. Requires custom field and process design tied to fundraising outreach.

### US-320 — Volunteers manage their own shifts

**Status:** Partial

Guest signup works for initial registration. Volunteers cannot view, update, or cancel shifts through a portal. No logged-in Experience Cloud experience. Capacity enforcement on signup is not confirmed.

### US-321 — See volunteer availability and skills

**Status:** Partial

NPC provides PersonCompetency, PersonLocationAvailability, and Search Volunteers — but not as simple Contact fields. Requires configuration, sample data, and possibly enabling Interest Tags and Group Membership.

### US-328 — Auto-update hours after event completion

**Status:** Partial

Manual attendance updates exist (Mark Complete/Absent). No automation triggers when an event/shift is marked Completed. Rollup flows are sample-only per VMQS documentation.

### US-329 — Automated shift reminders

**Status:** Missing

No reminder flows, templates, activity logging, or configurable timing. VMQS README suggests building scheduled flows as a post-install step.

### US-330 — Waitlist management

**Status:** Missing

Shift capacity fields exist (`MaximumAttendeesCount`, `RemainingCapacity`) but no waitlist status, promotion workflow, or notifications.

### US-331 — Volunteer milestone recognition

**Status:** Missing

Hours tracked per assignment only. No cumulative hours on Contact, milestone thresholds, alerts, or recognition reporting.

### US-340 — Employer/affiliation data quality

**Status:** Partial

Contacts to Multiple Accounts and AccountContactRelation support employer-as-Account pattern, but validation, deduplication, and data quality dashboards are not configured.

### US-341 — Track skills and availability for matching

**Status:** Partial

Overlaps with US-321. NPC competency and availability model can support this with configuration; not aligned to Contact-field acceptance criteria as written.

### US-68 — Integrate shadow spreadsheets

**Status:** Partial

Standard Data Import available. No VM-specific import template, mapping documentation, or duplicate rules for volunteer data.

### US-69 — Minimize clicks for attendance updates

**Status:** Partial

Mark Complete/Absent quick actions and inline-edit list views exist in VMQS. Needs validation with sample data and list view configuration in the org.

---

## What VMQS supports well (out of the box)

- Staff creating initiatives, positions, and shifts
- Guest signup via personalized links (no login required)
- Staff marking attendance (Complete / Absent)
- Shift capacity reporting
- Skills-based matching via Search Volunteers (once competencies are configured)
- Location and address on job positions

## What VMQS explicitly does not include

Per accelerator documentation:

- Volunteer applications or interest intake workflows
- Automated email reminders (suggested as custom scheduled flow)
- Waitlist management
- Milestone recognition
- Background check provider integration
- Campaign-based event tracking

---

## Recommended build priority

### Phase 1 — Configuration and reframe (minimal build)

1. Reframe US-312 to Volunteer Initiative / Job Position Assignment terminology
2. Enable Group Membership, Interest Tags, and Person Location Availability
3. Create sample data (initiative, positions, shifts, person accounts with competencies)
4. Validate US-69 — list views and Mark Complete/Absent from shift related lists

### Phase 2 — High-value custom build

| Story | Recommended build |
|-------|-------------------|
| US-313 | `Non_Solicitable__c` on Contact + list views/report filters + fundraising exclusion |
| US-329 | Scheduled flow + email templates for shift reminders |
| US-328 | Record-triggered flow when shift/initiative → Completed → update assignment hours/status |
| US-320 | Extend Experience Cloud (logged-in portal) or flows for cancel/reschedule |

### Phase 3 — Larger features

| Story | Recommended build |
|-------|-------------------|
| US-330 | Waitlist status or object + promotion flow |
| US-331 | Cumulative hours rollup + milestone custom metadata + alert flow |
| US-340 | Affiliation validation, dedup rules, data quality reports |
| US-68 | Import template + mapping guide + duplicate rules |

---

## Suggested next steps

1. **Decide data model direction** — Campaign vs Volunteer Management (highest leverage decision)
2. **Review line-by-line CSV** — `Volunteer User Stories Gap Analysis.csv` in this folder
3. **Pick a pilot story set** — recommended demo path: US-312 (reframed) + US-69 + US-329
4. **Create sample data** — required before meaningful story validation in the org

---

## Related files

| File | Description |
|------|-------------|
| `Volunteer User Stories.csv` | Original user stories and acceptance criteria |
| `Volunteer User Stories Gap Analysis.csv` | Line-by-line gap analysis export |
| `Volunteer User Stories Gap Analysis Notes.md` | This document |

---

## Org reference

| Item | Value |
|------|-------|
| Scratch org alias | `volunteer-npc` |
| Username | `test-8loc2nn6vlnq@example.com` |
| Login URL | `https://fun-business-2052.scratch.my.salesforce.com` |
| Signup site URL | `https://fun-business-2052.scratch.my.site.com/volunteersignup/s/` |
| Network ID | `0DBdq000000SDDdGAO` |
| Guest user ID | `005dq00000MXuGTAA1` |

# Volunteer Management — Salesforce Configuration

Custom configuration built on top of the **Salesforce Nonprofit Cloud Volunteer Management (VMQS)** managed package. This repository contains only the additional metadata layered on top of VMQS — it does not include the base package, Experience Cloud site design, or page layouts.

---

## Prerequisites

The target org must have the following already installed and configured before deploying this package:

- **Nonprofit Cloud (NPC) with Volunteer Management** — managed package installed
- **Person Accounts** enabled
- **Experience Cloud** site provisioned (if using portal flows)
- **DLRS (Declarative Lookup Rollup Summaries)** installed — required to populate `Total_Volunteer_Hours__c` on the Person Account (see [Manual Setup](#manual-setup-steps))
- Salesforce CLI (`sf`) installed locally, or use the GitHub Actions workflow

---

## What This Package Deploys

| Category | Components |
|---|---|
| Custom Fields | 6 fields on Account (availability, hours, milestone), 2 on JobPositionAssignment (role, waitlist) |
| Custom Object | `Volunteer_Program_Interest__c` — junction linking volunteers to programs |
| Custom Metadata Type | `Volunteer_Milestone__mdt` — configurable milestone thresholds (Bronze/Silver/Gold/Platinum) |
| Custom Metadata Records | 4 milestone records (50 / 100 / 250 / 500 hours) |
| Flows | 9 VMQS flows — sign-up, cancel, reschedule, waitlist promotion, hours update, reminders, milestones |
| Permission Sets | `Volunteer_Coordinator` (internal CRUD), `Volunteer_Portal_User` (authenticated portal) |
| Sharing Rules | VolunteerInitiative and JobPositionShift portal sharing (see note below) |

> **Note on sharing rules:** Industries Cloud standard objects do not support sharing rule deployment via Metadata API. The files in `force-app/main/default/sharingRules/` are included for reference only. These must be created manually — see [Manual Setup](#manual-setup-steps).

---

## Deploy via GitHub Actions

The repository includes a manual deploy workflow. No code changes are required to trigger it.

### One-time setup

1. Generate an SFDX auth URL for your target org:
   ```bash
   sf org display --target-org <your-org-alias> --verbose --json | grep sfdxAuthUrl
   ```
2. Add the URL as a repository secret in GitHub:
   - Go to **Settings → Secrets and variables → Actions → New repository secret**
   - Name: `SFDX_AUTH_URL`
   - Value: the full `force://...` URL from step 1

### Running the workflow

1. Go to **Actions → Deploy to Salesforce → Run workflow**
2. Choose options:
   - **Validation only** — runs a check-only deploy to verify metadata is valid (no changes made)
   - **Test level** — `NoTestRun` is fine for metadata-only deployments; use `RunLocalTests` if deploying to production
3. Click **Run workflow**

---

## Deploy via CLI (Local)

```bash
# Authenticate to your target org
sf org login web --alias target-org

# Validate first (recommended)
sf project deploy validate \
  --manifest manifest/package.xml \
  --target-org target-org \
  --api-version 67.0 \
  --wait 30

# Deploy
sf project deploy start \
  --manifest manifest/package.xml \
  --target-org target-org \
  --api-version 67.0 \
  --wait 30
```

---

## Manual Setup Steps

These steps cannot be automated via metadata deployment and must be completed after the package deploy.

### 1. Configure DLRS for Total Volunteer Hours

`Total_Volunteer_Hours__c` on the Person Account is populated by a DLRS rollup. Configure it in Setup → Installed Packages → Declarative Lookup Rollup Summaries:

| Setting | Value |
|---|---|
| Parent Object | Account |
| Rollup Field | `Total_Volunteer_Hours__c` |
| Child Object | JobPositionAssignment |
| Relationship Field | `AssignedAccountId` |
| Aggregate Function | SUM |
| Aggregate Field | `ActualDuration` |
| Filter Criteria | `Status = 'Complete'` |

### 2. Create Portal Sharing Rules

In **Setup → Sharing Settings**, create criteria-based sharing rules for each object:

**VolunteerInitiative:**
- Share with: All Customer Portal Users
- Access level: Read Only
- Criteria: `IsPublished = True` AND `Status = In Progress`

**JobPositionShift:**
- Share with: All Customer Portal Users
- Access level: Read Only
- Criteria: `Status = In Progress`

### 3. Assign Permission Sets

- Assign `Volunteer_Coordinator` to internal staff who manage volunteer programs
- Assign `Volunteer_Portal_User` to Experience Cloud users (portal volunteers)

### 4. Add Coordinator Quick Action for Waitlist Promotion

On the **JobPositionShift** object, create a Quick Action of type **Flow** pointing to `VMQS_Coordinator_Promote_From_Waitlist`. Pass the record Id as the `varShiftId` input variable.

### 5. Add Related Lists to Page Layouts

Add the following related lists to the appropriate page layouts:

| Layout | Related List |
|---|---|
| Person Account | PersonCompetency (Volunteer Skills) |
| Person Account | Volunteer_Program_Interest__c |
| VolunteerInitiative | JobPositionAssignment |
| JobPositionShift | JobPositionAssignment |

---

## Flows Reference

| Flow | Type | Description |
|---|---|---|
| VMQS_Authenticated_Sign_Up | Screen Flow | Portal volunteer signs up for a shift; handles waitlist automatically |
| VMQS_Authenticated_Cancel_Shift | Screen Flow | Portal volunteer cancels an upcoming assignment |
| VMQS_Authenticated_Reschedule_Shift | Screen Flow | Portal volunteer moves from one shift to another |
| VMQS_Coordinator_Promote_From_Waitlist | Screen Flow | Coordinator promotes a waitlisted volunteer to Upcoming |
| VMQS_Auto_Update_Volunteer_Hours | Record-Triggered | Stamps ActualDuration on assignments when Initiative completes |
| VMQS_Send_Shift_Reminders | Scheduled | Daily 8 AM — emails volunteers with shifts the next day |
| VMQS_Volunteer_Milestone_Alert | Record-Triggered | Creates a Task when a volunteer crosses a milestone threshold |
| VMQS_Sign_Up_Guest_User | Screen Flow | Legacy guest signup — retain for reference; SSO orgs can deactivate |
| VMQS_Sign_Up_Guest_User_Volunteer_Handling | Screen Flow | Legacy guest volunteer creation — see above |

---

## Known Gaps / Deferred Items

| Item | Notes |
|---|---|
| US-313: Volunteer conflict detection | Deferred — confirm if field exists in production org |
| US-68: Data migration | Separate workstream — not in scope for this package |
| US-340: Data quality validation rules and dashboards | Follow-on sprint |
| DLRS configuration | Must be completed manually (Step 1 above) |
| Customer Community Plus licensing | Portal end-to-end test requires CCP license — High Volume Customer Portal licenses cannot access Industries Cloud objects |

---

## Repository Structure

```
force-app/main/default/
  customMetadata/        Milestone threshold records
  flows/                 VMQS automation flows
  objects/
    Account/fields/      Volunteer profile custom fields
    JobPositionAssignment/fields/  Role and waitlist fields
    Volunteer_Milestone__mdt/      CMT definition
    Volunteer_Program_Interest__c/ Junction object
  permissionsets/        Coordinator + portal user permission sets
  sharingRules/          Reference only — deploy manually
  experiences/           Experience Cloud site — excluded from package deploy
  networks/              Network config — excluded from package deploy

manifest/
  package.xml            Deployable component manifest (excludes Experience Cloud)

.github/workflows/
  deploy.yml             GitHub Actions deploy workflow

docs/
  Volunteer Management - Test Scripts.docx
  Volunteer Management - Admin Documentation.docx
```

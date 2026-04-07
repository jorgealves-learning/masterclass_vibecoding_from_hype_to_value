# SPEC.md — Contract & Project Analysis Tracking System

> **What is this document?**  
> This specification describes **what the system should do** from a business perspective. It explains the goals, features, and rules in plain language for anyone to understand.
>
> **Who is this for?**  
> Product owners, stakeholders, and anyone who needs to understand what this application does without reading code.

---

## 1. What Problem Does This Solve?

Currently, when the company needs to analyze a new contract or project, multiple departments are involved: Legal, Data Protection, Tax, Cybersecurity, and Business. Each department reviews the request and provides their assessment.

**The Problem**: Today, this process happens through emails and spreadsheets. Information gets lost, it's hard to know the current status, and there's no single place to see all ongoing requests.

**The Solution**: A web application that serves as a single source of truth for all contract and project analysis requests. Everyone can see what's being analyzed, who is responsible, what issues exist, and what the final decision is.

---

## 2. Who Will Use This System?

- **Internal Staff**: Non-technical employees who create requests and track their progress
- **Department Reviewers**: Staff from Legal, Data Protection, Tax, Cybersecurity, and Business who analyze requests and record their assessments
- **Managers**: People who need visibility into all ongoing analyses and their statuses

The interface will be in **Portuguese (Portugal)**.

---

## 3. Core Concepts

### 3.1 What is a Request?

A **Request** is the central item in the system. Someone creates a request when they need a contract or project to be analyzed by the various departments.

Each request contains:
- **Basic Information**: Title, description, type (contract or project), who is requesting it, who owns it internally
- **Supplier**: The external company involved (if any)
- **Criticality**: How urgent or important this analysis is (Low, Medium, High)
- **Geographic Scope**: Which countries or regions are affected

### 3.2 What is an Area Analysis?

Each request must be reviewed by five departments (called "Areas"):
1. **Legal** — Reviews legal terms and compliance
2. **Data Protection** — Assesses privacy and GDPR implications
3. **Tax** — Evaluates fiscal impact
4. **Cybersecurity** — Analyzes security risks
5. **Business** — Confirms business value and alignment

Each area provides:
- **Status**: Not Started, In Progress, or Completed
- **Responsible Person**: Who is handling the review
- **Comments**: Their observations and concerns
- **Decision**: Their final position — No Objections, With Reservations, or Blocking

### 3.3 What is an Open Issue?

During analysis, reviewers may identify problems that need resolution before proceeding. These are tracked as **Open Issues**.

Each issue has:
- A description of the problem
- Which area raised it
- Priority level (Low, Medium, High, Critical)
- Due date (optional)
- Current status (Open, In Progress, Resolved)

### 3.4 What is a Final Decision?

Once all areas have completed their analysis, a final decision is recorded:
- **Approved** — Proceed with no concerns
- **Approved with Conditions** — Proceed but certain conditions must be met
- **Rejected** — Do not proceed
- **Pending** — Not yet decided

The decision includes who approved it, when, and a summary of the analysis.

### 3.5 What is the Timeline?

Every significant action is recorded in a timeline, creating an audit trail. This includes when the request was created, when statuses changed, when areas were updated, when issues were added or resolved, and when decisions were made.

---

## 4. Features

### 4.1 Dashboard

The main screen shows all active requests at a glance.

**Users can:**
- See all requests in a list or card format
- Filter requests by status, type, or criticality
- Search by title, supplier, or owner name
- Sort by creation date
- See progress at a glance (how many areas have completed their review)
- See warning indicators for requests with open issues
- Click on any request to see its full details
- Create a new request

#### Examples: Dashboard Usage

| Scenario | What Happens | Success or Failure |
|----------|--------------|-------------------|
| Maria searches for "Acme" to find all contracts with that supplier | The dashboard filters to show only requests containing "Acme" in the title or supplier field | ✅ Success |
| João filters by "Has Issues" status to see requests needing attention | Only requests with open issues appear, clearly showing which need action | ✅ Success |
| Ana searches for "contract" but spells it "contrat" | No results appear because exact matching fails | ❌ Failure — system should show "no results" message and suggest checking spelling |
| Pedro clicks a request card expecting details but nothing happens | The card doesn't respond to clicks due to a loading error | ❌ Failure — system should always navigate to detail view or show an error message |

---

### 4.2 Creating a New Request

When someone needs to start an analysis, they fill out a form with:
- Title and description of what needs to be analyzed
- Whether it's a contract or a project
- Which internal area is requesting this
- Who internally owns this request
- The supplier name (if applicable)
- How critical this is
- Which countries or regions are involved
- Any initial notes or context

After creation, the request appears on the dashboard with status "New" and all five area analyses ready for departments to begin their reviews.

#### Examples: Creating Requests

| Scenario | What Happens | Success or Failure |
|----------|--------------|-------------------|
| Sofia fills all required fields and clicks "Create" | Request is saved, she sees a confirmation message, and is redirected to the new request's detail page | ✅ Success |
| Carlos submits a form with title "IT" (too short) | Form shows clear error: "Title must be at least 5 characters" next to the title field | ✅ Success |
| Luísa fills the form but her internet disconnects when she clicks "Create" | She sees an error message explaining the save failed, and her entered data is preserved so she can retry | ✅ Success |
| Miguel submits a valid form but sees no confirmation | He's unsure if it saved, creates it again, and now has duplicates | ❌ Failure — system must always confirm success or failure |
| Rita leaves description empty and submits | Form submits anyway with blank description, causing confusion later | ❌ Failure — required fields must be enforced |

---

### 4.3 Viewing a Request

The detail view shows everything about a single request, organized into sections:

**General Information**
- All the basic details submitted during creation
- Current status with ability to change it
- Overall progress indicator showing how many areas have completed review
- Recent activity from the timeline

**Area Analyses**
- Five cards, one for each reviewing department
- Each shows the responsible person, their comments, and their decision
- Reviewers can update their area's information and record their decision
- Visual indicators show which areas are blocking or have reservations

**Open Issues**
- List of all problems identified during review
- Ability to add new issues
- Ability to mark issues as resolved
- Filtering by status or responsible area

**Documents**
- List of related documents (contracts, proposals, policies)
- Ability to add document references with names and links

**Final Decision**
- Summary of the current state
- Ability to record the final decision with approver name and comments
- Option to auto-generate a summary from all the area analyses

#### Examples: Viewing and Updating Requests

| Scenario | What Happens | Success or Failure |
|----------|--------------|-------------------|
| Teresa opens a request to check its status | Page loads quickly showing all information organized in clear tabs | ✅ Success |
| Bruno from Legal updates his area's comments and clicks "Save" | His changes are saved, he sees a confirmation, and the timeline shows "Legal area updated by Bruno" | ✅ Success |
| Carla changes request status from "New" to "In Analysis" | Status updates, timeline records the change, and the dashboard reflects the new status | ✅ Success |
| Manuel edits comments but accidentally closes the browser tab | He loses all unsaved work with no warning | ❌ Failure — system should warn before leaving with unsaved changes |
| Paula saves her analysis but the page shows old data | She refreshes and sees conflicting information, unsure what was saved | ❌ Failure — display must update immediately after save |

---

### 4.4 Managing Open Issues

#### Examples: Issue Tracking

| Scenario | What Happens | Success or Failure |
|----------|--------------|-------------------|
| Data Protection identifies a GDPR concern and adds an issue | Issue appears in the list with correct priority, responsible area, and open status | ✅ Success |
| Legal resolves a contractual issue and marks it resolved | Issue status changes to "Resolved", timeline records who resolved it and when | ✅ Success |
| Cybersecurity adds a critical issue, and the request card on dashboard shows warning | Dashboard displays a warning indicator (e.g., "3 open issues") on the request card | ✅ Success |
| André tries to delete an issue but it disappears with no confirmation | He accidentally deleted the wrong issue and cannot recover it | ❌ Failure — deletion must require confirmation |
| Beatriz marks an issue resolved but it still shows as open | The status didn't save, causing confusion about actual state | ❌ Failure — status changes must persist immediately |

---

### 4.5 Recording Final Decisions

#### Examples: Final Decision Process

| Scenario | What Happens | Success or Failure |
|----------|--------------|-------------------|
| All 5 areas complete with "No Objections", manager records "Approved" | Decision saves with approver name, date, and optional summary. Request status updates. | ✅ Success |
| Manager clicks "Generate Summary" | System creates text summarizing: request details, each area's decision, and issue count. Manager can edit before saving. | ✅ Success |
| Manager tries to approve while Legal has a "Blocking" decision | System shows warning: "Cannot approve — Legal has blocking concerns. Please resolve before approving." | ✅ Success |
| Manager approves a request that has 3 unresolved open issues | System allows approval without any warning about pending issues | ❌ Failure — should warn about open issues before final decision |
| Decision is saved but no one can tell who approved it or when | Missing audit information makes the decision untraceable | ❌ Failure — approver and date are required |

---

### 4.6 Warning for Blocking Issues

If any department marks their decision as "Blocking," the system prominently displays a warning. This ensures no request proceeds to approval while critical concerns remain unaddressed.

#### Examples: Blocking Behavior

| Scenario | What Happens | Success or Failure |
|----------|--------------|-------------------|
| Tax marks decision as "Blocking" due to fiscal risk | A prominent red banner appears at top of request: "⛔ BLOCKING: Tax has registered a blocking concern" | ✅ Success |
| User tries to change status to "Approved" while blocking exists | System prevents change and shows: "Cannot approve while blocking concerns exist" | ✅ Success |
| Blocking is resolved, Tax changes to "No Objections" | Warning banner disappears, approval is now possible | ✅ Success |
| Two areas have blocking decisions but only one warning shows | User misses the second blocking concern | ❌ Failure — all blocking areas must be listed |
| Blocking exists but user can still set status to "Approved" | Critical concern is overlooked, contract proceeds with unresolved risk | ❌ Failure — this is the primary scenario the system must prevent |

---

## 5. Business Rules

### 5.1 Progress Calculation

Progress is calculated as the percentage of areas that have completed their review. If 2 out of 5 areas are done, progress is 40%.

#### Examples: Progress Display

| Scenario | Expected Progress | Success or Failure |
|----------|-------------------|-------------------|
| New request, no areas have started | 0% (0/5 areas completed) | ✅ Success |
| Legal and Tax completed, others in progress | 40% (2/5 areas completed) | ✅ Success |
| All 5 areas marked as completed | 100% (5/5 areas completed) | ✅ Success |
| 3 areas completed but progress shows 50% | Incorrect calculation misleads users | ❌ Failure |

---

### 5.2 Request Status

A request moves through these statuses:
- **New** — Just created, no analysis started
- **In Analysis** — At least one area is reviewing
- **Awaiting Information** — Paused, waiting for more details
- **Has Issues** — Open issues exist that need resolution
- **Approved** — All areas agree, final decision is positive
- **Approved with Conditions** — Approved but with requirements
- **Rejected** — Final decision is negative
- **Closed** — No longer active (archived)

The user changes status manually. The system provides suggestions (for example, suggesting "Approved" when all areas are complete and have no objections) but never forces automatic changes.

#### Examples: Status Management

| Scenario | What Happens | Success or Failure |
|----------|--------------|-------------------|
| All areas complete with no objections, user sees suggestion | System shows hint: "All areas approved. Consider changing status to Approved." | ✅ Success |
| User manually sets status to "Awaiting Information" | Status changes, timeline records the change | ✅ Success |
| Request has open issues but user sets "Approved" | System allows it but shows warning: "Note: 2 open issues remain unresolved" | ✅ Success |
| Status changes automatically without user action | User loses control of the workflow | ❌ Failure — changes must always be manual |
| Status dropdown shows options that don't make sense (e.g., "Approved" for new request) | All options shown regardless of context, causing confusion | ❌ Failure — could show contextual suggestions |

---

### 5.3 Blocking Prevention

If any area has recorded a "Blocking" decision:
- A prominent warning appears on the request
- The system warns if someone tries to set status to "Approved"
- This ensures critical concerns cannot be accidentally overlooked

### 5.4 Data Retention

Records are never permanently deleted. When someone "deletes" a request or issue, it is marked as inactive but retained for audit purposes. This preserves the historical record of all decisions.

#### Examples: Data Retention

| Scenario | What Happens | Success or Failure |
|----------|--------------|-------------------|
| User deletes a request they created by mistake | Request disappears from dashboard but data is preserved in database | ✅ Success |
| Manager needs to review a decision from a deleted request | Administrators can access archived records for audit | ✅ Success |
| User deletes a request and all its history is permanently lost | Audit trail is destroyed, compliance risk created | ❌ Failure |

---

## 6. User Experience Expectations

### 6.1 Performance
- Pages should load quickly
- The system should work well on desktop computers and tablets
- Mobile phones should be functional but are not the primary focus

### 6.2 Feedback
- Users should see confirmation messages when they save or update information
- Users should be asked to confirm before deleting anything
- Forms should clearly indicate what's required and show helpful error messages
- Loading indicators should appear during any waiting periods

### 6.3 Navigation
- Users should always know where they are (breadcrumb trail)
- Users should be able to go back easily
- Users should be warned if they try to leave a page with unsaved changes

#### Examples: User Experience

| Scenario | What Happens | Success or Failure |
|----------|--------------|-------------------|
| User saves a form and sees "Changes saved successfully" | Clear feedback confirms action completed | ✅ Success |
| User clicks delete and sees "Are you sure? This cannot be undone." | Confirmation prevents accidental deletion | ✅ Success |
| Page takes 5 seconds to load with no indicator | User thinks system is frozen, clicks multiple times | ❌ Failure — should show loading spinner |
| User on detail page can't tell which request they're viewing | Missing breadcrumb or title causes confusion | ❌ Failure — context must always be clear |

---

## 7. What This System Does NOT Do (MVP Limitations)

To keep the first version simple and focused, the following are explicitly excluded:

- **No Login System** — Anyone with access can use the system
- **No Email Notifications** — Users must check the system manually
- **No File Uploads** — Documents are referenced by link only
- **No Exports** — No PDF or Excel downloads
- **No External Integrations** — No connection to other company systems
- **No Automated Workflows** — Humans control all status changes
- **No Analytics** — No reports or metrics dashboards
- **No Offline Access** — Requires internet connection
- **No Multi-Language** — Portuguese only

---

## 8. Future Possibilities

These features may be added in later versions:

**Collaboration Improvements**
- Login with company credentials
- Email notifications when action is needed
- Ability to mention colleagues in comments
- Task assignments with reminders

**Automation**
- Automatic status updates based on area completion
- Integration with document management systems
- Ability to export summaries as PDF
- API for connecting other systems

**Analytics**
- Dashboard showing average analysis times
- Approval rate statistics
- SLA tracking
- Historical reporting

---

## 9. Sample Data for Testing

The system should include sample requests to demonstrate functionality:

| Request | Type | Status | Criticality |
|---------|------|--------|-------------|
| IT Outsourcing Contract — Acme Corp | Contract | In Analysis | High |
| CRM Implementation — Salesforce | Project | Approved with Conditions | Medium |
| NDA — XYZ Corporation | Contract | Approved | Low |
| Analytics Platform — DataCorp | Project | Has Issues | High |
| Maintenance Contract — SoftWare+ | Contract | New | Medium |

Each sample should have realistic data across all areas, some open issues, attached documents, and timeline events showing activity history.

---

## 10. Success Criteria

The system is successful when:

1. **Visibility**: Anyone can quickly see all ongoing analyses and their current state
2. **Accountability**: It's clear who is responsible for each review and what their assessment is
3. **Traceability**: There's a complete history of all actions and decisions
4. **Blocking Prevention**: Critical concerns cannot be accidentally ignored
5. **Simplicity**: Non-technical staff can use it without training

### Overall System Examples

| Scenario | Expected Outcome | Success or Failure |
|----------|------------------|-------------------|
| New employee uses system for first time without training | They can create a request and understand the dashboard within 5 minutes | ✅ Success |
| Manager asks "What's the status of the Acme contract?" | Answer is found in under 30 seconds via search or filter | ✅ Success |
| Auditor requests history of a decision made 6 months ago | Complete timeline with all changes and responsible parties is available | ✅ Success |
| Legal marked a contract as blocking but it was approved anyway | System failed to prevent the most critical error it was designed to stop | ❌ Critical Failure |
| User spends 10 minutes trying to figure out how to add an issue | Interface is not intuitive, training would be required | ❌ Failure |
| Data is lost after a browser refresh | System failed to persist data correctly | ❌ Critical Failure |

---

*Document Version: 3.1 — Business specification with task examples for stakeholder review*

# Feature Specification: Request Detail View

> **Feature ID**: `request-detail`  
> **Priority**: P2 — Request Management  
> **Parent Document**: [SPEC.md](../../SPEC.md)

---

## Description

The Request Detail View is the central workspace for managing a single analysis request. It provides comprehensive access to all information about a request, organized into logical sections (tabs). Users come here to review request information, track analysis progress across departments, manage issues, attach documents, and record final decisions.

This view is where the actual work happens — department reviewers update their analyses, issues are tracked and resolved, and final decisions are recorded. It serves as the single source of truth for everything related to a specific contract or project analysis.

### Business Context

When multiple departments need to analyze the same contract or project, coordination is challenging. The Request Detail View solves this by:
- Showing all department assessments in one place
- Tracking open issues that need resolution
- Maintaining a complete history of all actions
- Providing a clear path to final decision

### User Stories

- As a **department reviewer**, I want to see the full context of a request so I can provide an informed assessment
- As a **request owner**, I want to monitor progress across all areas so I can follow up where needed
- As a **manager**, I want to see the complete picture of a request including any blocking concerns
- As an **auditor**, I want to review the complete history of decisions and changes

### Interface Language

All visible text, labels, and messages must be in **Portuguese (Portugal)**.

---

## Page Structure

### Header Section

The top of the page displays:
- **Breadcrumb**: "Dashboard > [Request Title]"
- **Request Title**: Prominently displayed
- **Status Badge**: Current status with appropriate color
- **Criticality Badge**: Low/Medium/High indicator
- **Progress Bar**: Visual representation of areas completed (e.g., "3/5 áreas concluídas")

### Blocking Alert Banner

If any area has a "Blocking" decision, display a prominent alert:
- Position: Immediately below header, full width
- Style: Red/danger background, white text
- Content: "⛔ BLOQUEIO: [Area Name] registou uma preocupação bloqueante"
- Behavior: Shows all blocking areas if multiple exist

### Tab Navigation

The content is organized into five tabs:

| Tab | Portuguese Label | Purpose |
|-----|------------------|---------|
| General | Informação Geral | Basic request details and status |
| Areas | Análises por Área | Five department review cards |
| Issues | Pontos em Aberto | Issue tracking and management |
| Documents | Documentos | Attached document references |
| Decision | Decisão Final | Final decision recording |

**Tab behavior:**
- Active tab is visually highlighted
- Tab shows badge count where applicable (e.g., "Pontos em Aberto (3)")
- Tab content loads without full page refresh
- URL may update to include tab identifier (e.g., `/pedidos/[id]?tab=issues`)

---

## Functional Requirements

### FR-1: Page Loading

- Page must load request data by ID from URL parameter
- If request ID doesn't exist, show 404 error with link to dashboard
- If request is soft-deleted, show appropriate message
- Loading state must be shown while data is being fetched

### FR-2: Header Information

**Display requirements:**
- Request title (full text, may wrap to multiple lines)
- Type badge: "Contrato" or "Projeto"
- Status badge with color coding per design system
- Criticality badge with color coding
- Creation date formatted as "Criado em DD/MM/YYYY"
- Internal owner name

**Progress bar:**
- Shows fraction: "X/5 áreas concluídas"
- Visual bar fills proportionally (0%, 20%, 40%, 60%, 80%, 100%)
- Updates automatically when area statuses change

### FR-3: Status Change

Users must be able to change the request status:

**Available statuses:**
- Novo (New)
- Em Análise (In Analysis)
- Aguarda Informação (Awaiting Information)
- Com Pontos em Aberto (Has Issues)
- Aprovado (Approved)
- Aprovado com Condições (Approved with Conditions)
- Rejeitado (Rejected)
- Fechado (Closed)

**Status change behavior:**
- Dropdown or select input showing all status options
- On change: update status, record timestamp, add timeline event
- If changing to "Approved" while blocking exists: show warning dialog
- If changing to "Approved" while open issues exist: show info notification
- Status change must persist immediately

### FR-4: Blocking Alert Logic

**Display conditions:**
- Show alert if ANY area has `decision === 'blocking'`
- List ALL areas with blocking decisions, not just the first

**Alert content:**
- Area name(s) with blocking decision
- Link to jump to that area's card in the Areas tab

**Behavior:**
- Cannot dismiss the alert while blocking exists
- Alert disappears automatically when all blocking decisions are changed

### FR-5: Tab Content Loading

- Each tab's content loads on first access
- Tab content is preserved when switching between tabs
- Unsaved changes in one tab are preserved when switching tabs
- If tab has unsaved changes, show indicator on tab label

---

## Acceptance Criteria

### AC-1: Page Load
- [ ] Page loads correctly when navigating from dashboard
- [ ] Page shows loading indicator while fetching data
- [ ] Page displays all header information correctly
- [ ] Invalid request ID shows 404 message
- [ ] Deleted request shows appropriate message

### AC-2: Header Display
- [ ] Title displays fully (no truncation)
- [ ] Type badge shows correct value
- [ ] Status badge shows correct value and color
- [ ] Criticality badge shows correct value and color
- [ ] Progress bar shows correct fraction and fill percentage
- [ ] Creation date is formatted correctly

### AC-3: Status Change
- [ ] Status dropdown shows all available statuses
- [ ] Selecting new status updates immediately
- [ ] Status change creates timeline event
- [ ] Warning appears when approving with blocking areas
- [ ] Notification appears when approving with open issues

### AC-4: Blocking Alert
- [ ] Alert appears when any area has blocking decision
- [ ] Alert lists all blocking areas (not just one)
- [ ] Alert links to relevant area cards
- [ ] Alert disappears when blocking is resolved

### AC-5: Tab Navigation
- [ ] All five tabs are visible and labeled correctly
- [ ] Clicking tab shows corresponding content
- [ ] Active tab is visually distinguished
- [ ] Tab badges show correct counts
- [ ] Tab switching preserves unsaved changes

### AC-6: Breadcrumb Navigation
- [ ] Breadcrumb shows "Dashboard > [Request Title]"
- [ ] Clicking "Dashboard" navigates back to dashboard
- [ ] Current page (request title) is not clickable

---

## Success and Failure Examples

### Success Examples ✅

| Scenario | User Action | Expected Result |
|----------|-------------|-----------------|
| **View request details** | Teresa clicks a request card on dashboard | Detail page loads within 1 second showing all request information, defaulting to General tab |
| **Check progress** | Manuel views the header | Progress bar shows "3/5 áreas concluídas" with 60% visual fill |
| **See blocking alert** | User opens request where Tax has blocking decision | Red banner appears: "⛔ BLOQUEIO: Tax registou uma preocupação bloqueante" |
| **Change status** | User changes status from "New" to "In Analysis" | Status badge updates immediately, timeline shows "Status alterado para Em Análise" |
| **Navigate tabs** | User clicks "Pontos em Aberto" tab | Tab content shows issues list, tab label shows count badge "(3)" |
| **Return to dashboard** | User clicks "Dashboard" in breadcrumb | Browser navigates to dashboard, preserving any active filters |
| **Multiple blocking areas** | Legal and Cybersecurity both have blocking decisions | Alert shows both: "⛔ BLOQUEIO: Legal, Cybersecurity registaram preocupações bloqueantes" |
| **Attempt premature approval** | User tries to set status to "Approved" while Tax is blocking | Dialog appears: "Não é possível aprovar. Tax tem preocupação bloqueante." with Cancel/View Area options |

### Failure Examples ❌

| Scenario | User Action | Incorrect Result | Why It's Wrong |
|----------|-------------|------------------|----------------|
| **Slow page load** | User clicks request card | Page takes 5+ seconds with no loading indicator | Users think it's broken; must show loading state |
| **Missing blocking alert** | User views request with blocking area | No alert banner appears | Critical concern could be overlooked |
| **Partial blocking display** | Two areas are blocking | Alert only shows one area | User might resolve one and think they're done |
| **Silent status change** | User changes status | Status updates but no confirmation | User unsure if change was saved |
| **Lost unsaved changes** | User edits area, switches tabs | Changes are lost | Must preserve unsaved changes across tab switches |
| **Broken breadcrumb** | User clicks Dashboard link | Nothing happens | Navigation must always work |
| **Approval allowed with blocking** | User sets status to Approved while blocking exists | Status changes without warning | This defeats the primary safety feature |
| **Timeline not updated** | User changes status | Timeline shows no new event | Audit trail is broken |
| **Progress wrong** | 4 areas completed | Progress shows "2/5" | Incorrect calculation misleads users |
| **Tab badge missing** | 5 open issues exist | Issues tab shows no count badge | User doesn't know issues need attention |

---

## Edge Cases and Boundary Conditions

### Request States

| State | Expected Behavior |
|-------|------------------|
| Brand new request | All areas show "Not Started", progress is 0/5 |
| All areas completed | Progress shows 100%, status suggestions may appear |
| Request with no issues | Issues tab shows empty state: "Sem pontos em aberto" |
| Request with no documents | Documents tab shows empty state with add button |
| Fully approved request | All information read-only except status |
| Closed request | All information read-only, show archive notice |

### Navigation Scenarios

| Scenario | Expected Behavior |
|----------|------------------|
| Direct URL access | Page loads correctly from URL (e.g., `/pedidos/abc123`) |
| Invalid ID in URL | Show 404 page with "Request not found" message |
| Browser back button | Returns to previous page (usually dashboard) |
| Refresh on tab | Page reloads on the same tab if URL includes tab param |
| Deep link to tab | `/pedidos/[id]?tab=issues` opens directly to Issues tab |

### Concurrent Editing

| Scenario | Expected Behavior |
|----------|------------------|
| Two users view same request | Both see current data |
| User A edits while User B views | User B sees old data until refresh (MVP limitation) |
| User edits, network fails | Error shown, data preserved for retry |

---

## User Interface Guidelines

### Layout Structure

```
┌─────────────────────────────────────────────────────────────┐
│  Dashboard > IT Outsourcing Contract — Acme Corp           │
├─────────────────────────────────────────────────────────────┤
│  IT Outsourcing Contract — Acme Corp                       │
│  [Contrato] [Em Análise] [Alta]                            │
│  Responsável: João Silva    Criado: 15/01/2024            │
│  ████████████░░░░░░░░░░░░ 60% (3/5 áreas concluídas)       │
├─────────────────────────────────────────────────────────────┤
│  ⛔ BLOQUEIO: Tax registou uma preocupação bloqueante      │
├─────────────────────────────────────────────────────────────┤
│  [Info Geral] [Análises] [Pontos (2)] [Docs] [Decisão]     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│                      Tab Content Area                       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Responsive Behavior

**Desktop (≥1024px):**
- Full header with all badges on one line
- Tabs displayed horizontally
- Tab content uses full width

**Tablet (768-1023px):**
- Header may wrap to two lines
- Tabs remain horizontal but may scroll
- Tab content uses full width

**Mobile (<768px):**
- Header stacks vertically
- Tabs may become dropdown or scrollable
- Tab content uses full width

---

## Definition of Done

This feature is complete when:

- [ ] **Page routing**: `/pedidos/[id]` route works correctly
- [ ] **Data loading**: Request data loads and displays correctly
- [ ] **Header display**: All header elements render with correct data
- [ ] **Progress bar**: Shows correct fraction and visual percentage
- [ ] **Status change**: Works with confirmation and creates timeline event
- [ ] **Blocking alert**: Appears correctly when blocking decisions exist
- [ ] **Tab navigation**: All five tabs navigate correctly
- [ ] **Tab badges**: Show correct counts for issues and documents
- [ ] **Breadcrumb**: Navigation back to dashboard works
- [ ] **Error states**: 404 and deleted request states handled
- [ ] **Loading state**: Spinner shown while fetching data
- [ ] **Portuguese text**: All labels and messages in Portuguese
- [ ] **Responsive**: Works on desktop and tablet
- [ ] **Accessible**: Keyboard navigation works for tabs
- [ ] **Tests**: Unit tests for status change logic and blocking detection
- [ ] **Code review**: Approved by at least one team member

---

## Related Features

- **[Dashboard](../dashboard/SPEC.md)**: Navigation source and return destination
- **[Area Analysis](../area-analysis/SPEC.md)**: Content for Areas tab
- **[Open Issues](../open-issues/SPEC.md)**: Content for Issues tab
- **[Documents](../documents/SPEC.md)**: Content for Documents tab
- **[Final Decision](../final-decision/SPEC.md)**: Content for Decision tab
- **[Timeline](../timeline/SPEC.md)**: Activity history shown in General tab
- **[Blocking Alerts](../blocking-alerts/SPEC.md)**: Alert banner logic

---

## Technical Notes

> For developer reference. Non-technical stakeholders may skip this section.

- Use Next.js dynamic route: `/src/app/(features)/pedidos/[id]/page.tsx`
- Page should be a Server Component fetching initial data
- Tab content may use Client Components for interactivity
- Consider using URL search params for tab state (`?tab=issues`)
- Blocking detection logic: `request.areas.some(a => a.decision === 'blocking')`
- Status change should call service layer and handle optimistic updates
- Timeline events should be created server-side for consistency

---

*Specification Version: 1.0*

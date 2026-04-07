# Feature Specification: Open Issues

> **Feature ID**: `open-issues`  
> **Priority**: P3 — Issues & Decisions  
> **Parent Document**: [SPEC.md](../../SPEC.md)

---

## Description

The Open Issues feature provides a centralized system for tracking problems, concerns, and action items identified during the analysis of a contract or project request. When department reviewers identify issues that need resolution before proceeding, they log them here. Issues can be tracked through their lifecycle from identification to resolution.

This feature ensures that no concern gets lost in the analysis process. It creates visibility around what problems exist, who is responsible for addressing them, and what has been resolved.

### Business Context

During contract and project analysis, reviewers frequently identify problems that need attention:
- A contract clause may need renegotiation
- Additional documentation may be required
- A security vulnerability needs assessment
- Tax implications require clarification

Before this system, these concerns would be scattered across emails and meeting notes. The Open Issues feature provides:
- A single list of all identified problems
- Clear ownership and priority
- Status tracking from open to resolved
- Historical record for audit purposes

### User Stories

- As a **department reviewer**, I want to log issues I identify so that they are tracked and not forgotten
- As a **request owner**, I want to see all open issues so I can coordinate resolution efforts
- As a **manager**, I want to understand what's blocking approval so I can help remove obstacles
- As an **auditor**, I want to see the history of issues raised and how they were resolved

### Interface Language

All visible text, labels, and messages must be in **Portuguese (Portugal)**.

---

## Data Model Concepts

### Issue Properties

| Property | Portuguese Label | Description |
|----------|-----------------|-------------|
| Title | Título | Brief summary of the issue (required) |
| Description | Descrição | Detailed explanation of the problem (optional) |
| Responsible Area | Área Responsável | Which department should address this issue (required) |
| Priority | Prioridade | How urgent is this issue (required) |
| Due Date | Data Limite | When should this be resolved (optional) |
| Status | Estado | Current state of the issue (required) |
| Notes | Notas | Additional context or updates (optional) |
| Created At | Criado em | When the issue was logged (automatic) |
| Updated At | Atualizado em | When the issue was last modified (automatic) |

### Priority Levels

| Priority ID | Portuguese Label | Color | When to Use |
|-------------|-----------------|-------|-------------|
| `critical` | Crítica | Red-600 | Blocks all progress, needs immediate attention |
| `high` | Alta | Red-500 | Significant impact, needs urgent attention |
| `medium` | Média | Amber-500 | Important but not urgent |
| `low` | Baixa | Green-500 | Minor concern, address when convenient |

### Status Values

| Status ID | Portuguese Label | Meaning |
|-----------|-----------------|---------|
| `open` | Aberto | Issue identified, not yet addressed |
| `in_progress` | Em Tratamento | Someone is actively working on resolution |
| `resolved` | Resolvido | Issue has been addressed and closed |

---

## Detailed Requirements

### FR-1: Issue List Display

The Issues tab must display all issues associated with the request.

**List requirements:**
- Show all issues in a table or card format
- Display key information at a glance: title, area, priority, due date, status
- Sort by priority (Critical → High → Medium → Low) by default
- Visual differentiation between open, in progress, and resolved issues
- Show issue count in tab badge: "Pontos em Aberto (X)"

**Empty state:**
- Message: "Não existem pontos em aberto registados."
- Prominent "Add Issue" button

### FR-2: Filtering and Search

Users must be able to filter and find specific issues.

**Filter options:**
- By status: Open, In Progress, Resolved
- By responsible area: Legal, Data Protection, Tax, Cybersecurity, Business

**Search:**
- Search by title
- Search results update as user types

### FR-3: Create New Issue

Users must be able to add new issues to a request.

**Trigger:** "Adicionar Ponto" (Add Issue) button

**Form appears in:** Modal dialog

**Form fields:**

| Field | Type | Required | Constraints |
|-------|------|----------|-------------|
| Title | Text input | Yes | Min 5 chars, Max 200 chars |
| Description | Textarea | No | Max 1000 chars |
| Responsible Area | Select dropdown | Yes | One of the 5 areas |
| Priority | Select or radio | Yes | One of 4 priority levels |
| Due Date | Date picker | No | Must be today or future |
| Notes | Textarea | No | Max 500 chars |

**On submission:**
1. Validate all fields
2. Create issue with status "Open"
3. Set timestamps (created, updated)
4. Add timeline event: "Issue added: [title]"
5. Update issue count badge
6. Close modal and show confirmation
7. Update dashboard card to reflect new issue count

### FR-4: Edit Issue

Users must be able to modify existing issues.

**Trigger:** Click on issue row/card or explicit "Edit" button

**Form:** Same as create, pre-populated with existing values

**Constraints:**
- Cannot edit resolved issues (show message if attempted)
- All original validations apply

**On save:**
1. Validate changes
2. Update issue data
3. Update `updatedAt` timestamp
4. Add timeline event if significant change
5. Show confirmation

### FR-5: Resolve Issue

Users must be able to mark issues as resolved.

**Trigger:** "Marcar como Resolvido" button or status change

**Confirmation:** Show dialog "Tem certeza que pretende marcar este ponto como resolvido?"

**On confirmation:**
1. Change status to "resolved"
2. Update `updatedAt` timestamp
3. Add timeline event: "Issue resolved: [title]"
4. Update issue count badge (only count open + in progress)
5. Update dashboard card
6. Show confirmation toast

**Resolution behavior:**
- Resolved issues remain visible but visually subdued
- Resolved issues can be filtered out
- Resolved issues cannot be edited (but can be reopened in future phases)

### FR-6: Delete Issue

Users must be able to remove issues.

**Trigger:** Delete button/action on each issue

**Confirmation:** Required - "Tem certeza que pretende eliminar este ponto?"

**On confirmation:**
1. Soft delete the issue (set deletedAt timestamp)
2. Remove from visible list
3. Add timeline event: "Issue deleted: [title]"
4. Update counts
5. Show confirmation

### FR-7: Issue Impact on Request

Issues affect the overall request state.

**Dashboard indicator:**
- Request cards show "⚠️ X pontos em aberto" when open/in-progress issues exist
- Warning icon and count are visible

**Status suggestions:**
- If open issues exist, system may suggest status "Has Issues"
- When approving with open issues, show warning

---

## Acceptance Criteria

### AC-1: Issue List
- [ ] All issues for the request are displayed
- [ ] Each issue shows title, area, priority, status, due date
- [ ] Default sort is by priority (critical first)
- [ ] Tab badge shows count of open + in progress issues
- [ ] Empty state displays correctly when no issues exist

### AC-2: Filtering
- [ ] Filter by status works correctly
- [ ] Filter by area works correctly
- [ ] Multiple filters can be combined
- [ ] Clear filters option is available
- [ ] Search by title works

### AC-3: Create Issue
- [ ] "Adicionar Ponto" button opens modal form
- [ ] All form fields display correctly
- [ ] Required field validation works
- [ ] Character limits are enforced
- [ ] Due date cannot be in the past
- [ ] Successful creation shows confirmation
- [ ] New issue appears in list immediately
- [ ] Timeline records creation event

### AC-4: Edit Issue
- [ ] Clicking issue opens edit form
- [ ] Form is pre-populated with current values
- [ ] Changes save correctly
- [ ] Resolved issues cannot be edited
- [ ] Timeline records significant changes

### AC-5: Resolve Issue
- [ ] Resolve action is available for open/in progress issues
- [ ] Confirmation dialog appears before resolving
- [ ] Status changes to resolved after confirmation
- [ ] Resolved issues appear visually distinct
- [ ] Issue count updates correctly
- [ ] Timeline records resolution

### AC-6: Delete Issue
- [ ] Delete action requires confirmation
- [ ] Deleted issues are removed from list
- [ ] Timeline records deletion
- [ ] Counts update correctly

### AC-7: Request Impact
- [ ] Dashboard shows issue warning when issues exist
- [ ] Approving with open issues shows warning
- [ ] Progress considers issue state in status suggestions

---

## Success and Failure Examples

### Success Examples ✅

| Scenario | User Action | Expected Result |
|----------|-------------|-----------------|
| **Add first issue** | Maria clicks "Adicionar Ponto" and fills form | Issue appears in list, tab badge shows "(1)", dashboard card shows "⚠️ 1 ponto em aberto" |
| **Log detailed issue** | João adds issue with full description, notes, and due date | All information saves correctly and displays in issue detail |
| **Filter by area** | Ana clicks filter for "Jurídico" area | Only issues assigned to Legal area are displayed |
| **Find specific issue** | Pedro searches for "NDA" | Only issues with "NDA" in title appear |
| **Resolve issue** | Carla marks an issue as resolved | Status changes, issue becomes visually subdued, count decreases, timeline shows resolution |
| **Track overdue** | Bruno views issues list | Issue with past due date is highlighted as overdue |
| **Edit and update** | Sofia changes priority from Medium to Critical | Priority updates, visual indicator changes to red |
| **View history** | Manager checks timeline | Sees complete history: "Issue added", "Issue updated", "Issue resolved" |
| **Clear issues** | All issues resolved | Tab badge shows no count, dashboard warning disappears |

### Failure Examples ❌

| Scenario | User Action | Incorrect Result | Why It's Wrong |
|----------|-------------|------------------|----------------|
| **Missing validation** | User submits issue with 3-char title | Issue is created with too-short title | Minimum 5 characters must be enforced |
| **Silent deletion** | User clicks delete | Issue disappears with no confirmation | Confirmation dialog must appear first |
| **Count mismatch** | 3 open issues exist | Tab badge shows "(2)" | Count must be accurate |
| **Lost data** | User fills long description, submits fails | Form clears, description lost | Data must be preserved on error |
| **No feedback** | User creates issue | Modal closes, nothing happens | Must show success confirmation |
| **Edit resolved** | User tries to edit resolved issue | Edit form opens normally | Should show message that resolved issues cannot be edited |
| **Past due date** | User sets due date to yesterday | Date is accepted | Past dates must be rejected |
| **Resolve without confirm** | User clicks resolve | Issue immediately resolved | Must show confirmation dialog first |
| **Dashboard not updated** | Issue added | Dashboard still shows no issues | Dashboard must reflect current state |
| **Timeline missing** | Issue resolved | No timeline entry | All significant actions must be logged |

---

## Edge Cases and Boundary Conditions

### Input Boundaries

| Field | Boundary | Expected Behavior |
|-------|----------|------------------|
| Title | Exactly 5 characters | Accepted (minimum valid) |
| Title | 4 characters | Rejected with error |
| Title | 200 characters | Accepted (maximum valid) |
| Title | 201 characters | Rejected or truncated |
| Description | Empty | Accepted (optional) |
| Description | 1000 characters | Accepted (maximum valid) |
| Due Date | Today | Accepted |
| Due Date | Yesterday | Rejected with error |
| Due Date | Empty | Accepted (optional) |

### Status Transitions

| From Status | To Status | Allowed | Notes |
|-------------|-----------|---------|-------|
| Open | In Progress | Yes | Normal workflow |
| Open | Resolved | Yes | Direct resolution |
| In Progress | Open | Yes | Re-open for more work |
| In Progress | Resolved | Yes | Normal completion |
| Resolved | Open | No (MVP) | Future: allow reopening |
| Resolved | In Progress | No (MVP) | Future: allow reopening |

### Edge Scenarios

| Scenario | Expected Behavior |
|----------|------------------|
| Request has 50+ issues | List scrolls, consider pagination for performance |
| All issues are resolved | Show all as subdued, option to hide resolved |
| Issue due today | Highlight as due soon, not overdue |
| Issue due yesterday | Highlight as overdue with visual indicator |
| Create issue on closed request | Should be allowed (requests can be reopened) |
| Same title as existing issue | Allowed (not enforcing uniqueness) |

---

## User Interface Guidelines

### Issue List Layout

```
┌──────────────────────────────────────────────────────────────────┐
│ Pontos em Aberto                            [Adicionar Ponto]   │
├──────────────────────────────────────────────────────────────────┤
│ Filtrar: [Status ▼] [Área ▼]        Pesquisar: [____________]   │
├──────────────────────────────────────────────────────────────────┤
│ ┌────────────────────────────────────────────────────────────┐   │
│ │ 🔴 Cláusula de rescisão precisa revisão       [Jurídico]   │   │
│ │    Prioridade: Crítica    Data Limite: 20/01/2024         │   │
│ │    Estado: Aberto                        [Editar] [Resolver]│  │
│ └────────────────────────────────────────────────────────────┘   │
│ ┌────────────────────────────────────────────────────────────┐   │
│ │ 🟡 Clarificar tratamento de dados pessoais  [Prot. Dados]  │   │
│ │    Prioridade: Média    Data Limite: 25/01/2024           │   │
│ │    Estado: Em Tratamento                 [Editar] [Resolver]│  │
│ └────────────────────────────────────────────────────────────┘   │
│ ┌────────────────────────────────────────────────────────────┐   │
│ │ ✅ Documentação fiscal em falta               [Fiscal]      │   │
│ │    Prioridade: Alta    Resolvido: 15/01/2024              │   │
│ │    Estado: Resolvido                                       │   │
│ └────────────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────────┘
```

### Create/Edit Modal

```
┌──────────────────────────────────────────────────────────────┐
│ Adicionar Ponto em Aberto                              [X]   │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│ Título *                                                     │
│ [__________________________________________]                 │
│                                                              │
│ Descrição                                                    │
│ ┌──────────────────────────────────────────────────────────┐ │
│ │                                                          │ │
│ └──────────────────────────────────────────────────────────┘ │
│                                                              │
│ Área Responsável *              Prioridade *                 │
│ [Selecione...        ▼]        ○ Baixa  ○ Média             │
│                                 ○ Alta   ○ Crítica           │
│                                                              │
│ Data Limite                     Notas                        │
│ [__/__/____]                   [_________________________]  │
│                                                              │
│                              [Cancelar] [Guardar]            │
└──────────────────────────────────────────────────────────────┘
```

### Priority Colors

| Priority | Icon | Color |
|----------|------|-------|
| Critical | 🔴 or filled circle | Red-600 |
| High | 🟠 or filled circle | Red-500 |
| Medium | 🟡 or filled circle | Amber-500 |
| Low | 🟢 or filled circle | Green-500 |

### Status Styling

| Status | Badge Style | Row/Card Style |
|--------|-------------|----------------|
| Open | Red background, white text | Normal styling |
| In Progress | Amber background, dark text | Normal styling |
| Resolved | Green background, dark text | Subdued (gray text, lower opacity) |

---

## Definition of Done

This feature is complete when:

- [ ] **Issue List**: All issues display correctly with key information
- [ ] **Sorting**: Default sort by priority works
- [ ] **Filtering**: Status and area filters work correctly
- [ ] **Search**: Title search works with real-time results
- [ ] **Create**: Modal form creates issues with all fields
- [ ] **Validation**: All field constraints enforced with clear messages
- [ ] **Edit**: Existing issues can be modified (except resolved)
- [ ] **Resolve**: Confirmation flow works, status updates correctly
- [ ] **Delete**: Soft delete with confirmation works
- [ ] **Tab Badge**: Shows accurate count of open + in progress issues
- [ ] **Dashboard Impact**: Request cards show issue warning
- [ ] **Timeline**: All CRUD operations create timeline events
- [ ] **Empty State**: Displays correctly with clear CTA
- [ ] **Persistence**: All changes persist across page refresh
- [ ] **Error Handling**: Network errors show clear messages
- [ ] **Portuguese**: All text in Portuguese (Portugal)
- [ ] **Accessibility**: Keyboard navigation, proper labels
- [ ] **Tests**: Unit tests for validation, integration tests for CRUD
- [ ] **Code Review**: Approved by team member

---

## Related Features

- **[Request Detail](../request-detail/SPEC.md)**: Parent container with Issues tab
- **[Timeline](../timeline/SPEC.md)**: Records all issue events
- **[Area Analysis](../area-analysis/SPEC.md)**: Areas may create issues related to their concerns
- **[Final Decision](../final-decision/SPEC.md)**: Open issues affect approval decisions
- **[Dashboard](../dashboard/SPEC.md)**: Shows issue count on request cards

---

## Technical Notes

> These notes are for developer reference. Non-technical stakeholders may skip this section.

- Issues are separate entities linked to requests by `requestId`
- Soft delete: set `deletedAt` timestamp, filter in queries
- Consider optimistic UI updates for better perceived performance
- Modal forms should use React Hook Form for validation
- Issue list may need virtualization if many issues (future optimization)
- Due date handling must account for timezone (store as date-only, not datetime)
- Priority sort order: critical=0, high=1, medium=2, low=3 (ascending sort)

---

*Specification Version: 1.0*

# Feature Specification: Area Analysis

> **Feature ID**: `area-analysis`  
> **Priority**: P2 — Request Management  
> **Parent Document**: [SPEC.md](../../SPEC.md)

---

## Description

The Area Analysis feature allows department reviewers to record their assessments of contract and project requests. Each request must be reviewed by five distinct departments (called "Areas"), and this feature provides the interface for each department to document their review status, findings, and final decision.

This is the core workflow of the system — where the actual analysis work happens. Without Area Analysis, requests would remain static with no departmental input or decision-making capability.

### Business Context

When a contract or project requires approval, multiple departments must weigh in:
- **Legal** ensures terms are compliant and protect the company
- **Data Protection** assesses GDPR and privacy implications
- **Tax** evaluates fiscal impact and obligations
- **Cybersecurity** analyzes security risks and requirements
- **Business** confirms value alignment and strategic fit

Before this system, each department would send their feedback via email, leading to fragmented information, unclear accountability, and difficulty tracking who had completed their review. The Area Analysis feature centralizes all departmental input in one place.

### User Stories

- As a **Legal reviewer**, I want to record my analysis and decision so that others know Legal has completed their review
- As a **Data Protection officer**, I want to document GDPR concerns so that they are visible to all stakeholders
- As a **manager**, I want to see which areas have completed their review so I can track overall progress
- As an **internal owner**, I want to see each area's decision so I can understand the overall recommendation

### Interface Language

All visible text, labels, and messages must be in **Portuguese (Portugal)**.

---

## The Five Areas

| Area ID | Portuguese Name | English Name | Responsibility |
|---------|-----------------|--------------|----------------|
| `legal` | Jurídico | Legal | Legal compliance, contract terms, liability |
| `data_protection` | Proteção de Dados | Data Protection | GDPR, privacy, data handling |
| `tax` | Fiscal | Tax | Tax implications, fiscal compliance |
| `cybersecurity` | Cibersegurança | Cybersecurity | Security risks, technical safeguards |
| `business` | Negócio | Business | Business value, strategic alignment |

---

## Detailed Requirements

### Area Analysis Card Structure

Each area is displayed as an independent card within the request detail view. All five areas are visible simultaneously, allowing users to see the complete picture at a glance.

**Card Header:**
- Area name (Portuguese)
- Current status badge (Not Started / In Progress / Completed)
- Last updated timestamp (if updated)

**Card Body:**
- Responsible person field (text input)
- Comments field (textarea)
- Decision radio buttons (only visible when status is "Completed")

**Card Footer:**
- "Guardar Análise" (Save Analysis) button
- Last updated information

### Field Specifications

| Field | Portuguese Label | Type | Required | Constraints |
|-------|-----------------|------|----------|-------------|
| `responsible` | Responsável | Text input | No | Maximum 100 characters |
| `comments` | Comentários | Textarea | No | Maximum 2000 characters |
| `status` | Estado | Derived | N/A | Automatically set based on content |
| `decision` | Decisão | Radio buttons | Yes (when completing) | Three options (see below) |

### Status Values

| Status ID | Portuguese Label | When Applied |
|-----------|-----------------|--------------|
| `not_started` | Não Iniciado | No responsible person or comments entered |
| `in_progress` | Em Progresso | Responsible or comments entered, but no decision |
| `completed` | Concluído | Decision has been recorded |

**Status Derivation Logic:**
- If `decision` is set → status is `completed`
- Else if `responsible` OR `comments` has content → status is `in_progress`
- Else → status is `not_started`

### Decision Values

| Decision ID | Portuguese Label | Meaning | Visual Indicator |
|-------------|-----------------|---------|------------------|
| `no_objections` | Sem Objeções | Area approves without concerns | Green |
| `with_reservations` | Com Reservas | Area approves but has noted concerns | Amber/Yellow |
| `blocking` | Bloqueante | Area cannot approve; critical issues exist | Red |

---

## Functional Requirements

### FR-1: Display All Five Areas

When viewing a request detail, all five area analysis cards must be visible, regardless of their status.

**Requirements:**
- Cards must be displayed in a consistent order (Legal → Data Protection → Tax → Cybersecurity → Business)
- Each card must show current status even if no data has been entered
- Empty areas must show "Not Started" status with empty fields ready for input

### FR-2: Edit Area Analysis

Any user can edit any area's analysis at any time.

**Requirements:**
- Editing does not require explicit "Edit mode" — fields are always editable
- Changes are not saved until user clicks "Save Analysis"
- Multiple areas can be edited before saving (each saves independently)
- Unsaved changes in one area do not affect other areas

### FR-3: Save Area Analysis

When user clicks "Save Analysis" for an area:

1. Validate field constraints (character limits)
2. Update the area's data
3. Recalculate and update status based on content
4. Set `updatedAt` timestamp to current time
5. Add timeline event: "Area [name] updated by [user]"
6. Recalculate request progress (completed areas / 5)
7. Show confirmation toast message
8. Refresh the card to show updated information

### FR-4: Decision Recording

Users can record a decision for their area.

**Requirements:**
- Decision radio buttons must show all three options
- Selecting "Blocking" must trigger the blocking alert system (see [Blocking Alerts](../blocking-alerts/SPEC.md))
- Decision can be changed after initial recording (not locked)
- When decision is set, status automatically becomes "Completed"

### FR-5: Progress Impact

Area completion affects the overall request progress.

**Requirements:**
- Progress = (number of areas with status "completed") / 5 × 100%
- Progress must update immediately when an area's status changes
- Progress is displayed on both the request detail header and dashboard card

### FR-6: Visual Decision Indicators

Completed areas must clearly display their decision.

**Requirements:**
- "No Objections" — Green checkmark or border
- "With Reservations" — Amber/Yellow warning indicator
- "Blocking" — Red alert indicator, highly prominent

---

## Acceptance Criteria

### AC-1: Area Display
- [ ] All five area cards display on request detail view
- [ ] Cards appear in correct order (Legal → Data Protection → Tax → Cybersecurity → Business)
- [ ] Each card shows Portuguese area name
- [ ] Each card shows current status badge
- [ ] Empty areas show "Não Iniciado" status

### AC-2: Editing Fields
- [ ] Responsible field accepts text input up to 100 characters
- [ ] Comments field accepts text input up to 2000 characters
- [ ] Fields are editable without clicking an "Edit" button
- [ ] Character limits are enforced with clear feedback

### AC-3: Saving Changes
- [ ] "Guardar Análise" button is visible on each card
- [ ] Clicking save persists the data
- [ ] Confirmation message appears after successful save
- [ ] Timestamp updates to show when last saved
- [ ] Timeline records the update event

### AC-4: Status Transitions
- [ ] Status shows "Não Iniciado" when area is empty
- [ ] Status changes to "Em Progresso" when responsible or comments are added
- [ ] Status changes to "Concluído" when decision is recorded
- [ ] Status badge color matches the status

### AC-5: Decision Recording
- [ ] Three decision options are available: Sem Objeções, Com Reservas, Bloqueante
- [ ] Selecting a decision and saving marks the area as completed
- [ ] Decision can be changed after initial selection
- [ ] Blocking decision triggers blocking alert banner on request

### AC-6: Progress Calculation
- [ ] Request progress shows 0% when no areas are completed
- [ ] Progress shows 20% per completed area (20%, 40%, 60%, 80%, 100%)
- [ ] Progress updates immediately after saving an area
- [ ] Progress is accurate on both detail view and dashboard

### AC-7: Visual Indicators
- [ ] Completed areas with "No Objections" show green indicator
- [ ] Completed areas with "With Reservations" show amber indicator
- [ ] Completed areas with "Blocking" show red indicator
- [ ] Visual indicators are accessible (not color-only)

---

## Success and Failure Examples

### Success Examples ✅

| Scenario | User Action | Expected Result |
|----------|-------------|-----------------|
| **First area update** | Ana from Legal enters her name as responsible and adds comments | Status changes from "Não Iniciado" to "Em Progresso", save button enabled |
| **Complete an area** | Bruno selects "Sem Objeções" as decision and clicks save | Status becomes "Concluído", green indicator appears, progress updates to 20% |
| **Record blocking decision** | Carla from Data Protection selects "Bloqueante" | Red indicator appears on card, blocking alert banner appears at top of request |
| **Update existing analysis** | Daniel changes his comments after initial save | New content saves successfully, timestamp updates, timeline shows new event |
| **All areas complete** | Fifth and final area is marked complete | Progress shows 100%, all five areas show completion status |
| **Change decision** | Eva reconsiders and changes from "Com Reservas" to "Sem Objeções" | Decision updates, indicator changes from amber to green |
| **View without editing** | Fernando opens a request to check progress | All areas visible with current status, no accidental changes made |
| **Quick status scan** | Gabriela glances at area cards | She can immediately see 3 completed (green), 1 in progress (amber badge), 1 not started (gray) |

### Failure Examples ❌

| Scenario | User Action | Incorrect Result | Why It's Wrong |
|----------|-------------|------------------|----------------|
| **Unsaved work lost** | Hugo enters detailed comments then accidentally navigates away | All his work is lost with no warning | System should warn about unsaved changes before navigation |
| **Save fails silently** | Inês clicks save but network fails | Button returns to normal state, no error shown | User thinks save succeeded; must show clear error message |
| **Progress miscalculation** | 3 areas completed | Progress shows 50% instead of 60% | Math error; 3/5 = 60%, not 50% |
| **Missing area** | User opens request detail | Only 4 area cards visible | All 5 areas must always be present and visible |
| **Blocking not highlighted** | User sets blocking decision | Card looks same as "with reservations" | Blocking must be prominently red and trigger alert banner |
| **Status doesn't update** | User enters responsible name | Status still shows "Não Iniciado" | Should change to "Em Progresso" when any content is added |
| **Can't clear decision** | User accidentally clicks wrong decision | No way to undo before saving | Radio buttons should allow changing before save |
| **Duplicate timeline events** | User saves same content twice | Timeline shows duplicate "area updated" events | Should only create event if content actually changed |
| **Other areas affected** | User saves Legal area | Data Protection area loses unsaved changes | Areas must be completely independent |
| **Stale data shown** | User saves, then views request | Old timestamp shows instead of just-saved time | Display must refresh after save |

---

## Edge Cases and Boundary Conditions

### Input Boundaries

| Field | Boundary | Expected Behavior |
|-------|----------|------------------|
| Responsible | Empty | Accepted (optional field) |
| Responsible | Exactly 100 characters | Accepted (maximum valid) |
| Responsible | 101 characters | Rejected or truncated with warning |
| Comments | Empty | Accepted (optional field) |
| Comments | Exactly 2000 characters | Accepted (maximum valid) |
| Comments | 2001 characters | Rejected or truncated with warning |

### Special Content

| Input | Expected Behavior |
|-------|------------------|
| Portuguese characters (ã, ç, é) | Stored and displayed correctly |
| Line breaks in comments | Preserved and displayed |
| HTML in comments | Sanitized/escaped (not rendered as HTML) |
| Very long single word | Displayed with word-wrap |
| Only whitespace in responsible | Treated as empty (trimmed) |

### Status Edge Cases

| Situation | Expected Status |
|-----------|----------------|
| Responsible filled, comments empty, no decision | In Progress |
| Responsible empty, comments filled, no decision | In Progress |
| Both filled, no decision | In Progress |
| Only decision selected (others empty) | Completed |
| Decision removed after being set | In Progress (if content exists) or Not Started |

### Concurrent Editing

| Scenario | Expected Behavior |
|----------|------------------|
| Two users edit same area simultaneously | Last save wins; consider showing warning |
| User saves while another user is viewing | Viewer sees stale data until refresh |
| Rapid save clicks | Debounce to prevent duplicate requests |

---

## User Interface Guidelines

### Card Layout

```
┌──────────────────────────────────────────────────────────┐
│ Jurídico                                    [Em Progresso] │
├──────────────────────────────────────────────────────────┤
│                                                          │
│ Responsável: [_____________________________]             │
│                                                          │
│ Comentários:                                             │
│ ┌──────────────────────────────────────────────────────┐│
│ │                                                      ││
│ │                                                      ││
│ │                                                      ││
│ └──────────────────────────────────────────────────────┘│
│                                                          │
│ Decisão:                                                 │
│ ○ Sem Objeções  ○ Com Reservas  ○ Bloqueante            │
│                                                          │
│ Última atualização: 15/01/2024 às 14:32                 │
│                                                          │
│                              [Guardar Análise]           │
└──────────────────────────────────────────────────────────┘
```

### Status Badge Colors

| Status | Background | Text |
|--------|------------|------|
| Não Iniciado | Gray-100 | Gray-600 |
| Em Progresso | Amber-100 | Amber-700 |
| Concluído | Green-100 | Green-700 |

### Decision Indicator Colors

| Decision | Indicator Color | Border Color |
|----------|----------------|--------------|
| Sem Objeções | Green-500 | Green-300 |
| Com Reservas | Amber-500 | Amber-300 |
| Bloqueante | Red-600 | Red-400 |

---

## Definition of Done

This feature is complete when:

- [ ] **Five Areas Display**: All five area cards render correctly with Portuguese labels
- [ ] **Edit Functionality**: Users can edit responsible and comments fields on any area
- [ ] **Save Works**: Clicking save persists data and shows confirmation
- [ ] **Status Updates**: Status automatically changes based on content (not started → in progress → completed)
- [ ] **Decision Recording**: All three decision options work and affect status
- [ ] **Progress Updates**: Request progress recalculates when areas are completed
- [ ] **Timeline Events**: Saving an area creates a timeline event
- [ ] **Visual Indicators**: Decision colors display correctly (green/amber/red)
- [ ] **Blocking Integration**: Blocking decision triggers alert banner
- [ ] **Validation**: Character limits are enforced with clear messages
- [ ] **Independence**: Editing one area doesn't affect others
- [ ] **Persistence**: Data survives page refresh
- [ ] **Error Handling**: Network errors show clear messages, preserve unsaved data
- [ ] **Accessibility**: All inputs labeled, keyboard navigable
- [ ] **Code Quality**: Passes linting, no TypeScript errors
- [ ] **Tests**: Unit tests for status derivation logic, integration test for save flow
- [ ] **Review**: Code reviewed and approved

---

## Related Features

- **[Request Detail](../request-detail/SPEC.md)**: Parent container that displays area analysis cards
- **[Blocking Alerts](../blocking-alerts/SPEC.md)**: Triggered when any area records a blocking decision
- **[Timeline](../timeline/SPEC.md)**: Records all area update events
- **[Final Decision](../final-decision/SPEC.md)**: Uses area decisions to inform final approval

---

## Technical Notes

> These notes are for developer reference. Non-technical stakeholders may skip this section.

- Each area analysis is a separate entity linked to the request by `requestId`
- Status should be computed, not stored (derive from content on read)
- Consider optimistic UI updates for save operations
- Debounce save button to prevent accidental double-clicks
- Auto-save could be a future enhancement but is not in MVP scope
- Area cards should be individual client components to enable independent updates

---

*Specification Version: 1.0*

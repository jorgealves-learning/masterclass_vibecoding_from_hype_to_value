# Feature Specification: Final Decision

> **Feature ID**: `final-decision`  
> **Priority**: P3 — Issues & Decisions  
> **Parent Document**: [SPEC.md](../../SPEC.md)

---

## Description

The Final Decision feature enables authorized users to record the official outcome of a contract or project analysis. After all departments have completed their reviews and any issues have been addressed, this feature captures the formal decision: approved, approved with conditions, rejected, or pending.

This is the culmination of the entire analysis workflow. It transforms all the individual area assessments, issue resolutions, and document reviews into a single, documented decision that can be referenced and audited.

### Business Context

Every contract or project analysis must conclude with a clear decision. Before this system:
- Decisions were communicated via email, often ambiguously
- There was no standardized format for recording approvals
- It was difficult to determine who approved what and when
- Summaries were inconsistent or non-existent

The Final Decision feature provides:
- A standardized decision recording process
- Clear accountability (who approved, when)
- Automatic summary generation from area analyses
- A permanent record for audit and compliance

### User Stories

- As a **manager**, I want to record the final decision so that everyone knows the official outcome
- As an **auditor**, I want to see who made the decision and when so I can verify compliance
- As a **request owner**, I want a summary of the analysis so I can communicate it to stakeholders
- As a **department reviewer**, I want to see how the overall decision reflects my area's input

### Interface Language

All visible text, labels, and messages must be in **Portuguese (Portugal)**.

---

## Decision Types

| Decision ID | Portuguese Label | Meaning | Status Implication |
|-------------|-----------------|---------|-------------------|
| `pending` | Pendente | Decision not yet made | No change |
| `approved` | Aprovado | Fully approved without concerns | Request status → Approved |
| `approved_with_conditions` | Aprovado com Condições | Approved but with specific requirements | Request status → Approved w/ Conditions |
| `rejected` | Rejeitado | Not approved, cannot proceed | Request status → Rejected |

---

## Detailed Requirements

### Decision Data Structure

| Field | Portuguese Label | Type | Required | Constraints |
|-------|-----------------|------|----------|-------------|
| `decision` | Decisão | Radio/Select | Yes (when recording) | One of four decision types |
| `finalApprover` | Aprovador Final | Text input | Yes (when decision ≠ pending) | Minimum 3 characters, maximum 100 characters |
| `decisionDate` | Data da Decisão | Date picker | Yes (when decision ≠ pending) | Must be today or past |
| `summary` | Resumo | Textarea | No | Maximum 3000 characters |

---

## Functional Requirements

### FR-1: Decision Tab Display

The Final Decision tab shows the current decision state and allows recording/updating the decision.

**Display components:**

1. **Status Summary Panel** — Shows current state at a glance:
   - Current request status
   - Count of completed areas (X/5)
   - Count of open issues (warning if > 0)
   - Blocking areas (list if any)

2. **Decision Form** — For recording the decision:
   - Decision selection (four options)
   - Final approver name
   - Decision date
   - Summary text area

3. **Generate Summary Button** — Auto-generates summary from analysis data

### FR-2: Status Summary Panel

Before recording a decision, users should see the current state of the request.

**Requirements:**
- Display current request status with badge
- Show progress: "X de 5 áreas concluídas"
- Show open issues count with warning styling if > 0: "⚠️ X pontos em aberto"
- If any area has blocking decision, show prominent alert
- This panel is informational only, not editable

### FR-3: Record Decision

Users can record or update the final decision.

**Requirements:**
- Decision is required before approver and date become active
- If decision is "Pending", approver and date are optional
- If decision is Approved/Rejected/Approved with Conditions:
  - Final approver is required
  - Decision date is required and defaults to today
- Decision can be changed after initial recording (not locked)

**Validation rules:**
- Final approver: minimum 3 characters, maximum 100 characters
- Decision date: cannot be in the future
- Summary: maximum 3000 characters

**On save:**
1. Validate all required fields
2. Update request's final decision data
3. Update request status to match decision
4. Add timeline event: "Decision recorded: [decision type] by [approver]"
5. Update `updatedAt` timestamp
6. Show confirmation toast

### FR-4: Generate Summary

Users can automatically generate a summary based on the request data.

**Trigger:** "Gerar Resumo" (Generate Summary) button

**Generated content includes:**
- Request title and type
- Current status
- Each area's decision with responsible person name
- Count of total vs. open vs. resolved issues
- Criticality level
- Any blocking concerns

**Requirements:**
- Generated text populates the summary textarea
- User can edit the generated text before saving
- Generation does not automatically save; user must click save
- Button is always available regardless of decision state

**Sample generated format:**
```
RESUMO DA ANÁLISE

Pedido: [Title]
Tipo: [Contract/Project]
Criticidade: [High/Medium/Low]

ANÁLISES POR ÁREA:
• Jurídico: [Decision] — [Responsible Name]
• Proteção de Dados: [Decision] — [Responsible Name]
• Fiscal: [Decision] — [Responsible Name]
• Cibersegurança: [Decision] — [Responsible Name]
• Negócio: [Decision] — [Responsible Name]

PONTOS EM ABERTO:
• Total: X | Resolvidos: Y | Em aberto: Z

[If blocking exists:]
⚠️ PREOCUPAÇÕES BLOQUEANTES:
• [Area Name]: [Comments excerpt]
```

### FR-5: Blocking Prevention

The system must prevent or warn against approval when blocking concerns exist.

**Requirements:**
- If any area has `decision === 'blocking'`, show prominent warning
- When user selects "Approved" while blocking exists:
  - Show modal warning: "Existem preocupações bloqueantes registadas. Tem a certeza que pretende aprovar?"
  - Require explicit confirmation to proceed
  - Log this override in timeline: "Approved despite blocking concerns from [Area]"
- When user selects "Approved" while open issues exist:
  - Show info notification: "Nota: Existem X pontos em aberto por resolver"
  - Allow approval to proceed (warning only, not blocking)

### FR-6: Decision State Transitions

**Status auto-update:**
When decision is saved, request status updates automatically:
- `approved` → Request status becomes "Aprovado"
- `approved_with_conditions` → Request status becomes "Aprovado com Condições"
- `rejected` → Request status becomes "Rejeitado"
- `pending` → No status change (manual status management)

---

## Acceptance Criteria

### AC-1: Status Summary Display
- [ ] Current request status displays with correct badge
- [ ] Completed areas count shows "X de 5 áreas concluídas"
- [ ] Open issues count displays with warning styling when > 0
- [ ] Blocking areas are prominently displayed if present

### AC-2: Decision Form
- [ ] All four decision options are available
- [ ] Selecting non-pending decision enables approver and date fields
- [ ] Approver field validates minimum 3 characters
- [ ] Date field defaults to today
- [ ] Date field rejects future dates
- [ ] Summary field accepts up to 3000 characters

### AC-3: Save Decision
- [ ] Save validates required fields
- [ ] Successful save shows confirmation
- [ ] Request status updates to match decision
- [ ] Timeline records decision event with details
- [ ] Form shows saved values after refresh

### AC-4: Generate Summary
- [ ] "Gerar Resumo" button is visible
- [ ] Clicking generates text in summary field
- [ ] Generated text includes all required information
- [ ] Generated text is editable before save
- [ ] Generation works regardless of current decision state

### AC-5: Blocking Prevention
- [ ] Warning appears when approving with blocking areas
- [ ] Confirmation dialog requires explicit action
- [ ] Timeline records approval despite blocking
- [ ] Info notification appears when approving with open issues

### AC-6: Status Synchronization
- [ ] Saving "Approved" decision changes request status to "Aprovado"
- [ ] Saving "Approved with Conditions" changes status to "Aprovado com Condições"
- [ ] Saving "Rejected" changes request status to "Rejeitado"
- [ ] Saving "Pending" does not change request status

---

## Success and Failure Examples

### Success Examples ✅

| Scenario | User Action | Expected Result |
|----------|-------------|-----------------|
| **View analysis state** | Manager opens Decision tab | Sees summary showing 4/5 areas complete, 2 open issues, no blocking |
| **Generate summary** | User clicks "Gerar Resumo" | Summary textarea fills with formatted analysis summary |
| **Edit generated summary** | User modifies auto-generated text | Changes are preserved in textarea until save |
| **Approve request** | All areas complete, user selects "Aprovado", enters name and date | Decision saves, request status becomes "Aprovado", timeline records event |
| **Approve with conditions** | User selects "Aprovado com Condições" and adds conditions to summary | Decision saves with summary explaining conditions |
| **Reject request** | Legal has blocking concern, manager selects "Rejeitado" | Decision saves as rejected, status updates |
| **Approve despite blocking** | User selects "Aprovado" while Tax is blocking | Warning modal appears, user confirms, decision saves with timeline noting override |
| **Track approval** | Auditor checks decision | Sees approver name "João Silva", date "15/01/2024", and complete summary |
| **Update decision** | Manager changes from "Pendente" to "Aprovado" | New decision replaces pending state, status updates |

### Failure Examples ❌

| Scenario | User Action | Incorrect Result | Why It's Wrong |
|----------|-------------|------------------|----------------|
| **Missing approver** | User selects "Aprovado" but leaves approver blank | Decision saves anyway | Approver is required for non-pending decisions |
| **Future date** | User sets decision date to tomorrow | Date is accepted | Decision date must be today or past |
| **No blocking warning** | User approves while Legal has blocking | Approval proceeds with no warning | Must warn and require confirmation for blocking override |
| **Status not updated** | User saves "Aprovado" decision | Request status stays "Em Análise" | Status must sync with decision |
| **Summary lost** | User generates summary, navigation occurs | Summary disappears without save | Should warn about unsaved changes |
| **Timeline missing** | Decision is recorded | No timeline entry for decision | All decisions must create timeline events |
| **Silent save failure** | User clicks save, network fails | Form resets, no error shown | Must show error and preserve entered data |
| **Wrong summary generation** | User clicks generate summary | Only partial information appears | Must include all areas, issues, and warnings |
| **Date validation bypass** | User types invalid date format | Form submits with invalid date | Date format must be validated |
| **Double submission** | User clicks save rapidly | Two timeline events created | Must debounce or prevent duplicate saves |

---

## Edge Cases and Boundary Conditions

### Input Boundaries

| Field | Boundary | Expected Behavior |
|-------|----------|------------------|
| Final Approver | Exactly 3 characters | Accepted (minimum valid) |
| Final Approver | 2 characters | Rejected with error |
| Final Approver | 100 characters | Accepted (maximum valid) |
| Final Approver | 101 characters | Rejected or truncated |
| Summary | Empty | Accepted (optional) |
| Summary | 3000 characters | Accepted (maximum valid) |
| Summary | 3001 characters | Rejected or truncated |
| Decision Date | Today | Accepted |
| Decision Date | Yesterday | Accepted |
| Decision Date | Tomorrow | Rejected with error |

### State Combinations

| Areas Status | Issues Status | Blocking | User Tries | Expected Behavior |
|--------------|---------------|----------|------------|------------------|
| 5/5 complete | 0 open | None | Approve | Clean approval, no warnings |
| 5/5 complete | 3 open | None | Approve | Info notification about issues, allows approval |
| 4/5 complete | 0 open | None | Approve | Allowed (completion not required for decision) |
| 5/5 complete | 0 open | Tax blocking | Approve | Warning modal, requires confirmation |
| 5/5 complete | 2 open | Legal blocking | Approve | Warning modal for blocking, info for issues |
| 3/5 complete | 0 open | None | Reject | Allowed, manager can reject early |

### Special Content

| Input | Expected Behavior |
|-------|------------------|
| Portuguese characters in approver | Stored and displayed correctly |
| Line breaks in summary | Preserved and displayed |
| HTML in summary | Sanitized, not rendered as HTML |
| Generated summary with missing area data | Shows "Não iniciado" for areas without decisions |

### Concurrent Changes

| Scenario | Expected Behavior |
|----------|------------------|
| Area decision changes while user on Decision tab | Summary panel may be stale; generate summary gets fresh data |
| Two users try to record decision simultaneously | Last save wins; consider warning |
| Issue resolved while user viewing Decision tab | Status summary may be stale until refresh |

---

## User Interface Guidelines

### Decision Tab Layout

```
┌──────────────────────────────────────────────────────────────────┐
│ Decisão Final                                                    │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│ ┌──────────────────────── ESTADO ATUAL ────────────────────────┐│
│ │                                                              ││
│ │ Estado do Pedido: [Em Análise]                               ││
│ │ Progresso: 4 de 5 áreas concluídas                          ││
│ │ Pontos em Aberto: ⚠️ 2 por resolver                         ││
│ │                                                              ││
│ │ ⛔ PREOCUPAÇÃO BLOQUEANTE: Fiscal                            ││
│ │                                                              ││
│ └──────────────────────────────────────────────────────────────┘│
│                                                                  │
│ ┌──────────────────────── DECISÃO ─────────────────────────────┐│
│ │                                                              ││
│ │ Decisão: *                                                   ││
│ │ ○ Pendente  ○ Aprovado  ○ Aprovado c/ Condições  ○ Rejeitado ││
│ │                                                              ││
│ │ Aprovador Final: *          Data da Decisão: *              ││
│ │ [_____________________]     [__/__/____]                    ││
│ │                                                              ││
│ │ Resumo:                                    [Gerar Resumo]   ││
│ │ ┌──────────────────────────────────────────────────────────┐││
│ │ │                                                          │││
│ │ │                                                          │││
│ │ │                                                          │││
│ │ │                                                          │││
│ │ └──────────────────────────────────────────────────────────┘││
│ │                                                              ││
│ │                                    [Guardar Decisão]         ││
│ │                                                              ││
│ └──────────────────────────────────────────────────────────────┘│
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Blocking Warning Modal

```
┌────────────────────────────────────────────────────┐
│ ⚠️ Atenção — Preocupações Bloqueantes             │
├────────────────────────────────────────────────────┤
│                                                    │
│ As seguintes áreas registaram preocupações         │
│ bloqueantes:                                       │
│                                                    │
│ • Fiscal: Implicações fiscais não resolvidas      │
│                                                    │
│ Tem a certeza que pretende aprovar este pedido    │
│ apesar destas preocupações?                       │
│                                                    │
│ Esta ação será registada no histórico.            │
│                                                    │
│        [Cancelar]    [Aprovar Mesmo Assim]        │
│                                                    │
└────────────────────────────────────────────────────┘
```

### Decision Badge Colors

| Decision | Background | Text |
|----------|------------|------|
| Pendente | Gray-100 | Gray-600 |
| Aprovado | Green-100 | Green-700 |
| Aprovado c/ Condições | Teal-100 | Teal-700 |
| Rejeitado | Red-100 | Red-700 |

---

## Definition of Done

This feature is complete when:

- [ ] **Status Summary**: Panel shows accurate progress, issues, and blocking info
- [ ] **Decision Options**: All four decision types selectable
- [ ] **Required Fields**: Approver and date required for non-pending decisions
- [ ] **Validation**: All field constraints enforced with Portuguese error messages
- [ ] **Save Works**: Decision persists and shows confirmation
- [ ] **Status Sync**: Request status updates to match decision type
- [ ] **Timeline Events**: Decision creates detailed timeline entry
- [ ] **Summary Generation**: Auto-generates from area analyses and issues
- [ ] **Summary Editable**: Generated summary can be modified before save
- [ ] **Blocking Warning**: Modal appears when approving with blocking areas
- [ ] **Blocking Override**: Confirmation required, logged in timeline
- [ ] **Issues Warning**: Info notification when approving with open issues
- [ ] **Date Validation**: Future dates rejected
- [ ] **Persistence**: Decision survives page refresh
- [ ] **Error Handling**: Network errors show clear messages, preserve data
- [ ] **Portuguese**: All labels and messages in Portuguese
- [ ] **Accessibility**: Form fields labeled, keyboard navigable
- [ ] **Responsive**: Works on desktop and tablet viewports
- [ ] **Tests**: Unit tests for validation, integration tests for save flow
- [ ] **Code Review**: Approved by team member

---

## Related Features

- **[Request Detail](../request-detail/SPEC.md)**: Parent container with Decision tab
- **[Area Analysis](../area-analysis/SPEC.md)**: Provides input for summary generation
- **[Open Issues](../open-issues/SPEC.md)**: Issue counts appear in status summary
- **[Blocking Alerts](../blocking-alerts/SPEC.md)**: Integrates with approval prevention
- **[Timeline](../timeline/SPEC.md)**: Records decision events

---

## Technical Notes

> These notes are for developer reference. Non-technical stakeholders may skip this section.

- Final decision is stored as a nested object within the Request
- Status sync should happen atomically with decision save
- Summary generation should fetch fresh data from all areas and issues
- Consider optimistic UI for decision save with rollback on failure
- Blocking detection: `request.areas.some(a => a.decision === 'blocking')`
- Decision override should include `overrideReason` field in timeline event
- Date field should use native date picker with Portuguese locale
- Summary textarea should auto-resize or have generous height
- Modal for blocking warning should use shadcn/ui AlertDialog

---

*Specification Version: 1.0*

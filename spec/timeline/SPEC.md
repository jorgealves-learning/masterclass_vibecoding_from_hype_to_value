# Feature Specification: Timeline

> **Feature ID**: `timeline`  
> **Priority**: P4 — Supporting Feature  
> **Parent Document**: [SPEC.md](../../SPEC.md)

---

## Description

The Timeline feature provides a chronological audit trail of all significant actions taken on a request. Every change, decision, update, and event is recorded with a timestamp and (where applicable) the user who performed it. This creates a complete history that supports accountability, traceability, and compliance requirements.

The Timeline is not a feature users interact with directly — they don't "use" the timeline, they observe it. However, it is critical infrastructure that other features depend on for recording their actions.

### Business Context

In regulated environments and complex approval processes, knowing *what happened* is often as important as knowing *what the current state is*. The Timeline provides:

- **Accountability**: Who did what and when
- **Traceability**: How did the request reach its current state
- **Compliance**: Auditable record of all decisions
- **Debugging**: Understanding unexpected states
- **Communication**: Team members can see what's changed since they last looked

Before this system, understanding the history of a contract analysis required piecing together email threads, meeting notes, and spreadsheet versions. The Timeline consolidates this into a single, authoritative record.

### User Stories

- As a **manager**, I want to see when the status was changed so I can track progress over time
- As an **auditor**, I want a complete history of all actions so I can verify compliance
- As a **department reviewer**, I want to see what changed since my last review so I can catch up quickly
- As a **request owner**, I want to know who made the final decision and when

### Interface Language

All visible text, labels, and messages must be in **Portuguese (Portugal)**.

---

## Event Types

The Timeline records the following types of events:

| Event Type ID | Portuguese Label | Description | Triggered By |
|---------------|------------------|-------------|--------------|
| `request_created` | Pedido criado | Request was created | Create Request |
| `status_changed` | Estado alterado | Request status was changed | Status change |
| `area_updated` | Área atualizada | Area analysis was modified | Area Analysis save |
| `issue_added` | Ponto adicionado | New issue was created | Create Issue |
| `issue_resolved` | Ponto resolvido | Issue was marked resolved | Resolve Issue |
| `issue_deleted` | Ponto removido | Issue was deleted | Delete Issue |
| `document_added` | Documento adicionado | Document was attached | Add Document |
| `document_removed` | Documento removido | Document was deleted | Delete Document |
| `decision_recorded` | Decisão registada | Final decision was recorded | Final Decision save |
| `summary_generated` | Resumo gerado | Summary was auto-generated | Generate Summary action |
| `approval_blocked` | Aprovação bloqueada | Approval attempt was blocked | Blocked approval attempt |

---

## Timeline Event Data Structure

Each timeline event contains:

| Field | Type | Description | Required |
|-------|------|-------------|----------|
| `id` | String | Unique identifier for the event | Auto-generated |
| `requestId` | String | Reference to parent request | Auto-set |
| `type` | EventType | One of the defined event types | Yes |
| `description` | String | Human-readable description in Portuguese | Yes |
| `user` | String | Name or identifier of who performed the action | When available |
| `timestamp` | ISO 8601 | When the event occurred | Auto-set |
| `metadata` | Object | Additional context-specific data | Optional |

### Metadata Examples

Different event types may include relevant metadata:

**status_changed:**
```json
{
  "previousStatus": "new",
  "newStatus": "in_analysis"
}
```

**area_updated:**
```json
{
  "areaName": "legal",
  "previousDecision": null,
  "newDecision": "no_objections"
}
```

**issue_resolved:**
```json
{
  "issueTitle": "Cláusula de rescisão problemática",
  "resolvedBy": "Ana Silva"
}
```

**approval_blocked:**
```json
{
  "blockingAreas": ["legal", "tax"],
  "attemptedStatus": "approved"
}
```

---

## Functional Requirements

### FR-1: Event Recording

The system must automatically record timeline events when significant actions occur.

**Requirements:**
- Events are recorded atomically with the action that triggers them
- Events are immutable — once created, they cannot be edited or deleted
- Events are always associated with a request via `requestId`
- Timestamp is set server-side to ensure accuracy
- User information is captured when available

### FR-2: Timeline Display

The Timeline is displayed within the General Information tab of the Request Detail view.

**Requirements:**
- Events are displayed in reverse chronological order (newest first)
- Each event shows: icon, description, user (if available), and relative/absolute timestamp
- By default, show the 10 most recent events
- "Ver mais" (See more) link/button to expand and show all events
- If no events exist (impossible in practice), show appropriate message

### FR-3: Event Description Generation

Each event type should generate a clear, Portuguese description.

**Templates:**

| Event Type | Description Template |
|------------|---------------------|
| `request_created` | "Pedido criado" (optionally: "por [User]") |
| `status_changed` | "Estado alterado de [Previous] para [New]" |
| `area_updated` | "Área [AreaName] atualizada" (optionally: "por [User]") |
| `issue_added` | "Ponto em aberto adicionado: [IssueTitle]" |
| `issue_resolved` | "Ponto em aberto resolvido: [IssueTitle]" |
| `issue_deleted` | "Ponto em aberto removido: [IssueTitle]" |
| `document_added` | "Documento adicionado: [DocumentName]" |
| `document_removed` | "Documento removido: [DocumentName]" |
| `decision_recorded` | "Decisão registada: [DecisionType] por [Approver]" |
| `summary_generated` | "Resumo gerado automaticamente" |
| `approval_blocked` | "Tentativa de aprovação bloqueada devido a [Area1, Area2]" |

### FR-4: Timestamp Display

Timestamps should be user-friendly with both relative and absolute options.

**Requirements:**
- Recent events: Show relative time ("há 5 minutos", "há 2 horas", "ontem")
- Older events: Show date ("15/01/2024 às 14:32")
- Hover/tooltip should show full timestamp
- All times displayed in user's local timezone

### FR-5: Event Icons

Each event type should have a distinctive icon for quick scanning.

| Event Type | Icon Suggestion |
|------------|-----------------|
| `request_created` | ➕ Plus or document icon |
| `status_changed` | 🔄 Refresh/cycle icon |
| `area_updated` | ✏️ Edit/pencil icon |
| `issue_added` | ⚠️ Warning/alert icon |
| `issue_resolved` | ✅ Checkmark icon |
| `issue_deleted` | 🗑️ Trash icon |
| `document_added` | 📄 Document icon |
| `document_removed` | 📄❌ Document with X |
| `decision_recorded` | ⚖️ Scale/gavel icon |
| `summary_generated` | 📝 Notes icon |
| `approval_blocked` | ⛔ Stop/blocked icon |

---

## Acceptance Criteria

### AC-1: Event Recording
- [ ] `request_created` event is recorded when new request is submitted
- [ ] `status_changed` event is recorded when status is modified
- [ ] `area_updated` event is recorded when any area is saved with changes
- [ ] `issue_added` event is recorded when new issue is created
- [ ] `issue_resolved` event is recorded when issue is marked resolved
- [ ] `issue_deleted` event is recorded when issue is deleted
- [ ] `document_added` event is recorded when document is attached
- [ ] `document_removed` event is recorded when document is deleted
- [ ] `decision_recorded` event is recorded when final decision is saved
- [ ] `approval_blocked` event is recorded when approval is prevented

### AC-2: Event Data
- [ ] All events have unique IDs
- [ ] All events have accurate timestamps
- [ ] Events include user information when available
- [ ] Events include relevant metadata for context

### AC-3: Timeline Display
- [ ] Timeline appears in General Information tab
- [ ] Events are sorted newest first
- [ ] Default shows 10 most recent events
- [ ] "Ver mais" expands to show all events
- [ ] Each event shows icon, description, and timestamp

### AC-4: Event Descriptions
- [ ] All descriptions are in Portuguese
- [ ] Descriptions include relevant details (names, titles)
- [ ] Descriptions are human-readable and clear

### AC-5: Timestamp Display
- [ ] Recent times show relative format
- [ ] Older times show absolute format
- [ ] Full timestamp available on hover

### AC-6: Event Immutability
- [ ] Events cannot be edited after creation
- [ ] Events cannot be deleted
- [ ] No API endpoint allows event modification

---

## Success and Failure Examples

### Success Examples ✅

| Scenario | System Behavior | Expected Outcome |
|----------|-----------------|------------------|
| **Request creation** | User submits new request form | Timeline shows "Pedido criado por [User]" as first event |
| **Status change** | Manager changes status to "Em Análise" | Timeline shows "Estado alterado de Novo para Em Análise" |
| **Area completion** | Legal marks decision as "Sem Objeções" | Timeline shows "Área Jurídico atualizada por Ana" |
| **Issue lifecycle** | Issue created, then resolved | Timeline shows two events: addition then resolution |
| **Complete audit** | Auditor views timeline | Sees complete history from creation to current state |
| **Recent activity** | User checks timeline 10 minutes after change | Sees "há 10 minutos" for recent event |
| **Expand history** | User clicks "Ver mais" | All events load, extending the visible list |
| **Blocked approval** | Attempt to approve with blocking | Timeline records "Tentativa de aprovação bloqueada devido a Fiscal" |

### Failure Examples ❌

| Scenario | Incorrect Behavior | Why It's Wrong |
|----------|-------------------|----------------|
| **Missing event** | User changes status but no timeline entry | Audit trail is incomplete |
| **Wrong user** | Event shows "System" when João made the change | Accountability is lost |
| **Wrong timestamp** | Event shows yesterday but happened today | Temporal accuracy is critical |
| **Events edited** | Administrator modifies event description | Immutability violated |
| **Events deleted** | Old events disappear from timeline | Audit trail tampered with |
| **Out of order** | Events display in random order | Chronology is essential |
| **Missing details** | Status change shows no previous/new status | Context is lost |
| **Truncated permanently** | Only 10 events ever visible, no expand option | History is inaccessible |
| **English descriptions** | Events show "Request created" | Interface must be Portuguese |
| **Timezone errors** | Event shows wrong time due to timezone confusion | Accuracy compromised |

---

## Edge Cases and Boundary Conditions

### Volume Scenarios

| Scenario | Expected Behavior |
|----------|------------------|
| Request with 1 event (just created) | Shows single event, no "Ver mais" |
| Request with 10 events | Shows all 10, no "Ver mais" needed |
| Request with 11+ events | Shows 10, "Ver mais" available |
| Request with 100+ events | Paginate or lazy-load for performance |
| Very long event description | Wrap text, don't truncate meaning |

### Timing Scenarios

| Scenario | Expected Behavior |
|----------|------------------|
| Event just now | "agora mesmo" or "há menos de 1 minuto" |
| Event 5 minutes ago | "há 5 minutos" |
| Event 2 hours ago | "há 2 horas" |
| Event yesterday | "ontem às 14:32" |
| Event 7 days ago | "há 7 dias" or "08/01/2024" |
| Event months ago | "15/10/2023 às 09:15" |

### Concurrent Events

| Scenario | Expected Behavior |
|----------|------------------|
| Two events at same second | Both recorded, order may be arbitrary |
| Rapid successive changes | All changes recorded individually |
| Bulk operations (future) | Each action recorded separately |

### User Attribution

| Scenario | User Field Value |
|----------|-----------------|
| Known user makes change | User's name |
| User info unavailable | "Utilizador desconhecido" or omit field |
| System-generated event | "Sistema" or omit field |
| MVP (no auth) | Whatever user info is available, or omit |

---

## User Interface Guidelines

### Timeline Display Layout

```
┌──────────────────────────────────────────────────────────────────┐
│ Atividade Recente                                                │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│ ⚖️ Decisão registada: Aprovado por João Silva                    │
│    há 2 horas                                                    │
│                                                                  │
│ ✏️ Área Jurídico atualizada por Ana Costa                        │
│    há 4 horas                                                    │
│                                                                  │
│ ✅ Ponto em aberto resolvido: Cláusula de rescisão               │
│    ontem às 16:45                                                │
│                                                                  │
│ ⚠️ Ponto em aberto adicionado: Cláusula de rescisão              │
│    15/01/2024 às 10:30                                           │
│                                                                  │
│ 🔄 Estado alterado de Novo para Em Análise                       │
│    14/01/2024 às 09:15                                           │
│                                                                  │
│ ➕ Pedido criado por Maria Santos                                 │
│    14/01/2024 às 08:50                                           │
│                                                                  │
│                        [Ver mais ↓]                              │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Styling Guidelines

| Element | Style |
|---------|-------|
| Container | Light background, subtle border |
| Event icon | Colored appropriately to event type |
| Event description | Normal weight, dark text |
| Timestamp | Smaller size, muted/gray color |
| Spacing | Clear separation between events |
| "Ver mais" | Link style, centered |

### Responsive Behavior

| Viewport | Behavior |
|----------|----------|
| Desktop | Full width within content area |
| Tablet | Full width, may stack elements |
| Mobile | Full width, compact but readable |

---

## Definition of Done

This feature is complete when:

- [ ] **Event Recording**: All 11 event types are recorded correctly
- [ ] **Automatic Triggers**: Events are created automatically by their source features
- [ ] **Event Data**: All events have ID, type, description, timestamp, and metadata
- [ ] **Immutability**: Events cannot be modified or deleted
- [ ] **Display**: Timeline renders in General Information tab
- [ ] **Ordering**: Events display newest first
- [ ] **Pagination**: Default 10 events with "Ver mais" expansion
- [ ] **Descriptions**: All descriptions in Portuguese with proper grammar
- [ ] **Timestamps**: Relative and absolute formats display correctly
- [ ] **Icons**: Each event type has appropriate icon
- [ ] **Performance**: Timeline loads quickly even with many events
- [ ] **Portuguese**: All labels and text in Portuguese (Portugal)
- [ ] **Responsive**: Works on desktop and tablet
- [ ] **Accessible**: Screen readers can navigate timeline
- [ ] **Tests**: Unit tests for event creation, integration tests for recording
- [ ] **Code Review**: Approved by team member

---

## Integration Points

The Timeline is a cross-cutting feature that integrates with all other features:

| Feature | Creates Events |
|---------|---------------|
| [Create Request](../create-request/SPEC.md) | `request_created` |
| [Request Detail](../request-detail/SPEC.md) | `status_changed` |
| [Area Analysis](../area-analysis/SPEC.md) | `area_updated` |
| [Open Issues](../open-issues/SPEC.md) | `issue_added`, `issue_resolved`, `issue_deleted` |
| [Documents](../documents/SPEC.md) | `document_added`, `document_removed` |
| [Final Decision](../final-decision/SPEC.md) | `decision_recorded`, `summary_generated` |
| [Blocking Alerts](../blocking-alerts/SPEC.md) | `approval_blocked` |

---

## Technical Notes

> These notes are for developer reference. Non-technical stakeholders may skip this section.

- Timeline events are stored as an array within the Request object
- Events should be appended, never mutated or removed
- Consider a `TimelineService` with methods like `addEvent(requestId, type, description, metadata)`
- All event creation should happen in service layer, not in components
- Timestamps must use server time, not client time
- For MVP, user attribution may be limited since there's no authentication
- Consider using Intl.RelativeTimeFormat for relative timestamps
- Virtualization may be needed for requests with very long timelines
- Events should be created within the same transaction as the action they record
- Timeline component should be a Client Component for dynamic time updates

### Event Creation Helper

```typescript
// Example interface for timeline service
interface TimelineService {
  addEvent(
    requestId: RequestId,
    type: EventType,
    description: string,
    metadata?: Record<string, unknown>,
    user?: string
  ): Promise<TimelineEvent>;
}
```

---

## Future Enhancements (Out of Scope)

These may be added in future versions:

- Real-time updates (events appear without refresh)
- Event filtering by type or date range
- Event search functionality
- Event export for compliance reports
- User mentions and notifications based on events
- Integration with external audit systems
- Event analytics (most common events, peak activity times)
- Undo capability based on event history

---

*Specification Version: 1.0*

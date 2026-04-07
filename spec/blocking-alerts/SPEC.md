# Feature Specification: Blocking Alerts

> **Feature ID**: `blocking-alerts`  
> **Priority**: P2 — Critical Safety Feature  
> **Parent Document**: [SPEC.md](../../SPEC.md)

---

## Description

The Blocking Alerts feature is the primary safety mechanism of the Contract & Project Analysis Tracking System. When any reviewing department marks their decision as "Blocking," the system must prominently alert all users that a critical concern exists which prevents approval. This feature ensures that no contract or project proceeds to approval while unresolved blocking concerns remain.

This is arguably the **most important feature** in the entire system. The core purpose of multi-department review is to catch problems before they become costly mistakes. If a blocking concern can be overlooked or bypassed, the entire system fails to deliver its primary value.

### Business Context

When a department marks a request as "Blocking," they are signaling:
- A critical problem exists that cannot be ignored
- Proceeding with approval would expose the company to unacceptable risk
- The issue must be addressed before any further progress

**Real-world examples of blocking concerns:**
- Legal discovers a liability clause that could cost millions
- Data Protection identifies GDPR violations that could result in regulatory fines
- Cybersecurity finds vulnerabilities that could lead to data breaches
- Tax identifies fiscal structures that could trigger audits or penalties

Before this system, such critical concerns could be lost in email threads or overlooked in the rush to close deals. The Blocking Alerts feature makes it impossible to miss these critical signals.

### User Stories

- As a **manager**, I need to immediately see when a critical concern has been raised so I can take action
- As a **request owner**, I need to be prevented from accidentally approving a blocked request
- As a **department reviewer**, I need confidence that my blocking concern will be seen and addressed
- As a **compliance officer**, I need assurance that critical concerns cannot be bypassed

### Interface Language

All visible text, labels, and messages must be in **Portuguese (Portugal)**.

---

## What Triggers a Blocking Alert

A blocking alert is triggered when **any** area analysis has:

```
decision === 'blocking'
```

The blocking alert system must monitor all five areas:
- Legal (Jurídico)
- Data Protection (Proteção de Dados)
- Tax (Fiscal)
- Cybersecurity (Cibersegurança)
- Business (Negócio)

### Alert Activation Rules

| Condition | Alert Status |
|-----------|--------------|
| No areas have blocking decisions | Alert NOT shown |
| One area has blocking decision | Alert shown, naming that area |
| Multiple areas have blocking decisions | Alert shown, naming ALL blocking areas |
| Area changes from blocking to another decision | Alert disappears (if no other blocking exists) |
| Request is closed or archived | Alert still shown if blocking exists |

---

## Functional Requirements

### FR-1: Alert Banner Display

When blocking exists, display a prominent alert banner at the top of the request detail view.

**Requirements:**
- Position: Immediately below the header, spanning full width
- Visibility: Cannot be dismissed, hidden, or collapsed while blocking exists
- Style: High-contrast danger/error styling (red background, white or light text)
- Icon: Warning icon (⛔ or equivalent)
- Persistence: Remains visible regardless of which tab is active

**Content:**
- Single area blocking: "⛔ BLOQUEIO: [Area Name] registou uma preocupação bloqueante"
- Multiple areas blocking: "⛔ BLOQUEIO: [Area 1], [Area 2] registaram preocupações bloqueantes"

### FR-2: Alert Linking

The alert should provide direct navigation to the blocking area(s).

**Requirements:**
- Each area name in the alert should be a clickable link
- Clicking the link navigates to the Areas tab and scrolls to that area's card
- The blocking area card should be highlighted momentarily after navigation

### FR-3: Status Change Protection

The system must prevent or warn against approving a request with blocking concerns.

**When user attempts to set status to "Approved" or "Approved with Conditions" while blocking exists:**

1. Display a blocking dialog (modal):
   - Title: "Aprovação Bloqueada"
   - Message: "Não é possível aprovar este pedido. Existem preocupações bloqueantes por resolver."
   - List: Show all areas with blocking decisions
   - Actions: 
     - "Ver Áreas" — Navigate to Areas tab
     - "Fechar" — Close dialog (do NOT change status)

2. Do NOT allow the status change to proceed

3. Log the attempt in the timeline: "Approval attempted but blocked due to [Area Name] concerns"

### FR-4: Visual Indicators on Area Cards

Areas with blocking decisions must be visually distinct.

**Requirements:**
- Blocking area cards have red/danger border or background
- Decision indicator shows red icon and "Bloqueante" text
- The visual distinction is immediately apparent without reading text

### FR-5: Dashboard Awareness (Optional Enhancement)

Requests with blocking decisions should be indicated on the dashboard.

**Requirements:**
- Request card shows additional warning indicator when blocking exists
- Consider: "⛔ Bloqueio" badge alongside other badges
- This is supplementary to the main alert on the detail view

### FR-6: Alert Removal

The alert must automatically disappear when blocking is resolved.

**Requirements:**
- When the last blocking decision is changed to another option, alert disappears
- No manual dismissal is required or allowed
- Page does not require refresh — update should be immediate after save

---

## Alert States

### State 1: No Blocking (Normal State)

```
┌─────────────────────────────────────────────────────────────┐
│ Dashboard > Contrato de Outsourcing IT — Acme Corp         │
├─────────────────────────────────────────────────────────────┤
│ Contrato de Outsourcing IT — Acme Corp                     │
│ [Contrato] [Em Análise] [Alta]                              │
│ ...                                                         │
```

No alert banner is shown. Normal request detail view.

### State 2: Single Area Blocking

```
┌─────────────────────────────────────────────────────────────┐
│ Dashboard > Contrato de Outsourcing IT — Acme Corp         │
├─────────────────────────────────────────────────────────────┤
│ ⛔ BLOQUEIO: Fiscal registou uma preocupação bloqueante     │
│                                              [Ver Área →]   │
├─────────────────────────────────────────────────────────────┤
│ Contrato de Outsourcing IT — Acme Corp                     │
│ [Contrato] [Em Análise] [Alta]                              │
│ ...                                                         │
```

### State 3: Multiple Areas Blocking

```
┌─────────────────────────────────────────────────────────────┐
│ Dashboard > Contrato de Outsourcing IT — Acme Corp         │
├─────────────────────────────────────────────────────────────┤
│ ⛔ BLOQUEIO: Jurídico, Cibersegurança registaram            │
│             preocupações bloqueantes           [Ver Áreas →]│
├─────────────────────────────────────────────────────────────┤
│ Contrato de Outsourcing IT — Acme Corp                     │
│ [Contrato] [Em Análise] [Alta]                              │
│ ...                                                         │
```

### State 4: Approval Blocked Dialog

```
┌─────────────────────────────────────────────────────────────┐
│                    Aprovação Bloqueada                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ⛔ Não é possível aprovar este pedido.                    │
│                                                             │
│   Existem preocupações bloqueantes por resolver:            │
│                                                             │
│   • Fiscal                                                  │
│   • Cibersegurança                                          │
│                                                             │
│   Resolva estas preocupações antes de aprovar.             │
│                                                             │
│                     [Ver Áreas]  [Fechar]                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Acceptance Criteria

### AC-1: Alert Appearance
- [ ] Alert banner appears when any area has blocking decision
- [ ] Alert spans full width below header
- [ ] Alert has high-contrast danger styling (red background)
- [ ] Alert includes warning icon (⛔ or equivalent)
- [ ] Alert cannot be dismissed while blocking exists

### AC-2: Alert Content
- [ ] Single blocking area shows correct Portuguese message with area name
- [ ] Multiple blocking areas lists all area names
- [ ] Area names use correct Portuguese labels
- [ ] Grammar is correct (singular vs. plural verb forms)

### AC-3: Alert Linking
- [ ] Area names in alert are clickable
- [ ] Clicking navigates to Areas tab
- [ ] View scrolls to the relevant area card
- [ ] Target area card is highlighted briefly

### AC-4: Status Protection
- [ ] Attempting to approve shows blocking dialog
- [ ] Dialog lists all blocking areas
- [ ] Status does NOT change when blocking exists
- [ ] "Ver Áreas" button navigates to Areas tab
- [ ] "Fechar" button closes dialog without changes
- [ ] Attempt is logged in timeline

### AC-5: Area Card Indicators
- [ ] Blocking area cards have distinctive red styling
- [ ] Decision shows "Bloqueante" with red indicator
- [ ] Visual distinction is clear without reading text

### AC-6: Alert Removal
- [ ] Alert disappears when blocking decision is changed
- [ ] Disappearance is immediate (no page refresh needed)
- [ ] Status change becomes possible after blocking resolved

### AC-7: Multi-Area Consistency
- [ ] System correctly tracks blocking across all 5 areas
- [ ] Adding second blocking area updates alert to show both
- [ ] Resolving one of two blocking areas updates alert to show remaining one
- [ ] Resolving all blocking removes alert entirely

---

## Success and Failure Examples

### Success Examples ✅

| Scenario | System Behavior | Expected Outcome |
|----------|-----------------|------------------|
| **Tax marks blocking** | User selects "Bloqueante" for Tax area and saves | Red alert banner appears: "⛔ BLOQUEIO: Fiscal registou uma preocupação bloqueante" |
| **Second area blocks** | Legal also marks blocking while Tax is already blocking | Alert updates: "⛔ BLOQUEIO: Jurídico, Fiscal registaram preocupações bloqueantes" |
| **Attempt approval** | Manager tries to change status to "Approved" | Modal appears preventing approval, listing both blocking areas |
| **Navigate to blocking area** | User clicks area name in alert | Page switches to Areas tab, scrolls to that area, area card flashes to draw attention |
| **Resolve one blocker** | Tax changes decision from "Bloqueante" to "Com Reservas" | Alert updates to show only Legal as blocking |
| **Resolve all blockers** | Legal changes decision from "Bloqueante" to "Sem Objeções" | Alert disappears entirely, approval becomes possible |
| **Tab persistence** | User views Issues tab while blocking exists | Alert remains visible at top, not hidden by tab content |
| **Refresh persistence** | User refreshes page with blocking request | Alert reappears immediately based on current data |

### Failure Examples ❌

| Scenario | Incorrect Behavior | Why It's Critical |
|----------|-------------------|-------------------|
| **Missing alert** | Tax marks blocking but no alert appears | Users might approve a blocked request unknowingly |
| **Dismissible alert** | User clicks X and alert disappears | Blocking concern becomes hidden, defeating the purpose |
| **Approval succeeds** | User changes status to "Approved" despite blocking | The core safety feature has failed completely |
| **Partial listing** | Two areas are blocking but alert only mentions one | Second blocking concern could be overlooked |
| **Wrong area named** | Tax is blocking but alert says "Legal" | User addresses wrong department, real concern remains |
| **Alert on wrong request** | Request A has blocking, user views Request B | Confusion about which request is actually blocked |
| **Stale alert** | Tax resolves blocking but alert persists | User might think approval is still blocked |
| **No navigation** | User clicks area name, nothing happens | Cannot quickly find and address the blocking concern |
| **Status changes anyway** | Dialog appears but clicking "Fechar" approves request | Dialog provides false sense of protection |
| **Timeline missing** | Approval attempt occurs but not logged | Audit trail is incomplete |

---

## Edge Cases and Boundary Conditions

### Timing Scenarios

| Scenario | Expected Behavior |
|----------|------------------|
| Blocking set during page load | Alert appears once page fully loads |
| Blocking changed by another user | Alert state may be stale until refresh (MVP limitation) |
| Rapid blocking/unblocking | System reflects most recent saved state |
| Blocking set then immediately unset | If final state is not blocking, no alert |

### Request State Scenarios

| Request State | Blocking Alert Behavior |
|---------------|------------------------|
| New request | Can have blocking (unlikely but possible) |
| In Analysis | Normal blocking behavior |
| Approved (historical) | Alert shows if blocking exists (data inconsistency) |
| Closed/Archived | Alert shows if blocking exists |
| Deleted (soft delete) | Not visible, alert not relevant |

### Area Decision Scenarios

| Scenario | Alert Status |
|----------|--------------|
| 0 of 5 areas have decisions | No alert |
| 5 areas with "No Objections" | No alert |
| 4 "No Objections" + 1 "Blocking" | Alert shows the one |
| 3 "Reservations" + 2 "Blocking" | Alert shows the two |
| 5 areas all "Blocking" | Alert shows all five |
| Mixed: some complete, some not started, one blocking | Alert shows the one blocking |

### Grammar Considerations (Portuguese)

| Count | Verb Form | Example |
|-------|-----------|---------|
| 1 area | Singular | "Fiscal **registou** uma preocupação bloqueante" |
| 2+ areas | Plural | "Fiscal, Jurídico **registaram** preocupações bloqueantes" |

---

## User Interface Guidelines

### Alert Banner Styling

| Property | Value | Rationale |
|----------|-------|-----------|
| Background | `red-600` or `bg-destructive` | High visibility, danger signal |
| Text | White (`white`) | Contrast against red background |
| Icon | ⛔ or `AlertTriangle` | Universal warning symbol |
| Padding | Comfortable (py-3 px-4) | Easy to read |
| Font weight | Semi-bold or bold | Emphasis |
| Position | Sticky below header | Always visible while scrolling |

### Area Card Blocking Styling

| Property | Normal Card | Blocking Card |
|----------|-------------|---------------|
| Border | Gray or transparent | Red-500 (2px solid) |
| Background | White | Red-50 or subtle red tint |
| Decision badge | Green/Amber/Gray | Red-600 with white text |
| Overall impression | Neutral | Clearly problematic |

### Modal Styling

| Property | Value |
|----------|-------|
| Width | 400-500px, centered |
| Background | White with overlay behind |
| Header | "Aprovação Bloqueada" with warning icon |
| List style | Bullet points for blocking areas |
| Buttons | "Ver Áreas" (primary), "Fechar" (secondary) |

---

## Definition of Done

This feature is complete when:

- [ ] **Alert Display**: Banner appears immediately when any area has blocking decision
- [ ] **Alert Styling**: Uses high-contrast red/danger theme, cannot be dismissed
- [ ] **Alert Content**: Correctly lists all blocking areas with proper Portuguese grammar
- [ ] **Alert Linking**: Clicking area name navigates to Areas tab and highlights the card
- [ ] **Status Protection**: Cannot approve while blocking exists
- [ ] **Blocking Dialog**: Modal prevents approval and lists all blocking areas
- [ ] **Timeline Logging**: Blocked approval attempts are recorded
- [ ] **Area Indicators**: Blocking areas have distinctive red styling
- [ ] **Real-time Updates**: Alert appears/disappears immediately after save (no refresh needed)
- [ ] **Multi-area Support**: Correctly handles 1, 2, 3, 4, or 5 areas blocking simultaneously
- [ ] **Tab Persistence**: Alert remains visible across all tabs
- [ ] **Responsive**: Works on desktop and tablet viewports
- [ ] **Accessible**: Alert is announced to screen readers, modal is keyboard accessible
- [ ] **Portuguese**: All text in Portuguese (Portugal)
- [ ] **Code Quality**: Passes linting, no TypeScript errors
- [ ] **Tests**: Unit tests for blocking detection logic, integration test for status protection
- [ ] **Review**: Code reviewed with explicit verification of safety mechanism

---

## Testing Requirements

Given the critical nature of this feature, additional testing rigor is required:

### Required Test Cases

1. **Single blocking detection**: Set one area to blocking, verify alert appears
2. **Multiple blocking detection**: Set two areas to blocking, verify both listed
3. **Blocking resolution**: Resolve blocking, verify alert disappears
4. **Approval prevention**: Attempt approval with blocking, verify blocked
5. **All five areas**: Test with each area being the blocking one
6. **Grammar**: Verify singular/plural Portuguese verb forms
7. **Navigation**: Click area link, verify scroll and highlight
8. **Edge case**: Blocking on closed request

### Manual QA Checklist

- [ ] Visually verify alert is impossible to miss
- [ ] Verify alert cannot be dismissed by any user action
- [ ] Attempt to bypass approval prevention (click fast, keyboard shortcuts, etc.)
- [ ] Verify status is truly unchanged after blocked approval attempt
- [ ] Check timeline for logged approval attempt

---

## Related Features

- **[Request Detail](../request-detail/SPEC.md)**: Container where alert is displayed
- **[Area Analysis](../area-analysis/SPEC.md)**: Source of blocking decisions
- **[Final Decision](../final-decision/SPEC.md)**: Affected by blocking state
- **[Timeline](../timeline/SPEC.md)**: Records blocked approval attempts
- **[Dashboard](../dashboard/SPEC.md)**: May show blocking indicator on cards

---

## Technical Notes

> These notes are for developer reference. Non-technical stakeholders may skip this section.

- Blocking detection: `request.areas.filter(a => a.decision === 'blocking')`
- Consider a custom hook: `useBlockingAreas(request)` returning array of blocking areas
- Alert component should be rendered at Request Detail layout level, not within tabs
- Status change validation should happen both client-side (immediate feedback) and server-side (authoritative)
- Timeline event for blocked attempts should include which areas were blocking
- Consider adding `data-testid` attributes for critical elements to enable reliable E2E testing
- Modal should trap focus and be closable with Escape key
- Screen readers should announce alert when it appears (use `role="alert"` or `aria-live`)

---

## Security Considerations

This feature has security/compliance implications:

- **Audit Trail**: All approval attempts (successful or blocked) must be logged
- **Server Enforcement**: Status change validation must happen server-side, not just client-side
- **No Workarounds**: Ensure no API endpoint allows bypassing the blocking check
- **Data Integrity**: Blocking decisions should only be changeable through the proper Area Analysis flow

---

*Specification Version: 1.0*

# Feature Specification: Dashboard

> **Feature ID**: FEAT-001  
> **Priority**: P1 — Core Feature  
> **Parent Document**: [SPEC.md](../../SPEC.md)

---

## Description

The Dashboard is the primary entry point to the application. It provides users with a comprehensive overview of all active contract and project analysis requests in a single, scannable view. Users can quickly assess the current state of all ongoing analyses, identify requests that need attention, and navigate to specific requests for detailed work.

The Dashboard serves three main purposes:
1. **Visibility** — Show all requests at a glance with key status information
2. **Discovery** — Help users find specific requests through search and filters
3. **Navigation** — Provide quick access to request details and request creation

### User Stories

- As an **internal staff member**, I want to see all my pending requests so I can track their progress
- As a **department reviewer**, I want to filter requests by status so I can focus on those needing my attention
- As a **manager**, I want to see requests with issues highlighted so I can prioritize my oversight
- As a **new user**, I want to understand what requests exist without clicking into each one

### Interface Language

All visible text, labels, and messages must be in **Portuguese (Portugal)**.

---

## Functional Requirements

### FR-1: Request List Display

The dashboard must display all active (non-deleted) requests in a card or list format.

**Each request card must show:**
- Request title (prominently displayed)
- Request type badge (Contract or Project)
- Current status badge (with appropriate color coding)
- Criticality indicator (Low, Medium, High)
- Internal owner name
- Creation date (formatted as DD/MM/YYYY)
- Progress indicator showing X of 5 areas completed (visual bar + text)
- Open issues count with warning indicator if > 0

**Card behavior:**
- The entire card must be clickable, navigating to the request detail view
- Hover state must provide visual feedback indicating the card is interactive
- Cards must maintain consistent height for visual alignment

### FR-2: Filtering Capabilities

Users must be able to filter the displayed requests by:

| Filter | Options | Behavior |
|--------|---------|----------|
| Status | New, In Analysis, Awaiting Info, Has Issues, Approved, Approved w/ Conditions, Rejected, Closed | Multiple selection allowed |
| Type | Contract, Project | Single or multiple selection |
| Criticality | Low, Medium, High | Single or multiple selection |

**Filter behavior:**
- Filters must be combinable (AND logic between different filter types)
- Active filters must be visually indicated
- A "Clear filters" action must be available when any filter is active
- Filter state should persist during the session (not across page refreshes in MVP)

### FR-3: Search Functionality

Users must be able to search for requests by text.

**Search must match against:**
- Request title
- Supplier name
- Internal owner name

**Search behavior:**
- Search must be case-insensitive
- Search must match partial strings (e.g., "Acm" matches "Acme Corp")
- Search results must update as the user types (debounced, 300ms delay)
- When no results match, display a friendly "No results found" message with the search term shown
- Search can be combined with filters

### FR-4: Sorting

Requests must be sortable by creation date.

**Default sort:** Newest first (descending by creation date)

**Future consideration:** Additional sort options (by status, by criticality) may be added but are not required for MVP.

### FR-5: Empty States

The dashboard must handle empty states gracefully:

| Condition | Message (Portuguese) | Additional UI |
|-----------|---------------------|---------------|
| No requests exist | "Não existem pedidos registados. Crie o primeiro pedido para começar." | Show prominent "Create Request" button |
| No results match filters/search | "Nenhum pedido encontrado para os critérios selecionados." | Show "Clear filters" link |

### FR-6: Request Count

Display a count of visible requests: "X pedidos encontrados" (X requests found)

This count must update dynamically as filters and search are applied.

### FR-7: Create Request Action

A prominent "Novo Pedido" (New Request) button must be visible at all times, navigating to the Create Request form.

**Button placement:** Top-right of the dashboard, always visible (not scrolling with content)

---

## Non-Functional Requirements

### NFR-1: Performance

- Dashboard must load and display all requests within 1 second
- Filter and search operations must respond within 200ms
- Smooth scrolling with large numbers of requests (tested with 100+ requests)

### NFR-2: Responsiveness

- Desktop (≥1024px): Display requests in a grid (3-4 columns)
- Tablet (768-1023px): Display requests in a grid (2 columns)
- Mobile (<768px): Display requests in a single column list

### NFR-3: Accessibility

- All interactive elements must be keyboard accessible
- Color coding must not be the only indicator of status (include text labels)
- Sufficient color contrast for all text (WCAG AA minimum)

---

## Acceptance Criteria

### AC-1: Request List
- [ ] All active requests are displayed on page load
- [ ] Deleted (soft-deleted) requests are not shown
- [ ] Each card displays all required information (title, type, status, criticality, owner, date, progress, issues)
- [ ] Progress bar accurately reflects completed areas / total areas
- [ ] Open issues count displays correctly with warning styling when > 0

### AC-2: Navigation
- [ ] Clicking any request card navigates to `/pedidos/[id]`
- [ ] Clicking "Novo Pedido" navigates to `/pedidos/novo`
- [ ] Navigation is responsive (no delay or double-clicks required)

### AC-3: Filtering
- [ ] Status filter shows all 8 status options
- [ ] Type filter shows Contract and Project options
- [ ] Criticality filter shows Low, Medium, High options
- [ ] Multiple filters can be active simultaneously
- [ ] Filter results update immediately
- [ ] Clear filters resets all filters and shows all requests

### AC-4: Search
- [ ] Search field is visible and clearly labeled
- [ ] Typing in search filters results in real-time
- [ ] Search matches title, supplier, and owner fields
- [ ] Search is case-insensitive
- [ ] Empty search results show appropriate message

### AC-5: Empty States
- [ ] New system with no requests shows welcome message and create button
- [ ] No matching results shows clear message about criteria
- [ ] Empty states do not break the page layout

### AC-6: Request Count
- [ ] Count displays total number of visible requests
- [ ] Count updates when filters or search change
- [ ] Count uses correct Portuguese grammar (singular/plural)

---

## Success and Failure Examples

### Success Examples ✅

| Scenario | User Action | Expected Result |
|----------|-------------|-----------------|
| View all requests | User opens the dashboard | All active requests appear as cards, sorted by newest first, within 1 second |
| Search by supplier | User types "Acme" in search field | Only requests with "Acme" in title, supplier, or owner are shown; count updates to reflect filtered results |
| Filter by status | User selects "Has Issues" status filter | Only requests with status "Has Issues" appear; filter badge shows as active |
| Combine search and filter | User searches "Contract" and filters by "High" criticality | Only high-criticality requests containing "Contract" appear |
| Clear all filters | User clicks "Limpar filtros" | All filters reset, all requests reappear, search field clears |
| Navigate to request | User clicks on a request card | Browser navigates to `/pedidos/[id]` showing the request detail view |
| Create new request | User clicks "Novo Pedido" button | Browser navigates to `/pedidos/novo` showing the create form |
| View empty system | New user opens dashboard with no requests | Friendly message appears with prominent create button |
| Identify issues quickly | User scans dashboard | Requests with open issues show warning badge (e.g., "⚠️ 3 pontos em aberto") |
| Check progress | User looks at a request card | Progress bar shows "2/5 áreas concluídas" with 40% visual fill |

### Failure Examples ❌

| Scenario | User Action | Incorrect Result | Why It's Wrong |
|----------|-------------|------------------|----------------|
| Search with typo | User searches "contrat" (typo) | Page shows no results with no guidance | Should show "No results" message with suggestion to check spelling |
| Slow loading | User opens dashboard | Page takes 5+ seconds to show requests | Users will think the system is broken; must load within 1 second |
| Click unresponsive | User clicks request card | Nothing happens, no navigation | Cards must always respond to clicks; if loading, show indicator |
| Deleted requests appear | User opens dashboard | A deleted request still appears in the list | Soft-deleted requests must be filtered out |
| Filter doesn't work | User selects "Approved" status | All requests still show, including non-approved | Filter logic is broken; only filtered results should appear |
| Misleading count | 15 requests exist, filter shows 3 | Count still says "15 pedidos encontrados" | Count must reflect currently visible (filtered) requests |
| Missing progress | User views request card | No progress indicator visible | Progress is critical information; must always display |
| Status colors only | User is colorblind | Cannot distinguish approved from rejected | Status must include text labels, not just colors |
| Broken empty state | No requests exist | Page shows error or blank white screen | Empty state must show helpful message and create action |
| Filters persist incorrectly | User navigates away and returns | Old filters still active unexpectedly | In MVP, filters should reset on page reload for simplicity |

---

## Visual Design Guidelines

### Status Badge Colors

| Status | Background | Text | Portuguese Label |
|--------|------------|------|------------------|
| New | Blue-100 | Blue-700 | Novo |
| In Analysis | Amber-100 | Amber-700 | Em Análise |
| Awaiting Info | Orange-100 | Orange-700 | Aguarda Informação |
| Has Issues | Red-100 | Red-700 | Com Pontos em Aberto |
| Approved | Green-100 | Green-700 | Aprovado |
| Approved w/ Conditions | Teal-100 | Teal-700 | Aprovado c/ Condições |
| Rejected | Red-200 | Red-800 | Rejeitado |
| Closed | Gray-100 | Gray-500 | Fechado |

### Criticality Indicators

| Criticality | Color | Portuguese Label |
|-------------|-------|------------------|
| High | Red-500 | Alta |
| Medium | Amber-500 | Média |
| Low | Green-500 | Baixa |

### Type Badges

| Type | Style | Portuguese Label |
|------|-------|------------------|
| Contract | Outlined, neutral | Contrato |
| Project | Filled, neutral | Projeto |

---

## Definition of Done

This feature is complete when:

- [ ] **Functionality**: All functional requirements (FR-1 through FR-7) are implemented and working
- [ ] **Acceptance Criteria**: All acceptance criteria (AC-1 through AC-6) pass testing
- [ ] **Responsiveness**: Dashboard displays correctly on desktop, tablet, and mobile viewports
- [ ] **Performance**: Page loads within 1 second with mock data (50+ requests)
- [ ] **Accessibility**: All interactive elements are keyboard accessible
- [ ] **Empty States**: Both empty state scenarios display correctly
- [ ] **Portuguese**: All user-facing text is in Portuguese (Portugal)
- [ ] **Error Handling**: Network errors show user-friendly error messages
- [ ] **Code Quality**: Code passes linting and has no TypeScript errors
- [ ] **Testing**: Unit tests exist for filter logic and search functionality
- [ ] **Review**: Code reviewed and approved by at least one team member

---

## Dependencies

- **Data Layer**: Requires `requestService.findAll()` to retrieve all requests
- **Routing**: Requires Next.js App Router configuration for `/` route
- **Components**: Requires base UI components from shadcn/ui (Card, Badge, Input, Button)
- **Types**: Requires `Request`, `RequestStatus`, `Criticality`, `RequestType` types defined

---

## Out of Scope for This Feature

- Pagination (MVP assumes manageable number of requests)
- Saved filter presets
- Export functionality
- Advanced sorting options beyond creation date
- Real-time updates (no WebSocket/polling)
- Drag-and-drop reordering
- Bulk actions on multiple requests

---

*Specification Version: 1.0*

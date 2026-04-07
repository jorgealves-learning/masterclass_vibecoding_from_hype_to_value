# Feature Specification: Create Request

> **Feature ID**: `create-request`  
> **Priority**: P1 — Core Feature  
> **Parent Document**: [SPEC.md](../../SPEC.md)

---

## Description

The Create Request feature allows internal staff members to initiate a new contract or project analysis request. This is the primary entry point for all work in the system. When a user creates a request, they provide essential information about what needs to be analyzed, and the system initializes all necessary structures for the five reviewing departments to begin their assessments.

### Business Context

Before this system, employees would send emails to various departments or fill out spreadsheets to request analysis of contracts and projects. This led to:
- Lost requests in email threads
- Inconsistent information gathering
- No standardized process
- Difficulty tracking who requested what and when

The Create Request form standardizes the intake process, ensuring all necessary information is captured upfront and creating a traceable record from the moment work begins.

### User Story

> As an **internal staff member**, I want to **submit a request for contract or project analysis** so that **all relevant departments can review it and provide their assessments in a coordinated manner**.

---

## Detailed Requirements

### Form Fields

| Field | Label (Portuguese) | Type | Required | Constraints |
|-------|-------------------|------|----------|-------------|
| `title` | Título | Text input | Yes | Minimum 5 characters, maximum 200 characters |
| `description` | Descrição | Textarea | Yes | Minimum 20 characters, maximum 2000 characters |
| `type` | Tipo | Select dropdown | Yes | Values: "Contrato" (contract), "Projeto" (project) |
| `requestingArea` | Área Solicitante | Select dropdown | Yes | Predefined list of company departments |
| `internalOwner` | Responsável Interno | Text input | Yes | Minimum 3 characters, maximum 100 characters |
| `supplier` | Fornecedor | Text input | No | Maximum 200 characters |
| `criticality` | Criticidade | Radio buttons | Yes | Values: "Baixa" (low), "Média" (medium), "Alta" (high) |
| `countriesRegions` | Países/Regiões | Text input | No | Free text, comma-separated values accepted |
| `initialNotes` | Notas Iniciais | Textarea | No | Maximum 1000 characters |

### Requesting Area Options

The following departments should be available in the "Requesting Area" dropdown:
- Recursos Humanos (Human Resources)
- Financeiro (Finance)
- Operações (Operations)
- Comercial (Commercial/Sales)
- Marketing
- Tecnologia (Technology/IT)
- Administração (Administration)
- Outro (Other)

### System Behavior on Submission

When the user submits a valid form, the system must:

1. **Validate all fields** — Check required fields are filled and constraints are met
2. **Generate a unique identifier** — Create a new Request ID
3. **Set initial status** — Status becomes "New" (novo)
4. **Initialize area analyses** — Create five empty area analysis records (Legal, Data Protection, Tax, Cybersecurity, Business), each with status "Not Started"
5. **Record timestamps** — Set `createdAt` and `updatedAt` to current time
6. **Create timeline event** — Add a "request_created" event recording who created it and when
7. **Persist the data** — Save the complete request to storage
8. **Navigate to detail view** — Redirect user to the newly created request's detail page
9. **Show confirmation** — Display success message to user

### Form Behavior

- **Real-time validation**: Show validation errors as user types or when field loses focus
- **Portuguese error messages**: All validation messages must be in Portuguese
- **Preserve input on error**: If submission fails, do not clear the form
- **Cancel action**: Provide a way to abandon the form and return to dashboard
- **Unsaved changes warning**: If user tries to navigate away with data entered, show confirmation dialog

---

## Acceptance Criteria

### AC-1: Form Display
- [ ] Form displays all required fields with correct labels in Portuguese
- [ ] Required fields are clearly marked (e.g., with asterisk)
- [ ] Type field offers exactly two options: Contract and Project
- [ ] Criticality field displays three radio button options
- [ ] Requesting Area dropdown contains all predefined departments

### AC-2: Field Validation
- [ ] Title field rejects input shorter than 5 characters
- [ ] Title field rejects input longer than 200 characters
- [ ] Description field rejects input shorter than 20 characters
- [ ] Description field rejects input longer than 2000 characters
- [ ] Internal Owner field rejects input shorter than 3 characters
- [ ] All required fields must be filled before submission succeeds
- [ ] Validation errors appear next to the relevant field, not just at form top

### AC-3: Successful Submission
- [ ] Form submits when all required fields are valid
- [ ] User sees a confirmation message (e.g., "Pedido criado com sucesso")
- [ ] User is redirected to the new request's detail page
- [ ] New request appears on the dashboard with status "New"
- [ ] All five area analyses are initialized with "Not Started" status
- [ ] Timeline shows "Request created" as first event

### AC-4: Failed Submission Handling
- [ ] If network error occurs, user sees clear error message
- [ ] Form data is preserved after failed submission
- [ ] User can retry submission without re-entering data

### AC-5: Navigation and Cancellation
- [ ] Cancel button returns user to dashboard
- [ ] If form has data and user tries to leave, confirmation dialog appears
- [ ] Browser back button triggers unsaved changes warning if form has data

### AC-6: Accessibility and Usability
- [ ] Form can be completed using keyboard only
- [ ] Form fields have proper labels for screen readers
- [ ] Error states are announced to assistive technologies
- [ ] Submit button shows loading state during submission

---

## Success and Failure Examples

### Success Scenarios

| Scenario | User Action | Expected System Response | Why This is Success |
|----------|-------------|-------------------------|---------------------|
| **Complete valid submission** | Sofia fills all required fields correctly and clicks "Criar Pedido" | Form submits, confirmation toast appears, page redirects to new request detail view showing all entered information | The primary happy path works as intended |
| **Validation feedback helps user** | Carlos types "IT" in title field and tabs away | Field shows error "Título deve ter pelo menos 5 caracteres" immediately | User can correct mistake before attempting submission |
| **Network recovery** | Luísa submits form but network fails, then she retries | First attempt shows error "Falha ao guardar. Verifique a sua ligação.", form data remains, second attempt succeeds | System gracefully handles temporary failures |
| **Minimal required data** | João fills only required fields, leaving supplier and notes empty | Request is created successfully with optional fields empty | System doesn't force unnecessary data entry |
| **Maximum length input** | Ana writes a 2000-character description | Text is accepted, form submits successfully | Edge cases within limits work correctly |
| **Cancel without data** | Pedro opens form, enters nothing, clicks Cancel | Returns to dashboard immediately with no warning | No false positive on unsaved changes |
| **Unsaved changes protection** | Maria fills half the form, accidentally clicks browser back | Dialog appears: "Tem alterações não guardadas. Deseja sair?" with Stay/Leave options | Prevents accidental data loss |

### Failure Scenarios

| Scenario | What Goes Wrong | Why This is a Failure | How to Detect |
|----------|-----------------|----------------------|---------------|
| **Silent submission failure** | User clicks submit, nothing happens, no error shown | User doesn't know if request was created, may submit duplicates | No confirmation or error message appears after clicking submit |
| **Data loss on error** | Network fails during submission, form clears all fields | User must re-enter all information from scratch | Form is empty after seeing an error message |
| **Cryptic error message** | Validation fails with message "Error 422: Unprocessable entity" | User cannot understand what to fix | Error message contains technical jargon or codes |
| **Missing validation** | User submits with 3-character title, request is created | Invalid data enters the system, causing confusion later | Request exists with title shorter than 5 characters |
| **Broken cancel** | User clicks Cancel but nothing happens | User is trapped in form | Cancel button is unresponsive |
| **No loading indicator** | User clicks submit, button remains static for 3 seconds | User thinks nothing happened, clicks again, creates duplicates | No visual change after clicking submit button |
| **Wrong redirect** | Form submits successfully but redirects to dashboard instead of detail | User cannot verify their submission or continue working on it | After successful submission, user is not on the new request's page |
| **Area analyses not created** | Request is created but only 3 of 5 areas are initialized | Some departments cannot record their analysis | Request detail shows fewer than 5 area analysis cards |
| **Timeline not updated** | Request is created but timeline is empty | No audit trail of who created the request and when | Timeline section shows no events for a new request |
| **Duplicate submission** | Clicking submit twice creates two identical requests | Data integrity issue, confusing duplicates in system | Dashboard shows multiple requests with identical titles created at same time |

---

## Edge Cases and Boundary Conditions

### Input Boundaries

| Field | Boundary | Expected Behavior |
|-------|----------|------------------|
| Title | Exactly 5 characters | Accepted (minimum valid) |
| Title | Exactly 4 characters | Rejected with error message |
| Title | Exactly 200 characters | Accepted (maximum valid) |
| Title | 201 characters | Rejected or truncated with warning |
| Description | Exactly 20 characters | Accepted (minimum valid) |
| Description | Exactly 2000 characters | Accepted (maximum valid) |
| Supplier | Empty | Accepted (optional field) |
| Countries/Regions | Empty | Accepted (optional field) |

### Special Characters

| Input Type | Example | Expected Behavior |
|------------|---------|------------------|
| Portuguese characters | "Contrato de Prestação de Serviços" | Accepted, displayed correctly |
| Accented characters | "José António — Análise Prévia" | Accepted, stored and displayed correctly |
| Emoji | "Contract Review 📋" | Accepted or gracefully rejected |
| HTML tags | "<script>alert('test')</script>" | Sanitized, not executed |
| Very long word | "Antidisestablishmentarianism..." (50+ chars) | Displayed with word-wrap, not breaking layout |

### Concurrent Operations

| Scenario | Expected Behavior |
|----------|------------------|
| Two users create requests simultaneously | Both succeed with unique IDs |
| User double-clicks submit button | Only one request is created (debounce) |
| User submits, then immediately navigates away | Request is still created successfully |

---

## User Interface Guidelines

### Form Layout

The form should be presented as a single-page form with clear visual hierarchy:

1. **Header**: "Novo Pedido" (New Request)
2. **Required fields section**: Title, Description, Type, Requesting Area, Internal Owner, Criticality
3. **Optional fields section**: Supplier, Countries/Regions, Initial Notes
4. **Action buttons**: "Criar Pedido" (primary), "Cancelar" (secondary)

### Visual Indicators

- Required fields marked with red asterisk (*)
- Validation errors shown in red text below the field
- Valid fields may show green checkmark (optional enhancement)
- Submit button disabled while form is invalid (optional) or shows errors on click
- Loading spinner on submit button during submission

### Responsive Behavior

- Desktop: Two-column layout for shorter fields, full-width for textareas
- Tablet: Single-column layout with adequate touch targets
- Mobile: Functional but not optimized (per MVP scope)

---

## Definition of Done

This feature is considered complete when:

- [ ] **Form renders correctly** — All fields display with Portuguese labels, proper input types, and visual hierarchy
- [ ] **Validation works** — All field constraints are enforced with clear Portuguese error messages
- [ ] **Submission succeeds** — Valid form creates request with correct initial state (status: new, 5 areas initialized, timeline event recorded)
- [ ] **Confirmation shown** — User sees clear success message after submission
- [ ] **Navigation works** — User is redirected to new request detail page after success
- [ ] **Error handling works** — Network failures show clear error, preserve form data
- [ ] **Cancel works** — Cancel button returns to dashboard
- [ ] **Unsaved changes protected** — Navigation away triggers confirmation if form has data
- [ ] **Data persists** — Created request appears on dashboard after page refresh
- [ ] **Keyboard accessible** — Entire form can be completed without mouse
- [ ] **No duplicate submissions** — Double-clicking submit creates only one request
- [ ] **Code reviewed** — Implementation reviewed and approved by another developer
- [ ] **Tests pass** — Unit tests for validation logic, integration test for form submission flow

---

## Related Features

- **[Dashboard](../dashboard/SPEC.md)** — Where users navigate from and where new requests appear
- **[Request Detail](../request-detail/SPEC.md)** — Where users are redirected after creation
- **[Timeline](../timeline/SPEC.md)** — Records the "request_created" event
- **[Area Analysis](../area-analysis/SPEC.md)** — Five area records are initialized on creation

---

## Technical Notes

> These notes are for developer reference. Non-technical stakeholders may skip this section.

- Form state management should use React Hook Form for performance
- Validation schema should be defined with Zod and shared with backend
- Request ID generation should use UUID v4
- All timestamps should be stored in ISO 8601 format
- Area initialization should happen atomically with request creation
- Consider debouncing the submit button to prevent double submission

---

*Specification Version: 1.0*

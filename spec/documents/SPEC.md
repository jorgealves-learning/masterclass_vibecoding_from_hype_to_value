# Feature Specification: Documents

> **Feature ID**: `documents`  
> **Priority**: P4 — Supporting Feature  
> **Parent Document**: [SPEC.md](../../SPEC.md)

---

## Description

The Documents feature allows users to attach references to documents related to a contract or project analysis request. This provides a centralized place to link all relevant materials — contracts, proposals, addendums, policies, and other supporting documents — so that reviewers have easy access to the materials they need for their assessment.

### Business Context

When analyzing a contract or project, reviewers need access to various documents:
- The actual contract or proposal being analyzed
- Supporting documentation like policies or addendums
- Previous versions or related agreements
- Reference materials

Previously, these were shared via email attachments or file share links scattered across multiple messages. The Documents feature consolidates all document references in one place, making it easy for all reviewers to find what they need.

### MVP Limitation

**Important**: In the MVP version, there is no actual file upload functionality. Documents are represented by metadata (name, type) and an optional link (URL or path string). This keeps the system simple while still providing value through organization and reference tracking.

### User Stories

- As a **request creator**, I want to link the contract document so that reviewers can access it
- As a **department reviewer**, I want to see all related documents so I can make an informed assessment
- As a **manager**, I want to know what documents exist for a request so I can verify completeness
- As an **auditor**, I want to see what documents were available during the review process

### Interface Language

All visible text, labels, and messages must be in **Portuguese (Portugal)**.

---

## Document Types

| Type ID | Portuguese Label | English Name | Typical Use |
|---------|-----------------|--------------|-------------|
| `contract` | Contrato | Contract | The main contract being analyzed |
| `addendum` | Aditamento | Addendum | Additions or modifications to contracts |
| `proposal` | Proposta | Proposal | Initial proposals or quotations |
| `policy` | Política | Policy | Internal policies relevant to the review |
| `other` | Outro | Other | Any other document type |

---

## Detailed Requirements

### Document Data Structure

Each document record contains:

| Field | Portuguese Label | Type | Required | Constraints |
|-------|-----------------|------|----------|-------------|
| `id` | — | System-generated | Auto | Unique identifier |
| `requestId` | — | Reference | Auto | Link to parent request |
| `name` | Nome | Text input | Yes | Minimum 3 characters, maximum 200 characters |
| `type` | Tipo | Select dropdown | Yes | One of the five defined types |
| `link` | Ligação | Text input | No | Valid URL or path string, maximum 500 characters |
| `addedAt` | Adicionado em | Timestamp | Auto | Set when document is added |

---

## Functional Requirements

### FR-1: View Documents List

The Documents tab displays all documents attached to a request.

**Requirements:**
- Show documents in a list format, sorted by most recently added first
- Each item displays: document name, type badge, link (if present), and date added
- If no documents exist, show empty state with prompt to add first document
- Document count should appear in the tab badge (e.g., "Documentos (3)")

### FR-2: Add Document

Users can add new document references to a request.

**Requirements:**
- "Adicionar Documento" button visible at top of documents list
- Clicking button opens a modal form with name, type, and link fields
- Name and type are required; link is optional
- On successful submission:
  1. Document is added to the request
  2. Timeline event is created: "Document '[name]' added"
  3. Modal closes
  4. Document list refreshes to show new item
  5. Confirmation toast appears
- On validation failure, show errors inline in the modal

### FR-3: View Document Link

If a document has a link, users can access it.

**Requirements:**
- Link is displayed as clickable text or dedicated button
- Clicking link opens in a new browser tab
- If link is empty, show "Sem ligação" (No link) in muted text
- Do not attempt to validate if URL is actually accessible

### FR-4: Delete Document

Users can remove document references from a request.

**Requirements:**
- Each document item has a delete action (button or icon)
- Clicking delete shows confirmation dialog: "Tem a certeza que deseja remover este documento?"
- On confirmation, document is removed (soft delete if applicable)
- Timeline event is created: "Document '[name]' removed"
- Document list refreshes
- Confirmation toast appears

### FR-5: Empty State

When no documents are attached to a request.

**Requirements:**
- Show friendly message: "Nenhum documento anexado"
- Show prompt: "Adicione documentos relevantes para esta análise"
- Show prominent "Adicionar Documento" button

---

## Acceptance Criteria

### AC-1: Documents List Display
- [ ] Documents tab shows all attached documents
- [ ] Tab badge shows correct count of documents
- [ ] Documents are sorted by most recently added first
- [ ] Each document shows name, type badge, link status, and date
- [ ] Empty state displays when no documents exist

### AC-2: Add Document Form
- [ ] "Adicionar Documento" button is visible and clickable
- [ ] Modal opens with form fields for name, type, and link
- [ ] Name field validates minimum 3 characters
- [ ] Type dropdown shows all five document types
- [ ] Link field accepts valid URLs and path strings
- [ ] Form submits successfully with valid data
- [ ] Validation errors appear inline next to invalid fields

### AC-3: Document Creation
- [ ] New document appears in list after successful creation
- [ ] Document has correct name, type, and link values
- [ ] Timeline records "Document added" event
- [ ] Confirmation message appears
- [ ] Modal closes after successful submission
- [ ] Tab badge count increments

### AC-4: Document Links
- [ ] Documents with links show clickable link
- [ ] Clicking link opens in new tab
- [ ] Documents without links show "Sem ligação"
- [ ] Links are not validated for accessibility (intentional MVP limitation)

### AC-5: Document Deletion
- [ ] Delete action is available for each document
- [ ] Confirmation dialog appears before deletion
- [ ] Document is removed from list after confirmation
- [ ] Timeline records "Document removed" event
- [ ] Confirmation message appears
- [ ] Tab badge count decrements
- [ ] Canceling confirmation does not delete document

### AC-6: Accessibility and Usability
- [ ] All form fields have proper labels
- [ ] Modal can be closed with Escape key
- [ ] Focus is trapped within modal when open
- [ ] Form can be submitted with keyboard

---

## Success and Failure Examples

### Success Examples ✅

| Scenario | User Action | Expected Result |
|----------|-------------|-----------------|
| **Add first document** | User clicks "Adicionar Documento" on empty list | Modal opens, user fills form, document appears in previously empty list |
| **Add contract document** | User enters "Contrato Principal" with type "Contrato" and a SharePoint link | Document is saved, appears at top of list with type badge and clickable link |
| **Add document without link** | User adds document with name and type but leaves link empty | Document saves successfully, shows "Sem ligação" in link column |
| **Access linked document** | User clicks link on a document | New browser tab opens with the linked URL |
| **Delete unwanted document** | User clicks delete, confirms in dialog | Document removed, count decreases, timeline updated |
| **Cancel delete** | User clicks delete, then clicks "Cancelar" in dialog | Document remains, no changes made |
| **View document list** | User opens Documents tab with 5 documents | All 5 documents visible with correct information, tab shows "(5)" |
| **Add multiple documents** | User adds three documents in succession | All three appear in list, ordered by most recent first |

### Failure Examples ❌

| Scenario | User Action | Incorrect Result | Why It's Wrong |
|----------|-------------|------------------|----------------|
| **Form submits with empty name** | User leaves name blank and clicks save | Document is created with empty name | Name is required; validation should prevent this |
| **Modal won't close** | User clicks outside modal or presses Escape | Nothing happens | Modal should close on outside click or Escape |
| **Silent deletion** | User clicks delete | Document disappears with no confirmation | Must show confirmation dialog before destructive action |
| **Lost document data** | User fills form, network fails | Form clears, data lost | Should preserve form data and show error for retry |
| **Duplicate documents** | User double-clicks save button | Two identical documents created | Should debounce or disable button during submission |
| **Missing timeline event** | User adds document | Document appears but timeline has no new entry | Must record all document actions in timeline |
| **Wrong count** | User has 4 documents | Tab badge shows "(3)" | Count must accurately reflect number of documents |
| **Delete without authorization** | Any user deletes any document | Document removed with no tracking of who deleted it | Should track who performed deletion in timeline |
| **Broken link display** | Document has link | Link shows as plain text, not clickable | Links must be interactive |
| **Type not saving** | User selects "Aditamento" type | Document saves but shows "Contrato" | Selected type must be persisted correctly |

---

## Edge Cases and Boundary Conditions

### Input Boundaries

| Field | Boundary | Expected Behavior |
|-------|----------|------------------|
| Name | Exactly 3 characters | Accepted (minimum valid) |
| Name | 2 characters | Rejected with error: "Nome deve ter pelo menos 3 caracteres" |
| Name | Exactly 200 characters | Accepted (maximum valid) |
| Name | 201 characters | Rejected or truncated with warning |
| Link | Empty | Accepted (optional field) |
| Link | Exactly 500 characters | Accepted (maximum valid) |
| Link | 501 characters | Rejected with error |
| Link | Invalid URL format | Accepted (no URL validation in MVP) |

### Special Content

| Input | Expected Behavior |
|-------|------------------|
| Portuguese characters in name (ã, ç, é) | Stored and displayed correctly |
| Spaces in link URL | Preserved (let browser handle encoding) |
| File path instead of URL | Accepted (e.g., "\\\\server\\share\\file.pdf") |
| Very long file name | Truncated with ellipsis in display, full name in tooltip |
| HTML in name | Sanitized, not rendered as HTML |

### Document Limits

| Scenario | Expected Behavior |
|----------|------------------|
| Request has 0 documents | Empty state displays |
| Request has 50 documents | All documents displayed (no pagination in MVP) |
| Same document added twice | Both entries created (no duplicate prevention in MVP) |

### Link Handling

| Link Type | Example | Expected Behavior |
|-----------|---------|------------------|
| HTTPS URL | `https://company.sharepoint.com/doc.pdf` | Opens in new tab |
| HTTP URL | `http://internal.server/file.doc` | Opens in new tab |
| File path | `\\server\share\document.pdf` | Browser attempts to open (behavior varies) |
| Relative path | `/documents/contract.pdf` | Opens relative to current domain |
| Empty string | `` | Shows "Sem ligação" |

---

## User Interface Guidelines

### Documents List Layout

```
┌──────────────────────────────────────────────────────────────────┐
│ Documentos (3)                                                   │
│                                                   [Adicionar ➕] │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│ ┌────────────────────────────────────────────────────────────┐  │
│ │ 📄 Contrato Principal v2.0                    [Contrato]   │  │
│ │    🔗 https://sharepoint.com/...              [🗑️ Remover] │  │
│ │    Adicionado: 15/01/2024                                  │  │
│ └────────────────────────────────────────────────────────────┘  │
│                                                                  │
│ ┌────────────────────────────────────────────────────────────┐  │
│ │ 📄 Proposta Comercial                         [Proposta]   │  │
│ │    🔗 https://drive.google.com/...            [🗑️ Remover] │  │
│ │    Adicionado: 14/01/2024                                  │  │
│ └────────────────────────────────────────────────────────────┘  │
│                                                                  │
│ ┌────────────────────────────────────────────────────────────┐  │
│ │ 📄 Política de Privacidade                    [Política]   │  │
│ │    Sem ligação                                [🗑️ Remover] │  │
│ │    Adicionado: 13/01/2024                                  │  │
│ └────────────────────────────────────────────────────────────┘  │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Empty State Layout

```
┌──────────────────────────────────────────────────────────────────┐
│ Documentos (0)                                                   │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│                         📁                                       │
│                                                                  │
│               Nenhum documento anexado                           │
│                                                                  │
│         Adicione documentos relevantes para esta análise         │
│                                                                  │
│                   [Adicionar Documento]                          │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Add Document Modal

```
┌────────────────────────────────────────────────────┐
│ Adicionar Documento                           [✕] │
├────────────────────────────────────────────────────┤
│                                                    │
│ Nome do Documento *                                │
│ ┌────────────────────────────────────────────────┐│
│ │                                                ││
│ └────────────────────────────────────────────────┘│
│                                                    │
│ Tipo *                                             │
│ ┌────────────────────────────────────────────────┐│
│ │ Selecione o tipo...                         ▼ ││
│ └────────────────────────────────────────────────┘│
│                                                    │
│ Ligação (URL ou caminho)                           │
│ ┌────────────────────────────────────────────────┐│
│ │                                                ││
│ └────────────────────────────────────────────────┘│
│                                                    │
│                                                    │
│           [Cancelar]    [Adicionar Documento]     │
│                                                    │
└────────────────────────────────────────────────────┘
```

### Document Type Badge Colors

| Type | Background | Text |
|------|------------|------|
| Contrato | Blue-100 | Blue-700 |
| Aditamento | Purple-100 | Purple-700 |
| Proposta | Amber-100 | Amber-700 |
| Política | Gray-100 | Gray-700 |
| Outro | Gray-100 | Gray-600 |

---

## Definition of Done

This feature is complete when:

- [ ] **List Display**: Documents tab shows all documents with correct information
- [ ] **Tab Badge**: Count accurately reflects number of documents
- [ ] **Add Modal**: Opens correctly with all required fields
- [ ] **Validation**: Name minimum length and required fields are enforced
- [ ] **Create Works**: New documents appear in list after successful creation
- [ ] **Timeline Integration**: Adding documents creates timeline events
- [ ] **Links Work**: Document links open in new tab correctly
- [ ] **No Link State**: "Sem ligação" displays when link is empty
- [ ] **Delete Confirmation**: Confirmation dialog appears before deletion
- [ ] **Delete Works**: Documents are removed after confirmation
- [ ] **Empty State**: Displays correctly when no documents exist
- [ ] **Error Handling**: Network errors show clear messages
- [ ] **Form Preservation**: Failed submissions preserve entered data
- [ ] **Portuguese**: All labels and messages in Portuguese
- [ ] **Accessibility**: Form fields labeled, keyboard navigable, modal trap focus
- [ ] **Responsive**: Works on desktop and tablet viewports
- [ ] **Code Quality**: Passes linting, no TypeScript errors
- [ ] **Tests**: Unit tests for validation, integration test for add/delete flow
- [ ] **Review**: Code reviewed and approved

---

## Related Features

- **[Request Detail](../request-detail/SPEC.md)**: Parent container with Documents tab
- **[Timeline](../timeline/SPEC.md)**: Records document add/remove events
- **[Create Request](../create-request/SPEC.md)**: Documents can be added after request creation

---

## Technical Notes

> These notes are for developer reference. Non-technical stakeholders may skip this section.

- Documents are stored as array within the Request object
- Each document needs a unique ID (UUID) for deletion targeting
- Modal should use shadcn/ui Dialog component
- Link field should use `type="url"` for mobile keyboard optimization but accept any string
- Consider `target="_blank" rel="noopener noreferrer"` for external links
- Document list should be a Client Component for real-time updates after add/delete
- Empty state should be its own component for reusability
- Soft delete may be overkill for documents; hard delete acceptable if timeline captures the event

---

## Future Enhancements (Out of Scope)

These may be added in future versions:

- Actual file upload with storage
- File preview within the application
- Version tracking for documents
- Document categories or tags
- Search within documents
- Access logging (who viewed which document)
- File type icons based on extension
- Drag-and-drop file attachment

---

*Specification Version: 1.0*

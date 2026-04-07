# Engineering Standards & Architectural Rules
# Next.js TypeScript Applications on Vercel

## 1.0 Guiding Principles

This document defines the mandatory engineering standards and architectural rules for all Next.js projects. It is not a tutorial; it is a set of non-negotiable constraints and patterns that ensure quality, scalability, and maintainability. Adherence is required.

- **1.1. Rule of Law**: The standards herein are authoritative. Deviations require formal architectural review and approval.
- **1.2. Intentionality**: Every architectural decision must be deliberate. Default to the patterns defined in this document.
- **1.3. Automation as a Mandate**: All processes that can be automated must be automated. This includes testing, linting, formatting, and deployment.
- **1.4. Security by Design**: Security is a foundational requirement, not an afterthought. All development must adhere to the security standards defined.

---

## 2.0 Software Architecture Standards

These rules govern the structure and organization of the codebase.

### 2.1. Directory Structure Mandate

- **2.1.1. `src` is Root**: All application source code must reside within the `/src` directory.
- **2.1.2. Feature-Based Routing**: The `/src/app` directory must be organized by features using route groups. For example, `(features)/auth`, `(features)/dashboard`.
- **2.1.3. Component Stratification**: The `/src/components` directory must be stratified as follows:
    - `/ui`: Contains only generic, unstyled, or minimally styled primitive components (e.g., a base `Button`, `Input`). These are the building blocks.
    - `/layout`: Contains components responsible for page structure (e.g., `Header`, `Sidebar`, `Footer`).
    - `/features`: Contains components that are specific to a particular feature and are composed of `ui` and `layout` components.
- **2.1.4. Logic Encapsulation**:
    - `/src/lib`: For shared, framework-agnostic logic, utilities, and third-party API clients.
    - `/src/hooks`: For custom React hooks that encapsulate stateful logic.
- **2.1.5. API Abstraction**: All API route handlers must be located within `/src/app/(api)`. Direct database or external service calls from components are forbidden.

### 2.2. Component Architecture Rules

- **2.2.1. Server-First Mandate**: All components must be React Server Components (RSCs) by default.
- **2.2.2. Client Component Justification**: The `'use client'` directive may only be used when a component requires state, lifecycle effects, or browser-only APIs. A comment justifying its use is required.
- **2.2.3. Data Fetching**: Data fetching must occur within RSCs or in Route Handlers. Client-side data fetching is an anti-pattern and is prohibited unless explicitly justified for dynamic, real-time UI updates.
- **2.2.4. State Management**:
    - **Local State**: `useState` and `useReducer` are for component-local state only.
    - **Global State**: For global client-side state, Zustand is the only approved library.
    - **Server State**: Server-side state management must be handled via Next.js caching, revalidation (`revalidatePath`, `revalidateTag`), and Server Actions.

---

## 3.0 TypeScript & Coding Standards

These rules define the required level of quality and consistency for all code.

### 3.1. Type System Rules

- **3.1.1. Strictness is Mandatory**: The `tsconfig.json` file must always have `"strict": true`.
- **3.1.2. The `any` Type is Prohibited**: The `any` type is forbidden. Use `unknown` for type-unsafe data and perform runtime validation.
- **3.1.3. Explicit Boundaries**: All function signatures, API responses, and public-facing module exports must have explicit type annotations. Type inference may be used within function bodies.
- **3.1.4. Nominal Typing via Branded Types**: For primitive types that have specific business meaning (e.g., `UserId`, `ProductId`), branded types must be used to prevent logical errors.
- **3.1.5. Zod for Validation**: All external data (API responses, form submissions, environment variables) must be validated at the application boundary using Zod. Failure to validate will be treated as a critical defect.

### 3.2. Code Quality & Style Rules

- **3.2.1. Linter is Law**: All code must pass ESLint checks without any warnings or errors. No exceptions.
- **3.2.2. Formatter is Final**: All code must be formatted with Prettier. No unformatted code may be committed.
- **3.2.3. Single Responsibility Principle**: Components and functions must be small and have a single, well-defined responsibility. A function exceeding 25 lines or a component exceeding 50 lines requires justification.
- **3.2.4. No Default Exports**: Use named exports exclusively for all modules. This enforces consistency and improves refactorability.

---

## 4.0 DevOps & Deployment Standards

These rules govern the deployment and operational aspects of the application on Vercel.

- **4.1. Vercel as the Single Source of Truth**: Vercel is the only approved deployment platform. All operations must be optimized for its ecosystem.
- **4.2. Git-Driven Deployments**: The `main` branch is the production branch. Every push to `main` must trigger a production deployment. All other branches are deployed as preview environments. Direct deployments via `vercel deploy --prod` are forbidden except in declared emergencies.
- **4.3. Environment Variable Security**:
    - **Centralized Management**: All environment variables must be managed in the Vercel project settings.
    - **No Local Secrets**: The `.env.local` file is for non-sensitive, development-only overrides and must be in `.gitignore`. It must never contain secrets.
    - **Runtime Validation**: Environment variables must be validated on application startup via a Zod schema in `/src/lib/env.ts`. The application must fail to start if validation fails.
- **4.4. Testing Mandates**:
    - **Unit & Integration Tests**: Jest and React Testing Library are the required frameworks. All business logic and complex components must have unit tests.
    - **End-to-End Tests**: Playwright is the required framework for E2E testing. All critical user flows must be covered.
    - **CI/CD Enforcement**: All tests must run and pass in the CI/CD pipeline before code can be merged to `main`.

---

## 5.0 Mandated Technology Stack

The use of this stack is mandatory. No other technologies may be introduced for the same purpose without formal architectural approval.

- **Framework**: Next.js (App Router)
- **Language**: TypeScript
- **Styling**: Tailwind CSS
- **UI Primitives**: Radix UI (via shadcn/ui)
- **State Management**: Zustand (for global client state)
- **Forms**: React Hook Form
- **Schema & Validation**: Zod
- **Testing**: 
    - Jest & React Testing Library (Unit/Integration)
    - `@playwright/test` (End-to-End)
- **Deployment**: Vercel

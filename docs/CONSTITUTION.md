# Nexus E-Commerce Platform Engineering Constitution

**Version:** 1.0.0

> **Authority Declaration:** This document is the supreme engineering authority for the Nexus E-Commerce Platform. All code, architecture, tooling, and AI-assisted development must strictly adhere to these rules. No pull request may be merged if it violates this constitution.

*(Assumption: As the project brief, shape, maturity, and primary languages were delegated, this constitution assumes a Production-maturity Full-Stack Web Application (Next.js/Node.js) for a high-performance e-commerce platform written in TypeScript.)*

## Mission
To build a modern, high-performance headless e-commerce backend and storefront, supporting multi-tenant product catalogs, secure checkout, and real-time inventory management with zero downtime and absolute data integrity.

## Core Values
*   **Correctness over speed:** A working, bug-free feature is better than a fast, broken one.
*   **Security over convenience:** Never compromise user data or system integrity for developer ease.
*   **Simplicity over cleverness:** Code must be readable and maintainable by the next engineer.
*   **Maintainability over shortcuts:** Technical debt must be documented and paid down promptly.
*   **Observability over assumptions:** If it isn't logged and monitored, it doesn't exist in production.
*   **Explicitness over magic:** Avoid hidden side effects and overly abstracted frameworks.
*   **Automation over manual processes:** CI/CD, testing, and formatting must be automated.
*   **Testing over trust:** All critical paths must be provably correct via automated tests.

## Technology Stack

### Required Technologies
| Category | Technology | Purpose |
| :--- | :--- | :--- |
| **Language** | TypeScript (Strict) | Primary language for all application code |
| **Frontend** | Next.js / React | Storefront UI and server-side rendering |
| **Backend** | Node.js / Express | Core API and business logic |
| **Database** | PostgreSQL & Prisma | Primary relational data store and ORM |
| **Caching** | Redis | Session management and inventory caching |
| **Testing** | Jest / Playwright | Unit, integration, and end-to-end testing |

### Forbidden Technologies & Practices
*   **Plain JavaScript:** Forbidden in application code; TypeScript is mandatory.
*   **Unmaintained dependencies:** No libraries without active updates in the last 12 months.
*   **Experimental libraries:** Forbidden in production without explicit architectural approval.
*   **Direct Database Access from Frontend:** All database queries must route through the Backend API.

## Repository Structure
The project utilizes a monorepo structure managed by Turborepo to enforce boundary separation.

```text
nexus-platform/
├── apps/
│   ├── web/                # Next.js storefront application
│   └── api/                # Node.js/Express backend API
├── packages/
│   ├── ui/                 # Shared React component library
│   ├── database/           # Prisma schema and generated client
│   └── config/             # Shared ESLint, Prettier, and TS configs
├── docs/                   # Architecture Decision Records (ADRs)
└── package.json
```

## Language/Code Standards

*   **TypeScript Strict Mode:** `strict: true` and `noImplicitAny: true` are mandatory.
*   **Component/File Size Limits:** 
    *   Target: 300 lines of code.
    *   Mandatory Refactor: 500 lines of code.

### Naming Conventions
| Element | Convention | Example |
| :--- | :--- | :--- |
| Files/Directories | `kebab-case` | `shopping-cart.tsx`, `user-controller.ts` |
| Variables/Functions | `camelCase` | `calculateTotal()`, `cartItems` |
| Classes/Interfaces | `PascalCase` | `PaymentProcessor`, `IUser` |
| React Components | `PascalCase` | `ProductCard`, `CheckoutForm` |
| Constants/Enums | `UPPER_SNAKE_CASE` | `MAX_RETRY_COUNT`, `OrderStatus.PENDING` |

## Frontend Standards
*   **Server Components:** Default to React Server Components (RSC) in Next.js. Use `'use client'` only when interactivity or browser APIs are strictly required.
*   **State Management:** Prefer URL state and server state (via React Query/SWR) over global client-side state (e.g., Redux) unless managing complex, localized UI flows.
*   **Styling:** Use Tailwind CSS. Avoid inline styles.

## Backend/API & Validation Standards
*   **RESTful Design:** APIs must follow strict REST conventions (resource-based URLs, correct HTTP methods, standard status codes).
*   **Input Validation:** All incoming data (body, query, params) MUST be validated at the boundary using Zod.
*   **Statelessness:** The API must remain stateless. Session state must be managed via Redis or JWTs.

## Error Handling
*(Assumption: Error categories were not provided; standard production e-commerce categories are assumed.)*

*   **Never expose stack traces** to the client in production.
*   **Standardized Response:** All API errors must return a consistent JSON structure:

```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "Invalid email format",
    "details": [
      { "field": "email", "issue": "Must be a valid email address" }
    ],
    "requestId": "req-12345abcde"
  }
}
```

### Error Categories
1.  `Validation`: Bad client input (HTTP 400).
2.  `Authentication`: Missing or invalid credentials (HTTP 401).
3.  `Authorization`: Insufficient permissions (HTTP 403).
4.  `NotFound`: Resource does not exist (HTTP 404).
5.  `Internal`: Unhandled exceptions or database failures (HTTP 500).

## Logging
*   **Format:** JSON format is mandatory in production. Human-readable formats are permitted in local development.
*   **Required Structured Fields:**
    *   `event`: Action being performed (e.g., `checkout_initiated`).
    *   `timestamp`: ISO 8601 format.
    *   `requestId`: For distributed tracing.
    *   `userId?`: If the user is authenticated.
    *   `metadata?`: Contextual data (e.g., `cartId`, `totalAmount`).
*   **PII/PCI:** NEVER log passwords, credit card numbers, or raw authentication tokens.

## Security
*   **Authentication & Authorization Location:** Must occur server-side. Client-side checks are for UX only, never for security.
*   **Secrets Handling:**
    *   Must be loaded from environment variables.
    *   Must be provisioned via a secure secret manager (e.g., AWS Secrets Manager, HashiCorp Vault).
    *   Never commit secrets to version control.
*   **Dependency Policy:**
    *   Must pass automated security scans (e.g., `npm audit`, Snyk).
    *   Must pass license review (no copyleft licenses like GPL in proprietary code).
    *   Must be actively maintained.
    *   Prefer building custom logic over adding a dependency when the required functionality is small.

## Accessibility
*   **Standard:** All storefront UI components must comply with WCAG 2.1 AA standards.
*   **Enforcement:** Automated accessibility testing (e.g., axe-core) must run in CI. Semantic HTML and proper ARIA attributes are mandatory.

## Performance
*   **Core Web Vitals:** The storefront must maintain "Good" scores for LCP (< 2.5s), INP (< 200ms), and CLS (< 0.1).
*   **API Latency:** 95th percentile (p95) response times for critical API endpoints (e.g., Add to Cart, Checkout) must be under 200ms.

## Testing
*   **Minimum Coverage:** 
    *   80% minimum overall codebase coverage.
    *   95% minimum for critical business logic (pricing, checkout, inventory).
*   **Required Test Types:**
    *   *Unit Tests (Jest):* For isolated functions, utilities, and components.
    *   *Integration Tests (Jest/Supertest):* For API endpoints and database interactions.
    *   *End-to-End Tests (Playwright):* For critical user journeys (e.g., user registration, full checkout flow).

## CI/CD
*   **CI Gates per PR:** Every pull request MUST pass the following automated gates before merge is allowed:
    1.  Linting (ESLint/Prettier).
    2.  Typechecking (`tsc --noEmit`).
    3.  Unit tests.
    4.  Integration tests.
*   **Deployment:** Main branch deployments to staging are automated. Production deployments require manual approval after staging verification.

## Documentation
*   **READMEs:** Every package and app in the monorepo must have a `README.md` detailing setup, scripts, and architecture.
*   **API Specs:** All backend endpoints must be documented using OpenAPI (Swagger).
*   **ADRs:** Any significant architectural change must be documented in `docs/adr/` before implementation.

## Observability
*   **Distributed Tracing:** OpenTelemetry must be implemented across the frontend and backend to trace requests end-to-end via `requestId`.
*   **Metrics:** Prometheus must scrape application metrics (CPU, memory, event loop lag, HTTP response times).
*   **Centralized Logging:** All logs must be aggregated in a centralized platform (e.g., Datadog, ELK) with alerting configured for elevated 5xx error rates.

## AI Development Rules
*   **Trust Boundary:** AI-generated code is UNTRUSTED. It must be reviewed, tested, and validated by a human engineer before merging.
*   **Agent Restrictions (Without Human Approval):**
    *   May NOT deploy to production.
    *   May NOT rotate credentials or manage secrets.
    *   May NOT modify infrastructure (Terraform/CDK).
    *   May NOT approve pull requests.

## Prompt/MCP/RAG Standards
*   **Prompts:** Must be version-controlled, documented, and tested. Changes to system prompts require standard code review.
*   **MCP Integrations:** Must operate on a least-privilege basis, be fully auditable, and be instantly revocable.
*   **RAG Sources:** Must be trusted, versioned, and source-attributed in the output.

## Code Review Standards
Every Pull Request description MUST answer the following questions:
1.  **What changed?** (Summary of modifications)
2.  **Why?** (Business or technical justification)
3.  **Risks?** (Potential side effects or performance impacts)
4.  **Rollback plan?** (How to revert if things go wrong)
5.  **Testing evidence?** (Screenshots, test output, or coverage reports)

## Git Standards
*   **Branch Conventions:**
    *   `feature/*` (New functionality)
    *   `bugfix/*` (Non-critical fixes)
    *   `hotfix/*` (Critical production fixes)
    *   `chore/*` (Maintenance, dependencies)
*   **Commit Conventions (Conventional Commits):**
    *   `feat:` A new feature
    *   `fix:` A bug fix
    *   `refactor:` Code change that neither fixes a bug nor adds a feature
    *   `test:` Adding or correcting tests
    *   `docs:` Documentation only changes
    *   `perf:` A code change that improves performance
    *   `chore:` Changes to the build process or auxiliary tools

## Dependency Rules
*   Dependencies must be locked using `package-lock.json` or `pnpm-lock.yaml`.
*   Adding a new production dependency requires justification in the PR (size impact, security posture, and maintenance status).
*   Dependabot (or equivalent) must be configured for automated security updates.

## Definition of Done
*(Assumption: DoD items were not provided; standard production criteria are assumed.)*

A task is only "Done" when:
1.  Code meets all Language and Architecture standards.
2.  Unit and Integration tests are written and passing.
3.  CI pipeline is 100% green.
4.  Code has been reviewed and approved by at least one peer.
5.  Documentation (OpenAPI, ADRs, READMEs) is updated.
6.  Feature is deployed to and verified in the Staging environment.

## Non-Negotiable Rules (NEVER / ALWAYS)
*   **NEVER** commit secrets, credentials, or API keys to version control.
*   **NEVER** bypass CI checks or force-push to the `main` branch.
*   **NEVER** trust client-side data; **ALWAYS** validate input at the server boundary.
*   **ALWAYS** handle errors gracefully and log them with context.
*   **ALWAYS** leave the codebase cleaner than you found it.

## Amendment Process
*(Assumption: Amendment process was not provided; a standard governance process is assumed.)*

This constitution is a living document. To amend it:
1.  Open a Pull Request modifying this file.
2.  Provide a detailed rationale for the change in the PR description.
3.  The PR requires approval from at least two core maintainers or engineering leads.
4.  Once merged, the version number at the top of this document must be incremented.
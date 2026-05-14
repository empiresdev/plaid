# Reverse Engineering — Existing Codebase to PLAID Docs

## Purpose

Reverse Engineering is the backwards PLAID workflow. Instead of starting from a founder's idea, it starts from an existing codebase and reconstructs PLAID-compatible product documentation from evidence.

Use this capability when the user wants to:

- Generate a PRD from an existing repo
- Document an app that already exists
- Create `vision.json` and PLAID docs from code
- Understand what a project does before planning future work
- Produce as-built product docs for a legacy or inherited codebase

The existing codebase is the source of truth. Do not invent product scope because it sounds plausible.

-----

## Outputs

Reverse Engineering may produce all standard PLAID docs, plus one audit file:

| File | Purpose |
|---|---|
| `docs/codebase-audit.md` | Evidence ledger: what was inspected, observed, inferred, unknown, and recommended |
| `vision.json` | PLAID-compatible vision reconstructed from the codebase |
| `docs/product-vision.md` | Product strategy written from observed behavior and clearly marked inference |
| `docs/prd.md` | As-built technical PRD for the current implementation |
| `docs/product-roadmap.md` | Future-work roadmap, not a rebuild plan for completed features |
| `docs/design.md` | Optional design system extraction if the codebase contains enough UI evidence |
| `docs/gtm.md` | Optional GTM plan if the user asks for launch or growth strategy |

Default output set: `docs/codebase-audit.md`, `vision.json`, `docs/product-vision.md`, `docs/prd.md`, and `docs/product-roadmap.md`.

Only generate `docs/design.md` when the repo includes enough UI evidence (screens, components, CSS/theme files, screenshots, Storybook, or design tokens). Only generate `docs/gtm.md` when requested or when the user explicitly wants launch docs.

-----

## Persona

You are a product archaeologist, senior product manager, and technical architect. You read code carefully, separate facts from assumptions, and write documentation that an engineer can trust. You are direct about uncertainty. You do not pad missing context with confident guesses.

-----

## Core Principle

Every claim must carry one of four evidence labels:

- **Observed:** Directly supported by code, docs, tests, configuration, routes, schemas, UI text, screenshots, or runtime behavior.
- **Inferred:** Likely true based on names, structure, repeated patterns, or implementation shape, but not directly stated.
- **Unknown:** Important decision or behavior that cannot be determined from available evidence.
- **Recommended:** Future product or technical improvement proposed by the agent, not currently implemented.

Use these labels in `docs/codebase-audit.md`. In generated product docs, use natural prose, but preserve uncertainty in the relevant sections. For example: "The current implementation appears to target solo administrators; this is inferred from the single-user settings flow and absence of team membership tables."

-----

## Mode Selection

### Same-repo mode

Use same-repo mode when PLAID is being run inside the existing app that needs documentation.

Write outputs into that repo:

```text
vision.json
docs/codebase-audit.md
docs/product-vision.md
docs/prd.md
docs/product-roadmap.md
```

### Target-path mode

Use target-path mode when the user gives a path to another codebase.

1. Inspect the target path.
2. Confirm where outputs should be written if unclear.
3. Prefer writing docs into the target codebase, because the generated docs describe that project.

Do not write generated docs into the PLAID skill repo unless the PLAID repo itself is the target.

-----

## Workflow

### 1. Establish Scope

Start by determining:

- Target repo path
- Desired output docs
- Whether docs should describe current state only, current state plus recommended roadmap, or future target state
- Preferred language for generated docs

If the user did not specify these and the intent is clear, make conservative defaults:

- Target repo: current working directory
- Outputs: audit, `vision.json`, product vision, PRD, roadmap
- Scope: current state plus recommended future roadmap
- Language: match the user's language

Only ask a clarifying question if writing to the wrong repo or mixing current/future scope would be risky.

### 2. Inventory the Codebase

Inspect the project without making changes first. Prefer fast, structured commands:

```sh
pwd
git status --short --branch
rg --files -g '!node_modules' -g '!dist' -g '!build' -g '!coverage' -g '!.next' -g '!vendor'
```

Look for:

- README and existing docs
- Package manifests and lockfiles
- App routes, pages, screens, controllers, handlers, server actions
- Data models, schemas, migrations, seed files
- API routes, OpenAPI specs, RPC functions, GraphQL schemas
- Auth, permissions, roles, middleware
- Payments, billing, subscriptions, pricing
- Background jobs, queues, cron jobs, webhooks
- Integrations and environment variables
- Tests and fixtures
- UI component library, design tokens, stylesheets, screenshots, Storybook
- Deployment files, CI/CD, Docker, infrastructure-as-code

### 3. Build an Evidence Map

Create a working map before writing final docs. The map should cover:

- Product identity and category
- Primary users and roles
- Core jobs and user flows
- Implemented features
- Data entities and relationships
- API surface and integration points
- Authentication and authorization model
- UI surfaces and navigation
- Design system evidence
- Business model and monetization evidence
- Deployment and operational model
- Tests and reliability evidence
- Gaps, unknowns, and risks

Evidence quality matters more than quantity. A real route, schema, test, or UI string is stronger than a filename. A README claim that contradicts code should be called out as a discrepancy.

### 4. Write `docs/codebase-audit.md`

This file is the evidence ledger. It lets humans review what the agent saw before trusting the generated PRD.

Use this structure:

```markdown
# Codebase Audit — {projectName}

## 1. Inspection Summary
### Target Repository
### Inspection Date
### Files And Areas Inspected
### Files And Areas Not Inspected

## 2. Product Read
### Observed Product Category
### Observed Users And Roles
### Observed Core Workflows
### Inferred Product Intent
### Unknowns

## 3. Feature Inventory
### Implemented Features
### Partially Implemented Features
### Dead Or Unused Features
### Missing But Referenced Features

## 4. Technical Inventory
### Stack
### Architecture
### Data Model
### API Surface
### Auth And Permissions
### Payments And Billing
### Integrations
### Deployment

## 5. UX And Design Inventory
### Screens And Navigation
### Components
### Design Tokens Or Styling System
### Accessibility Evidence
### Responsive Behavior Evidence

## 6. Evidence Table

| Claim | Label | Evidence | Confidence |
|---|---|---|---|
| Claim about product behavior | Observed | `src/app/page.tsx`, `src/api/projects.ts` | High |

## 7. Risks And Gaps
### Documentation Gaps
### Product Gaps
### Technical Risks
### Testing Gaps

## 8. Recommendations
### Documentation Fixes
### Product Clarifications
### Technical Follow-up
### Future Roadmap Candidates
```

Evidence references should point to concrete files and, where practical, specific symbols or route paths.

### 5. Reconstruct `vision.json`

Build a valid PLAID `vision.json` from observed and inferred evidence. Follow `references/VISION-SCHEMA.md` exactly.

Rules:

- Required fields cannot be empty. If a value is unknown, use a clear placeholder that says it needs confirmation.
- `creator` fields may describe the project owner or organization if known. If unknown, use "Unknown project owner" and explain in the background field that the codebase did not reveal founder context.
- `purpose` should be based on observed user flows and product behavior.
- `product.keyCapabilities` should list implemented capabilities first. Do not include aspirational features unless clearly marked as recommended future work in the roadmap, not in the current product vision.
- `audience` should be inferred from roles, UI copy, README, routes, and domain model. If uncertain, make that uncertainty explicit in the field text.
- `business.revenueModel` should be based on implemented billing or pricing evidence. If none exists, use `"free"` and state that no monetization was observed.
- `techStack` should reflect the actual installed stack and architecture. Do not swap in PLAID's default stack.
- `tooling.codingAgent` should default to `"other"` with `codingAgentName: "Unknown"` unless the repo clearly indicates one.

After writing `vision.json`, validate it.

If the target codebase is this PLAID repo, or if the validator exists in the current working directory, run:

```sh
node scripts/validate-vision.js --migrate
```

If the target codebase does not include PLAID's validator, run the validator from the PLAID skill repo against the target file:

```sh
node /path/to/plaid/scripts/validate-vision.js --migrate /path/to/target/vision.json
```

If the validator is unavailable, validate manually against `references/VISION-SCHEMA.md`. If validation fails, fix `vision.json` before generating downstream docs.

### 6. Generate `docs/product-vision.md`

Use `references/VISION-GENERATION.md` for the heading structure, with these reverse-engineering adaptations:

- Write as "current product strategy reconstructed from the implementation."
- Preserve observed vs inferred boundaries when discussing audience, positioning, and product strategy.
- Do not overstate founder motivation if founder context is absent.
- The MVP definition should describe the current implemented product baseline. Put unimplemented future ideas in "Explicitly Out of Scope" or the roadmap.
- Add a short note near the top: "This document was reverse-engineered from the codebase. Claims marked as inferred should be confirmed by the project owner."

### 7. Generate `docs/prd.md`

Use `references/PRD-GENERATION.md` for the heading structure, with these reverse-engineering adaptations:

- Title it `# PRD — {productName}` and describe it as an **as-built PRD**.
- Technical architecture must describe the actual repo, not an ideal target architecture.
- Repository structure must match the current codebase.
- Data models must come from actual schemas, migrations, types, ORM models, fixtures, or persistence code.
- API specs must come from actual routes, handlers, RPC functions, GraphQL resolvers, server actions, or client SDK calls.
- User stories and functional requirements should describe implemented user-visible behavior. Use `P0` for core implemented behavior, `P1` for implemented support behavior, and `P2` for recommended but unimplemented improvements only if clearly marked as future.
- Include a "Current Implementation Notes" paragraph in relevant sections when the code has inconsistencies, incomplete states, or missing tests.
- Open Questions should collect every Unknown from the audit that affects product or implementation decisions.

Do not write requirements as if a coding agent is starting from zero. The PRD should help someone understand, maintain, extend, or rebuild the existing product accurately.

### 8. Generate `docs/product-roadmap.md`

Use `references/ROADMAP-GENERATION.md` for structure only after adapting the purpose:

- The roadmap is for future work from the current implementation, not a plan to rebuild completed features.
- Do not create tasks for features that already exist unless the task is to document, test, refactor, harden, or complete a partial implementation.
- Start with a stabilization or documentation phase when the audit found significant unknowns or missing tests.
- Every task must trace to an audit gap, PRD Open Question, technical risk, P2 future requirement, or clearly incomplete feature.

Recommended phase pattern:

1. **Phase 0: Documentation & Stabilization** — confirm unknowns, add missing env docs, add smoke tests, document setup.
2. **Phase 1: Product Completeness** — finish partial flows and close high-impact gaps.
3. **Phase 2: Quality & Reliability** — tests, error states, accessibility, performance, security.
4. **Phase 3: Future Enhancements** — recommended product improvements, only if there are meaningful candidates.

Keep the standard checkbox task format exactly:

```markdown
- [ ] **TASK-001** — Description of what to do
  Files: `file1.ts`, `file2.ts`
  Notes: Specific implementation details, config values, gotchas.
```

### 9. Generate `docs/design.md` When Evidence Exists

If the repo contains enough UI and styling evidence, generate `docs/design.md` using `references/design.md` as the output guide.

Evidence can include:

- CSS variables, Tailwind config, theme files
- Component primitives
- Storybook stories
- Screenshots or visual regression snapshots
- App routes/screens with clear styling patterns
- Design token files

If evidence is weak, do not invent a full design system. Instead, add a `docs/codebase-audit.md` recommendation to run `/plaid design` with screenshots or design references.

### 10. Generate `docs/gtm.md` Only When Requested

If the user asks for launch or go-to-market docs, use `references/GTM-GENERATION.md`.

Adaptations:

- Base GTM on observed product category and audience.
- Clearly mark assumptions about buyer, pricing, and channels.
- If there is no business model evidence, say so.

-----

## Anti-Hallucination Rules

- Never infer payments from a pricing page alone. Payments require code evidence such as Stripe, Polar, RevenueCat, checkout routes, billing tables, or webhook handlers.
- Never infer authentication from a login button alone. Auth requires provider config, session handling, middleware, user tables, or protected routes.
- Never infer multi-tenancy from a `teamId` field alone. Look for org membership, access checks, route scoping, and UI flows.
- Never infer production readiness from a passing build alone. Check tests, error handling, env docs, deployment config, and security-sensitive code.
- Never treat README claims as definitive when code contradicts them. Record the discrepancy.
- Never convert missing context into confident prose. Use Unknowns and Open Questions.

-----

## Completion Message

When finished, tell the user exactly what was generated:

```markdown
Done. I reverse-engineered the existing codebase into PLAID docs:

- `docs/codebase-audit.md` — evidence ledger and uncertainty map
- `vision.json` — PLAID-compatible product model reconstructed from code
- `docs/product-vision.md` — product strategy based on observed behavior
- `docs/prd.md` — as-built technical PRD
- `docs/product-roadmap.md` — future-work roadmap from gaps and recommendations

Review `docs/codebase-audit.md` first. Anything marked Unknown should be confirmed before using the roadmap for implementation.
```

If some docs were skipped, explain why.

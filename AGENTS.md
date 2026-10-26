# Repository Guidelines

## Project Structure & Module Organization

The repository context describes a TypeScript/Express backend under `src/backend`. The application pipeline is in `app.ts`, startup is in `server.ts`, and routers are registered by domain in `shared/`, `b2b/`, `b2c/`, `admin/`, and `infra/`. Domain modules generally follow `routes → controller → service → repository`; keep validators and tests beside the module they cover. Prisma's schema and migrations belong in `src/backend/prisma/`. Consult the actual source files as the authority for behavior; `contexto.md` is an architectural reference, not a substitute for implementation.

## Build, Test, and Development Commands

No package manifest or runnable source tree is present in this working directory, so project commands cannot be confirmed here. Once the backend checkout is available, inspect its `package.json` and use the scripts defined there for development, build, lint, and tests. Do not assume a command or introduce a new one without checking the project configuration.

## Coding Style & Naming Conventions

Use TypeScript and follow the formatting and naming already used by the neighboring module. Keep HTTP concerns in routes/controllers and domain rules in services. Name files for their role, such as `*.routes.ts`, `*.controller.ts`, `*.service.ts`, `*.repository.ts`, and `*.validator.ts`. Validate request input with the module's established Zod patterns and return expected application errors through the shared error handling flow.

## Testing Guidelines

No test framework or coverage threshold is verifiable from the files in this directory. When working in the source checkout, follow its configured test runner and colocated test naming convention. Add or update focused tests for behavior changes, especially tenant isolation, authorization, financial state, and scheduling invariants; run only commands defined by the project.

## Security & Configuration

Derive salon scope from the authenticated identity, enforce role and resource permissions, and validate input at the boundary. Keep secrets out of source control; use `.env.example` for documented variable names and local environment configuration for values. Preserve webhook signature checks, idempotency, and transaction boundaries when modifying integrations or multi-record operations.

## Commit & Pull Request Guidelines

Git history is unavailable in this directory, so no commit convention can be confirmed. Use a concise, imperative commit subject consistent with the source repository's history when available. Pull requests should explain the domain and behavior changed, note configuration or migration impact, link related work, and include relevant test results; add screenshots only for user-facing changes.

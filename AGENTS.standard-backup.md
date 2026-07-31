# Global Node.js Project Rules

## 1. Purpose and Trigger

Apply this file whenever the user asks to create, configure, extend, or maintain a Node.js project, including:

```text
create project <project_name>
```

This phrase is an AI task trigger, not a terminal command or request for a project-generator CLI.

---

## 2. Rule Authority and Protection

- Read this file before planning or changing the project.
- Treat every applicable rule as mandatory.
- Treat this file as read-only: do not edit, delete, rename, move, override, bypass, or weaken it without the user's explicit approval for the exact rule change.
- Do not create a nested `AGENTS.md`, `AGENTS.override.md`, or conflicting AI instruction file without explicit approval.
- When a request conflicts with this file, stop the conflicting action, explain the rule conflict, and offer a compliant alternative.

---

## 3. Core Working Principles

- Understand the project before changing it: inspect requirements, existing files, `package.json`, lock files, database schema, installed versions, and conventions.
- Make the smallest change that fully satisfies the approved task.
- Do not refactor, rename, reformat, replace, or delete unrelated working code.
- Do not assume that all projects need the same framework, database, ORM, authentication, storage, testing, or deployment setup.
- Ask only for missing decisions that materially affect the project; never repeat information already provided.
- Use simple architecture for simple projects and modular architecture for complex projects.
- Never claim a command, test, migration, build, or installation succeeded unless it was actually run successfully.

---

## 4. Mandatory Discovery for New Projects

Before creating files or selecting architecture-specific packages, confirm only the relevant missing requirements:

- Project purpose and main features
- JavaScript or TypeScript
- Node.js version and package manager
- Application type: REST API, GraphQL, WebSocket, worker, scheduled job, CLI, or combination
- Framework: Express, Fastify, NestJS, native Node.js, or another choice
- Database: none, PostgreSQL, MySQL, MongoDB, SQLite, or another database
- Data access: Prisma, Sequelize, Mongoose, native driver, query builder, or none
- Authentication: none, JWT, sessions, OAuth, or external identity provider
- File storage: none, local filesystem, AWS S3, S3-compatible storage, Azure Blob, Google Cloud Storage, or another provider
- Testing requirements
- Deployment target when it affects design
- Optional services only when relevant: Docker, Redis, queues, email, payments, OpenAPI, monitoring, audit logs, or scheduled tasks

Do not ask about an ORM when no database is required. Do not ask about uploads when the project has no file-handling requirement.

---

## 5. Architecture and Package Approval Gate

Before creating the project or changing its technology stack, present:

1. Proposed project size and architecture
2. Language, runtime, framework, and package manager
3. Database and ORM/data-access choice
4. Authentication and storage approach, when required
5. Exact direct production dependencies
6. Exact direct development dependencies
7. Short reason for each direct package
8. High-level folder structure
9. Important trade-offs, risks, external costs, or vendor lock-in

Then wait for explicit user approval.

The AI must never install, remove, upgrade, downgrade, or replace a package without explicit approval. This includes editing dependencies in `package.json` directly.

Approval is required again when a later task introduces or changes:

- Framework or runtime tooling
- Database or ORM
- Authentication or authorization technology
- Local, cloud, or hybrid storage
- Validation, logging, testing, linting, formatting, or documentation packages
- Cloud SDKs, Redis, queues, email, payments, monitoring, or other external integrations

An instruction such as “continue,” “fix it,” or “use best practices” is not package approval unless the package list was already shown and clearly approved.

---

## 6. Project Size and Structure

Choose the smallest structure that supports the approved requirements.

### Simple Layered Project

Use for small APIs, utilities, prototypes, or services with limited business logic.

```text
src/
├── config/
├── controllers/
├── routes/
├── services/
├── validators/
├── middlewares/
├── utils/
├── app.js
└── server.js
tests/
```

### Feature-Based Modular Project

Use for medium projects with several resources or ongoing development.

```text
src/
├── modules/
│   └── <feature>/
│       ├── <feature>.controller.js
│       ├── <feature>.service.js
│       ├── <feature>.repository.js
│       ├── <feature>.routes.js
│       └── <feature>.validator.js
├── config/
├── database/
├── middlewares/
├── utils/
├── app.js
└── server.js
tests/
```

### Domain or Enterprise Modular Project

Use for ERP, RMA, inventory, e-commerce, financial, multi-tenant, or workflow-heavy systems.

```text
src/
├── modules/
│   ├── authentication/
│   ├── users/
│   ├── products/
│   ├── orders/
│   └── inventory/
├── shared/
│   ├── database/
│   ├── errors/
│   ├── middleware/
│   ├── validation/
│   ├── logging/
│   ├── storage/
│   └── utilities/
├── config/
├── jobs/
├── events/
├── app.js
└── server.js
tests/
```

Create only folders and files the approved project needs. Do not create empty enterprise layers for a small project.

---

## 7. File Responsibilities

- `server.js`: load validated configuration, connect required infrastructure, start the server, and handle graceful shutdown.
- `app.js`: configure the application, global middleware, routes, not-found handling, and central error handling without starting the server.
- Routes: define endpoints and attach middleware; contain no business logic.
- Controllers: translate HTTP input/output and call services; contain no database queries or complex business rules.
- Services: contain business logic, workflows, transactions, and coordination.
- Repositories: contain ORM or database queries only.
- Validators: define request, parameter, query, and payload validation.
- Middleware: contain reusable request-processing concerns such as authentication, authorization, validation, uploads, rate limiting, and error handling.
- Config: validate environment variables and expose typed or structured configuration.
- Utilities: contain small reusable helpers; do not become a miscellaneous business-logic folder.

Do not duplicate responsibilities across layers.

---

## 8. Database and ORM Rules

- Never choose Prisma, Sequelize, Mongoose, or another data-access tool silently.
- Do not mix multiple ORMs unless the user explicitly approves the complexity.
- Keep database queries out of routes and controllers.
- Use transactions for multi-step operations that must succeed or fail together.
- Add indexes and constraints based on real query and integrity requirements.
- Do not edit or delete applied migrations without explicit approval and a migration-risk explanation.
- Do not run migrations, seeds, resets, truncations, or destructive scripts until the active environment and database target are confirmed.
- Never run development or destructive database actions against production without explicit approval.
- Before destructive database work, require an approved backup and rollback plan.

Use only the structure required by the selected tool:

```text
Prisma:     prisma/schema.prisma, prisma/migrations/, src/database/prisma.js
Sequelize:  src/models/, src/database/migrations/, src/database/seeders/
Mongoose:   feature-owned models or src/models/, based on approved architecture
```

---

## 9. File Upload and Storage Rules

- Never choose local storage, S3, or another provider without user approval.
- Validate file type, MIME type, extension, size, and authorization.
- Generate safe storage names or object keys; never trust the original filename as a path.
- Prevent path traversal, executable uploads, and public exposure of private files.
- Keep storage logic behind a service interface so providers can be changed safely.
- Store only required metadata in the database.

For local storage:

- Keep uploaded content outside source-code folders.
- Exclude uploaded files from Git while preserving required placeholder directories.
- Document persistence, backup, cleanup, and multi-instance limitations.

For S3 or object storage:

- Use least-privilege credentials and private buckets by default.
- Prefer signed URLs when controlled access is required.
- Document bucket, region, object-key, retention, and deletion behaviour.
- Never hard-code cloud credentials.

---

## 10. Security and Data Protection

- Validate all untrusted input and reject unknown or unsafe fields where appropriate.
- Enforce authentication and authorization on every protected action.
- Prevent mass assignment by explicitly selecting writable fields.
- Use parameterized queries or approved ORM APIs; never concatenate untrusted SQL.
- Protect against XSS, CSRF when applicable, injection, path traversal, unsafe redirects, brute force, and denial-of-service abuse.
- Apply controlled CORS, request-size limits, rate limits, and secure headers appropriate to the application.
- Hash passwords using an approved password-hashing algorithm; never store plaintext passwords.
- Keep secrets out of source code, logs, errors, tests, and documentation.
- Do not expose stack traces, SQL, tokens, internal paths, or sensitive records to clients.
- Redact sensitive values from structured logs.
- Apply least privilege to database, cloud, filesystem, and API credentials.
- Never delete data, reset databases, remove files, or make irreversible changes without explicit approval and a clear warning.

---

## 11. Environment and Configuration

- Keep real secrets only in environment variables or an approved secret manager.
- Commit `.env.example`, never a real `.env` file.
- Include variable names and safe examples or descriptions without real secrets.
- Validate required variables at startup and fail with a clear configuration error.
- Separate development, test, staging, and production configuration where necessary.
- Confirm the active environment before executing data-changing commands.

---

## 12. Dependency and Version Management

- Preserve the existing package manager and lock-file format.
- Prefer supported, maintained, stable packages with minimal dependency overlap.
- Inspect installed versions before writing version-specific code.
- Use official documentation matching the installed version when behaviour or syntax is uncertain.
- Never invent package APIs, options, commands, or features.
- Do not update unrelated dependencies while adding an approved package.
- Do not delete or regenerate the lock file unnecessarily.
- Explain migration and compatibility risks before any major-version upgrade.
- Use built-in Node.js functionality when it securely meets the requirement without unnecessary complexity.
- Before approval, check proposed packages for known vulnerabilities, maintenance status, and licence compatibility; disclose material risks.

---

## 13. Git and Existing Work Protection

Before changes:

- Check Git status when the project is under version control.
- Identify uncommitted user changes and avoid overwriting them.
- Preserve the existing architecture, conventions, package manager, and formatting unless a change is approved.

Never run destructive Git commands without explicit approval, including:

```text
git reset --hard
git clean -fd
git checkout -- <file>
git restore --source=<ref>
force push
```

Do not create a remote repository, commit, push, rewrite history, or force-update branches unless explicitly requested.

After changes, review the diff and remove accidental or unrelated modifications.

---

## 14. API and Error-Handling Standards

- Use consistent route naming, status codes, response shapes, pagination, filtering, and versioning.
- Return `201` for successful creation, `204` only when no response body is returned, and appropriate `4xx` or `5xx` codes for errors.
- Use centralized error handling and application-specific error types where useful.
- Log operational details securely while returning safe, actionable client messages.
- Do not silently swallow errors or expose internal implementation details.
- Add idempotency, concurrency control, or request correlation when the business workflow requires it.
- Preserve backward compatibility for existing APIs and database schemas unless a breaking change is explicitly approved and documented.

---

## 15. Testing and Verification

For complex work, define a short plan and completion criteria before editing.
- Run existing tests before changes when practical to establish a baseline and separate pre-existing failures from new ones.

After each meaningful change, run the narrowest applicable checks available in the project:

- Syntax or startup validation
- Lint and format checks
- Unit, integration, or end-to-end tests
- Type checks
- Build commands
- Migration validation
- Security-relevant tests

Rules:

- Test important success, validation, authorization, not-found, conflict, and failure paths.
- Do not weaken or delete tests merely to make a failing change pass.
- Fix errors caused by the AI’s changes.
- If a check cannot be run, state exactly why and do not report it as passed.
- After two unsuccessful speculative attempts, stop, reassess the root cause, inspect logs and official documentation, and explain the findings before making further broad changes.
- Remove temporary debugging code, test credentials, and generated clutter before completion.

A task is not complete until changed code has been checked for syntax errors, broken imports, configuration issues, dependency compatibility, and consistency with the approved architecture.

---

## 16. Required Project Files

Create only applicable files, normally including:

- `package.json` and lock file
- `src/` application code
- `.gitignore`
- `.env.example`
- `README.md`
- Tests required by the approved scope
- ORM schema, migrations, or models when a database is used
- Docker or deployment files only when approved

Recommended scripts should reflect installed tools and actual project needs, for example:

```json
{
  "scripts": {
    "dev": "...",
    "start": "...",
    "test": "...",
    "lint": "...",
    "format": "..."
  }
}
```

Do not add scripts for tools that are not installed.

---

## 17. Documentation Requirements

Keep `README.md` accurate and project-specific. Include, when applicable:

- Purpose and approved stack
- Prerequisites and installation
- Environment variables
- Database, migration, and seed setup
- Storage and external-service setup
- Development, production, test, lint, type-check, and build commands that actually exist
- Main folder structure and entry points
- API overview
- Security and deployment notes

Do not leave generic template text or document commands that were not implemented.

---

## 18. Completion Report

Before reporting completion, verify:

- This rule file was not changed except through explicit user authorization.
- Only approved architecture and packages were used.
- No unrelated files or user changes were overwritten.
- Required checks were run and their results are known.
- Remaining risks, failures, skipped checks, and user actions are disclosed.

Provide a concise report containing:

- Files created, changed, moved, or deleted
- Approved dependencies added, removed, or changed
- Commands executed
- Verification results
- Unresolved issues or skipped checks
- Exact command to run or start the project

Do not paste every generated file unless requested.

---

## 19. Required Workflow Summary

For `create project <project_name>`:

```text
1. Read the rules and inspect the workspace.
2. Ask only the relevant missing requirement questions.
3. Select the smallest suitable architecture.
4. Present the architecture and exact direct-package plan.
5. Wait for explicit approval.
6. Create only the approved files and install only approved packages.
7. Verify the project with applicable checks.
8. Review the diff and protect existing work.
9. Report results honestly and identify remaining actions.
10. Ask for approval again before any future package or architecture change.
```

# Node.js Project Rules

## 1. Scope

Apply these rules whenever creating, configuring, extending, or maintaining a Node.js project, including:

```text
create project <project_name>
```

This is an AI task trigger, not a terminal command or request for a project-generator CLI.

## 2. Rule Protection

- Read this file before planning or changing anything.
- Treat every applicable rule as mandatory.
- Do not edit, delete, rename, move, override, weaken, or bypass this file without explicit approval for the exact rule change.
- Do not create conflicting or nested AI instruction files without explicit approval.
- If a request conflicts with these rules, stop the conflicting action, explain the conflict, and offer a compliant alternative.

## 3. General Working Rules

- Inspect requirements, existing files, `package.json`, lock files, schemas, versions, Git status, and project conventions before changing code.
- Make the smallest change that fully satisfies the approved task.
- Do not modify, reformat, rename, delete, or refactor unrelated working code.
- Preserve uncommitted user changes and existing behaviour unless a change is approved.
- Ask only for missing decisions that materially affect implementation; do not repeat answered questions.
- Never claim that a command, test, build, migration, or installation succeeded unless it actually ran successfully.

## 4. New-Project Discovery

Before creating files or selecting packages, confirm only the relevant missing requirements:

- Purpose and main features
- JavaScript or TypeScript
- Node.js version and package manager
- Application type and framework
- Database and ORM/data-access tool
- Authentication and authorization
- File storage: none, local, object storage, or hybrid
- Testing, deployment, and external-service needs

Do not ask about technology that the project does not require.

## 5. Mandatory Approval Gate

Before creating a new project or changing its stack, present:

1. Proposed architecture and folder structure
2. Language, framework, database, ORM, authentication, and storage choices
3. Exact direct production and development packages
4. Brief reason for each package
5. Important risks, trade-offs, costs, or vendor lock-in

Wait for explicit approval before proceeding.

Never install, remove, upgrade, downgrade, replace, or add a dependency to `package.json` without explicit approval. Approval is required again for later changes involving frameworks, databases, ORMs, authentication, storage, validation, logging, testing, cloud SDKs, Redis, queues, email, payments, monitoring, or other integrations.

“Continue,” “fix it,” or “use best practices” is not package approval unless the exact package plan was already shown and approved.

## 6. Architecture

Choose the smallest structure that supports the approved requirements:

- **Simple:** shared `routes/`, `controllers/`, `services/`, `validators/`, and `middlewares/`.
- **Modular:** `src/modules/<feature>/` containing the feature’s route, controller, service, repository, and validator.
- **Enterprise/domain:** domain modules plus shared database, errors, validation, logging, storage, events, and jobs.

Create only required folders and files. Do not add empty enterprise layers to a small project.

Responsibilities:

- `server`: configuration, infrastructure startup, server startup, and graceful shutdown.
- `app`: middleware, routes, not-found handling, and central error handling.
- Routes: endpoint mapping only.
- Controllers: HTTP input/output only.
- Services: business logic, workflows, and transactions.
- Repositories: database queries only.
- Validators: request, parameter, query, and payload validation.
- Middleware: reusable request-processing concerns.
- Config: validated environment configuration.
- Utilities: small reusable helpers, not business logic.

## 7. Database and Storage

- Never select or mix ORMs, databases, or storage providers without approval.
- Keep database queries out of routes and controllers.
- Use transactions for operations that must succeed or fail together.
- Add constraints and indexes from real integrity and query requirements.
- Do not edit or delete applied migrations without explicit approval and a risk explanation.
- Confirm the active environment and database before migrations, seeds, resets, truncations, or data-changing scripts.
- Never perform destructive or development database actions against production without explicit approval.
- Require an approved backup and rollback plan before destructive database work.
- Preserve API and schema backward compatibility unless a breaking change is explicitly approved and documented.

For uploads:

- Validate authorization, size, MIME type, extension, and safe filenames or object keys.
- Prevent path traversal, executable uploads, and unintended public access.
- Keep storage behind a service interface and credentials outside source code.
- Exclude local uploads from Git; use private object storage and signed access where appropriate.

## 8. Security and Configuration

- Validate untrusted input and explicitly select writable fields.
- Enforce authentication and authorization on every protected action.
- Use parameterized queries or safe ORM APIs; never concatenate untrusted SQL.
- Protect against injection, mass assignment, XSS, CSRF when applicable, path traversal, brute force, unsafe redirects, and request abuse.
- Apply appropriate CORS, secure headers, rate limits, and request-size limits.
- Hash passwords with an approved algorithm; never store plaintext passwords.
- Keep secrets out of code, Git, logs, errors, tests, and documentation.
- Store real secrets in environment variables or an approved secret manager.
- Commit `.env.example`, never real `.env` values.
- Validate required environment variables at startup.
- Redact sensitive log data and never expose stack traces, SQL, tokens, internal paths, or private records to clients.
- Never perform irreversible file or data deletion without explicit approval and a clear warning.

## 9. Dependencies, Versions, and Git

- Preserve the existing package manager and lock-file format.
- Inspect installed versions before writing version-specific code.
- Use official documentation when package behaviour or syntax is uncertain.
- Never invent APIs, configuration options, commands, or package features.
- Prefer maintained, stable packages with minimal overlap and use built-in Node.js functionality when suitable.
- Before approval, check proposed packages for known vulnerabilities, maintenance status, licence compatibility, and major compatibility risks.
- Do not update unrelated dependencies or regenerate the lock file unnecessarily.
- Explain migration risks before major-version upgrades.
- Do not commit, push, rewrite history, create remotes, force-push, or run destructive Git commands without explicit approval.
- Review the final diff and remove accidental or unrelated changes.

## 10. API and Error Handling

- Use consistent routes, response shapes, status codes, pagination, filtering, and versioning.
- Use centralized error handling and meaningful application errors.
- Return safe client messages and log operational details securely.
- Do not swallow errors or expose internal implementation details.
- Add idempotency, concurrency control, or request correlation when required by the workflow.

## 11. Testing and Verification

- For complex work, define a short plan and completion criteria before editing.
- Run existing tests first when practical to identify pre-existing failures.
- After changes, run the narrowest applicable syntax, lint, format, type, build, test, migration, and security checks.
- Test important success, validation, authorization, not-found, conflict, and failure paths.
- Do not weaken or delete tests merely to make changes pass.
- Fix failures caused by the AI’s changes.
- If a check cannot run, state why; never report it as passed.
- After two failed speculative attempts, reassess using logs and official documentation before making broader changes.
- Remove temporary debugging code, credentials, and generated clutter.

A task is incomplete until changed code is checked for syntax errors, broken imports, configuration problems, dependency compatibility, and consistency with the approved architecture.

## 12. Required Files and Documentation

Create only applicable files, normally including:

- `package.json` and its lock file
- `src/`
- `.gitignore`
- `.env.example`
- `README.md`
- Approved tests
- Database schema, models, or migrations when required
- Deployment files only when approved

Scripts and documentation must match tools and commands that actually exist. Keep `README.md` project-specific and include setup, environment, database, storage, commands, structure, and deployment information when applicable.

## 13. Completion Report

Before completion, verify that:

- This rule file was not changed without authorization.
- Only approved architecture and packages were used.
- No unrelated code or user work was overwritten.
- Applicable checks were run and results are known.
- Risks, failures, skipped checks, and required user actions are disclosed.

Report concisely:

- Files changed
- Dependency changes
- Commands run
- Verification results
- Unresolved issues
- Exact start or run command

## 14. Required Workflow

```text
1. Read rules and inspect workspace.
2. Ask only relevant missing questions.
3. Propose the smallest suitable architecture and exact package plan.
4. Wait for explicit approval.
5. Create only approved files and install only approved packages.
6. Verify with applicable checks.
7. Review the diff and protect existing work.
8. Report results honestly.
9. Ask again before future package or architecture changes.
```

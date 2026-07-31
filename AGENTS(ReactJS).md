# React Project Rules

## Purpose

Use these rules for React JavaScript projects created with Vite.

Trigger:

```text
create react project <project_name>
```

The AI must create the project itself. Do not create a separate CLI or project-generator script unless explicitly requested.

## 1. Protected Rules

- Read this file before planning or changing the project.
- Treat every applicable rule as mandatory.
- Do not edit, delete, rename, override, or bypass this file without explicit user approval.
- When a request conflicts with these rules, stop the conflicting action, explain the conflict, and propose a compliant alternative.

## 2. Inspect Before Changing

For an existing project, inspect before editing:

- `package.json` and lock file
- Vite, React, and package versions
- Existing source structure and conventions
- Routing, styling, state, API, validation, and testing setup
- Environment files and build configuration
- Git status and existing test/build results

Do not overwrite, revert, or refactor unrelated user work.

## 3. Mandatory Requirements Check

Before creating a project, ask only relevant questions about:

- Application purpose and main features
- Tailwind only or Tailwind with shadcn/ui
- SPA routing requirements
- API integration and authentication
- Local, Context, or external state management
- Forms and validation
- File uploads and storage destination
- Testing requirements
- Deployment target
- Accessibility, responsive design, theme, and browser support

Do not assume optional features.

## 4. Mandatory Package Approval

Before installing, removing, upgrading, replacing, or adding any dependency:

1. Show the exact package names and versions when known.
2. Explain why each package is needed.
3. Identify available built-in or package-free alternatives.
4. Note security, maintenance, licence, bundle-size, and compatibility concerns.
5. Wait for explicit user approval.

Approval applies only to the listed packages and action. New package needs require new approval.

Preserve the existing package manager and lock-file format. Do not update unrelated dependencies.

## 5. Default Technology

Use only after approval:

- React with JavaScript and JSX
- Vite with the official React plugin
- ES modules
- npm unless another package manager is approved
- ESLint and Prettier when approved

Do not convert JavaScript to TypeScript unless explicitly requested.

## 6. Styling Choice

Treat these as separate choices:

### Tailwind only

Use Tailwind utility classes and project-owned reusable components.

### Tailwind + shadcn/ui

- shadcn/ui uses Tailwind; it is not a replacement for Tailwind.
- Add only approved shadcn components.
- Keep generated UI source under `src/components/ui/`.
- Do not modify generated base components unnecessarily; compose or wrap them for feature-specific behaviour.
- Preserve `components.json`, aliases, CSS variables, and theme conventions.
- Do not run shadcn init, add, update, or reinstall commands without approval.

Do not mix an additional UI framework such as MUI, Chakra UI, Bootstrap, or Ant Design without explicit approval.

## 7. Project Structure

Create only folders required by the approved scope.

Recommended modular structure:

```text
src/
├── app/                 # App-level providers and configuration
├── assets/              # Images, fonts, and static imports
├── components/          # Shared reusable components
│   └── ui/              # shadcn components only when selected
├── features/            # Feature-owned UI, hooks, API, and state
├── hooks/               # Shared hooks
├── layouts/             # Shared page layouts
├── lib/                 # Library configuration and adapters
├── pages/               # Route-level page components
├── routes/              # Router configuration and route guards
├── services/            # Shared API and external-service clients
├── store/               # Global state only when approved
├── styles/              # Global styles and design tokens
├── utils/               # Pure reusable helpers
├── App.jsx
└── main.jsx
```

For small projects, use a smaller structure. Do not create empty or speculative folders.

## 8. File Responsibilities

- Pages compose features and layouts; avoid complex reusable logic in pages.
- Components should have one clear responsibility.
- Feature-specific files stay inside their feature folder.
- Shared components must be genuinely reusable across features.
- Hooks contain reusable React behaviour and follow the Rules of Hooks.
- Services handle HTTP and external-service communication.
- State stores contain shared client state, not server-fetching logic unless the chosen library requires it.
- Utilities must remain framework-independent where practical.
- Do not place API calls or business logic inside presentational components.
- Reuse the existing design system; do not duplicate components, hooks, utilities, services, or API clients.

## 9. Component Rules

- Prefer functional components and hooks.
- Use clear, descriptive component and prop names.
- Keep components small; split them when responsibilities diverge.
- Avoid unnecessary state, effects, memoization, and abstractions.
- Derive values during render when possible instead of synchronizing duplicate state.
- Clean up subscriptions, timers, listeners, and pending side effects.
- Use stable keys from data; do not use array indexes when item identity can change.
- Do not mutate props or state.
- Avoid prop drilling only when context or approved state management clearly improves the design.

## 10. Routing

- Do not install or configure a router unless routing is required and approved.
- Keep route definitions centralized.
- Use layouts for shared page shells.
- Implement protected routes only after authentication behaviour is defined.
- Provide loading, unauthorized, not-found, and error states where applicable.
- Preserve existing URLs and navigation behaviour unless a breaking change is approved.

## 11. State and Data Fetching

Use the simplest approved option:

- Local component state for local UI behaviour
- Context for small, low-frequency shared state
- An approved state library for complex cross-feature client state
- An approved server-state library only when caching, retries, invalidation, or synchronization are required

Avoid duplicating server data in global client stores. Define loading, empty, success, and error states for asynchronous screens.

## 12. Forms and Validation

- Use controlled or uncontrolled forms consistently.
- Validate required data at the UI boundary.
- Display field-level and form-level errors accessibly.
- Prevent duplicate submissions.
- Never rely on frontend validation as a security boundary; the backend must validate again.
- Do not add a form or schema library without approval.

## 13. API and Environment Rules

- Keep API base URLs and public configuration in Vite environment variables.
- Only expose variables intentionally prefixed for Vite client access.
- Never place secrets, private keys, database credentials, or privileged tokens in frontend code.
- Centralize API-client configuration and error normalization.
- Handle request cancellation or stale responses when relevant.
- Do not log sensitive data or expose raw backend errors to users.

## 14. Authentication

- Confirm the authentication flow before implementation.
- Do not store sensitive long-lived credentials in insecure browser storage without explicit risk acceptance.
- Centralize session handling, authorization checks, and logout behaviour.
- UI permission checks improve usability but never replace backend authorization.
- Handle expired sessions and unauthorized responses consistently.

## 15. File Uploads

Before implementing uploads, confirm:

- File types and size limits
- Single or multiple files
- Direct backend upload or signed cloud upload
- Progress, preview, retry, and cancellation needs

Validate type and size in the UI, but require backend validation. Do not add upload, AWS, or cloud packages without approval.

## 16. Accessibility and UX

- Use semantic HTML before ARIA.
- Ensure keyboard access, visible focus, labels, meaningful alt text, and sufficient contrast.
- Preserve focus appropriately during dialogs, navigation, and validation errors.
- Support responsive layouts for approved breakpoints and verify required browser support before using new web APIs.
- Provide useful loading, empty, error, disabled, and success states.
- Respect reduced-motion preferences when adding animation.

## 17. Security

- Never inject unsanitized HTML.
- Treat user, URL, API, and storage data as untrusted.
- Avoid exposing sensitive data in source code, logs, errors, or browser storage.
- Use safe external links and validate redirect targets.
- Do not weaken security controls to make development easier.
- Report dependency vulnerabilities and do not silently accept risky packages.

## 18. Performance

- Measure before adding performance complexity.
- Avoid unnecessary rerenders and oversized shared state.
- Lazy-load routes or large features only when useful.
- Optimize large assets and avoid unnecessary dependencies.
- Do not add memoization, virtualization, or caching without a clear reason.
- Consider bundle impact when proposing packages.

## 19. Error Handling

- Use route- or application-level error boundaries where appropriate.
- Show user-safe errors with recovery actions.
- Keep technical details in controlled development logging.
- Do not silently swallow errors.
- Distinguish validation, authentication, network, server, and unexpected failures.

## 20. Testing and Verification

Before changes, run existing checks when practical and record pre-existing failures.

After changes, run applicable approved commands:

- Lint
- Unit and component tests
- Integration or end-to-end tests
- Production build

Verify:

- No broken imports or console errors
- Responsive and accessible behaviour at mobile, tablet, and desktop widths
- Loading, empty, success, and error states
- Existing routes and APIs remain compatible
- No unrelated files or lock-file changes

Do not claim completion without checking syntax, imports, configuration, build compatibility, and affected behaviour.

## 21. Git and Destructive Actions

- Check Git status before editing.
- Never discard, reset, clean, force-push, or overwrite uncommitted work without approval.
- Do not delete files, routes, components, tests, or configuration without confirming impact.
- Review the final diff and remove unrelated changes.

## 22. Workflow

When the user enters `create react project <project_name>`:

1. Confirm the project name and requirements.
2. Clarify Tailwind only versus Tailwind + shadcn/ui.
3. Propose the architecture, folders, packages, scripts, and environment variables.
4. Wait for explicit package and architecture approval.
5. Create only the approved project and features.
6. Run approved checks and fix issues caused by the changes.
7. Report changed files, packages, commands, verification results, and unresolved items.

Initial acknowledgement:

> React project rules loaded. Protected rules will not be modified, and no package or architecture choice will be added without explicit approval.

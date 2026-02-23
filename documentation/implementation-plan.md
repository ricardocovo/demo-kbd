# Implementation Plan Template

Purpose
- Provide a reproducible plan template for implementing features, projects, or migrations. Capture scope, tasks, owners, and validation steps.

Template structure

1. Title
- Short, descriptive title of the implementation.

2. Objective
- One-paragraph summary of goals and success criteria.

3. Scope (In-scope / Out-of-scope)
- Clear bullet lists of what is included and excluded.

4. Deliverables
- Concrete artifacts to be produced (APIs, services, migrations, docs).

5. Milestones & timeline (no dates required)
- Milestone A: description and acceptance criteria
- Milestone B: description and acceptance criteria

6. Tasks & owners
- Task 1: concise description — Owner (github handle)
- Task 2: concise description — Owner

7. Dependencies
- Other teams, libraries, infra changes, or vendor services required.

8. Risks & mitigations
- Risk: impact — Mitigation: action

9. Testing & validation
- Unit tests, integration tests, perf benchmarks, security scans to run.
- Rollout strategy (canary, dark launch, feature flag) and rollback plan.

10. Operational readiness
- Monitoring dashboards, alerts, runbooks required, and on-call owner.

11. Communication & stakeholder sign-off
- Who needs to approve releases and what channels to use (Slack/email). Sign-off checklist.

12. Post-release tasks
- Cleanup, technical debt tickets, and retrospective notes.

How to use
- Copy this template into a project-specific implementation plan file (e.g., `docs/plans/feature-x-plan.md`).
- Link the plan from the PR that implements the change and update as tasks complete.

Integration with todos
- For execution, reflect tasks into the session SQL `todos` table and track progress (see project onboarding docs for example commands).

Guidance for Copilot sessions
- When generating an implementation plan, populate the template with concrete tasks and suggest reasonable splits for parallel work; avoid scheduling dates without explicit input from the user.

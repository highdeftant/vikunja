# Agentic Life OS

## Status

Initial architecture checkpoint: 2026-09-14.

This fork is the foundation for a self-hosted personal operating system built on
Vikunja's task, project, permission, and calendar primitives. The goal is not to
replace Vikunja's core immediately. The goal is to add an agent-friendly layer
that can safely turn ideas into researched, scheduled, and verified work.

## Product direction

The system should organize:

- projects and tasks;
- time-blocked work;
- health and homelab data;
- recurring routines and reminders;
- research notes and follow-up actions;
- agent proposals and approvals;
- notifications across phone, laptop, and desktop.

Agents may inspect context, research options, propose changes, and create draft
work. Destructive or externally visible actions should require explicit approval
until a policy grants them permission.

## Keep from upstream

Keep upstream-compatible wherever practical:

- existing task/project/user models;
- permissions and sharing;
- recurring tasks and due dates;
- Kanban, list, table, and Gantt views;
- CalDAV compatibility;
- generated API clients;
- normal Vikunja web UI behavior.

New API routes must use `/api/v2`. Do not extend frozen `/api/v1` routes.

## Add as a separate product layer

### 1. Work blocks

A task can become a scheduled work block without changing its basic task
semantics. A work block needs:

- task ID;
- start and end time;
- timezone;
- source (`user`, `agent`, `recurrence`, or `import`);
- status (`planned`, `active`, `completed`, `skipped`);
- optional notification policy.

The first implementation should use existing start/end dates where possible and
avoid a second scheduler until the current model is proven insufficient.

### 2. Agent activity and proposals

Agents need an auditable record of what they inspected and proposed:

- actor/provider identifier;
- request or trigger;
- affected entity IDs;
- proposed mutation;
- evidence links or notes;
- approval state;
- execution result;
- timestamps.

Agents should be able to create a proposal instead of directly mutating a task
when confidence, scope, or safety is unclear.

### 3. Research-to-task workflow

A research run should be able to produce:

- sources;
- claims and uncertainty;
- recommended next steps;
- draft tasks;
- estimated duration;
- dependencies;
- a suggested schedule.

The user can approve the resulting task set as a batch.

### 4. Notification policy

Keep notification delivery separate from task storage. The current ntfy bridge
is the first delivery adapter. Future adapters can include desktop, Android,
email, or other gateways without coupling them to the task model.

A notification should include:

- stable event ID;
- task/project reference;
- severity;
- title and body;
- deduplication key;
- delivery attempts and result.

### 5. Agent access

The existing Vikunja MCP server remains a compatibility adapter. Long term, add
a first-party agent API/MCP surface that supports:

- list/search tasks and projects;
- inspect schedules and available work windows;
- create draft tasks and proposals;
- approve or reject proposals;
- schedule or reschedule work blocks;
- record research evidence;
- request notifications.

The first-party surface should reuse authorization and service logic rather than
calling the public HTTP API from inside the server.

## Initial implementation order

1. Add this product boundary and document the domain vocabulary.
2. Inventory existing task/date/reminder models and generated v2 API patterns.
3. Implement read-only schedule and availability queries.
4. Add agent proposal storage and audit records.
5. Add work-block scheduling with conflict detection.
6. Add approval and batch execution.
7. Replace the external MCP adapter with first-party agent operations.
8. Integrate scale and OpenStrap health data as separate sources.

## Design constraints

- Preserve upstream mergeability until the new layer is stable.
- Prefer existing models and services over duplicate storage.
- Use migrations for persistent schema changes.
- Use generated clients for frontend access to new routes.
- Keep raw health/device data outside the task database unless there is a clear
  relational need; link to source records instead.
- No autonomous destructive actions by default.
- Every agent mutation must be attributable and reviewable.
- Every claimed external action needs read-back verification.

## Explicit non-goals for the first iteration

- replacing Vikunja's task UI;
- building a full autonomous calendar optimizer;
- importing every health metric into the task database;
- adding a proprietary notification transport;
- breaking compatibility with standard Vikunja clients.

## Success criteria

The first useful release should let an agent:

1. read the user's open projects and available schedule;
2. create a researched proposal containing tasks and time blocks;
3. show conflicts and uncertainty before execution;
4. receive user approval;
5. apply the batch safely;
6. notify subscribed devices;
7. leave an audit trail that another agent or the user can inspect.

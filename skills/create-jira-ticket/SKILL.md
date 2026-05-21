---
name: create-jira-ticket
description: Create a Jira ticket (optionally with subtasks) from a description, Slack thread, or conversation context. Confirm a small set of inputs first, then create the parent and any subtasks via the Jira MCP tools. Use when the user asks to "create a Jira ticket" / "raise a ticket" / "make a ticket with subtasks".
---

# Create Jira Ticket

Goal: turn an unstructured request (Slack thread excerpt, free-form description, conversation context) into a well-formed Jira issue — and, when asked, a small set of subtasks linked to it.

## Inputs to confirm with the user (only if missing)

Ask these as a single short numbered list, then proceed once answered. Do not ask anything else.

1. **Project key** (e.g. `OPH`). Infer from the context (channel, recent ticket prefixes) and ask the user to confirm.
2. **Parent summary** — propose one based on the source material; ask only if it should be different.
3. **Subtask breakdown** — propose the splits explicitly (e.g. "Subtask 1: …, Subtask 2: …") and confirm.
4. **Reporter / assignee** — default reporter is the user themselves; default assignee is unassigned. Confirm if unclear.
5. **Labels / components / priority** — only set if the user names them. Otherwise leave defaults.
6. **Description** — confirm whether to include a quoted/summarised source excerpt (e.g. Slack thread).

> **Exception:** If the user gives explicit direction (e.g. "single ticket under OPH-100", "make it short", "no subtasks"), skip confirmation and execute directly.

## Steps

1. **Look up project conventions** — for unfamiliar projects, fetch one recent issue (`jira_get_issue`) on the same project to learn:
   - Required custom fields (e.g. `OPH` requires `customfield_10310` "NU - Investment area" and `customfield_10311` "NU - Investment Category").
   - The valid sub-issue type name (e.g. `Sub-task`, **not** `Subtask`).
   - Default values for those custom fields (copy what neighbouring tickets use).
2. **Create the parent** with `jira_create_issue`:
   - `issue_type`: `Task` by default. Use `Story` when the user says "story", when the parent is an Epic, or when the work is user-facing. Use `Bug` when the user says "bug".
   - `description`: Markdown. Headings as `## Heading`, lists as `-`, inline code with backticks. Jira converts markdown to wiki markup automatically — keep headings short so they translate cleanly.
   - Pass required custom fields via `additional_fields` JSON, e.g. `{"customfield_10310": {"value": "Platform Transformation"}, "customfield_10311": {"value": "Shaping/Transformational Work"}}`.
3. **Create each subtask** with `jira_create_issue` (only when the user explicitly asked for subtasks or when work items are genuinely independent and assignable to different people):
   - `issue_type`: `Sub-task` (use the project's actual sub-issue type — confirm via the lookup in step 1).
   - Link to parent in `additional_fields`: `{"parent": "OPH-341"}` — **string key**, not an object. `{"parent": {"key": "OPH-341"}}` is rejected with `expected 'key' property to be a string`.
   - Carry the same required custom fields as the parent unless the user says otherwise.
4. **Report** the created issue keys + URLs back to the user as a short bullet list (parent first, then subtasks). Do not transition statuses, add watchers, or change anything else without being asked.

## Single-ticket mode (preferred when user says "short" / "single ticket")

When the user asks for a short ticket or a single ticket (no subtasks), merge all work items into the **same Story or Task**:

- Use numbered sub-sections (`### 1. …`, `### 2. …`) under a `## Work` heading instead of creating formal subtasks.
- Keep each sub-section to 2-3 bullet points.
- Keep acceptance criteria to 4-5 bullets max.
- Drop the `## Source` section unless the user specifically asks for it or the provenance is critical.
- Drop `## Out of scope` unless the user explicitly wants it.

This produces a compact, readable ticket that Jira renders well.

## Description style

- Use Markdown headings (`##`) — Jira converts these to `h2.` automatically.
- Keep it factual and structured: **Background**, **Goals** / **Acceptance criteria**, **Out of scope**, **Source** (when based on a Slack thread or other external discussion).
- For sibling subtasks that depend on each other, add a short `## Depends on` section naming the sibling. Do not create a formal Jira issue link unless explicitly asked.
- When summarising a Slack thread, attribute the participants (e.g. "Slack thread in `#channel-name` — Person A / Person B"). Do not paste the entire transcript.
- **Brevity rule:** When the user says "short", condense by merging work items into numbered sub-sections, dropping non-essential sections, and limiting AC to the 2-3 most important bullets.

## Hard rules

- **Never assume the project key.** Ask if not stated, unless the user explicitly named it.
- **Never set assignee** unless the user named one.
- **Never invent labels / components / priority.** Leave defaults if the user didn't specify.
- **Never create issue links** (`Blocks`, `Relates to`, etc.) automatically — only when the user asks for them.
- **Never transition** the new issue (don't move it out of the project's default initial status).
- Sub-issue type is project-specific — verify against an existing ticket in the project before creating.
- Parent linkage uses `{"parent": "<KEY>"}` — string, not object.
- If a required custom field is missing on creation, the API returns `<FieldName> is required`. Look up the field via `jira_search_fields`, copy the value from a recent neighbour issue, retry. Do **not** ask the user for custom-field values they wouldn't know.

## Common pitfalls

- `issue_type: "Subtask"` → rejected. Use `Sub-task`.
- `additional_fields.parent: {"key": "OPH-X"}` → rejected. Use `additional_fields.parent: "OPH-X"`.
- `jira_get_field_options` against a global context returns `[]` for project-scoped option fields — fall back to inspecting a recent issue in the same project.

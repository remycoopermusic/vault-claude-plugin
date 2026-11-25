---
name: action-management
description: Safely find and manage VAULT actions. Use for action lookup, creation, field updates, and confirmed completion state transitions.
---

# Action management

## Overview

Keep action changes tied to the authenticated VAULT workspace. Resolve the intended action before changing fields or transitioning it to complete.

## Tool boundary

- Locate actions with `vault_list_actions`; inspect the selected action with `vault_get_action`.
- Create with `vault_create_action`, update fields with `vault_update_action`, and complete with `vault_complete_action`.
- Treat completion as a state transition. Do not claim deletion or any action operation outside these tools.

## Mutation gate

For every action mutation, follow this order exactly:

1. **Read** the current action object.
2. **Clarify** any ambiguous match using title, due date, assignee, and stable ID.
3. **Summarize** every changed field and the resulting state transition.
4. **Confirm** the mutation. Explicitly confirm completion because it is a state transition.
5. Create one stable `idempotencyKey` for the logical mutation and reuse that key on a retry.
6. **Write** with `vault_create_action`, `vault_update_action`, or `vault_complete_action`.
7. **Verify** with `vault_get_action` and report the saved final state.

## Resolve and change actions

- Use `vault_list_actions` to find candidates. If multiple candidates match, show title, due date, assignee, and ID, then ask the user to choose; never infer the target.
- Read before an update or completion: call `vault_get_action` and preserve all unchanged fields from the full current object.
- For `vault_create_action`, clarify title, due date, assignee, and requested initial state. Summarize the new action before creating it and verify the created ID.
- For `vault_update_action`, summarize field-level old and proposed values. Require confirmation for changes that alter status or assignment.
- For `vault_complete_action`, state the current status and proposed completed status, then obtain explicit confirmation immediately before writing.
- Use one stable mutation key as the `idempotencyKey` per logical mutation. If a request times out, retry with that same key and do not issue a second create or completion.
- Verify every saved action with `vault_get_action` and report only the final action fields needed by the user.
- Use `vault_get_action` to verify the final action state after every successful write or replay.

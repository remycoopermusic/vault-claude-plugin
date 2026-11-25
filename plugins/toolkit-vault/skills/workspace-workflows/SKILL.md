---
name: workspace-workflows
description: Discover and manage VAULT Inboxes, Briefs, and workspace custom metadata through the hosted action catalog. Use when a request involves submissions, briefs, custom fields, or an unfamiliar workspace workflow.
---

# Workspace workflows

## Find the supported action

1. **Read** the connected identity with `vault_get_me`, then call `vault_list_capabilities` with `domain: "inboxes"`, `domain: "briefs"`, or `domain: "metadata"`. These domains include Inbox submissions, Brief publishing and receiving, and workspace custom fields.
2. **Clarify** the exact workspace, resource, and requested outcome. A matching action with `available: false` is not permission to access data or make a change. Its scope or workspace capability may be missing. A manual action must be completed in VAULT by the user.
3. Call `vault_list_capabilities` again with the selected `actionId`. Use its exact input schema, mode, and required scopes. Never guess an action ID or argument shape from a title. If the five generic tools are absent in a personal ChatGPT developer connection, ask the user to open Settings → Plugins → VAULT → Manage → Information → Refresh and start a new chat. Packaged skill changes need a new plugin version.

## Request missing OAuth access

If the user explicitly requested a read action that reports `available: false`, call that exact read action once with its validated arguments. The server blocks access before returning data and may send an `insufficient_scope` challenge so ChatGPT can ask the user to approve the additional permission. Let the user review the OAuth consent screen. After reauthorization, discover the action again and continue only if it is available.

For a write or delete action, never call it only to test a permission. First inspect the target, summarize the exact proposed change, and get the user's explicit confirmation. If the selected action then returns `insufficient_scope`, let the user decide whether to grant it; after reauthorization, rediscover and recheck the target before any retry. A missing workspace capability or a declined scope remains a stop condition.

## Mutation gate

4. **Summarize** the current target and proposed change. For a Brief send, publication, audience change, submission change, or metadata replacement, show the affected people and content when available.
5. **Confirm** the user's request before writing. Consequential actions can return `awaiting_confirmation`; show the full returned preview and obtain a separate explicit approval of its exact targets and effects. Discovery and a prior generic request are not approval of that preview.
6. Reuse one stable `idempotencyKey` for the logical change. Do not invent a new key when retrying the same request.
7. **Write** with `vault_run_action` using the selected `actionId` and the typed `arguments`. If the result is `awaiting_confirmation`, call `vault_confirm_action` only after the explicit preview approval. Use `vault_cancel_action` if the user declines.
8. **Verify** the saved state with the matching read action or `vault_get_action_status` after a timeout. Report any partial or pending state instead of claiming completion.

Broad `vault.all.read`, `vault.all.write`, and `vault.all.delete` grants are optional. ChatGPT can request them only through an explicit action's scope challenge and the user's OAuth consent; advanced PAT clients can select them when creating a token. They never replace workspace permissions. A destructive action also requires its specific confirmation flow. Do not request new grants silently or claim a permission error means the VAULT server is offline.

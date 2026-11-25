---
name: playlist-and-sharing
description: Safely manage VAULT playlists and public sharing. Use for playlist membership, smart rules, and public-link changes that need access disclosure.
---

# Playlist and sharing

## Overview

Keep playlist membership and sharing changes explicit. Treat a public link as an external access effect and disclose its final access settings before any change.

## Tool boundary

- Locate playlists with `vault_list_playlists` and inspect one with `vault_get_playlist` before changing it.
- Create private playlist state with `vault_create_playlist` and change membership or rules with `vault_update_playlist`.
- Locate links with `vault_list_public_links` and inspect one with `vault_get_public_link` before creating or changing a link.
- Use `vault_create_public_link` and `vault_update_public_link` only for the supported public-link fields. Do not invent quick-link or recipient-specific mutations.

## Mutation gate

For every playlist or sharing mutation, follow this order exactly:

1. **Read** the current playlist or public-link object.
2. **Clarify** the target playlist, link, song IDs, rules, and intended access.
3. **Summarize** added and removed song IDs, rule changes, and every access or expiry change.
4. **Confirm** the proposed mutation. Require explicit confirmation before creating or changing a public link because it can be accessible outside VAULT.
5. Create one stable `idempotencyKey` for the logical mutation and reuse that key on a retry.
6. **Write** with the selected playlist or public-link tool.
7. **Verify** by fetching the saved playlist or public link and report its final state.

## Playlist membership and rules

- Before `vault_update_playlist`, inspect the current `vault_get_playlist` result, including smart rules and song membership.
- Build set differences by song ID. Do not add duplicate song IDs, and show the exact added and removed IDs in the summary.
- Validate proposed smart filters and rule fields before writing. Keep unchanged rules and membership unless the user asks to replace them.
- For `vault_create_playlist`, clarify its initial membership and rules, use a stable `idempotencyKey`, and verify with `vault_get_playlist`.
- For `vault_update_playlist`, require confirmation for membership removal or rule replacement, reuse the same `idempotencyKey` on retry, and verify with `vault_get_playlist`.

## Public-link disclosure

- Before `vault_create_public_link` or `vault_update_public_link`, inspect current link state with `vault_list_public_links` or `vault_get_public_link` when a link already exists.
- Explain that a public-link result can be accessible outside VAULT. Display proposed expiry and access fields, and require explicit confirmation immediately before the write.
- Use a stable `idempotencyKey` for the logical link mutation and reuse it on retry. Never expose tokens or secret URLs beyond the user-authorized result.
- After writing, fetch the saved public link with `vault_get_public_link` and report the final state without credentials or unapproved secret URLs.

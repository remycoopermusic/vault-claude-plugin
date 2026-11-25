---
name: splits-and-credits
description: Safely review and replace VAULT song splits and credits. Use for contact disambiguation, percentage validation, and explicit credit changes.
---

# Splits and credits

## Overview

Use stable contact and song identities when reviewing or replacing split participants. Treat a split update as a full replacement that requires an explicit user decision.

## Tool boundary

- Search contacts with `vault_search_contacts`; inspect a selected contact with `vault_get_contact`.
- Use `vault_create_contact` only after confirming that no existing contact is the intended person. Use `vault_update_contact` only after inspecting the current contact, and use `vault_link_contact_to_user` only when the user explicitly chooses the link.
- Resolve the song with `vault_get_song` before inspecting or changing its splits. Apply the replacement with `vault_update_song_splits`.

## Replacement gate

For every contact or split mutation, follow this order exactly:

1. **Read** the current song and every selected contact.
2. **Clarify** each participant by stable contact ID. Resolve multiple matches by asking for the intended ID.
3. **Summarize** old versus proposed participants and percentages, including every replacement and contact change.
4. **Confirm** explicitly before any replacement write, contact update, contact link, or contact creation that changes catalog identity.
5. Create one stable `idempotencyKey` for the logical mutation and reuse that key on a retry.
6. **Write** only after validation and confirmation.
7. **Verify** by rereading the saved song with `vault_get_song` and report the final participants and percentages.

## Contact and percentage validation

- Validate finite percentages after normalizing numeric values. Every percentage must be finite, not `NaN`, not infinite, and within the accepted percentage range.
- Require the normalized total split to equal exactly 100. Never infer missing shares, fill a remainder, or round a total into compliance.
- Never silently merge duplicate names. If two contacts have the same or similar name, show their stable IDs and ask which contact is intended.
- A missing contact can be created with `vault_create_contact` only after the user confirms the new identity. An existing contact can be changed with `vault_update_contact` only after its current object is shown.
- Link a contact with `vault_link_contact_to_user` only after displaying the selected contact ID and the target user relationship.

## Split replacement

- Fetch the song with `vault_get_song` and use its stable song ID. Show the old participants and percentages beside the proposed participants and percentages.
- Stop when any participant lacks a stable contact ID, any percentage is non-finite, or the normalized total is not exactly 100.
- Require explicit confirmation immediately before `vault_update_song_splits`. Include the stable `idempotencyKey` in the mutation request and reuse it for a retry of that same replacement.
- Verify the saved result with `vault_get_song`; report the final split state only after the reread succeeds.

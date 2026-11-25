---
name: catalog-management
description: Safely find, read, create, and update VAULT songs, tracks, and catalog metadata, including workspace custom fields. Use for catalog lookup, metadata maintenance, and contact references.
---

# Catalog management

## Overview

Keep catalog work bounded to the authenticated VAULT workspace. Resolve identity and current state before proposing or applying any song, track, or catalog metadata change.

## Tool boundary

- When workspace identity is unclear, begin with `vault_get_me` and show the returned workspace identity before continuing.
- Start song discovery with `vault_find_songs`. Use it before exact `vault_get_song` or `vault_list_song_tracks` reads. Use `vault_list_songs` only for a broad browse, then resolve the requested song with `vault_find_songs`.
- Treat an unfamiliar song property named by the user as a possible workspace custom field, even when they do not say "custom metadata". Use `vault_list_custom_fields` to discover its name, type, field ID, and select option IDs before filtering with `vault_find_songs`. For one song, read `vault_get_song` and, when needed, `vault_get_song_custom_fields`; resolve option labels with the definitions. An empty value means the field is unset, not that access is denied.
- If custom-field read tools are missing from the connected client, report that this client's VAULT tool registration is incomplete. If a custom-field read explicitly returns an authorization error, report the selected workspace and the denied resource without claiming that the entire VAULT connection is unavailable. Never infer permission from a song's visibility alone.
- For tracks, use `vault_list_song_tracks` after the song is resolved, then `vault_get_track` for the exact track object.
- For catalog contacts, use `vault_list_contacts`, `vault_search_contacts`, and `vault_get_contact`. Do not treat a display-name match as identity.
- Use `vault_create_song` for a new song, `vault_update_song_metadata` for metadata replacement, `vault_update_track` for track fields, and `vault_set_main_track` only after inspecting the candidate tracks.

## Mutation gate

For every mutation, follow this order exactly:

1. **Read** the full current song or track object and any related tracks before proposing a change.
2. **Clarify** the target using title, artist, and stable ID. If multiple songs or tracks match, stop and ask which one.
3. **Summarize** the field-level changes, including old and proposed values.
4. **Confirm** the request. Require explicit confirmation for destructive replacements, changing the main track, or any other overwrite.
5. Create one stable `idempotencyKey` for the logical mutation and reuse that same key on a retry.
6. **Write** with the selected mutation tool and caller-supplied `idempotencyKey`.
7. **Verify** by fetching the exact saved song or track and report the saved result.

## Lookup and disambiguation

- Show title, artist, and ID for every candidate returned by `vault_find_songs` or a track lookup. Do not guess when more than one candidate remains.
- Before `vault_get_song` or `vault_list_song_tracks`, retain the selected song ID from `vault_find_songs`.
- Before `vault_update_track` or `vault_set_main_track`, inspect the full current song and track objects. A main-track change must name both the song ID and the selected track ID.
- Before `vault_update_song_metadata`, inspect the full current song object, not just a search result. Present each changed field and retain unchanged fields.

## Create and update

- For `vault_create_song`, clarify required identity and metadata, summarize the proposed new record, then use a stable `idempotencyKey`. Verify with an exact song lookup returned by the create result or by `vault_find_songs` followed by `vault_get_song`.
- For `vault_update_song_metadata`, `vault_update_track`, and `vault_set_main_track`, apply the mutation gate and never send a partial replacement based on stale state.
- If a mutation fails or times out, retry with the same `idempotencyKey`; do not create a second key for the same logical change. Verify after a successful replay as well.

Do not claim or attempt audio upload, playback internals, or deletion. Audio upload is unsupported, playback internals are unsupported, and deletion is unsupported by this workflow.

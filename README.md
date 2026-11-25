# VAULT plugin for Claude

[VAULT](https://vault.toolkit.music) is a music management platform by [Toolkit](https://toolkit.music). This plugin connects Claude to your VAULT workspace through the hosted VAULT MCP server and adds five skills for catalog work, splits and credits, playlists and sharing, tasks, and Inboxes and Briefs.

## Install

In Claude Code:

```
/plugin marketplace add remycoopermusic/vault-claude-plugin
/plugin install toolkit-vault@toolkit
```

Then run `/mcp`, pick `vault` and sign in. Choose the workspace you want Claude to use and approve the requested access.

## Requirements

Direct access needs a VAULT Personal Pro plan or a seat in a paid Team workspace. Every connection is bound to one user and one workspace, and you can disconnect it at any time from the VAULT connection settings at https://vault.toolkit.music/account/mcp.

## What is included

- `vault`: the hosted VAULT MCP server at `https://mcp.vault.toolkit.music/mcp`, with OAuth sign-in
- Skills: `catalog-management`, `splits-and-credits`, `playlist-and-sharing`, `action-management`, `workspace-workflows`

The plugin reads and changes data only when the connection's permissions and your workspace role allow it. Actions that delete data or remove access ask for confirmation first. Audio uploads are not part of this plugin.

## Privacy and support

- Privacy policy: https://vault.toolkit.music/legal/privacy
- Terms: https://vault.toolkit.music/legal/terms
- Support: https://vault.toolkit.music/support

This repository is generated from the VAULT source. Do not edit the plugin folder by hand.

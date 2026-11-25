# VAULT Agent Plugin

Connect an eligible VAULT workspace to a compatible AI client. The plugin uses the hosted VAULT MCP service and five included skills for catalog and workspace workflows.

## Before you install

Direct connection requires Personal Pro or an eligible Team seat. Personal Creator uses Conductor in VAULT at `/songs/conductor` instead of a direct Personal connection. A Creator or Free user may still connect an eligible Team workspace, but a Team seat never grants direct access to a Personal workspace.

## Connect VAULT

1. Sign in to VAULT and open `https://vault.toolkit.music/account/mcp`.
2. Choose one eligible workspace and select ChatGPT, Codex, or Claude. VAULT shows whether the selected client's installation route is already available or still awaiting publication.
3. Follow the client-specific steps, review the requested scopes, select the same workspace during OAuth, and give consent. ChatGPT may request optional broad scopes later when an explicitly requested action needs them. Existing workspace permissions still apply.
4. Start a new conversation or session and ask: `Show which VAULT workspace is connected.` Check that the displayed workspace is the one you selected.
5. To connect a second workspace, create a separate client connection when the client supports it. Otherwise disconnect and authorize the other workspace.
6. Disconnect from the same VAULT page when access should stop. The client-side plugin tile can remain installed, but server access stops immediately.

## Client compatibility

- ChatGPT and Codex use the shared OpenAI plugin listing, hosted MCP connection, and five included skills after the plugin has been registered and approved. Codex installation uses a supported ChatGPT desktop surface or the `/plugins` browser in Codex CLI, not the IDE extension or mobile.
- Codex can also connect directly to the hosted MCP tools before the plugin listing is available: run `codex mcp add vault-hosted --url https://mcp.vault.toolkit.music/mcp`. If authorization does not open automatically, run `codex mcp login vault-hosted`. Select an eligible workspace during authorization and start a new Codex task. This direct connection does not install the five packaged skills.
- Clients that implement the portable Agent Plugins format can load the root `plugin.json`, `mcp.json`, and `skills/` directory.
- Claude can use the same hosted VAULT tools as a custom remote connector. Claude Team and Enterprise owners must add the connector for their organization before members connect it. In Claude Code, run `claude mcp add --transport http vault https://mcp.vault.toolkit.music/mcp` and complete OAuth from `/mcp`. This connector route does not include the five packaged skills; the VAULT plugin for Claude does, once it is listed in the Claude plugin directory.

Client support changes independently of VAULT. Run a private connection test in every client you plan to list before release.

<details>
<summary>Advanced MCP clients: use a personal access token</summary>

PAT setup is for advanced clients that cannot complete OAuth. Create a PAT from VAULT settings, copy the value once, and store it in the client's secret store. Do not place a PAT in a plugin file, prompt, shell history, or source control. Revoke it from the same connection list when it is no longer needed.

</details>

## Private developer testing

Use a real registered developer app ID supplied for your private test environment. Keep it in an environment variable and do not commit it:

```bash
export OPENAI_VAULT_APP_ID='your-registered-developer-app-id'
AGENT_PLUGIN_BUILD_PROFILE=release OPENAI_VAULT_APP_ID="$OPENAI_VAULT_APP_ID" pnpm agent-plugin:build
AGENT_PLUGIN_BUILD_PROFILE=release OPENAI_VAULT_APP_ID="$OPENAI_VAULT_APP_ID" pnpm agent-plugin:validate
python3 /Users/remyborsboom/.codex/skills/.system/plugin-creator/scripts/validate_plugin.py dist/agent-plugin/toolkit-vault
```

To install a confirmed local Codex marketplace, use the commands below. They update the local catalog through the Codex CLI, not by editing marketplace or Codex configuration JSON:

```bash
codex plugin marketplace add "$PWD/dist/agent-plugin"
codex plugin add toolkit-vault@toolkit-vault-local
```

Start a new task or thread after reinstall so the client loads updated skills and tools. The synthetic CI identifier is reserved for automated validation and is not a usable developer app ID.

## Scope and privacy

The plugin can read and change data when its connection scopes and the selected workspace permissions allow them. Every connection is bound to one user and one workspace. VAULT can revoke access independently of the client installation. The hosted action catalog includes confirmed destructive actions when explicitly granted. Audio uploads and playback internals remain outside this plugin.

Custom metadata is scoped to the connected workspace. The hosted MCP server exposes direct custom-field tools and the generic `vault_list_capabilities` / `vault_run_action` workflow. A song can be visible through a share while its source workspace's custom fields remain inaccessible to this connection.

The MCP URL serves live tool calls, but a personal developer-mode ChatGPT connection keeps a snapshot of tool names and schemas. After a tool-contract change, open ChatGPT **Settings → Plugins → VAULT → Manage → Information → Refresh**, then start a new conversation. The five generic action tools use the live action catalog, so adding an action behind their stable schemas does not require another tool refresh. Packaged skills and plugin presentation are separate snapshots: update those with **Upload new version**. The default OAuth request remains `vault.read vault.write`. Inbox, Brief, and metadata actions need their domain grants or the opt-in broad grants. When an explicitly requested action lacks a scope, the server returns an `insufficient_scope` challenge so ChatGPT can ask the user to authorize that scope. Refreshing tools or replacing the plugin package does not expand an existing OAuth grant. Workspace roles and feature access remain enforced separately.

ChatGPT's generated personal plugin can have a `dev-...` package name and an `asdk_app_...` mapping, even when the portable package is named `toolkit-vault`. Download the installed plugin ZIP first and read its `.app.json`. Build the release package as above, then export a compatible update using that existing app ID:

```bash
OPENAI_VAULT_PERSONAL_APP_ID='asdk_app_<existing-id>' pnpm agent-plugin:build-personal
```

When packaged skills or plugin presentation change, zip the generated `dist/agent-plugin/personal/dev-...` folder and use **Upload new version** on the same personal plugin. Keep the generated package name and app ID unchanged; a new ZIP must have a higher manifest version. The export contains the five skills and the existing app mapping without credentials. Do not upload the portable `toolkit-vault` ZIP to an existing `dev-...` plugin: the names differ.

A published OpenAI plugin uses continuous review for new and changed MCP tools, so its users do not reinstall it for each server release. New tools can be delayed or held by review. Server-only behavior changes take effect through the live URL when they preserve the published tool contract. Changes to packaged skills or plugin information still require a new plugin version.

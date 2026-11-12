# Connect the CRM MCP

Read this only when setting up or troubleshooting the connection. Installing a standalone Skill does not register an MCP server; neither a Skill nor a plugin grants CRM account access.

Use the environment requested by the user. Production uses the connection name `oneulink-crm` and Streamable HTTP endpoint `https://mcp.oneulink.com/mcp/channel`; OAuth is issued by `https://app.oneulink.com`. The endpoint is bound to the OneULink company. It does not offer company switching, and switching companies in the CRM webpage does not change the authorized MCP identity.

## Installed plugin

When this Skill is bundled with the OneULink CRM plugin, use the plugin connection's native sign-in and consent flow. Inspect its connection state and discover the tools actually exposed by the host; names may carry a host prefix. Do not create a duplicate standalone MCP connection just because a CLI does not list the plugin-managed connection. For 401 responses use native reconnect; for forbidden operations explain the missing account permission.

The user signs in and confirms the intended account and OneULink company on the authorization page. Keep the host's callback listener or login process running until authorization completes or failure is reported. Use a fresh login after an expired request. After connecting, call `crm_context` and verify with an appropriate read-only query; never test writes merely to establish a connection. If the plugin or connection is unavailable in the current host, explain that limitation and use its supported setup rather than pretending another client's CLI is available.

Keep existing standalone connections unless the user requests a change. Installing a plugin does not migrate, remove or revoke them. Removing local configuration does not revoke a server-side grant; use the CRM personal account page's MCP authorization management when revocation is requested.

## Standalone Codex

Inspect an existing connection before adding it:

```sh
codex mcp get oneulink-crm
```

If it is absent and setup is requested, add it:

```sh
codex mcp add oneulink-crm --url https://mcp.oneulink.com/mcp/channel
```

If sign-in did not complete during registration, or an existing connection needs to sign in again:

```sh
codex mcp login oneulink-crm
```

## Standalone Claude Code

Inspect the connection:

```sh
claude mcp get oneulink-crm
```

If it is absent and setup is requested, add it at user scope:

```sh
claude mcp add --scope user --transport http oneulink-crm https://mcp.oneulink.com/mcp/channel
claude mcp login oneulink-crm
```

Use project scope instead when the user requests a project-only connection. These commands were checked against the client help on 2026-10-27. If an installed client lacks `mcp login`, use its native MCP authentication interface and consult its current help; do not invent a token-copy workflow.

## Complete and verify

Preserve an existing connection to another environment; do not silently overwrite it. If inspecting configuration fails for a reason other than a missing entry, resolve that error before making changes. Keep any login process and loopback callback listener running until the user completes authorization or the client reports failure. For an expired request, initiate a fresh login rather than reusing a previous authorization URL.

The user signs in and confirms the account and OneULink company in the authorization page. Use the client's supported CIMD flow; CRM does not offer DCR. Do not provide client secrets, inject bearer headers, or read the host's token files to bypass login. An unavailable account needs its OneULink membership or MCP connection permission checked by an administrator.

After connection, discover the actual tool list and call `crm_context`. Share the returned environment, account and company after a new connection, when requested, or when resolving an identity mismatch; lead routine business replies with the requested result. Verify with a read-only query appropriate to the user's request. Do not test writes merely to prove setup succeeded. A successful login does not grant every tool; the account's existing API permissions still apply.

## Other hosts

Use a host's native MCP setup rather than pretending it has a Codex or Claude Code CLI. CRM also supports ChatGPT's official web CIMD client: configure the remote URL with OAuth in a custom MCP app when available to that account/workspace. The local `skills` installation does not automatically install this Skill into ChatGPT web, and unsupported client identities may be rejected.

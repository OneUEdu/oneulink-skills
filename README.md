# OneULink Skills

Private skills for OneULink staff. This repository is the source of truth for the skills; it does not contain the CRM backend or MCP server implementation.

## Available skills

| Skill | Purpose |
| --- | --- |
| [oneulink-crm](skills/oneulink-crm/SKILL.md) | Query and update existing school/course records, maintain course major options, investigate website feedback, and apply sourced batch updates through the internal OneULink CRM MCP. |

Instructions are in English with Chinese interface labels where useful. Agents follow the user's language and preserve literal API names, enum values, and source data. References are loaded by task rather than all at once.

## Install

You need Node.js/npm, a supported agent client, and read access to this private GitHub repository. Use your existing Git, GitHub CLI, or SSH authentication. Repository access allows downloading the Skill; CRM access is separately controlled by your account's OAuth grant and API permissions.

Install the Skill for Codex and Claude Code:

```sh
DISABLE_TELEMETRY=1 npx skills add OneUEdu/oneulink-skills \
  --skill oneulink-crm -g -a codex -a claude-code
```

Keep only the `-a` option for the client you use. Remove `-g` for a project-scoped installation. To inspect the repository before installing:

```sh
DISABLE_TELEMETRY=1 npx skills add OneUEdu/oneulink-skills --list
```

If an existing installation was made with the former OneULink installer or edited locally, preserve those local changes before replacing it. Use `skills` for subsequent updates; do not alternate installation managers on the same directory. Restart or open a new client session if needed for discovery.

Skill installation does not register the MCP connection or complete OAuth. Ask your agent to use `oneulink-crm` and connect it, or follow [MCP setup](skills/oneulink-crm/references/setup.md). The production MCP URL is `https://mcp.oneulink.com/mcp/channel`. Login and consent remain on OneULink; do not paste tokens into chat or this repository.

## Update

```sh
DISABLE_TELEMETRY=1 npx skills update oneulink-crm -g
```

Run this command to install available updates; merely pushing a repository change does not refresh every installed copy. Skill wording changes do not require a new CRM authorization. Removing a local Skill or MCP configuration does not revoke a server-side grant; use the CRM personal account page's MCP authorization management to revoke it.

`DISABLE_TELEMETRY=1` disables the installer's telemetry for these commands. See the upstream [skills CLI documentation](https://github.com/vercel-labs/skills/blob/main/docs/commands.md) and [telemetry setting](https://www.skills.sh/docs/cli).

## Maintain

Edit `skills/<skill-name>/SKILL.md` and its referenced files here. Keep the Skill name stable, link every supporting reference from the relevant entrypoint, and preserve established field and permission semantics. Do not duplicate the Skill back into the frontend repository or include credentials, real feedback records, or generated client configuration.

Before publishing a change, validate the Skill format, check relative links, and confirm the `skills` CLI can list the Skill. Test installation in an isolated temporary project rather than overwriting a user's installed copy. MCP protocol/schema changes belong in the CRM frontend repository's `services/crm-mcp`; CRM API and authorization changes belong in `oneu_connect`. This repository has no npm publishing or server-deployment step.

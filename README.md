# OneULink Skills

Skills for OneULink staff. The repository is public so anyone can install the skills; using them requires signing in to OneULink, and CRM access is controlled by each account's OAuth grant and API permissions. This repository is the source of truth for the skills; it does not contain the CRM backend or MCP server implementation.

## Available skills

| Skill | Purpose |
| --- | --- |
| [oneulink-crm](skills/oneulink-crm/SKILL.md) | Query and update existing school/course records, investigate website feedback, and apply sourced batch updates through the internal OneULink CRM MCP. |

Instructions are in English with Chinese interface labels where useful. Agents follow the user's language and preserve literal API names, enum values, and source data. References are loaded by task rather than all at once.

## Install

You need Node.js/npm and a supported agent client. The repository is public, so downloading the Skill needs no GitHub sign-in; CRM access is controlled separately by your OneULink sign-in, your account's OAuth grant and API permissions.

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

Pushing changes under `skills/` to `main` immediately starts [Sync OneULink plugin on skill updates](https://github.com/OneUEdu/oneulink-skills/actions/workflows/sync-plugin.yml). The workflow reads that exact source commit and runs the plugin repository's existing sync, validation and regression tests before committing generated files to `OneUEdu/oneulink-plugin/main`. Plugin connection and presentation overrides remain in the plugin repository. There is no polling schedule. Updating the workflow file also triggers a setup check; changes only to this README do not.

The workflow uses `sync --bump-patch`: an actual change to the packaged Skill content increments the plugin patch version and updates both manifests and the version/source changelog entry. Repeated runs, source-lock-only changes and source instructions fully replaced by plugin overrides do not increment it. Generated version changes are committed only after validation and tests succeed; this updates the repository version, while platform publication remains a separate step.

Cross-repository Git access uses a dedicated write deploy key attached only to `oneulink-plugin`, stored as the repository Actions secret `ONEULINK_PLUGIN_SYNC_KEY` here. The built-in `GITHUB_TOKEN` has only source read permission. Run manually with `dry_run` for validation without a plugin commit. Concurrent updates are serialized; a superseded source commit is skipped before publishing, and plugin pushes never use force. Failures are visible in Actions and use each maintainer's GitHub notification settings. Installed clients still require their normal update or release process.

Before publishing a change, validate the Skill format, check relative links, and confirm the `skills` CLI can list the Skill. Test installation in an isolated temporary project rather than overwriting a user's installed copy. MCP protocol/schema changes belong in the CRM frontend repository's `services/crm-mcp`; CRM API and authorization changes belong in `oneu_connect`. This repository has no npm publishing or server-deployment step.

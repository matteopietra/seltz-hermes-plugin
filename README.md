# Seltz for Hermes Agent

![Seltz](assets/logo.svg)

Seltz is a web search API built for AI agents. This plugin connects [Hermes Agent](https://github.com/NousResearch/hermes-agent) to Seltz's hosted MCP server at `https://mcp.seltz.ai/mcp` (streamable HTTP) and adds a skill that tells Hermes how to search, cite, and treat the results.

The server is remote. This package is a manifest, an MCP connection, one Markdown skill, and the Seltz logo. It runs no local code.

Sign in with a Seltz account. Hermes opens the [Seltz Console](https://console.seltz.ai), you pick an organization, and you click Allow. An API key in `SELTZ_API_KEY`, sent as `Authorization: Bearer <key>`, is the fallback for headless setups.

- Product: [seltz.ai](https://seltz.ai)
- Console: [console.seltz.ai](https://console.seltz.ai)
- MCP reference: [docs.seltz.ai/integrations/mcp](https://docs.seltz.ai/integrations/mcp)

Requires Hermes Agent 0.22.2 or later, with its MCP support installed. Use the same Hermes profile for installation, configuration, sign-in, and chat.

## Install

After this plugin is listed in the Hermes catalog:

```bash
hermes plugins install seltz --enable
```

Until then, install from this repository:

```bash
hermes plugins install seltz-ai/seltz-hermes-plugin --enable
```

Portable packages install disabled unless you pass `--enable`. If you skipped it:

```bash
hermes plugins enable seltz
```

If Hermes reports that the MCP SDK or HTTP transport is unavailable, install Hermes's MCP optional feature using the instructions for your Hermes version.

## Sign in (default)

The Agent Plugins manifest cannot declare `auth: oauth`. Browser sign-in is a native entry in the active profile's `config.yaml`. That entry replaces the packaged server of the same name. Replace any existing `mcp_servers.seltz` entry as a whole and keep your other servers. The full snippet is [examples/oauth.yaml](examples/oauth.yaml):

```yaml
mcp_servers:
  seltz:
    url: https://mcp.seltz.ai/mcp
    auth: oauth
    strict_redirect_headers: true
```

Then sign in from a terminal:

```bash
hermes mcp login seltz
```

Hermes discovers Seltz's OAuth server, opens the Console, and stores the tokens. The authorization scope the server advertises is `seltz:read`. You do not need `SELTZ_API_KEY` for this connection. Do not add an `Authorization` header: Hermes supplies the OAuth bearer token.

Cron jobs and other unattended runs cannot finish a browser sign-in. Sign in first, or use an API key for unattended work. If a refresh is rejected, run `hermes mcp login seltz` again.

Hermes documents the flow at [OAuth-authenticated HTTP servers](https://hermes-agent.nousresearch.com/docs/user-guide/features/mcp#oauth-authenticated-http-servers).

## API key (headless fallback)

Create a key in the [Seltz Console](https://console.seltz.ai). Seltz API keys use the `sk_` prefix. Save it on the active profile and keep it out of chat, URLs, and this repository:

```bash
hermes config set SELTZ_API_KEY sk_your_key
```

With no native `mcp_servers.seltz` entry, [mcp.json](mcp.json) sends the key as `Authorization: Bearer ${SELTZ_API_KEY}`. Hermes resolves the placeholder from the active profile.

To switch from OAuth to the key, replace the native entry as a whole so `auth: oauth` is removed. [examples/api-key.yaml](examples/api-key.yaml) is that entry. Leaving both `auth: oauth` and an `Authorization` header on the same entry is not a supported setup.

## Use

Start a new session. Ask Hermes to find and load the `seltz` skill with `skills_list` and `skill_view`, then ask for current sources. Hermes prefixes skill names, so use `skills_list` for the full name.

The documented MCP tool is `search` (`query` required, `max_results` optional, default 10). Hermes exposes it as `mcp__seltz__search`. The live tool schema from the server is the authority if it adds tools or changes parameters. Check the connection with `hermes mcp test seltz` after the native entry is in place.

Example:

> Search for the latest official guidance on EU AI Act transparency obligations and link the primary sources.

## What this plugin does on your machine and the network

- Sends the queries you ask Seltz to search to `https://mcp.seltz.ai/mcp` over HTTPS.
- OAuth sign-in also contacts `https://console.seltz.ai`, which is the authorization server Seltz publishes for that resource. Hermes owns the browser flow, token storage, and refresh.
- The API-key fallback sends `Authorization: Bearer` with the profile's `SELTZ_API_KEY`. The plugin never reads credential files and never writes tokens.
- Runs no local process, shell command, background worker, or self-updater.
- The hosted server's tools can change independently of this package.

## Catalog

Listing `hermes plugins install seltz` is a separate pull request to `NousResearch/hermes-agent` that adds `plugin-catalog/seltz.yaml`, pinned to a full commit SHA. The proposed entry is in [docs/catalog-submission.md](docs/catalog-submission.md). Only a Seltz owner or major contributor should open that request, and only Nous maintainers can set `tier: verified`.

## License

[Apache-2.0](LICENSE), the same license as [seltz-ai/seltz-codex-plugin](https://github.com/seltz-ai/seltz-codex-plugin).

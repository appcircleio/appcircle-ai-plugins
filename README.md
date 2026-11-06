# Appcircle AI Plugins

Appcircle's official plugin for AI coding assistants. It connects your coding agent to the [Appcircle](https://appcircle.io) mobile CI/CD platform, covering build, code signing, distribution, publishing (App Store / Google Play / Huawei AppGallery), and testing, so you can ask Appcircle questions and look up your own organization's data right from chat.

## Supported Platforms

- **Claude Code** — this repo is itself a Claude Code plugin marketplace.

  ```shell
  /plugin marketplace add appcircleio/appcircle-ai-plugins
  /plugin install appcircle@appcircle
  /reload-plugins
  ```

- **claude.ai** — you can install this plugin's skills in claude.ai too:

  1. Open **Customize**.
  2. Click **Add plugin**.
  3. Choose **Create plugin**, then **Add marketplace**.
  4. Enter the repository URL: `https://github.com/appcircleio/appcircle-ai-plugins`.

  MCP tools are only available in Claude Code, Copilot CLI, and Codex for now; installing the plugin in claude.ai gets you the skills, not the MCP tools.

- **GitHub Copilot CLI** — this repo is also a Copilot CLI plugin marketplace.

  ```shell
  copilot plugin marketplace add appcircleio/appcircle-ai-plugins
  copilot plugin install appcircle@appcircle
  ```

  This plugin can also be added to **VS Code's Copilot Chat** via its Agent Plugins feature.

- **ChatGPT and Codex** — this repo is also a ChatGPT/Codex plugin marketplace.

  ```shell
  codex plugin marketplace add appcircleio/appcircle-ai-plugins
  ```

  Then restart the ChatGPT desktop app, open the **Plugins Directory**, pick the **Appcircle** source, and install the plugin. In Codex CLI, use `/plugins` to install it instead.

## Skills

| Skill | Purpose |
|-------|---------|
| `appcircle:appcircle-assistant` | Answers Appcircle questions from official docs, product, and blog sources |
| `appcircle:build-insights-report` | Renders a visual Build Insights Report from the `get_build_insights_report` MCP tool |

Your agent invokes a skill automatically when your question matches its purpose, or you can call it directly:

```
/appcircle:appcircle-assistant How do I configure iOS code signing for a build profile?
```

## MCP server

The plugin registers `mcp.appcircle.io` as an MCP server named `appcircle`, authenticated via a bearer-token header using an Appcircle access token. The MCP tools don't work out of the box on any platform: you need to provide the token first, as described below. Using a self-hosted Appcircle server? See [Self-hosted Appcircle](#self-hosted-appcircle) instead. For the full list of available tools and how it works, see the [Appcircle MCP documentation](https://docs.appcircle.io/appcircle-ai/ai-features-on-appcircle/appcircle-mcp).

### Configure the MCP server

The server authenticates with an `APPCIRCLE_ACCESS_TOKEN`, which you can obtain from either credential type:

- [Personal Access Key](https://docs.appcircle.io/account/my-organization/security/personal-access-key)
- [API Key](https://docs.appcircle.io/account/my-organization/security/api-keys)

> [!WARNING]
> On a self-hosted Appcircle server, don't set `APPCIRCLE_ACCESS_TOKEN`: the plugin would send your self-hosted token to `mcp.appcircle.io`. Follow [Self-hosted Appcircle](#self-hosted-appcircle) instead.

1. Export it in the environment where your coding agent runs:

   ```bash
   export APPCIRCLE_ACCESS_TOKEN=<your-token>
   ```
2. **Codex only:** the plugin can't pass the token to the server, so register the server yourself (this replaces the plugin's entry):

   ```bash
   codex mcp add appcircle --url https://mcp.appcircle.io --bearer-token-env-var APPCIRCLE_ACCESS_TOKEN
   ```
3. Restart/reload your agent so the MCP server picks up the token.
4. Re-export and reload when it expires.

Without this, the MCP tools won't be able to authenticate. The appcircle-assistant skill works regardless, since it only reads public documentation.

### Self-hosted Appcircle

Self-hosted Appcircle servers run their own MCP service at `https://mcp.<your-appcircle-domain>`. The plugin always points to the cloud server, so register your self-hosted server yourself.

Keep your self-hosted token in a separate variable, `APPCIRCLE_SELFHOSTED_TOKEN`, and leave `APPCIRCLE_ACCESS_TOKEN` unset. The plugin's own server only reads `APPCIRCLE_ACCESS_TOKEN`, so your self-hosted token never reaches `mcp.appcircle.io`.

1. Make sure `APPCIRCLE_ACCESS_TOKEN` is not set. If you used the cloud setup before, remove it from your shell profile (for example `~/.zshrc`), then unset it and check that nothing is printed:

   ```bash
   unset APPCIRCLE_ACCESS_TOKEN
   echo "$APPCIRCLE_ACCESS_TOKEN"
   ```
2. Export a token issued by your self-hosted server ([Personal Access Key](https://docs.appcircle.io/account/my-organization/security/personal-access-key) or [API Key](https://docs.appcircle.io/account/my-organization/security/api-keys)):

   ```bash
   export APPCIRCLE_SELFHOSTED_TOKEN=<your-self-hosted-token>
   ```
3. Register your server:

   - **Claude Code:** add your server, then disable the plugin's `appcircle` server from `/mcp` (select `appcircle`, then **Disable**):

     ```bash
     claude mcp add --scope user --transport http appcircle-selfhosted https://mcp.<your-appcircle-domain> \
       --header 'Authorization: Bearer ${APPCIRCLE_SELFHOSTED_TOKEN}'
     ```
   - **Copilot CLI:** disable the plugin's server, then add yours:

     ```bash
     copilot mcp disable appcircle
     copilot mcp add --transport http appcircle-selfhosted https://mcp.<your-appcircle-domain> \
       --header 'Authorization: Bearer ${APPCIRCLE_SELFHOSTED_TOKEN}'
     ```
   - **Codex:**

     ```bash
     codex mcp add appcircle-selfhosted --url https://mcp.<your-appcircle-domain> --bearer-token-env-var APPCIRCLE_SELFHOSTED_TOKEN
     ```

   For Claude Code and Copilot CLI, keep the single quotes around the header: the configuration stores `${APPCIRCLE_SELFHOSTED_TOKEN}` as is and reads the token each time the agent starts, so you don't need to register again when the token changes.
4. **Internal CA (Claude Code, Copilot CLI):** if your server's certificate is signed by an internal certificate authority, point `NODE_EXTRA_CA_CERTS` to the CA bundle (PEM). Internal CAs are not covered for Codex yet.

   ```bash
   export NODE_EXTRA_CA_CERTS=/path/to/internal-ca.pem
   ```
5. Restart or reload your agent.
6. Re-export and reload when the token expires.

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

  MCP tools are only available in Claude Code and Copilot CLI for now; installing the plugin in claude.ai gets you the skills, not the MCP tools.

- **GitHub Copilot CLI** — this repo is also a Copilot CLI plugin marketplace.

  ```shell
  copilot plugin marketplace add appcircleio/appcircle-ai-plugins
  copilot plugin install appcircle@appcircle
  ```

  This plugin can also be added to **VS Code's Copilot Chat** via its Agent Plugins feature.

## Skills

| Skill | Purpose |
|-------|---------|
| `appcircle:doc-assistant` | Answers Appcircle questions from official docs and product sources |
| `appcircle:build-insights-report` | Renders a visual Build Insights Report from the `get_build_insights_report` MCP tool |

Your agent invokes a skill automatically when your question matches its purpose, or you can call it directly:

```
/appcircle:doc-assistant How do I configure iOS code signing for a build profile?
```

## MCP server

The plugin registers `mcp.appcircle.io` as an MCP server named `appcircle`, authenticated via a bearer-token header using an Appcircle access token. For the full list of available tools and how it works, see the [Appcircle MCP](https://github.com/appcircleio/appcircle-mcp) repository.

### Configure the MCP server

The server authenticates with an `APPCIRCLE_ACCESS_TOKEN`, which you can obtain from either credential type:

- [Personal Access Key](https://docs.appcircle.io/account/my-organization/security/personal-access-key)
- [API Key](https://docs.appcircle.io/account/my-organization/security/api-keys)

1. Export it in the environment where your coding agent runs:

   ```bash
   export APPCIRCLE_ACCESS_TOKEN=<your-token>
   ```
2. Restart/reload your agent so the MCP server picks up the token.
3. Re-export and reload when it expires.

Without this, the MCP tools won't be able to authenticate. The doc-assistant skill works regardless, since it only reads public documentation.

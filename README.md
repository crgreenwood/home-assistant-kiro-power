<p align="center">
  <img src="https://brands.home-assistant.io/_/homeassistant/logo.png" alt="Home Assistant Logo" width="120"/>
</p>

<h1 align="center">Home Assistant — Kiro Power</h1>

<p align="center">
  <em>AI-powered smart home control and configuration for <a href="https://kiro.dev">Kiro</a>, combining the <a href="https://github.com/homeassistant-ai/ha-mcp">ha-mcp</a> server with <a href="https://github.com/homeassistant-ai/skills">Home Assistant Agent Skills</a>.</em>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/MCP_Tools-85+-blue" alt="85+ Tools">
  <img src="https://img.shields.io/badge/Transport-stdio-green" alt="stdio transport">
  <img src="https://img.shields.io/badge/License-MIT-yellow" alt="MIT License">
  <a href="https://github.com/homeassistant-ai/ha-mcp"><img src="https://img.shields.io/badge/ha--mcp-latest-purple" alt="ha-mcp"></a>
</p>

---

## What is this?

This Kiro Power gives your AI agent the ability to **control, configure, and debug your Home Assistant smart home** through natural language. It packages:

1. **[ha-mcp](https://github.com/homeassistant-ai/ha-mcp)** — A comprehensive MCP server with 85+ tools for device control, automation management, dashboard editing, system administration, and debugging.
2. **[Home Assistant Agent Skills](https://github.com/homeassistant-ai/skills)** — Best-practice domain knowledge covering automations, helpers, scripts, dashboards, device control, and safe refactoring patterns.

The MCP server provides the **tools**. The skills provide the **knowledge** to use them well.

## What can you do with it?

| You say | What happens |
|---------|--------------|
| *"Turn off all lights in the living room"* | Finds and controls the relevant entities |
| *"Create an automation that turns on the porch light at sunset"* | Creates the automation with proper triggers and actions |
| *"The motion sensor automation isn't working, debug it"* | Analyzes traces, identifies issues, suggests fixes |
| *"Add a weather card to my dashboard"* | Updates your Lovelace dashboard |
| *"Set up a utility meter for solar panels"* | Creates the helper with correct configuration |
| *"Check for updates and create a backup"* | Manages system tasks |

## Features

| Category | Capabilities |
|----------|--------------|
| 🔍 **Search** | Fuzzy entity search, deep config search, system overview |
| 🏠 **Control** | Any service call, bulk device control, real-time states |
| 🔧 **Manage** | Automations, scripts, helpers, dashboards, areas, zones, groups, calendars, blueprints |
| 📊 **Monitor** | History, statistics, camera snapshots, automation traces |
| 💾 **System** | Backup/restore, updates, apps, device registry |
| 🧠 **Best Practices** | Native constructs over templates, correct helper selection, safe refactoring |

## Installation

### Option 1: Install from GitHub

1. Open Kiro → **Powers panel**
2. Click **Add Custom Power**
3. Select **Import power from GitHub**
4. Paste this repository URL:

```
https://github.com/crgreenwood/home-assistant-kiro-power
```

5. Click **Install**

### Option 2: Install from local path

1. Clone this repository:
   ```bash
   git clone https://github.com/crgreenwood/home-assistant-kiro-power.git
   ```
2. Open Kiro → **Powers panel**
3. Click **Add Custom Power**
4. Select **Import power from a folder**
5. Select the cloned directory

## Prerequisites

- **Home Assistant** instance (any installation type)
- **Python 3.11+** with [`uvx`](https://docs.astral.sh/uv/getting-started/installation/) installed
- **Long-Lived Access Token** from your Home Assistant profile
- Network access from your machine to your HA instance

## Configuration

After installation, update the MCP configuration with your Home Assistant details:

```json
{
  "mcpServers": {
    "ha-mcp": {
      "command": "uvx",
      "args": ["ha-mcp@latest"],
      "env": {
        "HOMEASSISTANT_URL": "http://homeassistant.local:8123",
        "HOMEASSISTANT_TOKEN": "your-long-lived-access-token"
      }
    }
  }
}
```

**Getting a Long-Lived Access Token:**
1. Open your Home Assistant web UI
2. Click your profile (bottom-left)
3. Scroll to **Long-Lived Access Tokens**
4. Click **Create Token** → copy it

## Power Structure

```
home-assistant-kiro-power/
├── POWER.md              # Power documentation and onboarding instructions
├── mcp.json              # MCP server configuration (stdio transport)
├── steering/
│   └── best-practices.md # HA best practices and anti-patterns
└── README.md             # This file
```

## Best Practices Included

The power teaches your agent to:

- ✅ Use **native HA constructs** over Jinja2 templates
- ✅ Use **entity_id** over device_id (survives re-pairing)
- ✅ Select the **correct automation mode** (restart, queued, parallel, single)
- ✅ Choose **built-in helpers** before template sensors
- ✅ Use the **config API** instead of generating raw YAML
- ✅ Perform **impact analysis** before renaming entities
- ✅ Create **Template Helpers via UI/API**, not YAML

## Attribution

This power packages and documents work created by the [Home Assistant AI Toolkit](https://github.com/homeassistant-ai) community:

- **MCP Server:** [homeassistant-ai/ha-mcp](https://github.com/homeassistant-ai/ha-mcp) by [@julienld](https://github.com/julienld) and contributors
- **Agent Skills:** [homeassistant-ai/skills](https://github.com/homeassistant-ai/skills)

All credit for the underlying tools and knowledge goes to the original authors. This power provides the Kiro integration layer.

## Troubleshooting

| Problem | Solution |
|---------|----------|
| MCP server won't connect | Verify URL is reachable: `curl YOUR_URL/api/` |
| Token rejected | Generate a new Long-Lived Access Token |
| `uvx` not found | Install with `brew install uv` or `pip install uv` |
| "Unknown tool" errors | Reconnect/refresh the MCP server in Kiro |
| File tools not working | Install the `ha_mcp_tools` custom component via HACS |

## Contributing

Issues and pull requests are welcome. If you find a bug or have an improvement:

1. Open an issue describing the problem or enhancement
2. Fork the repo and make your changes
3. Submit a PR with a clear description

## License

MIT

## Links

- [ha-mcp GitHub](https://github.com/homeassistant-ai/ha-mcp)
- [Home Assistant Agent Skills](https://github.com/homeassistant-ai/skills)
- [Kiro Powers Documentation](https://kiro.dev/docs/powers/)
- [Home Assistant](https://www.home-assistant.io/)

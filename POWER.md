---
name: "home-assistant"
displayName: "Home Assistant"
description: "AI-powered smart home control and configuration using the ha-mcp server, paired with Home Assistant best-practice skills for automations, helpers, dashboards, and device management."
keywords: ["home-assistant", "ha-mcp", "smart-home", "automations", "iot"]
author: "crgreenwood"
---

# Home Assistant

## Overview

This power combines two community projects from the [Home Assistant AI Toolkit](https://github.com/homeassistant-ai) into a single guided experience for managing your Home Assistant instance through Kiro:

1. **[ha-mcp](https://github.com/homeassistant-ai/ha-mcp)** — A comprehensive MCP server (85+ tools) that enables AI-powered smart home control, configuration, and debugging via natural language.
2. **[Home Assistant Agent Skills](https://github.com/homeassistant-ai/skills)** — Domain knowledge that teaches best practices for automations, helpers, scripts, dashboards, and device control.

The MCP server gives you the *tools* to interact with Home Assistant. The skills give you the *knowledge* to do it well — native constructs over templates, correct helper selection, safe refactoring workflows, and proper automation modes.

> **Attribution:** This power packages and documents work created by the [homeassistant-ai](https://github.com/homeassistant-ai) community. All credit for the MCP server goes to [@julienld](https://github.com/julienld) and contributors. Skills authored by the [homeassistant-ai/skills](https://github.com/homeassistant-ai/skills) team.

## Available Steering Files

- **best-practices** — Home Assistant best practices: native constructs, helper selection, automation modes, anti-patterns, and decision workflows (sourced from homeassistant-ai/skills)

## Onboarding

When this power is first activated, walk the user through connection setup step by step.

### Step 1: Introduce the power

Tell the user:

"This power is built on two open-source community projects:
- **ha-mcp** — the MCP server providing 85+ tools: https://github.com/homeassistant-ai/ha-mcp
- **Home Assistant Agent Skills** — best-practice domain knowledge: https://github.com/homeassistant-ai/skills

To get started, we need to connect the ha-mcp server to your Home Assistant instance. This power uses the local stdio transport — ha-mcp runs on your machine via `uvx` and connects to your HA instance over your network."

### Step 2: Verify prerequisites

Check the following prerequisites by running the commands yourself (don't ask the user to run them — do it directly):

1. **Python 3.11+**: Run `python3 --version` and confirm it's 3.11 or higher.
2. **uvx**: Run `uvx --version` to confirm it's installed.
   - If uvx is not found, offer to install it: `brew install uv` (macOS) or `pip install uv`. See https://docs.astral.sh/uv/getting-started/installation/

Report the results to the user. If anything is missing, offer to install it for them (with their permission).

### Step 3: Create a Long-Lived Access Token

Walk the user through creating a token:

1. Open your Home Assistant web UI in a browser
2. Click your profile icon (bottom-left of the sidebar)
3. Scroll down to the **Long-Lived Access Tokens** section
4. Click **Create Token**
5. Name it something like "Kiro MCP"
6. Copy the token immediately — it won't be shown again

### Step 4: Configure the MCP connection

The user needs to update the `mcp.json` that was installed with this power. It's located at `~/.kiro/settings/mcp.json` under the powers section. They need to replace the placeholder values:

```json
{
  "mcpServers": {
    "ha-mcp": {
      "command": "uvx",
      "args": ["ha-mcp@latest"],
      "env": {
        "HOMEASSISTANT_URL": "http://homeassistant.local:8123",
        "HOMEASSISTANT_TOKEN": "YOUR_ACTUAL_TOKEN_HERE"
      }
    }
  }
}
```

Tell the user to replace:
- `HOMEASSISTANT_URL` — The URL they use to access HA in their browser (e.g., `http://homeassistant.local:8123` or `http://192.168.1.x:8123`)
- `HOMEASSISTANT_TOKEN` — The Long-Lived Access Token they just created

### Step 5: Verify the connection

After the user has configured the connection, ask them to say: **"Can you see my Home Assistant?"**

The agent should call `ha_get_overview` and return a summary of their devices, areas, and entities. If it works, onboarding is complete.

## Common Workflows

### Control Devices

Just say what you want naturally:

- "Turn off all lights in the living room"
- "Set the thermostat to 21°C"
- "Lock the front door"
- "What's the temperature in the bedroom?"

The agent uses `ha_call_service`, `ha_get_state`, and `ha_search` to find and control entities.

### Create Automations

Describe what you want and the agent builds it:

- "Create an automation that turns on the porch light at sunset"
- "When motion is detected in the hallway after 10pm, turn on the light for 2 minutes"
- "Notify me when the washing machine finishes"

Uses `ha_config_set_automation` with proper triggers, conditions, and actions.

### Manage Dashboards

- "Add a weather card to my main dashboard"
- "Create a new dashboard for the garden sensors"
- "Show me what's on the bedroom dashboard"

Uses `ha_config_get_dashboard` and `ha_config_set_dashboard`.

### Debug Automations

- "The motion sensor automation isn't working, debug it"
- "Show me the last trace for my morning routine"
- "Why didn't the lights turn off last night?"

Uses `ha_get_automation_traces`, `ha_get_history`, and `ha_get_logs`.

### Manage Helpers

- "Create an input_boolean to track vacation mode"
- "Set up a utility meter for my solar panels"
- "List all my helpers"

Uses `ha_config_set_helper` and `ha_config_list_helpers`.

### System Management

- "Check for updates"
- "Create a backup"
- "Restart Home Assistant"
- "What HACS integrations do I have?"

Uses `ha_get_updates`, `ha_manage_backup`, `ha_restart`, `ha_get_hacs_info`.

## Best Practices

When working with Home Assistant through this power, follow these principles (detailed in the steering file):

- **Use native constructs over templates** — numeric_state conditions, time conditions, built-in helpers
- **Use entity_id over device_id** — device_id breaks when devices are re-added
- **Select correct automation mode** — restart for motion lights, queued for sequential, parallel for independent
- **Use built-in helpers before template sensors** — min_max, threshold, derivative, utility_meter
- **Never edit .storage files directly** — use the HA API
- **Use the config API for automations** — don't generate raw YAML snippets
- **Create Template Helpers via UI/API** — not YAML template sensors

Read the `best-practices` steering file for the complete decision workflow and anti-pattern reference.

## Troubleshooting

### MCP Server Won't Connect

**Symptoms:** Tools return errors, agent can't see your HA

**Solutions:**
1. Verify your HA URL is reachable: `curl YOUR_HOMEASSISTANT_URL/api/`
2. Verify your token: `curl -H "Authorization: Bearer YOUR_TOKEN" YOUR_HOMEASSISTANT_URL/api/`
3. Check `uvx` is installed: `uvx --version`
4. Ensure Python 3.11+: `python3 --version`
5. Try running manually: `HOMEASSISTANT_URL=... HOMEASSISTANT_TOKEN=... uvx ha-mcp@latest`

### "Unknown tool" Errors

**Cause:** Tool catalog changed but client has stale cache.

**Solution:** Reconnect or refresh the MCP server in Kiro.

### Tool Search vs Full Catalog

For smaller/local LLMs, enable search-based discovery:
- Set `ENABLE_TOOL_SEARCH=true` in env
- This reduces the visible tool list and uses `ha_search_tools` for on-demand discovery

For Claude Sonnet/Opus models (deferred tool loading), leave it off — the full catalog has no idle context cost.

### Custom Component Required

Some tools (`ha_config_set_yaml`, `ha_list_files`, `ha_read_file`, `ha_write_file`, `ha_delete_file`) require the `ha_mcp_tools` custom component. Install via HACS or manually. These also require feature flags:
- `HAMCP_ENABLE_FILESYSTEM_TOOLS=true`
- `ENABLE_YAML_CONFIG_EDITING=true`

## MCP Config Placeholders

The `mcp.json` shipped with this power uses placeholder values. If you followed the onboarding steps above, these will already be filled in. For reference:

- **`YOUR_HOMEASSISTANT_URL`**: The URL of your Home Assistant instance (e.g., `http://homeassistant.local:8123`, `http://192.168.1.100:8123`).
- **`YOUR_LONG_LIVED_ACCESS_TOKEN`**: A long-lived access token from your HA profile page.

---

**MCP Server:** [homeassistant-ai/ha-mcp](https://github.com/homeassistant-ai/ha-mcp)
**Skills:** [homeassistant-ai/skills](https://github.com/homeassistant-ai/skills)
**Installation:** `uvx ha-mcp@latest`

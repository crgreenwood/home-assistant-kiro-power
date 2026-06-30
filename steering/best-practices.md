# Home Assistant Best Practices

> **Source:** This content is derived from [homeassistant-ai/skills](https://github.com/homeassistant-ai/skills) — the Home Assistant Agent Skills project. All credit to the original authors.

**Core principle:** Use native Home Assistant constructs wherever possible. Templates bypass validation, fail silently at runtime, and make debugging opaque.

## Decision Workflow

Follow this sequence when creating or modifying any automation:

### 0. Modifying existing config?

If your change affects entity IDs or cross-component references — renaming entities, replacing template sensors with helpers, converting device triggers, or restructuring automations — perform impact analysis first:

1. Search for the entity ID across automations, scripts, scenes, dashboards, and groups
2. Check Config Entry data for integrations that store entity_ids (Better Thermostat, Min/Max, Threshold)
3. Update all consumers before or atomically with the rename
4. Verify after the change

### 1. Check for native condition/trigger

Before writing any template, check for native alternatives:

| Template approach | Native alternative |
|---|---|
| `{{ states('x') \| float > 25 }}` | `numeric_state` condition with `above: 25` |
| `{{ is_state('x', 'on') and is_state('y', 'on') }}` | `condition: and` with state conditions |
| `{{ now().hour >= 9 }}` | `condition: time` with `after: "09:00:00"` |
| `wait_template: "{{ is_state(...) }}"` | `wait_for_trigger` with state trigger |

### 2. Check for built-in helper

Before creating a template sensor, check if a built-in helper exists:

| Need | Helper to use |
|------|--------------|
| Sum/average multiple sensors | `min_max` integration |
| Binary any-on/all-on logic | `group` helper |
| Rate of change | `derivative` integration |
| Cross threshold detection | `threshold` integration |
| Consumption tracking | `utility_meter` helper |
| Smoothing noisy sensors | `filter` integration |
| Random values | `random` integration |
| Time tracking | `history_stats` integration |
| Scheduling | `schedule` helper |

If no built-in helper fits, use a **Template Helper** (created via UI or config flow API) — not YAML `template:` sensors.

### 3. Select correct automation mode

Default `single` mode is often wrong:

| Scenario | Correct mode |
|----------|-------------|
| Motion light with timeout | `restart` |
| Sequential processing (door locks) | `queued` |
| Independent per-entity actions | `parallel` |
| One-shot notifications | `single` |

### 4. Use entity_id over device_id

`device_id` breaks when devices are re-added or migrated. Always use `entity_id` in service calls and triggers.

**Exception:** Zigbee2MQTT autodiscovered device triggers are acceptable. For ZHA, use `event` trigger with `device_ieee` (persistent).

### 5. Zigbee buttons/remotes

- **ZHA:** Use `event` trigger with `device_ieee` (persistent across re-pairing)
- **Z2M:** Use `device` trigger (autodiscovered) or `mqtt` trigger

## Critical Anti-Patterns

| Anti-pattern | Use instead | Why |
|---|---|---|
| `condition: template` with float comparison | `condition: numeric_state` | Validated at load, not runtime |
| `wait_template` with is_state | `wait_for_trigger` with state trigger | Event-driven, not polling |
| `device_id` in triggers | `entity_id` or `device_ieee` | device_id breaks on re-add |
| `mode: single` for motion lights | `mode: restart` | Re-triggers must reset the timer |
| Template sensor for sum/mean | `min_max` helper | Declarative, handles unavailable states |
| Template binary sensor with threshold | `threshold` helper | Built-in hysteresis support |
| Renaming entity IDs without impact analysis | Search all consumers first | Renames break dashboards, scripts, scenes silently |
| `template:` sensor in YAML | Template Helper (UI or config flow API) | Requires file edit and reload; harder to manage |
| Editing `.storage/` files directly | Use the HA REST/WebSocket API | Bypasses validation, risks corruption |
| Writing raw YAML for automations | Use the config API | API validates, avoids syntax errors |
| Generating YAML snippets | Use `ha_config_set_automation` tool | Programmatic creation with validation |
| `color_temp` (mireds) in light calls | `color_temp_kelvin` | `color_temp` was removed in HA 2026.3 |
| `enabled: false` in automations.yaml | `automation.turn_off` or entity registry disable | Not a valid top-level key |
| Telling user to edit configuration.yaml | Direct to Settings > Devices & Services | Most integrations are UI-configured |
| Referring to HA "add-ons" | Use "Apps" | Renamed in HA 2026.2 |

## Service Calls Best Practices

- Always use `entity_id` targeting (not `device_id`)
- Use `target:` block for multi-entity/area/device targeting
- Prefer area targeting when controlling all devices in a room
- Check entity state before destructive actions when appropriate

## Scene Best Practices

- Use `scene.create` for snapshot/restore patterns
- Use `scene.apply` for transient state without storing
- Scenes capture only entities explicitly listed — be thorough
- Prefer scenes over scripts for "set multiple entities to known state"

## Dashboard Best Practices

- Use Sections view type for responsive layouts
- Prefer built-in cards over custom HACS cards when possible
- Use conditional cards to reduce clutter
- Group related entities with area-based organization

## YAML-Only Integrations

Some integrations are YAML-only (no config flow): `command_line`, platform-based `mqtt`, `rest`, `template` (legacy), `group` (legacy).

For these:
- Use managed YAML editing with backup and validation
- Know the post-edit action: some require `reload`, others require restart
- Never edit without creating a backup first

## AppDaemon Guidelines

When AppDaemon is appropriate:
- Complex state machines with multiple timers
- Integration with external Python libraries
- Cross-app coordination patterns

Key rules:
- Register callbacks in `initialize()`, never in `__init__()`
- Cancel timers before creating new ones on repeated triggers
- Store persistent state in HA helpers, not instance variables
- Pass entity IDs via `self.args` in `apps.yaml`, don't hardcode

---

**Full reference:** [homeassistant-ai/skills](https://github.com/homeassistant-ai/skills)

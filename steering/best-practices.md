# Home Assistant Best Practices

> **Source:** This content is derived from [homeassistant-ai/skills](https://github.com/homeassistant-ai/skills) — the Home Assistant Agent Skills project. All credit to the original authors.

**Core principle:** Use native Home Assistant constructs wherever possible. Templates bypass validation, fail silently at runtime, and make debugging opaque.

---

## When to load the skill guide

Call `ha_get_skill_guide` BEFORE performing any of the following:

- Creating or editing automations, scripts, scenes, or dashboards
- Choosing between template sensors and built-in helpers
- Writing triggers, conditions, waits, or automation modes
- Setting up Zigbee button/remote automations (ZHA or Zigbee2MQTT)
- Renaming entities or refactoring existing configuration
- Configuring dashboard cards or selecting helpers
- Writing any Jinja2 template

Start with `ha_get_skill_guide(skill="home-assistant-best-practices")` to list all reference files, then load the specific files listed in the Reference Files table below.

---

## Decision Workflow

Follow this sequence when creating or modifying any automation:

### 0. Modifying existing config?

If your change affects entity IDs or cross-component references — renaming entities, replacing template sensors with helpers, converting device triggers, or restructuring automations — **read `references/safe-refactoring.md` first**.

Key steps:
1. Search for the entity ID across automations, scripts, scenes, dashboards, groups, and Config-Entry integrations (Better Thermostat, Min/Max, Threshold, Generic Thermostat, Generic Hygrostat)
2. Update all consumers before or atomically with the rename
3. Verify zero stale references after

### 1. Check for native condition/trigger

Before writing any template, check `references/automation-patterns.md` for native alternatives.

| Template approach | Native alternative |
|---|---|
| `{{ states('x') \| float > 25 }}` | `numeric_state` condition with `above: 25` |
| `{{ is_state('x', 'on') and is_state('y', 'on') }}` | `condition: and` with state conditions |
| `{{ now().hour >= 9 }}` | `condition: time` with `after: "09:00:00"` |
| `{{ is_state('sun.sun', 'below_horizon') }}` | `condition: sun` with `after: sunset` |
| `wait_template: "{{ is_state(...) }}"` | `wait_for_trigger` with state trigger |

Note: `wait_for_trigger` waits for a *change*; `wait_template` passes immediately if the condition is already true. They are not interchangeable — see `references/safe-refactoring.md#trigger-restructuring`.

### 2. Check for built-in helper

Before creating a template sensor, check `references/helper-selection.md`.

| Need | Helper to use |
|------|--------------|
| Sum/average multiple sensors | `min_max` integration |
| Statistical analysis over time | `statistics` integration |
| Binary any-on/all-on logic | `group` helper |
| Rate of change | `derivative` integration |
| Cross threshold detection | `threshold` integration |
| Consumption tracking | `utility_meter` helper |
| Time in state | `history_stats` integration |
| Power to energy conversion | `integration` (Riemann sum) |
| Smoothing noisy sensors | `filter` integration |
| Countdown timer | `timer` helper |
| Weekly schedules | `schedule` helper |
| Time-of-day binary sensor | `tod` helper |
| Thermostat from switch+sensor | `generic_thermostat` |
| Humidistat from switch+sensor | `generic_hygrostat` |
| Expose switch as light/lock/cover | `switch_as_x` |
| Random values | `random` integration |

If no built-in helper fits, use a **Template Helper** (created via UI or config flow API) — not YAML `template:` sensors.

### 3. Select correct automation mode

Default `single` mode is often wrong — read `references/automation-patterns.md#automation-modes`.

| Scenario | Correct mode |
|----------|-------------|
| Motion light with timeout | `restart` |
| Sequential processing (door locks) | `queued` |
| Independent per-entity actions | `parallel` |
| One-shot notifications | `single` |

### 4. Use entity_id over device_id

`device_id` breaks when devices are re-added or migrated. Always use `entity_id` in service calls and triggers. See `references/device-control.md#entity-id-vs-device-id`.

**Exception:** Zigbee2MQTT autodiscovered device triggers are acceptable. For ZHA, use `event` trigger with `device_ieee` (persistent across re-pairing).

### 5. Zigbee buttons/remotes

- **ZHA:** Use `event` trigger with `device_ieee` (persistent)
- **Z2M:** Use `device` trigger (autodiscovered) or `mqtt` trigger

See `references/device-control.md#zigbee-buttonremote-patterns`.

---

## Critical Anti-Patterns

| Anti-pattern | Use instead | Why |
|---|---|---|
| `condition: template` with float comparison | `condition: numeric_state` | Validated at load, not runtime |
| `wait_template` with `is_state` | `wait_for_trigger` with state trigger | Event-driven, not polling |
| `device_id` in triggers | `entity_id` or `device_ieee` | device_id breaks on re-add |
| `mode: single` for motion lights | `mode: restart` | Re-triggers must reset the timer |
| Template sensor for sum/mean | `min_max` helper | Declarative, handles unavailable states |
| Template binary sensor with threshold | `threshold` helper | Built-in hysteresis support |
| Renaming entity IDs without impact analysis | Follow `safe-refactoring.md` workflow | Renames break dashboards, scripts, scenes, Config-Entry data silently |
| Renaming members of UI groups without updating membership | Update group membership via Options Flow | Config Entry `options.entities` is NOT updated by entity registry renames |
| Renaming entities used by Config-Entry integrations (Better Thermostat, Min/Max, Threshold) | Scan and patch Config-Entry `data`/`options` | These integrations store entity_ids in Config Entry, not in the entity registry |
| `template:` sensor in YAML | Template Helper (UI or config flow API) | Requires file edit and reload; harder to manage |
| Editing `.storage/` files directly | Use the HA REST/WebSocket API | Bypasses validation, risks corruption |
| Writing raw YAML for automations | Use `ha_config_set_automation` | API validates and avoids syntax errors |
| Generating YAML snippets | Use the HA config API | Programmatic creation with validation |
| `color_temp` (mireds) in light calls | `color_temp_kelvin` | `color_temp` was removed in HA 2026.3 |
| `enabled: false` in automations.yaml | `automation.turn_off` or entity registry disable | Not a valid top-level key; automation loads as `unavailable` |
| Person/device tracker `entered_home`/`left_home` device triggers | `state` trigger with `to: "home"` | Removed in HA 2026.5 |
| Telling user to edit configuration.yaml | Direct to Settings > Devices & Services | Most integrations are UI-configured |
| Referring to HA "add-ons" | Use "Apps" | Renamed in HA 2026.2 |
| `entity_id` inside `data:` for service calls | `entity_id` inside `target:` | `data.entity_id` is deprecated |

---

## Reference Files

Load these skill files when needed:

| File | When to read |
|------|-------------|
| `references/automation-patterns.md` | Writing triggers, conditions, waits, automation modes, repeat, if/then, choose, trigger IDs, disabling automations |
| `references/helper-selection.md` | Deciding between built-in helpers vs template sensors; full decision matrix |
| `references/template-guidelines.md` | Confirming templates ARE appropriate; template sensor best practices; common patterns; error handling |
| `references/safe-refactoring.md` | Renaming entities, replacing helpers, restructuring automations, Config-Entry blind spots, dashboard storage updates |
| `references/device-control.md` | Service call structure, Zigbee button/remote patterns, domain-specific patterns (lights, climate, covers, media, vacuum) |
| `references/dashboard-guide.md` | Lovelace dashboard layout, view types, tile/grid/area cards, features, custom cards, CSS styling, HACS |
| `references/dashboard-cards.md` | Available card types and card-specific documentation |
| `references/yaml-only-integrations.md` | YAML-only integrations (command_line, mqtt platform, rest) — post-edit actions, restart vs reload |
| `references/domain-docs.md` | Integration/domain documentation for service calls, entity attributes, configuration |
| `references/examples.yaml` | Compound examples combining multiple best practices (motion light, ZHA button remote, etc.) |

---

## Service Call Structure

```yaml
actions:
  - action: domain.service_name   # Required
    target:                       # Use target:, not data.entity_id
      entity_id: entity.id        # Single or list
      area_id: area_name          # Single or list
    data:                         # Service-specific parameters
      parameter: value
    response_variable: result     # Capture response if needed
```

Key rules:
- Always use `target:` block (not `data.entity_id`)
- Prefer `entity_id` over `device_id` in target
- Area targeting is stable and expressive — use it when controlling a whole room

---

## Template Guidelines Summary

Templates are appropriate ONLY for:
- Dynamic service `data.*` values
- Notification message/title bodies
- `event_data` payloads
- `variables:` blocks
- Accessing `trigger.*` context in actions

Templates in `condition:`, `trigger:`, and `wait_for_trigger:` positions should be replaced with native constructs wherever possible. Test any template with `ha_eval_template()` before embedding in automation config.

---

## Dashboard Best Practices

- Use `sections` view type for responsive, grid-based layouts
- Use `tile` cards as the primary card type (replaces legacy entity/light/climate cards)
- Use `grid` cards for multi-column layouts within sections
- Create multiple views with navigation paths; avoid single-view endless scrolling
- Use built-in cards over custom HACS cards where possible
- New dashboard `url_path` values must contain a hyphen (e.g. `my-dashboard`)
- Use `lovelace` to target the built-in default dashboard

Recent additions: `distribution` card (2026.2), section background colors (2026.4), tile card trend graphs and bar gauges (2025.9+).

---

## YAML-Only Integrations

Some integrations have no config flow: `command_line`, platform-based `mqtt`, `rest`, `shell_command`, legacy `template:`.

For these:
- Use managed YAML editing with backup and `check_config` validation
- Know the post-edit action: `template`, `mqtt`, `group` support reload; `command_line`, `rest`, `shell_command` require restart
- Never edit without a backup
- Confirm with the user before triggering a restart

---

## AppDaemon Guidelines

When AppDaemon is appropriate (complex state machines, external Python libraries, cross-app coordination):

- Register callbacks in `initialize()`, never in `__init__()`
- Cancel timers before creating new ones on repeated triggers
- Store persistent state in HA helpers, not instance variables
- Pass entity IDs via `self.args` in `apps.yaml`, don't hardcode

---

**Full skill reference:** Run `ha_get_skill_guide(skill="home-assistant-best-practices")` to list all reference files, then `ha_get_skill_guide(skill="home-assistant-best-practices", file="<filename>")` to read any of them.

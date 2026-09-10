# HA Door/Window Card

A Home Assistant Lovelace card that shows **every** door and window contact
sensor at once — open or closed — instead of only surfacing the ones that
happen to be open right now.

Each tile shows:

- **Open / closed** state, with an animated icon while open
- **Times opened today**
- **Last opened** (time)
- **Current open duration** (live, ticking) while open, or **total time open
  today** while closed
- **Battery level**, for sensors that expose one
- A gentle pulsing highlight when something has been left open longer than a
  configurable threshold

Items are grouped into Doors and Windows sections, each sorted with
currently-open items first (longest-open first), then closed items by most
recently used.

## Configuration

```yaml
type: custom:ha-door-window-card
title: Døre og vinduer
subtitle: Alle åbninger, altid synlige
long_open_minutes: 30
items:
  - name: Hoveddør
    entity: binary_sensor.front_door_contact
    type: door
    icon_set: door        # door | sliding | garage | lock | window
    battery_entity: sensor.front_door_battery   # optional
  - name: Soveværelse
    entity: binary_sensor.bedroom_window
    type: window
```

`items` is the only required field. `type` controls which section (Doors vs.
Windows) an entry appears in; `icon_set` picks the open/closed icon pair
(falls back to a sensible default per `type`). `icon_open` / `icon_closed`
can be set per item to override the icon entirely.

Per-entity "times opened today" / "last opened" / "total open time" are
computed live from Home Assistant's history API — no extra helpers or
template sensors required, and the window resets automatically at local
midnight.

### Open-duration survives Home Assistant restarts

The "currently open for" counter does **not** use the live entity's
`last_changed` attribute, which resets to the moment of the last Home
Assistant Core restart even when the sensor's actual state hasn't changed
— that would otherwise make a door that's been open for hours suddenly
show "just opened" after any restart. Instead it prefers, in order:

1. The entity's own `last_tripped_time` attribute, when the integration
   exposes one — reported by the device/integration itself, so it's
   generally unaffected by HA restarts and is the most trustworthy source.
2. The recorder history's last state-change row, but only when the
   fetched window actually contains the transition into "on" (a
   preceding row with a different state) — if the whole lookback window
   was already "on", that row's timestamp is just an artifact of where
   the query happened to start, not a real event.
3. `live.last_changed`, as a last resort.

The card fetches a 10-day history window (`HISTORY_LOOKBACK_DAYS`) so the
true start of an
in-progress open period can be recovered even across a restart.

**v0.3.0 fix:** the history response is requested with `minimal_response`
to keep the payload small across many entities, which generally omits an
`entity_id` field from each row. Earlier versions matched each returned
series back to its entity by searching for that field and came up empty
for every entity, silently breaking "times opened today" and "total open
time today" (always showing "< 1 min"). The card now matches series to
entities by position in the response array instead, which the
`history/period` endpoint guarantees matches the requested
`filter_entity_id` order.

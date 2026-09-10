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

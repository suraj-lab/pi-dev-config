---
name: hyprland
description: Hyprland config, IPC, layer rules, keybinds, monitors, and desktop troubleshooting
---

# Hyprland Skill

## Principles
- Verify current Hyprland syntax from the wiki before using new or uncertain config/API features.
- Keep config edits minimal and reversible.
- Prefer testing with `hyprctl` before making broad config changes.

Useful wiki pages:
- Layer rules: https://wiki.hypr.land/Configuring/Layer-Rules/
- Binds: https://wiki.hypr.land/Configuring/Basics/Binds/
- Variables: https://wiki.hypr.land/Configuring/Basics/Variables/
- Monitors: https://wiki.hypr.land/Configuring/Monitors/
- Window rules: https://wiki.hypr.land/Configuring/Window-Rules/

## Config Locations
```text
~/.config/hypr/hyprland.conf
~/.config/hypr/hyprlock.conf
```

## Useful CLI
```bash
hyprctl activewindow
hyprctl clients -j
hyprctl workspaces
hyprctl monitors
hyprctl -j activeworkspace
hyprctl reload
```

## Common Config Tasks
```ini
bind = $mainMod, KEY, action
windowrulev2 = rule, class:^(app)$
monitor = name, res@hz, position, scale
exec-once = command
```

## Layer Rule Notes
Hyprland 0.54+ examples:
```ini
layerrule = blur on, match:namespace quickshell
layerrule = blur_popups on, match:namespace quickshell
layerrule = ignore_alpha 0.3, match:namespace quickshell
layerrule = animation none, match:namespace quickshell-launcher
layerrule = blur off, match:namespace quickshell-launcher
```

Syntax reminders:
- `blur on` / `blur off`
- `animation none`
- `ignore_alpha` uses underscore
- Give overlays unique namespaces when targeting layer rules

## Quickshell IPC
```qml
import Quickshell.Hyprland

Hyprland.dispatch("workspace 3")
Hyprland.dispatch("exec [float;size 40% 90%] pavucontrol")
Hyprland.focusedMonitor.name
Hyprland.workspaces.values
```

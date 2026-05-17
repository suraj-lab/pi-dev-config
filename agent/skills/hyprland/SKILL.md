---
name: hyprland
description: Hyprland config, IPC, layer rules, keybinds, monitors, and desktop debugging
---

# Hyprland Skill

## Important — Always Verify Syntax
The Hyprland Lua API is new and changing fast.
Before writing ANY hyprland.lua code, fetch the relevant
wiki page to confirm current syntax:

- Layer rules:  https://wiki.hypr.land/Configuring/Layer-Rules/
- Binds:        https://wiki.hypr.land/Configuring/Basics/Binds/
- Variables:    https://wiki.hypr.land/Configuring/Basics/Variables/
- Monitors:     https://wiki.hypr.land/Configuring/Monitors/
- Window rules: https://wiki.hypr.land/Configuring/Window-Rules/
- Start/overview: https://wiki.hypr.land/Configuring/Start/

Use web_fetch to get the page, read the relevant section,
then write the code. Never rely on the skill alone for
exact Lua API syntax — the wiki is the source of truth.

## Config Location
~/.config/hypr/hyprland.conf    # main config — stays here permanently
~/.config/hypr/hyprlock.conf    # lock screen config

## IPC via Quickshell
```qml
import Quickshell.Hyprland

Hyprland.dispatch("workspace 3")
Hyprland.dispatch("exec [float;size 40% 90%] pavucontrol")
Hyprland.focusedMonitor.name     // current focused monitor name
Hyprland.workspaces.values       // all workspaces
```

## Single-monitor overlays
```qml
property var launcherScreen: null

// Capture focused monitor before opening:
launcherScreen = Hyprland.focusedMonitor?.name ?? null

// In Variants — only show on captured screen:
visible: launcherOpen && modelData.name === launcherScreen
```

## Layer Rule Syntax (Hyprland 0.54+)
```ini
layerrule = blur on, match:namespace quickshell
layerrule = blur_popups on, match:namespace quickshell
layerrule = ignore_alpha 0.3, match:namespace quickshell
layerrule = animation none, match:namespace quickshell-launcher
layerrule = blur off, match:namespace quickshell-launcher
layerrule = animation none, match:namespace quickshell-wallpicker
layerrule = blur off, match:namespace quickshell-wallpicker
layerrule = animation none, match:namespace quickshell-session
layerrule = blur off, match:namespace quickshell-session
layerrule = animation none, match:namespace quickshell-screenshot
layerrule = blur off, match:namespace quickshell-screenshot
```
Syntax notes:
- `blur on` not `blur`
- `animation none` not `noanim`
- `blur off` not `noblur`
- `ignore_alpha` with underscore

## Custom Namespaces
Give each overlay a unique WlrLayershell.namespace so layer rules
can target it independently without affecting the bar.

## Current Keybinds
```ini
bind = $mainMod, Space, exec, qs msg launcher toggle
bind = $mainMod SHIFT, W, exec, qs msg wallpicker toggle
bind = $mainMod SHIFT, S, exec, qs msg screenshot area
bind = , Print, exec, qs msg screenshot screen
bind = CTRL ALT, L, exec, hyprlock
bind = , XF86AudioRaiseVolume, exec, wpctl set-volume @DEFAULT_AUDIO_SINK@ 2%+
bind = , XF86AudioLowerVolume, exec, wpctl set-volume @DEFAULT_AUDIO_SINK@ 2%-
bind = , XF86AudioMute, exec, wpctl set-mute @DEFAULT_AUDIO_SINK@ toggle
bind = , XF86AudioMicMute, exec, wpctl set-mute @DEFAULT_AUDIO_SOURCE@ toggle
```

## Monitor Commands (Phase 2 — system panel)
```bash
hyprctl monitors                                              # current state
hyprctl keyword monitor DP-2,3440x1440@175,0x0,1             # change res/hz
hyprctl keyword monitor HDMI-A-1,1920x1080@74,3440x0,1
```

## Useful CLI
```bash
hyprctl activewindow         # focused window
hyprctl clients -j           # all windows as JSON
hyprctl workspaces           # workspace list
hyprctl -j activeworkspace   # current workspace JSON
hyprctl reload               # reload Hyprland config
```
## Config Editing
- Main config: ~/.config/hypr/hyprland.conf
- Reload: `hyprctl reload`
- Check errors: `hyprctl -j activeworkspace` or journalctl
- Syntax: no semicolons, TOML-like sections
- Test a keybind: `hyprctl dispatch <action>`

## Common Tasks
- Add keybind: bind = $mainMod, KEY, action
- Add window rule: windowrulev2 = rule, class:^(app)$
- Add monitor: monitor = name, res@hz, position, scale
- Add autostart: exec-once = command
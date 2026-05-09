# Editor Theme Tokens

Quick reference for keys available via `EditorInterface.get_editor_theme()` in Godot 4.x.  
All lookups follow the pattern:
```gdscript
theme.get_color("key", "TypeName")
theme.get_icon("key", "EditorIcons")   # second arg is always "EditorIcons"
theme.get_font("key", "EditorFonts")   # second arg is always "EditorFonts"
theme.get_font_size("key", "EditorFonts")
theme.get_constant("key", "TypeName")
```

---

## Colors — `get_color(key, "Editor")`

| Key | Description |
|---|---|
| `accent_color` | Bright highlight color (blue by default) |
| `base_color` | Panel background |
| `dark_color_1` | Slightly darker than base (sidebar bg) |
| `dark_color_2` | Even darker (menu bg) |
| `dark_color_3` | Darkest background |
| `contrast_color_1` | Subtle border / separator |
| `contrast_color_2` | More prominent border |
| `font_color` | Default text color |
| `font_color_disabled` | Dimmed text |
| `font_color_readonly` | Read-only field text |
| `success_color` | Green (typically) |
| `warning_color` | Yellow/orange |
| `error_color` | Red |
| `property_color` | Property label text (slightly dimmed) |
| `readonly_color` | Read-only property text |
| `highlighted_font_color` | Highlighted / selected text |
| `disabled_highlight_color` | Disabled highlight bg |
| `box_selection_fill_color` | Selection rectangle fill |
| `box_selection_stroke_color` | Selection rectangle border |

---

## Icons — `get_icon(key, "EditorIcons")`

All return a `Texture2D` at 16×16 (scaled by editor scale).

### Actions
| Key | Description |
|---|---|
| `Add` | Plus / add |
| `Remove` | Minus / remove |
| `Edit` | Pencil |
| `Rename` | Rename |
| `Duplicate` | Duplicate/copy |
| `Load` | Open file |
| `Save` | Save |
| `Reload` | Reload/refresh |
| `Clear` | Clear / trash |
| `Search` | Magnifier |
| `Close` | X / close |
| `Collapse` | Collapse |
| `Expand` | Expand |
| `MoveUp` | Arrow up |
| `MoveDown` | Arrow down |
| `Play` | Play triangle |
| `Stop` | Stop square |
| `Pause` | Pause bars |

### Node / Scene types
| Key | Description |
|---|---|
| `Node` | Generic node |
| `Node2D` | 2D node |
| `Node3D` | 3D node |
| `Control` | Control node |
| `PackedScene` | Scene resource |
| `Script` | Script resource |
| `GDScript` | GDScript file |
| `Folder` | Folder |
| `FolderMediumThumb` | Folder (medium) |
| `File` | Generic file |

### Tools (for toolbar buttons)
| Key | Description |
|---|---|
| `ToolSelect` | Select tool |
| `ToolMove` | Move tool |
| `ToolRotate` | Rotate tool |
| `ToolScale` | Scale tool |
| `ToolCursor` | Cursor/pointer |

### Status
| Key | Description |
|---|---|
| `StatusSuccess` | Green check |
| `StatusWarning` | Warning triangle |
| `StatusError` | Red X |
| `StatusImmutable` | Lock icon |
| `Visible` | Eye open |
| `Hidden` | Eye closed / slashed |
| `Lock` | Padlock closed |
| `Unlock` | Padlock open |

### Editor-specific
| Key | Description |
|---|---|
| `Inspector` | Inspector panel |
| `FileSystem` | FileSystem panel |
| `Scene` | Scene panel |
| `Script` | Script panel |
| `Debugger` | Debugger |
| `AssetLib` | Asset Library |
| `EditorPlugin` | Plugin icon |
| `SnapGrid` | Snap to grid |
| `History` | History |
| `Help` | Question mark |
| `Settings` | Gear / settings |

---

## Fonts — `get_font(key, "EditorFonts")`

| Key | Description |
|---|---|
| `main` | Default editor body font |
| `bold` | Bold variant |
| `italic` | Italic variant |
| `mono` | Monospace (code font) |
| `title` | Larger heading font |
| `main_msdf` | MSDF version of main (sharper at small sizes) |

## Font Sizes — `get_font_size(key, "EditorFonts")`

| Key | Description |
|---|---|
| `main_size` | Default font size (px) |
| `bold_size` | Bold font size |
| `title_size` | Title font size |
| `mono_size` | Monospace font size |

---

## Constants — `get_constant(key, "TypeName")`

Useful for matching built-in spacing:

| Key | TypeName | Description |
|---|---|---|
| `separation` | `VBoxContainer` | Vertical gap between children |
| `separation` | `HBoxContainer` | Horizontal gap between children |
| `margin_left` | `MarginContainer` | Default left margin |
| `icon_separation` | `Button` | Space between icon and text |
| `h_separation` | `Tree` | Column separation in Tree |
| `draw_guides` | `GraphEdit` | Whether guides draw |

---

## Dock Slot Constants (for `add_control_to_dock`)

```gdscript
EditorPlugin.DOCK_SLOT_LEFT_UL   # Upper-left dock (Scene panel area)
EditorPlugin.DOCK_SLOT_LEFT_BL   # Lower-left dock (FileSystem area)
EditorPlugin.DOCK_SLOT_LEFT_UR   # Upper-left right column
EditorPlugin.DOCK_SLOT_LEFT_BR   # Lower-left right column
EditorPlugin.DOCK_SLOT_RIGHT_UL  # Upper-right (Inspector area)
EditorPlugin.DOCK_SLOT_RIGHT_BL  # Lower-right
EditorPlugin.DOCK_SLOT_RIGHT_UR  # Upper-right right column
EditorPlugin.DOCK_SLOT_RIGHT_BR  # Lower-right right column
```

## Container Constants (for `add_control_to_container`)

```gdscript
EditorPlugin.CONTAINER_TOOLBAR                  # Main top toolbar
EditorPlugin.CONTAINER_SPATIAL_EDITOR_MENU      # 3D editor toolbar
EditorPlugin.CONTAINER_SPATIAL_EDITOR_SIDE_LEFT
EditorPlugin.CONTAINER_SPATIAL_EDITOR_SIDE_RIGHT
EditorPlugin.CONTAINER_SPATIAL_EDITOR_BOTTOM
EditorPlugin.CONTAINER_CANVAS_EDITOR_MENU       # 2D editor toolbar
EditorPlugin.CONTAINER_CANVAS_EDITOR_SIDE_LEFT
EditorPlugin.CONTAINER_CANVAS_EDITOR_SIDE_RIGHT
EditorPlugin.CONTAINER_CANVAS_EDITOR_BOTTOM
EditorPlugin.CONTAINER_INSPECTOR_BOTTOM        # Below the Inspector
EditorPlugin.CONTAINER_PROJECT_SETTING_TAB_LEFT
EditorPlugin.CONTAINER_PROJECT_SETTING_TAB_RIGHT
```

---

## Gotchas

- Icon keys are **case-sensitive** and use PascalCase.
- If a key doesn't exist, `get_icon()` etc. return a fallback — check in the editor with `print(theme.get_icon_list("EditorIcons"))` to see all available keys at runtime.
- Icon names may differ slightly across Godot minor versions; test on your minimum supported version.
- `get_editor_theme()` is only available at runtime in the editor (`@tool` scripts). Calling it from a non-tool script will crash.
- Always guard theme calls: if `not Engine.is_editor_hint(): return`.

---
name: godot-plugin-ui
description: Design and implement Godot editor plugin UIs using GDScript and Control nodes. Use this skill whenever the user wants to build an EditorPlugin, tool script UI, custom dock, inspector plugin, or any editor-facing interface in Godot 4.x. Triggers include: "Godot plugin", "editor plugin", "EditorPlugin", "custom dock", "bottom panel", "inspector plugin", "tool script UI", "GDScript UI", "editor tool", or any request to build UI that lives inside the Godot editor itself (not in a game scene). Also trigger when the user asks to make a panel, form, or layout for an editor workflow in Godot, even if they don't say "plugin" explicitly.
---

# Godot Plugin UI Design Skill

This skill produces native-feeling Godot editor plugin UIs: GDScript code using Control nodes that match the editor's own look, spacing, and conventions. The output should feel indistinguishable from a built-in Godot panel.

---

## Workflow

1. **Clarify the layout** — ask or infer: What panel type? What data does it show/edit? (See Layout Patterns below.)
2. **Mockup in plain language** — describe the structure before writing code (e.g. "Left: ItemList of assets. Right: VBoxContainer with label, LineEdit, and a save button.").
3. **Implement in GDScript** — follow the conventions below. Output a complete `plugin.gd` + the UI scene or inline-built tree.
4. **Theme integration** — always pull colors, icons, and fonts from the editor theme, never hardcode them.

---

## Plugin Scaffolding

Every editor plugin needs this minimum structure:

```
addons/my_plugin/
├── plugin.cfg
└── plugin.gd
```

**plugin.cfg**
```ini
[plugin]
name="My Plugin"
description=""
author=""
version="1.0"
script="plugin.gd"
```

**plugin.gd skeleton**
```gdscript
@tool
extends EditorPlugin

var dock: Control

func _enter_tree() -> void:
    dock = preload("res://addons/my_plugin/MyDock.tscn").instantiate()
    # OR build inline — see Layout Patterns
    add_control_to_dock(DOCK_SLOT_LEFT_UL, dock)

func _exit_tree() -> void:
    if dock:
        remove_control_from_docks(dock)
        dock.free()
```

For a **bottom panel tab** instead of a dock:
```gdscript
add_control_to_bottom_panel(dock, "My Tool")
# remove: remove_control_from_bottom_panel(dock)
```

---

## Theme Integration — The Most Important Rule

Never hardcode colors, font sizes, or icons. Always pull from the editor theme:

```gdscript
func _ready() -> void:
    # Get the live editor theme
    var theme := EditorInterface.get_editor_theme()

    # Colors
    var accent   := theme.get_color("accent_color",   "Editor")
    var base_bg  := theme.get_color("base_color",     "Editor")
    var prop_fg  := theme.get_color("property_color", "Editor")  # label text

    # Icons (returns a Texture2D)
    var icon_add := theme.get_icon("Add", "EditorIcons")

    # Fonts
    var font_default := theme.get_font("main", "EditorFonts")
    var font_bold    := theme.get_font("bold", "EditorFonts")

    # Font size
    var font_size := theme.get_font_size("main_size", "EditorFonts")

    # Apply to nodes
    $MyLabel.add_theme_color_override("font_color", prop_fg)
    $MyButton.icon = icon_add
```

**Editor scale** — respect DPI scaling:
```gdscript
var scale := EditorInterface.get_editor_scale()
var margin := int(4 * scale)
```

For `StyleBoxFlat` panels, always set `bg_color` from the theme rather than a literal color.

---

## Control Node Reference

### Containers (reach for these first)

| Node | Use when |
|---|---|
| `VBoxContainer` | Stacking items vertically (most common) |
| `HBoxContainer` | Row of buttons or labels side by side |
| `HSplitContainer` | Left list + right detail panel |
| `VSplitContainer` | Top/bottom split (e.g. log below editor) |
| `MarginContainer` | Add padding around any subtree |
| `ScrollContainer` | Scrollable list of arbitrary controls |
| `GridContainer` | Property grid (set `columns`) |
| `PanelContainer` | Boxed section with a background |
| `TabContainer` | Multiple views in one panel |

### Interactive Controls

| Node | Use when |
|---|---|
| `Button` | Action; set `.flat = true` for toolbar-style |
| `OptionButton` | Dropdown selector |
| `CheckBox` | Boolean toggle |
| `CheckButton` | Toggle switch style |
| `LineEdit` | Single-line text input |
| `TextEdit` | Multi-line text input |
| `SpinBox` | Numeric input |
| `Slider` (H/V) | Continuous numeric range |
| `ItemList` | Scrollable list of selectable items |
| `Tree` | Hierarchical data (like the scene tree) |

### Display Controls

| Node | Use when |
|---|---|
| `Label` | Static or dynamic text |
| `RichTextLabel` | Formatted/colored text, logs |
| `TextureRect` | Show a Texture2D / icon |
| `Separator` (H/V) | Visual divider between sections |
| `ProgressBar` | Task progress |

---

## Spacing & Layout Conventions

Match the Godot editor's own spacing. These values assume `editor_scale = 1.0`; always multiply by `EditorInterface.get_editor_scale()`.

| Element | Value |
|---|---|
| Outer margin (panel edge) | 4 px |
| Gap between related controls | 4 px |
| Gap between sections | 8 px |
| Button height (standard) | 24 px |
| Icon size (toolbar) | 16 × 16 px |
| Section header font | `bold` EditorFont |
| Body / property font | `main` EditorFont |

Apply margins via `MarginContainer` add_theme_constant_override:
```gdscript
var margin := MarginContainer.new()
var s := int(4 * EditorInterface.get_editor_scale())
for side in ["left","right","top","bottom"]:
    margin.add_theme_constant_override("margin_" + side, s)
```

---

## Layout Patterns

Read `/references/layout-patterns.md` for complete annotated GDScript examples of each pattern. Summary:

- **Left list + right form** — `HSplitContainer` → `ItemList` | `VBoxContainer` with fields
- **Settings panel** — `ScrollContainer` → `VBoxContainer` of `GridContainer` sections
- **Bottom panel / log** — `VBoxContainer`: toolbar row (`HBoxContainer` of flat Buttons) + `RichTextLabel` (`.scroll_following = true`)
- **Toolbar strip** — `HBoxContainer` of `Button` (flat) nodes with icons; add `VSeparator` between groups
- **Inspector-style property list** — `VBoxContainer` of rows: each row is an `HBoxContainer` with a `Label` (min_size.x fixed) + input Control

---

## Common Patterns (Inline)

### Toolbar button with editor icon
```gdscript
var btn := Button.new()
btn.flat = true
btn.icon = EditorInterface.get_editor_theme().get_icon("Play", "EditorIcons")
btn.tooltip_text = "Run"
toolbar.add_child(btn)
```

### Section header
```gdscript
func make_section(title: String) -> Control:
    var lbl := Label.new()
    lbl.text = title
    lbl.add_theme_font_override("font",
        EditorInterface.get_editor_theme().get_font("bold", "EditorFonts"))
    lbl.add_theme_color_override("font_color",
        EditorInterface.get_editor_theme().get_color("accent_color", "Editor"))
    var sep := HSeparator.new()
    var vbox := VBoxContainer.new()
    vbox.add_child(lbl)
    vbox.add_child(sep)
    return vbox
```

### Property row (label + input)
```gdscript
func make_row(label_text: String, input: Control) -> HBoxContainer:
    var row := HBoxContainer.new()
    var lbl := Label.new()
    lbl.text = label_text
    lbl.custom_minimum_size.x = 120 * EditorInterface.get_editor_scale()
    row.add_child(lbl)
    row.add_child(input)
    input.size_flags_horizontal = Control.SIZE_EXPAND_FILL
    return row
```

### Saving plugin state across sessions
```gdscript
# In plugin.gd
func _get_state() -> Dictionary:
    return { "my_value": dock.get_value() }

func _set_state(state: Dictionary) -> void:
    dock.set_value(state.get("my_value", ""))
```

---

## Inspector Plugin Pattern

When extending the Inspector for a specific class:

```gdscript
@tool
extends EditorInspectorPlugin

func _can_handle(object: Object) -> bool:
    return object is MyCustomResource

func _parse_property(object, type, name, hint_type, hint_string, usage_flags, wide) -> bool:
    if name == "my_special_prop":
        var editor := MyCustomPropertyEditor.new()
        add_property_editor(name, editor)
        return true  # suppress default editor for this prop
    return false
```

Register it in `plugin.gd`:
```gdscript
var _inspector_plugin: EditorInspectorPlugin

func _enter_tree():
    _inspector_plugin = MyInspectorPlugin.new()
    add_inspector_plugin(_inspector_plugin)

func _exit_tree():
    remove_inspector_plugin(_inspector_plugin)
```

---

## Checklist Before Outputting Code

- [ ] All colors/fonts/icons come from `EditorInterface.get_editor_theme()`
- [ ] Margins and sizes multiplied by `EditorInterface.get_editor_scale()`
- [ ] `@tool` annotation on every script that runs in the editor
- [ ] `_exit_tree()` frees all added controls
- [ ] No `await` or long-running code on the main thread (use `Thread` or `EditorFileSystem` signals)
- [ ] `plugin.cfg` included with correct script path

---

## Reference Files

- **`references/layout-patterns.md`** — Full GDScript implementations of each layout pattern with comments. Read this when building any non-trivial panel.
- **`references/editor-theme-tokens.md`** — Comprehensive list of theme color keys, icon names, and font names available from `get_editor_theme()`. Read this when you need a specific color or icon and aren't sure of the key name.

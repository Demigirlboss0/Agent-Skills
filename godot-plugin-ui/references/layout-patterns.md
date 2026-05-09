# Layout Patterns — Full GDScript Implementations

Complete, copy-paste-ready GDScript for the most common Godot editor plugin panel layouts.

---

## 1. Left List + Right Form (HSplitContainer)

Classic asset-manager or item-editor layout.

```gdscript
@tool
extends Control

func _ready() -> void:
    var split := HSplitContainer.new()
    split.set_anchors_and_offsets_preset(Control.PRESET_FULL_RECT)
    split.split_offset = 180  # px from left; adjust to taste
    add_child(split)

    # --- Left: item list ---
    var left := VBoxContainer.new()
    left.custom_minimum_size.x = 160
    split.add_child(left)

    var list_label := Label.new()
    list_label.text = "Items"
    list_label.add_theme_font_override("font",
        EditorInterface.get_editor_theme().get_font("bold", "EditorFonts"))
    left.add_child(list_label)
    left.add_child(HSeparator.new())

    var list := ItemList.new()
    list.size_flags_vertical = Control.SIZE_EXPAND_FILL
    list.add_item("Item One")
    list.add_item("Item Two")
    list.item_selected.connect(_on_item_selected)
    left.add_child(list)

    var add_btn := Button.new()
    add_btn.text = "Add"
    add_btn.icon = EditorInterface.get_editor_theme().get_icon("Add", "EditorIcons")
    left.add_child(add_btn)

    # --- Right: detail form ---
    var right_margin := MarginContainer.new()
    right_margin.size_flags_horizontal = Control.SIZE_EXPAND_FILL
    var sc := int(4 * EditorInterface.get_editor_scale())
    for side in ["left","right","top","bottom"]:
        right_margin.add_theme_constant_override("margin_" + side, sc)
    split.add_child(right_margin)

    var form := VBoxContainer.new()
    right_margin.add_child(form)

    form.add_child(_make_section("Properties"))
    form.add_child(_make_row("Name", LineEdit.new()))
    form.add_child(_make_row("Value", SpinBox.new()))

    var save_btn := Button.new()
    save_btn.text = "Save"
    save_btn.size_flags_horizontal = Control.SIZE_SHRINK_END
    form.add_child(save_btn)

func _on_item_selected(index: int) -> void:
    pass  # populate form from data[index]

func _make_section(title: String) -> VBoxContainer:
    var vbox := VBoxContainer.new()
    var lbl := Label.new()
    lbl.text = title
    lbl.add_theme_font_override("font",
        EditorInterface.get_editor_theme().get_font("bold", "EditorFonts"))
    lbl.add_theme_color_override("font_color",
        EditorInterface.get_editor_theme().get_color("accent_color", "Editor"))
    vbox.add_child(lbl)
    vbox.add_child(HSeparator.new())
    return vbox

func _make_row(label_text: String, input: Control) -> HBoxContainer:
    var row := HBoxContainer.new()
    var lbl := Label.new()
    lbl.text = label_text
    lbl.custom_minimum_size.x = int(120 * EditorInterface.get_editor_scale())
    row.add_child(lbl)
    input.size_flags_horizontal = Control.SIZE_EXPAND_FILL
    row.add_child(input)
    return row
```

---

## 2. Settings Panel (Scrollable Property Grid)

Good for plugin configuration or bulk property editing.

```gdscript
@tool
extends Control

func _ready() -> void:
    var scroll := ScrollContainer.new()
    scroll.set_anchors_and_offsets_preset(Control.PRESET_FULL_RECT)
    scroll.horizontal_scroll_mode = ScrollContainer.SCROLL_MODE_DISABLED
    add_child(scroll)

    var vbox := VBoxContainer.new()
    vbox.size_flags_horizontal = Control.SIZE_EXPAND_FILL
    scroll.add_child(vbox)

    # Outer margin
    var margin := MarginContainer.new()
    var s := int(8 * EditorInterface.get_editor_scale())
    for side in ["left","right","top","bottom"]:
        margin.add_theme_constant_override("margin_" + side, s)
    vbox.add_child(margin)

    var content := VBoxContainer.new()
    margin.add_child(content)

    # Section: General
    content.add_child(_make_section("General"))
    var grid_general := GridContainer.new()
    grid_general.columns = 2
    content.add_child(grid_general)
    _add_prop(grid_general, "Plugin Name", LineEdit.new())
    _add_prop(grid_general, "Auto Save", CheckButton.new())

    content.add_child(HSeparator.new())

    # Section: Export
    content.add_child(_make_section("Export"))
    var grid_export := GridContainer.new()
    grid_export.columns = 2
    content.add_child(grid_export)
    var fmt := OptionButton.new()
    fmt.add_item("JSON")
    fmt.add_item("Binary")
    _add_prop(grid_export, "Format", fmt)

func _add_prop(grid: GridContainer, label_text: String, input: Control) -> void:
    var lbl := Label.new()
    lbl.text = label_text
    lbl.size_flags_horizontal = Control.SIZE_EXPAND_FILL
    grid.add_child(lbl)
    input.size_flags_horizontal = Control.SIZE_EXPAND_FILL
    grid.add_child(input)

func _make_section(title: String) -> Label:
    var lbl := Label.new()
    lbl.text = title
    lbl.add_theme_font_override("font",
        EditorInterface.get_editor_theme().get_font("bold", "EditorFonts"))
    lbl.add_theme_color_override("font_color",
        EditorInterface.get_editor_theme().get_color("accent_color", "Editor"))
    return lbl
```

---

## 3. Bottom Panel with Log

Output log with a toolbar above it — useful for build tools, linters, importers.

```gdscript
@tool
extends Control

var _log: RichTextLabel

func _ready() -> void:
    var vbox := VBoxContainer.new()
    vbox.set_anchors_and_offsets_preset(Control.PRESET_FULL_RECT)
    add_child(vbox)

    # Toolbar
    var toolbar := HBoxContainer.new()
    vbox.add_child(toolbar)

    var run_btn := Button.new()
    run_btn.flat = true
    run_btn.text = "Run"
    run_btn.icon = EditorInterface.get_editor_theme().get_icon("Play", "EditorIcons")
    run_btn.pressed.connect(_on_run)
    toolbar.add_child(run_btn)

    toolbar.add_child(VSeparator.new())

    var clear_btn := Button.new()
    clear_btn.flat = true
    clear_btn.text = "Clear"
    clear_btn.icon = EditorInterface.get_editor_theme().get_icon("Clear", "EditorIcons")
    clear_btn.pressed.connect(_on_clear)
    toolbar.add_child(clear_btn)

    # Spacer to push anything else right
    var spacer := Control.new()
    spacer.size_flags_horizontal = Control.SIZE_EXPAND_FILL
    toolbar.add_child(spacer)

    toolbar.add_child(HSeparator.new())  # visual bottom border of toolbar

    # Log
    _log = RichTextLabel.new()
    _log.size_flags_vertical = Control.SIZE_EXPAND_FILL
    _log.scroll_following = true
    _log.selection_enabled = true
    _log.bbcode_enabled = true
    vbox.add_child(_log)

func log_info(msg: String) -> void:
    _log.append_text("[color=gray]%s[/color]\n" % msg)

func log_error(msg: String) -> void:
    _log.append_text("[color=red][b]ERROR:[/b] %s[/color]\n" % msg)

func log_success(msg: String) -> void:
    _log.append_text("[color=green]✓ %s[/color]\n" % msg)

func _on_run() -> void:
    log_info("Running...")

func _on_clear() -> void:
    _log.clear()
```

---

## 4. Toolbar Strip (Standalone)

For injecting a toolbar into the main editor viewport area.

```gdscript
# In plugin.gd _enter_tree():
var toolbar := HBoxContainer.new()

var select_btn := Button.new()
select_btn.flat = true
select_btn.toggle_mode = true
select_btn.icon = EditorInterface.get_editor_theme().get_icon("ToolSelect", "EditorIcons")
select_btn.tooltip_text = "Select (Q)"
toolbar.add_child(select_btn)

var move_btn := Button.new()
move_btn.flat = true
move_btn.toggle_mode = true
move_btn.icon = EditorInterface.get_editor_theme().get_icon("ToolMove", "EditorIcons")
move_btn.tooltip_text = "Move (W)"
toolbar.add_child(move_btn)

toolbar.add_child(VSeparator.new())  # group separator

var snap_check := CheckButton.new()
snap_check.text = "Snap"
toolbar.add_child(snap_check)

add_control_to_container(EditorPlugin.CONTAINER_CANVAS_EDITOR_MENU, toolbar)
# remove: remove_control_from_container(EditorPlugin.CONTAINER_CANVAS_EDITOR_MENU, toolbar)
```

---

## 5. TabContainer Panel

Multiple views inside one dock or bottom panel.

```gdscript
@tool
extends Control

func _ready() -> void:
    var tabs := TabContainer.new()
    tabs.set_anchors_and_offsets_preset(Control.PRESET_FULL_RECT)
    add_child(tabs)

    # Tab 1
    var tab1 := VBoxContainer.new()
    tab1.name = "Overview"  # .name becomes the tab label
    tabs.add_child(tab1)
    tab1.add_child(_make_label("Overview content here"))

    # Tab 2
    var tab2 := VBoxContainer.new()
    tab2.name = "Settings"
    tabs.add_child(tab2)
    tab2.add_child(_make_label("Settings content here"))

func _make_label(text: String) -> Label:
    var lbl := Label.new()
    lbl.text = text
    return lbl
```

---

## Notes on Inline vs. Scene-Based Panels

**Inline (all GDScript, no .tscn)** — preferred for:
- Simple panels that don't need the visual editor
- Keeping the plugin self-contained in a single script
- Panels that are highly dynamic

**Scene-based (.tscn)** — preferred for:
- Complex layouts that benefit from the visual editor
- Panels with many static nodes
- Reuse across multiple plugins

When using scenes, add `@tool` to the root script and use `preload(...).instantiate()` in `plugin.gd`.

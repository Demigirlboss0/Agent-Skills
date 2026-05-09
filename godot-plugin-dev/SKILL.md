---
name: godot-plugin-dev
description: >
  Use this skill whenever the user is writing, debugging, reviewing, or generating
  code for a Godot 4.x editor plugin. Triggers include: any mention of EditorPlugin,
  plugin.gd, plugin.cfg, addons/, GDScript for editor tooling, custom editor panels,
  inspector plugins, resource generation from the editor, or any request to scaffold,
  fix, or review Godot editor plugin code. Also trigger when the user pastes a Godot
  parse error or runtime error from an editor plugin context. Do not trigger for
  general Godot game scripting unrelated to the editor plugin API.
---

# Godot 4.x Editor Plugin Development

This skill covers correct patterns, common pitfalls, and required boilerplate for
Godot 4.x editor plugins. Many LLMs produce Godot 3 syntax or hallucinate API calls
that do not exist in Godot 4. Always verify against the patterns in this skill before
emitting plugin code.

---

## Required Boilerplate

Every editor plugin script MUST begin with the `@tool` annotation and extend
`EditorPlugin`. Without `@tool`, Godot 4 will throw a parse error and refuse to load
the script entirely.

```gdscript
@tool
extends EditorPlugin
```

> **Common mistake**: Using the `tool` keyword (no `@`). This is Godot 3 syntax.
> Godot 4 removed it. Always use `@tool`.

Every plugin also needs a `plugin.cfg` at the root of the plugin folder:

```ini
[plugin]
name="My Plugin"
description="What it does"
author="You"
version="1.0"
script="plugin.gd"
```

Without `plugin.cfg`, Godot will not recognize the plugin in Project Settings → Plugins.

---

## Initialization and Teardown

Use `_notification()` for lifecycle management. Do NOT use both `_ready()` and
`_notification(NOTIFICATION_READY)` — they both fire on initialization and will cause
double-setup bugs.

**Correct pattern:**

```gdscript
@tool
extends EditorPlugin

var main_panel: Control

func _notification(what: int) -> void:
    if what == NOTIFICATION_READY:
        _setup()
    elif what == NOTIFICATION_EXIT_TREE:
        _teardown()

func _setup() -> void:
    main_panel = preload("res://addons/my_plugin/main_panel.tscn").instantiate()
    add_control_to_bottom_panel(main_panel, "My Plugin")

func _teardown() -> void:
    if main_panel:
        remove_control_from_bottom_panel(main_panel)
        main_panel.queue_free()
        main_panel = null
```

> **Common mistake**: Using `_ready()` for setup. In EditorPlugin, prefer
> `_notification(NOTIFICATION_READY)` to avoid conflicts with Godot's internal
> initialization order.

---

## Adding UI to the Editor

| Where | Method | Remove with |
|---|---|---|
| Bottom panel tab | `add_control_to_bottom_panel(control, "Label")` | `remove_control_from_bottom_panel(control)` |
| Left dock | `add_control_to_dock(DOCK_SLOT_LEFT_UL, control)` | `remove_control_from_docks(control)` |
| Container | `add_control_to_container(CONTAINER_TOOLBAR, control)` | `remove_control_from_container(...)` |

> **Common mistake**: Calling `add_control()` or `remove_control()` — these do not
> exist. Always use the full method names above.

Always free the control in teardown. Failing to call `queue_free()` after removing
from the panel will leak the node.

---

## File and Directory API

Godot 4 reorganized the file/directory API significantly from Godot 3. Use these
and nothing else.

### Checking existence

```gdscript
# File exists:
FileAccess.file_exists("res://path/to/file.gd")

# Directory exists (use absolute path form):
DirAccess.dir_exists_absolute("res://path/to/dir")
```

> **Common mistake**: `FileAccess.dir_exists()` — does not exist.
> `DirAccess.dir_exists()` — instance method, requires an open handle; prefer the
> static `dir_exists_absolute()` form for simplicity.

### Creating directories

```gdscript
# Create a single directory:
DirAccess.make_dir_absolute("res://path/to/dir")

# Create a full path recursively:
DirAccess.make_dir_recursive_absolute("res://path/to/deeply/nested/dir")
```

### Deleting files

```gdscript
# Delete a file:
DirAccess.remove_absolute("res://path/to/file.tres")
```

> **Common mistake**: `DirAccess.remove_file()` — does not exist.

### Listing directory contents

```gdscript
var dir = DirAccess.open("res://towers/")
if dir:
    dir.list_dir_begin()
    var file_name = dir.get_next()
    while file_name != "":
        if not dir.current_is_dir() and file_name.ends_with(".tres"):
            # process file
            pass
        file_name = dir.get_next()
    dir.list_dir_end()
```

### Reading and writing files

```gdscript
# Write:
var file = FileAccess.open("res://path/file.gd", FileAccess.WRITE)
if file:
    file.store_string(content)
    file.close()

# Read:
var file = FileAccess.open("res://path/file.gd", FileAccess.READ)
if file:
    var content = file.get_as_text()
    file.close()
```

---

## Resources

### Defining a custom Resource

```gdscript
@tool
extends Resource
class_name MyResource

@export var my_field: String = ""
@export var my_int: int = 0
```

The `@tool` annotation is required on Resource scripts used in editor plugins,
otherwise the editor cannot instantiate them at edit time.

### Saving and loading resources

```gdscript
# Save:
ResourceSaver.save(my_resource, "res://path/to/file.tres")

# Load:
var res = load("res://path/to/file.tres")
if res is MyResource:
    # use it
```

### Forcing the editor to recognize new/changed files

After writing a new `.gd` or `.tres` file to disk from plugin code, call:

```gdscript
EditorInterface.get_resource_filesystem().scan()
```

Without this, Godot's editor filesystem won't know the file exists until the user
manually triggers a rescan. New resource classes won't be available to `load()` or
`ResourceSaver` until the scan completes.

---

## Showing Dialogs and Popup Windows from a Plugin

### Simple confirmation/message dialogs

Use `AcceptDialog` or `ConfirmationDialog`, parented to `EditorInterface.get_base_control()`:

```gdscript
var dialog = AcceptDialog.new()
dialog.dialog_text = "Something happened."
EditorInterface.get_base_control().add_child(dialog)
dialog.popup_centered()
dialog.confirmed.connect(func(): dialog.queue_free())
```

### Custom popup windows

Use a `Window` node parented to `EditorInterface.get_base_control()`. Build your UI inside it normally:

```gdscript
func _show_my_popup() -> void:
    var window = Window.new()
    window.title = "My Popup"
    window.size = Vector2i(400, 300)
    window.exclusive = true
    window.close_requested.connect(func(): window.queue_free())
    EditorInterface.get_base_control().add_child(window)

    var vbox = VBoxContainer.new()
    vbox.set_anchors_preset(Control.PRESET_FULL_RECT)
    vbox.add_theme_constant_override("separation", 8)
    window.add_child(vbox)

    # ... add your UI nodes to vbox ...

    window.popup_centered()
```

> **Common mistake**: Building a fake modal by adding a `Control` + `ColorRect` overlay
> to `get_tree().root`, then placing a `PanelContainer` on top. This renders as a
> transparent dimmed overlay with floating UI elements — it doesn't break out into a
> real window. Always use `Window` + `EditorInterface.get_base_control()`.

> **Common mistake**: Adding any editor popup or dialog to `get_tree().root` directly.
> Always parent to `EditorInterface.get_base_control()` — this includes `Window`,
> `AcceptDialog`, `ConfirmationDialog`, and `EditorFileDialog`.

### File picker dialog

```gdscript
var dialog = EditorFileDialog.new()
dialog.file_mode = EditorFileDialog.FILE_MODE_OPEN_FILE
dialog.access = EditorFileDialog.ACCESS_RESOURCES
dialog.filters = ["*.tscn"]
dialog.file_selected.connect(func(path):
    var res = load(path)
    # use res
    dialog.queue_free()
)
dialog.canceled.connect(func(): dialog.queue_free())
EditorInterface.get_base_control().add_child(dialog)
dialog.popup_centered(Vector2(800, 600))
```

---



### Refreshing the editor after changes

```gdscript
# Notify the editor that a resource has changed:
EditorInterface.get_resource_filesystem().update_file(file_path)

# Reload a script in the editor:
EditorInterface.get_script_editor().get_current_script()
```

### Showing dialogs from a plugin

Use `EditorInterface` to get the base control for dialog parenting:

```gdscript
var dialog = AcceptDialog.new()
dialog.dialog_text = "Something happened."
EditorInterface.get_base_control().add_child(dialog)
dialog.popup_centered()
# Connect to dialog.confirmed / dialog.canceled to handle response
dialog.confirmed.connect(func(): dialog.queue_free())
```

> **Common mistake**: Adding dialogs as children of the plugin script node or the
> panel control. They must be children of the editor base control to display correctly.

---

## GDScript 4 Syntax Reminders

These are frequently regressed by models trained on mixed Godot 3/4 data:

| Godot 3 (WRONG) | Godot 4 (CORRECT) |
|---|---|
| `tool` | `@tool` |
| `export var x` | `@export var x` |
| `onready var x` | `@onready var x` |
| `yield(signal)` | `await signal` |
| `connect("sig", self, "_method")` | `signal.connect(_method)` |
| `OS.get_ticks_msec()` | `Time.get_ticks_msec()` |
| `File.new()` / `Directory.new()` | `FileAccess.open()` / `DirAccess.open()` |
| `.empty()` | `.is_empty()` |
| `rand_range(a, b)` | `randf_range(a, b)` |

---

## Debugging Tips

The Godot editor GUI shows minimal error detail. For full stack traces and line
numbers, **always run Godot from the terminal during development**:

```bash
godot --editor --path /path/to/your/project
```

Parse errors, script load failures, and null reference errors all print full context
to stdout. The in-editor error popup is nearly useless by comparison.

---

## See Also

- `references/resource-patterns.md` — deeper patterns for custom resource schemas,
  regeneration strategies, and `.tres` file layout (load when building resource-heavy
  plugins)

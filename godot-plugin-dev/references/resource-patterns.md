# Resource Patterns for Godot 4 Editor Plugins

Referenced from SKILL.md. Load this when the plugin involves generating, managing,
or versioning custom Resource schemas from editor code.

---

## Regenerate vs Patch

When a plugin generates a `.gd` resource class and the schema changes (e.g. a new
field is added), the safest strategy is **full regeneration** — overwrite the file
entirely from a template string in plugin code.

Patching (finding and inserting lines) is fragile: it assumes the file hasn't been
manually edited and that line structure is predictable. Full regeneration is simpler
to implement and reason about.

If you choose regeneration, document clearly that the generated file should not be
hand-edited. If developers need custom behavior, they should extend the generated
class in a separate script.

---

## Generating a Resource Class from Plugin Code

```gdscript
func _generate_resource_class(output_path: String, extra_fields: Array) -> void:
    var field_lines = ""
    for field in extra_fields:
        field_lines += "@export var %s: int = 0\n" % field

    var content = """@tool
extends Resource
class_name MyResource

@export var display_name: String = ""
@export var some_value: float = 1.0

%s
""" % field_lines

    var file = FileAccess.open(output_path, FileAccess.WRITE)
    if file:
        file.store_string(content)
        file.close()

    # Tell the editor the file exists/changed
    EditorInterface.get_resource_filesystem().scan()
```

Note the `@tool` at the top of the generated script — required for editor-time
instantiation.

---

## Updating Existing .tres Files After Schema Change

When a field is added to a Resource class, existing `.tres` files won't have it —
Godot will use the default value from the class definition, which is fine. No manual
migration needed for additions.

When a field is removed, existing `.tres` files will have orphaned data. Godot will
warn about unknown properties on load but will not crash. Orphaned data is harmless
unless it causes confusion.

When a field is renamed, existing `.tres` files will lose the value entirely (the old
key becomes orphaned, the new key gets the default). If migration matters, load each
`.tres`, read the old key before regenerating the class, and write the value to the
new key after.

---

## Iterating Over All Resources in a Directory

```gdscript
func get_all_resources_of_type(dir_path: String, type) -> Array:
    var results = []
    var dir = DirAccess.open(dir_path)
    if not dir:
        return results

    dir.list_dir_begin()
    var file_name = dir.get_next()
    while file_name != "":
        if not dir.current_is_dir() and file_name.ends_with(".tres"):
            var full_path = dir_path.path_join(file_name)
            var res = load(full_path)
            if res and res is type:
                results.append(res)
        file_name = dir.get_next()
    dir.list_dir_end()

    return results
```

---

## Slug / Filename Derivation from Display Name

A consistent pattern for turning a human-readable name into a safe filename:

```gdscript
func derive_filename(display_name: String, suffix: String) -> String:
    # Strip non-alphanumeric, non-space characters
    var cleaned = ""
    for ch in display_name:
        if ch.unicode_at(0) >= 65 and ch.unicode_at(0) <= 90:   # A-Z
            cleaned += ch
        elif ch.unicode_at(0) >= 97 and ch.unicode_at(0) <= 122: # a-z
            cleaned += ch
        elif ch.unicode_at(0) >= 48 and ch.unicode_at(0) <= 57:  # 0-9
            cleaned += ch
        elif ch == " ":
            cleaned += " "

    # Title-case each word and concatenate
    var words = cleaned.split(" ", false)
    var result = ""
    for word in words:
        if not word.is_empty():
            result += word.substr(0, 1).to_upper() + word.substr(1).to_lower()

    return result + suffix + ".tres"
```

Example: `"Archimedes' Siege Mirror"` → `ArchimedesSiegeMirrorTowerData.tres`

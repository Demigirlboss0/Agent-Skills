---
name: openscad
description: >
  Generate, edit, debug, and explain OpenSCAD code for 3D modeling and parametric CAD design.
  Use this skill whenever the user wants to: create a 3D-printable part or model, write or fix
  OpenSCAD (.scad) scripts, design parametric objects (boxes, brackets, enclosures, gears, threads,
  snap fits, hinges), convert a sketch or idea into a CAD script, or learn OpenSCAD syntax and
  patterns. Trigger on keywords: OpenSCAD, .scad, parametric CAD, 3D model, printable part,
  STL generation, CSG modeling, linear_extrude, rotate_extrude, or any request to "make a 3D
  model of X". Even casual requests like "make me a box with a lid" or "design a phone stand"
  should trigger this skill if 3D modeling context is present.
---

# OpenSCAD Skill

OpenSCAD is a script-based parametric 3D CAD tool. Models are defined entirely in code — no
mouse-dragging. This makes it ideal for parametric, reusable, and version-controlled designs.

---

## Core Principles

1. **Always write parametric code** — use variables at the top, never hardcode dimensions inline.
2. **Comment every parameter** with units and purpose.
3. **Use modules** for any shape used more than once.
4. **Set `$fn` (or `$fs`/`$fa`)** appropriately — low for development (16–32), high for export (64–128).
5. **Prefer `difference()` with a small tolerance** (0.1–0.2 mm) for cuts and fits.
6. **Follow the "preview first" convention** — keep render-heavy ops (hull, minkowski) noted with a `/* slow */` comment.

---

## File Structure Template

Every `.scad` file should follow this layout:

```scad
// ============================================================
// <Part Name>
// Description: <what it is and what it's for>
// Units: mm
// Author: <author>
// ============================================================

// --- Parameters ---
length   = 50;   // mm, overall length
width    = 30;   // mm, overall width
height   = 20;   // mm, overall height
wall     = 2;    // mm, wall thickness
tolerance = 0.2; // mm, fit tolerance for mating parts

// --- Resolution ---
$fn = 32;  // increase to 64–128 for final export

// --- Main Model ---
main();

// --- Modules ---
module main() {
    // top-level assembly here
}
```

---

## Language Quick Reference

### Primitives
```scad
cube([x, y, z], center=false);
sphere(r=10);
cylinder(h=20, r=5, center=false);
cylinder(h=20, r1=5, r2=3);   // truncated cone
polyhedron(points=[...], faces=[...]);
```

### 2D Primitives (for extrusion)
```scad
square([w, h], center=false);
circle(r=10);
polygon(points=[[0,0],[10,0],[5,8]]);
text("Hello", size=8, font="Liberation Sans:style=Bold");
```

### Transformations
```scad
translate([x, y, z]) child();
rotate([rx, ry, rz]) child();       // degrees
rotate(a=45, v=[0,0,1]) child();    // axis-angle form
scale([sx, sy, sz]) child();
mirror([1, 0, 0]) child();          // mirror across YZ plane
resize([newx, newy, newz]) child();
```

### Boolean Operations
```scad
union()       { a(); b(); }   // join shapes (default if no op)
difference()  { base(); hole(); }   // subtract second+ from first
intersection(){ a(); b(); }  // keep only overlap
```

### Extrusion
```scad
linear_extrude(height=10, twist=0, slices=20, scale=1.0)
    2d_shape();

rotate_extrude(angle=360, $fn=64)
    translate([r, 0]) circle(d=tube_d);
```

### Loops and Conditionals
```scad
for (i = [0 : step : max])  translate([i*spacing, 0, 0]) child();
for (pos = [[0,0], [10,5]]) translate(pos) child();

if (condition) { ... } else { ... }
```

### Functions and Modules
```scad
function hypotenuse(a, b) = sqrt(a*a + b*b);

module rounded_box(w, h, d, r) {
    hull() {
        for (x = [r, w-r], y = [r, h-r])
            translate([x, y, 0]) cylinder(r=r, h=d);
    }
}
```

### Useful Built-ins
```scad
len(list)          // list length
concat(a, b)       // join lists
sin(deg), cos(deg), tan(deg), atan2(y,x)
min(a,b), max(a,b), abs(x), floor(x), ceil(x), round(x)
echo("value =", myvar);   // debug output
```

---

## Common Design Patterns

### Parametric Box with Lid
See `references/patterns.md` → "Box with Lid"

### Rounded Box (hull trick)
```scad
module rounded_box(w, h, d, r=2) {
    hull()
        for (x=[r, w-r], y=[r, h-r])
            translate([x, y, 0])
                cylinder(r=r, h=d, $fn=32);
}
```

### Snap Fit Tab
```scad
module snap_tab(w=8, h=4, t=1.2, barb=0.8) {
    difference() {
        cube([w, t, h]);
        // angled barb cutout
        translate([0, t-barb, h-barb*2])
            rotate([45, 0, 0])
                cube([w, barb*2, barb*2]);
    }
}
```

### Mounting Hole Pattern
```scad
module bolt_holes(d=3.2, positions=[]) {
    for (p = positions)
        translate([p[0], p[1], 0])
            cylinder(d=d, h=100, center=true);
}
// Usage: difference() { base(); bolt_holes(positions=[[5,5],[45,5],[5,35],[45,35]]); }
```

### Text Label on a Surface
```scad
linear_extrude(height=0.4)
    text("LABEL", size=6, halign="center", valign="center",
         font="Liberation Sans:style=Bold");
```

### Thread (using external library)
For metric threads, recommend the **BOSL2** library:
```scad
include <BOSL2/std.scad>
include <BOSL2/threading.scad>

threaded_rod(d=10, l=20, pitch=1.5, $fn=32);
threaded_hole(d=10, l=10, pitch=1.5, $fn=32);  // inside difference()
```

---

## Quality Levels

| Use Case         | `$fn` | `$fs` |
|-----------------|-------|-------|
| Fast preview    | 16    | —     |
| Development     | 32    | —     |
| Final export    | 64–128 | 0.5  |
| Very smooth     | 256   | 0.2  |

Add this near the top of a file to switch easily:
```scad
DRAFT = false;  // set true for fast preview
$fn = DRAFT ? 16 : 64;
```

---

## Tolerances and Fit Guide

| Fit Type        | Clearance (mm) |
|----------------|---------------|
| Press fit       | 0.0 – 0.1     |
| Snug/sliding    | 0.1 – 0.2     |
| Easy slide      | 0.2 – 0.4     |
| Snap/removable  | 0.4 – 0.6     |

Always subtract mating parts with a `tolerance` variable:
```scad
tolerance = 0.2;
difference() {
    shell();
    translate([...]) scale([1,1,1]) part_to_fit(); // no tolerance here
    // OR
    translate([...]) cube([part_w + tolerance*2, part_d + tolerance*2, part_h + 1]);
}
```

---

## Debugging Tips

- `echo("var =", myvar);` — prints to console
- `%child();` — renders child as transparent ghost (preview only)
- `#child();` — renders child highlighted in red
- `!child();` — renders ONLY this child (isolate one piece)
- `*child();` — disables this child entirely

---

## Output Guidance

When generating OpenSCAD code:

1. **Always include a full, runnable `.scad` file** — not snippets unless specifically asked.
2. **Explain key parameters** inline with `//` comments.
3. **Use `echo()` to verify key computed values** during development.
4. **Note any external libraries required** (e.g., BOSL2) at the top.
5. **For 3D-printing advice**, mention orientation and supports if relevant.
6. **Offer variants or next steps** — e.g., "To add a lid, see the pattern in references/patterns.md."

---

## Extended Patterns

For complex patterns (gears, threads, enclosures, living hinges, etc.),
read `references/patterns.md` for ready-to-adapt code templates.

For BOSL2 library usage (threads, rounded shapes, attachments),
read `references/bosl2.md`.

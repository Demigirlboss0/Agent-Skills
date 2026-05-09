# BOSL2 Library Reference

BOSL2 (Better OpenSCAD Library 2) is the most popular OpenSCAD utility library.
Install: https://github.com/BelfrySCAD/BOSL2

Include at top of file:
```scad
include <BOSL2/std.scad>
```

---

## Rounded Shapes (cuboid, cylinders)

```scad
include <BOSL2/std.scad>

// Rounded cuboid
cuboid([80, 60, 20], rounding=3);

// Chamfered cuboid
cuboid([80, 60, 20], chamfer=2);

// Rounded cylinder
cyl(h=20, d=30, rounding=2);
cyl(h=20, d1=30, d2=20, chamfer=1);
```

---

## Attachment System

BOSL2's attachment system lets you place children relative to parent faces/edges/corners:

```scad
include <BOSL2/std.scad>

// Place a smaller box on top of a larger one
cuboid([60, 40, 20])
    attach(TOP, BOTTOM)
        cuboid([30, 20, 10]);

// Place at an edge
cuboid([60, 40, 20])
    attach(TOP+RIGHT, BOTTOM)
        cyl(h=10, d=8);
```

---

## Threading

```scad
include <BOSL2/std.scad>
include <BOSL2/threading.scad>

// Metric threaded rod
threaded_rod(d=10, l=30, pitch=1.5, $fn=32);

// Nut
threaded_nut(d=10, l=8, pitch=1.5, $fn=32);

// Inside a part (use inside difference())
module threaded_insert_hole(d=5, l=10) {
    threaded_rod(d=d, l=l+2, pitch=0.8, $fn=32, internal=true);
}
```

---

## Gears

```scad
include <BOSL2/std.scad>
include <BOSL2/gears.scad>

// Spur gear
spur_gear(mod=2, teeth=20, thickness=8, shaft_diam=5, $fn=64);

// Bevel gear pair
bevel_gear(mod=2, teeth=20, face_width=8, shaft_diam=5, $fn=64);

// Rack (linear gear)
rack(mod=2, teeth=15, thickness=8, height=5, $fn=64);
```

---

## Paths and Sweeps

```scad
include <BOSL2/std.scad>
include <BOSL2/paths.scad>

// Sweep a profile along a path
path = arc(n=32, r=30, angle=[0, 180]);  // semicircle path
shape = circle(d=8, $fn=16);
path_sweep(shape, path);

// Helix path (for springs, coils)
helix_path = helix(l=40, r=10, turns=4, $fn=64);
path_sweep(circle(d=2, $fn=16), helix_path);
```

---

## Masks and Fillets

```scad
include <BOSL2/std.scad>

// Fillet an inside corner
diff()
    cuboid([60, 60, 20], anchor=BOTTOM)
        tag("remove") inside_fillet(l=60, r=4, spin=0, orient=UP);

// Edge rounding with masks
diff()
    cuboid([60, 40, 20])
        tag("remove") edge_mask(TOP)
            rounding_edge_mask(l=40, r=3);
```

---

## Useful Utilities

```scad
include <BOSL2/std.scad>

// Linear array of copies
xcopies(spacing=15, n=5) sphere(d=8);
ycopies(spacing=15, n=4) cube([5,5,5]);
zcopies(spacing=10, n=3) cylinder(d=8, h=2);

// Grid array
grid_copies(spacing=[20, 20], size=[80, 60]) cylinder(d=5, h=10);

// Polar array
zrot_copies(n=6, r=30) sphere(d=8);

// Mirror + keep original
xflip_copy() cube([20, 10, 5]);

// Extrude with rounded ends
tube(h=20, od=30, id=20, $fn=32);

// Slot (rounded rectangle extrusion)
slot(h=5, l=30, d=10);
```

---

## Installation Note

BOSL2 is not bundled with OpenSCAD. To use it:

1. Download from https://github.com/BelfrySCAD/BOSL2/releases
2. Extract to your OpenSCAD libraries folder:
   - Linux: `~/.local/share/OpenSCAD/libraries/`
   - macOS: `~/Documents/OpenSCAD/libraries/`
   - Windows: `My Documents\OpenSCAD\libraries\`
3. Include with `include <BOSL2/std.scad>`

Always note required library at top of generated file:
```scad
// Requires BOSL2: https://github.com/BelfrySCAD/BOSL2
include <BOSL2/std.scad>
```

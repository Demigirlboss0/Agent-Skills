# OpenSCAD Extended Patterns

A collection of ready-to-adapt code templates for common mechanical and 3D-printing use cases.

---

## Box with Snap-Fit Lid

```scad
// ============================================================
// Parametric Box with Snap-Fit Lid
// ============================================================

inner_w    = 60;   // mm, interior width
inner_d    = 40;   // mm, interior depth
inner_h    = 25;   // mm, interior height
wall       = 2.0;  // mm, wall thickness
lip_h      = 3.0;  // mm, height of lid lip
snap_h     = 1.5;  // mm, snap barb height
tolerance  = 0.2;  // mm, lid-to-box fit clearance
$fn = 32;

SHOW_BOX = true;
SHOW_LID = true;

if (SHOW_BOX) box_body();
if (SHOW_LID) translate([0, inner_d + wall*2 + 5, 0]) lid();

module box_body() {
    difference() {
        // outer shell
        cube([inner_w + wall*2, inner_d + wall*2, inner_h + wall + lip_h]);
        // hollow interior
        translate([wall, wall, wall])
            cube([inner_w, inner_d, inner_h + lip_h + 1]);
        // lip cutout for lid
        translate([wall - tolerance, wall - tolerance, inner_h + wall])
            cube([inner_w + tolerance*2, inner_d + tolerance*2, lip_h + 1]);
    }
}

module lid() {
    union() {
        // flat top
        cube([inner_w + wall*2, inner_d + wall*2, wall]);
        // lip that inserts into box
        translate([wall - tolerance, wall - tolerance, -lip_h + wall])
            difference() {
                cube([inner_w + tolerance*2, inner_d + tolerance*2, lip_h]);
                // snap groove
                translate([-1, -1, lip_h - snap_h])
                    cube([inner_w + tolerance*2 + 2, inner_d + tolerance*2 + 2, snap_h + 1]);
            }
    }
}
```

---

## Enclosure with Screw Posts

```scad
// ============================================================
// Enclosure with M3 Screw Boss Corners
// ============================================================

inner_w  = 80;
inner_d  = 60;
inner_h  = 30;
wall     = 2.5;
boss_r   = 4;    // outer radius of screw post
screw_d  = 3.2;  // hole diameter (M3 clearance)
$fn = 32;

difference() {
    union() {
        // outer box
        cube([inner_w + wall*2, inner_d + wall*2, inner_h + wall]);
        // corner bosses inside
        for (x = [wall + boss_r, inner_w + wall - boss_r],
             y = [wall + boss_r, inner_d + wall - boss_r])
            translate([x, y, wall])
                cylinder(r=boss_r, h=inner_h - 2);
    }
    // hollow out
    translate([wall, wall, wall])
        cube([inner_w, inner_d, inner_h + 1]);
    // screw holes through bosses
    for (x = [wall + boss_r, inner_w + wall - boss_r],
         y = [wall + boss_r, inner_d + wall - boss_r])
        translate([x, y, -1])
            cylinder(d=screw_d, h=inner_h + wall + 2);
}
```

---

## Living Hinge Panel

```scad
// ============================================================
// Living Hinge (for flexible filament or thin PLA)
// ============================================================

panel_w     = 60;
panel_h     = 40;
hinge_w     = 10;   // width of hinge zone
thickness   = 0.8;  // mm, hinge web thickness (print thin!)
slit_w      = 0.8;  // mm, slit width
slit_gap    = 2.0;  // mm, gap between slits
num_slits   = floor(panel_h / (slit_w + slit_gap)) - 1;

// Left panel
cube([panel_w, panel_h, 3]);
// Hinge zone
translate([panel_w, 0, 0])
    difference() {
        cube([hinge_w, panel_h, 3]);
        // alternating slits
        for (i = [0 : 2 : num_slits - 1])
            translate([-1, i * (slit_w + slit_gap) + slit_gap, thickness])
                cube([hinge_w/2 + 1, slit_w, 10]);
        for (i = [1 : 2 : num_slits])
            translate([hinge_w/2, i * (slit_w + slit_gap) + slit_gap, thickness])
                cube([hinge_w/2 + 1, slit_w, 10]);
    }
// Right panel
translate([panel_w + hinge_w, 0, 0])
    cube([panel_w, panel_h, 3]);
```

---

## Gear (Involute) — requires BOSL2

```scad
include <BOSL2/std.scad>
include <BOSL2/gears.scad>

mod      = 2;    // module (tooth size)
teeth_a  = 20;
teeth_b  = 40;
thickness = 8;
$fn = 64;

// Driver gear
spur_gear(mod=mod, teeth=teeth_a, thickness=thickness, shaft_diam=5);

// Driven gear (offset so they mesh)
translate([mod*(teeth_a + teeth_b)/2, 0, 0])
    spur_gear(mod=mod, teeth=teeth_b, thickness=thickness, shaft_diam=8);
```

---

## Dovetail Joint

```scad
// ============================================================
// Dovetail Slide Joint
// ============================================================

dt_w     = 20;   // width of dovetail at widest
dt_narrow = 14;  // width at narrowest
dt_h     = 6;    // height of dovetail feature
dt_l     = 40;   // length of joint
tolerance = 0.2;
$fn = 1;         // no curves needed

module dovetail_male(l=dt_l) {
    hull() {
        translate([0, 0, 0])
            cube([dt_narrow, l, 0.01]);
        translate([(dt_w - dt_narrow)/2, 0, dt_h])
            cube([dt_narrow, l, 0.01]);  // wait, recalc:
    }
    // simpler version:
    linear_extrude(height=l, center=false)
        polygon([[0,0],[dt_w,0],
                 [dt_w - (dt_w-dt_narrow)/2, dt_h],
                 [(dt_w-dt_narrow)/2, dt_h]]);
}

module dovetail_female(l=dt_l) {
    // Use inside a difference():
    // add tolerance on all sides
    translate([-(dt_w-dt_narrow)/2 - tolerance, 0, -0.01])
        linear_extrude(height=l + 0.02, center=false)
            offset(delta=tolerance)
                polygon([[0,0],[dt_w,0],
                         [dt_w-(dt_w-dt_narrow)/2, dt_h],
                         [(dt_w-dt_narrow)/2, dt_h]]);
}
```

---

## Cable / Wire Clip

```scad
// ============================================================
// Parametric Cable Clip (prints flat, snaps over cable)
// ============================================================

cable_d    = 5;    // mm, cable outer diameter
clip_w     = 8;    // mm, width of clip
wall       = 1.6;  // mm, wall thickness
gap        = cable_d * 0.7;  // snap opening — slightly smaller than cable
$fn = 32;

difference() {
    // outer ring + base tab
    union() {
        cylinder(d = cable_d + wall*2, h = clip_w);
        translate([-(cable_d/2 + wall), 0, 0])
            cube([cable_d + wall*2, cable_d/2 + wall + 4, clip_w]);
    }
    // cable bore
    translate([0, 0, -1]) cylinder(d=cable_d, h=clip_w + 2);
    // snap gap
    translate([-gap/2, -(cable_d/2 + wall + 1), -1])
        cube([gap, cable_d/2 + wall + 2, clip_w + 2]);
    // mounting hole in tab
    translate([0, cable_d/2 + wall + 2, clip_w/2])
        rotate([90,0,0]) cylinder(d=3.2, h=10);
}
```

---

## Voronoi-Style Decorative Panel (approximate)

For true Voronoi, use an external tool or library. A simpler approach with
randomized holes that approximates the look:

```scad
// Random-hole lightweight panel
panel_w = 80;
panel_d = 60;
panel_h = 3;
hole_d  = 8;
rows    = 5;
cols    = 7;
seed    = 42;

difference() {
    cube([panel_w, panel_d, panel_h]);
    for (r=[0:rows-1], c=[0:cols-1]) {
        jx = rands(-3, 3, 1, seed + r*100 + c)[0];
        jy = rands(-3, 3, 1, seed + r*100 + c + 50)[0];
        translate([panel_w/(cols+1)*(c+1) + jx,
                   panel_d/(rows+1)*(r+1) + jy,
                   -1])
            cylinder(d=hole_d, h=panel_h + 2, $fn=6);
    }
}
```

---

## Hex Grid Infill Panel

```scad
// Parametric hexagonal grid panel
panel_w = 100;
panel_d = 80;
panel_h = 3;
hex_r   = 6;    // hex circumradius
gap     = 1.5;  // material between hexes
$fn = 6;

hex_pitch_x = (hex_r + gap) * 2;
hex_pitch_y = (hex_r + gap) * sqrt(3);
cols = ceil(panel_w / hex_pitch_x) + 2;
rows = ceil(panel_d / hex_pitch_y) + 2;

difference() {
    cube([panel_w, panel_d, panel_h]);
    for (r = [0:rows], c = [0:cols]) {
        x = c * hex_pitch_x + (r % 2) * hex_pitch_x / 2 - hex_pitch_x;
        y = r * hex_pitch_y - hex_pitch_y;
        translate([x, y, -1])
            cylinder(r=hex_r, h=panel_h + 2, $fn=6);
    }
}
```

---

## Parametric Phone Stand

```scad
// ============================================================
// Adjustable Phone Stand
// ============================================================

phone_w     = 78;   // mm, phone width
phone_t     = 10;   // mm, phone thickness (with case)
tilt_angle  = 70;   // degrees, viewing angle from horizontal
base_d      = 60;   // mm, base front-to-back
base_h      = 5;    // mm, base thickness
slot_depth  = 18;   // mm, how deep phone sits in cradle
wall        = 3;    // mm

$fn = 32;

// Base
cube([phone_w + wall*2, base_d, base_h]);

// Angled back support
translate([0, base_d, base_h])
    rotate([90 - tilt_angle, 0, 0])
        cube([phone_w + wall*2, slot_depth + wall, wall]);

// Front lip
translate([0, 0, base_h])
    difference() {
        cube([phone_w + wall*2, wall, phone_t + wall*2]);
        translate([wall, -1, wall])
            cube([phone_w, wall + 2, phone_t]);
    }
```

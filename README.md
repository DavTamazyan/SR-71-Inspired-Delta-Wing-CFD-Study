# SR-71 Inspired Delta Wing — CFD Study

A self-directed aerospace engineering project: I designed a simplified SR-71-inspired delta wing in SolidWorks and ran an external aerodynamics simulation in ANSYS Fluent to study pressure distribution, velocity fields, lift, drag, and L/D ratio at different angles of attack.

Built independently to develop a complete CFD workflow from scratch — CAD modeling through post-processing and force extraction.

> Wing-only study inspired by the SR-71's planform geometry, not a full aircraft simulation.

---

<img width="1192" height="378" alt="image" src="https://github.com/user-attachments/assets/6a425212-0311-4f17-9ae6-39232226234b" />

*Static pressure contours at 10° AoA*

---

## What I built

A thin, swept delta wing based on a NACA 0008 airfoil profile (root chord ~1200 mm), modeled in SolidWorks and simulated in ANSYS Fluent at 50 m/s. The geometry was deliberately simplified - no fuselage, nacelles, or chines - to keep the scope manageable and the results interpretable.

---

## Results

| AoA | Drag (N) | Lift (N) | Cd | Cl | L/D |
|----:|--------:|---------:|---:|---:|----:|
| 0°  | 6.71 | 0.063 | 0.00324 | ~0* | 0.009 |
| 10° | 7.42 | 1.344 | 0.00359 | ~0.00065* | 0.181 |

*\*Cl and Cd are pending final reference area verification (planform area = 0.5 × root chord × span). Raw forces and L/D are valid regardless.*

---

## Workflow

```
SolidWorks CAD
  → NACA 0008 airfoil import + trailing edge closure
  → Loft between root and tip profiles → delta wing planform

ANSYS SpaceClaim
  → Fluid enclosure creation
  → Boolean subtraction (enclosure minus wing) → clean fluid volume

ANSYS Meshing
  → Tetrahedral mesh with surface refinement near wing

ANSYS Fluent
  → Steady pressure-based solver, k-omega SST
  → Velocity inlet at 50 m/s
  → AoA via inlet flow direction vector (no geometry rotation needed)
  → Lift and drag force extraction + coefficient calculation
```

---

## Solver setup

| Parameter | Value |
|-----------|-------|
| Solver | Steady, pressure-based |
| Fluid | Air, 1.225 kg/m³ |
| Inlet velocity | 50 m/s |
| Turbulence model | k-omega SST |
| AoA method | Inlet flow direction vector |
| Reference area | ~1.35 m² (delta wing planform) |

---

## Aerodynamic theory and governing concepts

### Why wings generate lift - and it is not equal transit time

A common misconception is the *equal transit time* (or "longer path") theory: air splits at the leading edge, the top-surface air must travel further and therefore moves faster to "meet up" with the bottom-surface air at the trailing edge. This is wrong. There is no physical reason for the two parcels to arrive simultaneously, and experiments confirm they do not. The upper-surface air actually arrives *earlier*.

Lift is better explained by two complementary frameworks:

**1. Pressure difference (Bernoulli's principle)**

For steady, incompressible, inviscid flow along a streamline:

```
P + ½ρv² + ρgh = constant
```

Where P is static pressure, ρ is fluid density, v is flow velocity, and h is height. In aerodynamics the gravitational term is negligible, so:

```
P + ½ρv² = P_total  (constant along a streamline)
```

The wing's curved upper surface accelerates flow - velocity increases, so static pressure drops. The lower surface sees higher pressure. That pressure difference integrated across the wing surface is lift. The pressure contours in this simulation show exactly this: a low-pressure region on the upper surface that grows as AoA increases from 0° to 10°.

**2. Newton's third law - momentum change**

The wing deflects the incoming airstream downward. By Newton's third law, the air exerts an equal and opposite force upward on the wing - that is lift. Both the Bernoulli and momentum perspectives are correct and consistent; they are different ways of describing the same physical process. Bernoulli describes local pressure at a point; Newton describes the net momentum exchange across the whole flow field.

At 0° AoA with a symmetric NACA 0008 profile, the flow is deflected equally above and below - net downwash is zero, lift is near zero. This matches the simulation result (Lift = 0.063 N at 0°). At 10° AoA the wing inclines into the flow, increasing downwash and the pressure differential, producing measurably higher lift (1.344 N).

---

### Key aerodynamic formulae used

**Dynamic pressure**

```
q = ½ρv²
  = ½ × 1.225 × 50²
  = 1531.25 Pa
```

This is the baseline for all coefficient calculations. At 50 m/s the Reynolds number for this wing is approximately Re ≈ 3.4 × 10⁶ (assuming 1 m reference length), placing the flow well into turbulent territory - which is why k-omega SST was used rather than a laminar solver.

**Lift and drag coefficients**

```
Cl = L / (q × A_ref)
Cd = D / (q × A_ref)
```

Where L and D are the lift and drag forces extracted from the wing surface, and A_ref is the wing planform area. These coefficients non-dimensionalise the forces so results can be compared across different scales and flow speeds.

**Lift-to-drag ratio**

```
L/D = Lift / Drag = Cl / Cd
```

L/D is independent of reference area - the area cancels. The L/D value remained low compared with what would be expected from an idealized, well-resolved wing simulation. This likely reflects the simplified geometry, low effective aspect ratio, absence of boundary-layer inflation, and remaining uncertainty in reference-area definition. This is also closely related to Reynold's number.
​

**Force projection at nonzero AoA**

When the inlet flow is angled, global coordinate forces (Fx, Fy) are not directly drag and lift. They must be projected onto axes aligned with the free-stream direction:

```
Drag = Fx·cos(α) + Fy·sin(α)
Lift = -Fx·sin(α) + Fy·cos(α)

Drag was computed using the inlet flow direction vector. Lift was computed using the perpendicular direction vector.

```

Where α is the angle of attack. At 10°: cos(10°) ≈ 0.985, sin(10°) ≈ 0.174.

**Delta wing planform reference area**

```
A_ref = 0.5 × root chord × span
```

For a triangular delta planform this is the standard definition, equivalent to the area of the triangle viewed from above.

---

### What the NACA 0008 profile contributes

NACA 0008 is a symmetric, 8% thickness-to-chord ratio aerofoil. Symmetric means zero camber - no built-in curvature to generate lift at zero incidence. This was the right choice for this study because:

- At 0° AoA, any lift generated comes purely from AoA, not camber - making it easier to isolate the effect of angle of attack
- The thin profile is appropriate for higher-speed inspired geometries (the SR-71's actual surfaces are extremely thin)
- Symmetric profiles follow thin aerofoil theory closely: Cl ≈ 2π sin(α), giving a theoretical Cl of ~1.09 at 10° for an infinite-span wing

The delta planform reduces this significantly due to induced drag and vortex lift effects, so a lower effective Cl is expected compared to a straight wing with the same profile.

---

## Troubleshooting

This section is kept in because the problems I hit are common CFD beginner pitfalls - and solving them was most of the learning.

**Airfoil geometry not closing in SolidWorks** - Imported NACA coordinate curves looked closed but weren't recognised as closed sketch profiles, which blocked lofting. Fixed by converting to sketch geometry and manually closing the trailing edge within the same sketch.

**Wrong fluid domain setup** - First attempt left the wing and enclosure as separate bodies, creating interface/contact regions that wrecked convergence. Fixed by subtracting the wing solid from the enclosure so Fluent sees one clean fluid volume with a wing-shaped wall boundary inside it.

**Lift and drag direction at nonzero AoA** - Global X/Y forces don't equal drag/lift once the flow is angled. Fixed by projecting forces parallel and perpendicular to the actual inlet flow direction vector.

**Reference area selection** - Early coefficient values were wrong because the reference area was set to the enclosure face rather than the wing planform. For a delta wing: Aref = 0.5 × root chord × span.

---

## Limitations

- Wing only - no fuselage, chines, nacelles, or control surfaces
- Subsonic steady-state (the real SR-71 cruises at Mach 3.2)
- No boundary layer inflation layers - near-wall resolution could be improved
- No mesh independence study run
- No experimental or wind tunnel validation

---

## Skills

`SolidWorks` `ANSYS Fluent` `ANSYS SpaceClaim` `CFD` `k-omega SST` `External aerodynamics` `Mesh generation` `Boundary conditions` `NACA airfoil` `Force extraction` `Lift & drag analysis` `Boolean geometry operations`

---

## Possible extensions

- Add a simplified fuselage and SR-71-style chines to study body lift and vortex formation
- Compare wing-only vs wing-body vs wing-body-chine configurations
- Add inflation layers for better near-wall accuracy
- Run a mesh independence study
- Test at higher AoA approaching stall
- Model compressible/transonic flow

# 3D Spinning Top on Rectangular Block — Euler Symmetric Top with Two-Phase Contact

<p align="center">
  <img src="https://img.shields.io/badge/FreeFEM++-Simulation-blue?style=for-the-badge&logo=gnu&logoColor=white"/>
  <img src="https://img.shields.io/badge/Steel-Precision%20Top-silver?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Euler-Symmetric%20Top-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Two--Phase-Contact%20Mechanics-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Gyroscopic-Precession-purple?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/ParaView-VTK%20Export-green?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/License-MIT-lightgrey?style=for-the-badge"/>
</p>

<p align="center">
  A rigid-body dynamics simulation of a <b>steel precision spinning top</b> on a rectangular block
  using FreeFEM++. The top starts at <b>200 rad/s (~1900 rpm)</b> with a 5 degree initial tilt
  and undergoes gyroscopic precession governed by the <b>Euler symmetric top equations</b>.
  Spin decays exponentially due to tip friction. A <b>two-phase contact model</b> correctly
  captures the transition from tip-only contact to rim contact at the geometrically derived
  critical tilt angle of 31 degrees. All fields are exported as <i>VTU/PVD files</i>
  for smooth ParaView animation across ~500 frames.
</p>

<img width="1008" height="772" alt="spin" src="https://github.com/user-attachments/assets/71155ea0-4dcc-47da-93d6-28d3200555a2" />


---

## Physics

The simulation models the top as a rigid axisymmetric body governed by the Euler symmetric
top equations derived from the Lagrangian. Key features:

- Semi-implicit Euler integration of the tilt ODE under gravity, gyroscopic, and damping torques
- Quasi-static precession rate slaved to spin and tilt via the gyroscopic equilibrium quadratic
- Exponential spin decay modelling tip friction dissipation
- Two-phase contact: tip-only phase and rim-contact phase with geometrically derived transition angle
- Centrifugal hoop stress field computed analytically via the Lamé rotating disk solution
- 3D top geometry reconstructed from axisymmetric profile revolved over 36 angular slices
- VTU/PVD export for ParaView with block geometry written alongside animated top mesh

---

## Geometry

```
        z (vertical axis)
        |
        ●  stem tip    (z = H_total = 50 mm)
        |
        |  cylindrical stem  r = R_stem = 3 mm
        |
        ●  ogive start (z = H_ogive = 35 mm)
        |
        |  ogive taper: r decreases linearly R_max → R_stem
        |
        ●  flange top  (z = H_flange = 14 mm)
        |
       ═══  flat flange  r = R_max = 20 mm  (maximum radius)
        |
        |  conical tip: r = R_max * z / H_cone
        |
        ●  tip  (z = 0)  ← always at origin, touching block surface
        |
       ═══════════════════════  block top surface  z = 0
       |||||||||||||||||||||||   rectangular block  200 × 200 × 20 mm
       ═══════════════════════  block bottom  z = −20 mm

  Spin axis a = (sin(ψ)cos(φ), sin(ψ)sin(φ), cos(ψ))
  ψ = 0: axis vertical, top upright
  ψ = 31°: flange rim touches block surface → rim contact phase begins
```

---

## Top Shape Profile

| Section | z range | Radius formula | Notes |
|---|---|---|---|
| Conical tip | 0 to 12 mm | r = R_max · z / H_cone | Linear increase |
| Flat flange | 12 to 14 mm | r = R_max = 20 mm | Maximum radius |
| Ogive taper | 14 to 35 mm | r = linear R_max → R_stem | Smooth narrowing |
| Cylindrical stem | 35 to 50 mm | r = R_stem = 3 mm | Constant |

---

## Material Properties

### Steel Spinning Top

| Property | Symbol | Value | Unit |
|---|---|---|---|
| Density | ρ | 7 800 | kg/m³ |
| Young's modulus | E | 200 | GPa |
| Poisson ratio | ν | 0.30 | — |
| Total height | H | 50 | mm |
| Maximum radius | R_max | 20 | mm |
| Stem radius | R_stem | 3 | mm |
| Mass (computed) | m | 142.6 | g |
| Centre of mass height | z_CM | 16.6 | mm above tip |

### Moments of Inertia (numerically integrated)

| Property | Symbol | Value | Unit |
|---|---|---|---|
| Axial (spin axis) | I₃ | 1.8326 × 10⁻⁵ | kg·m² |
| Transverse at CM | I₁ | 1.6959 × 10⁻⁵ | kg·m² |
| Ratio | I₃/I₁ | 1.081 | — |
| Gravity torque arm | mgL = mgz_CM | 23.2 | mN·m |

### Rectangular Block

| Property | Value | Unit |
|---|---|---|
| Dimensions | 200 × 200 × 20 | mm |
| Top surface | z = 0 | m |
| Material | rigid (no deformation) | — |

---

## Process Parameters

| Parameter | Symbol | Value | Description |
|---|---|---|---|
| Total simulated time | T_total | 12.0 | s |
| Number of time steps | N_steps | 240 000 | — |
| Time step | dt | 0.05 | ms |
| Output interval | N_save | 240 steps | every 12 ms → ~500 frames |
| Initial tilt | ψ₀ | 5 | deg from vertical |
| Initial spin | Ω₀ | 200 | rad/s (~1 909 rpm) |
| Spin decay rate | κ_spin | 0.12 | 1/s |
| Critical spin | Ω_crit | 68.5 | rad/s (654 rpm) |
| Rim contact angle | ψ_rim | 31.0 | deg |

---

## Two-Phase Contact Model

### Phase 1 — Tip Contact (`ψ < 31°`)

The tip is fixed at the origin `(0, 0, 0)` on the block surface. The top precesses
freely under gyroscopic equilibrium. The tip contact imposes no rolling constraint —
the top spins in place.

```
  Tip position: (0, 0, 0) always — by geometric construction.
  Any point at height s along axis:  P = s·â + r(s)·ê_perp
  At s = 0: P = 0 regardless of ψ or φ. CORRECT.
```

### Phase 2 — Rim Contact (`ψ ≥ 31°`)

When the flange rim reaches the block surface, both the tip and the rim touch simultaneously.
The critical angle is derived geometrically:

```
  z of flange rim = H_cone·cos(ψ) − R_max·sin(ψ) = 0
  → ψ_rim = atan(H_cone / R_max) = atan(12/20) = 31.0 deg
```

This is not a tuned parameter — it follows exactly from the top's shape. After rim contact,
`ψ` is clamped at `ψ_rim`. Precession rate `phiD` decays exponentially under rim friction
until the top comes fully to rest.

---

## Equations of Motion

### Tilt ODE (Euler symmetric top, Lagrangian derivation)

```
  I₁·ψ̈ = I₁·φ̇²·sin(ψ)·cos(ψ)    ← gyroscopic centrifugal term
         − I₃·Ω·φ̇·sin(ψ)          ← gyroscopic restoring torque
         + mgL·sin(ψ)               ← gravity drives tilt away from vertical
         − κ_damp·ψ̇                ← tip friction damps nutation
```

Sign convention: `ψ` increases as top tilts away from vertical (falls).
Gravity term is positive — unlike the Euler disk where gravity drove the coin toward flat.

### Precession Rate (quasi-static slaving)

At gyroscopic equilibrium the bracket equals zero, giving the quadratic:

```
  I₁·φ̇²·cos(ψ) − I₃·Ω·φ̇ + mgL = 0
  φ̇ = (I₃·Ω − √(I₃²·Ω² − 4·I₁·mgL·cos(ψ))) / (2·I₁·cos(ψ))
```

This is the **slow precession root**. The fast root is unphysical for this initial condition.
Slaving `φ̇` to `Ω` and `ψ` eliminates the `φ̈` ODE entirely (reduced order modeling).

### Spin Decay

```
  Ω(t) = Ω₀ · exp(−κ_spin · t)
```

The top falls when `Ω` drops below the **critical spin**:

```
  Ω_crit = 2·√(I₁·mgL) / I₃ = 68.5 rad/s
```

Below this threshold the quadratic discriminant goes negative — no real precession rate
exists, meaning the gyroscopic support fails and the top falls.

### Centrifugal Hoop Stress (Lamé rotating disk)

At each frame, the exact analytical solution for a spinning disk is computed per vertex:

```
  σ_hoop(r) = ρ·Ω²·(3+ν)·(R_max² − r²) / 8
```

Maximum at the axis (`r = 0`), zero at the rim (`r = R_max`). Scales as `Ω²` —
visibly fades as the top slows down toward rest.

---

## Simulation Phases

```
Phase         Time range    ψ range       Ω range          Description
─────────────────────────────────────────────────────────────────────────
SLEEPING      0 – 6 s       5° – 10°      200 – 97 rad/s   Stable precession
NEAR_CRITICAL 6 – 8 s       10° – 20°     97 – 68 rad/s    Ω approaching Ω_crit
FALLING       8 – 9 s       20° – 31°     68 – 55 rad/s    Rapid tilt increase
RIM_CONTACT   9 – ~10 s     31° (clamped) 55 – 0            Rim on block, phiD decays
REST          ~10 s+        31° (static)  0                 Fully at rest
```

---

## Output Fields (ParaView)

### Top Geometry (`top_NNNN.vtu`)

| Field | Type | Description | Units |
|---|---|---|---|
| HoopStress_Pa | P0 nodal | Centrifugal hoop stress via Lamé solution | Pa |
| AxisHeight_m | P0 nodal | Height along symmetry axis (profile colour) | m |
| psiDeg | uniform | Tilt angle from vertical | deg |
| OmegaRads | uniform | Spin rate | rad/s |

Geometry: 60 profile rings × 36 circumferential points + tip + stem vertex = 2162 vertices.
Revolved from the axisymmetric profile. Tip vertex always placed at `(0, 0, 0)`.

### Block (`block.vtu`)

Single VTK hexahedron (type 12) with 8 corner vertices. Static — written once,
referenced every frame in the PVD. Top surface at `z = 0`.

### Trajectory CSV (`top_data.csv`)

| Column | Description | Unit |
|---|---|---|
| time_s | Physical time | s |
| psiDeg | Tilt angle from vertical | deg |
| phiDeg | Precession angle | deg |
| OmegaRads | Spin rate | rad/s |
| phiDrads | Precession rate | rad/s |
| EkinJ | Total kinetic energy | J |
| phase | SLEEPING / NEAR_CRITICAL / FALLING / RIM_CONTACT / REST | — |

---

## Repository Structure

```
spinning_top_v2.edp                    # Main FreeFEM++ simulation script
README.md                              # This file

D:\freefem++\spinning_top_v2\
├── scene.pvd                          # Master animation — block + top (open first)
├── top_data.csv                       # Time-series state data
├── block.vtu                          # Static rectangular block (written once)
├── top_0000.vtu                       # t = 0.0 s   (initial state, upright)
├── top_0001.vtu                       # t = 0.012 s
├── ...
└── top_NNNN.vtu                       # Final frame (top at rest on rim)
```

---

## How to Run

### Requirements

- FreeFEM++ v4.10 or later: https://freefem.org
- ParaView v5.x or later: https://www.paraview.org

### Step 1 — Run the simulation

```bash
FreeFem++ spinning_top_v2.edp
```

The script will:
1. Compute block VTU (static hexahedron)
2. Initialise top at ψ₀ = 5° with Ω₀ = 200 rad/s
3. For each of 240 000 time steps: advance tilt ODE, check rim contact, write output every 240 steps
4. Save ~500 VTU frames and complete PVD + CSV

Console output example:
```
 OmCrit=68.5rad/s
 psiRim=31.0deg (flange hits block)
 psi0=5deg  Omega0=200rad/s
 phiD0=6.53rad/s
t=0s   psi=5.00deg  Om=200.0  phiD=6.53   [SLEEPING]
t=1s   psi=5.00deg  Om=177.4  phiD=7.43   [SLEEPING]
...
t=8s   psi=15.2deg  Om=76.6   phiD=22.7   [NEAR_CRITICAL]
  ** RIM CONTACT at t=9.12s  psi=31.0deg  Omega=56.3rad/s
t=9s   psi=31.0deg  Om=55.2   phiD=18.4   [RIM_CONTACT]
t=10s  psi=31.0deg  Om=0.0    phiD=0.00   [REST]
```

### Step 2 — Open in ParaView

1. `File > Open` → navigate to `D:\freefem++\spinning_top_v2\`
2. Select `scene.pvd` → OK
3. Choose PVD Reader when prompted → OK
4. Click `Apply`
5. Change `vtkBlockColors` dropdown to `Solid Color` for the block (light gray)
6. Select the top dataset and set colour field to `HoopStress_Pa`

### Step 3 — Visualise

**Option A — Hoop stress decay (recommended first view)**
```
Color by HoopStress_Pa
Colormap: Plasma or Rainbow, range 0 to 51 000 Pa
Press Play
→ Bright stress at flange (r = R_max = 20 mm) at t = 0
→ Stress fades as Ω decays: scales as Ω²
→ Completely dark (zero stress) when top reaches rest
→ Centre always brighter than rim (Lamé profile)
```

**Option B — Profile shape visualisation**
```
Color by AxisHeight_m
Colormap: Cool-Warm
→ Shows the physical profile: tip (blue) → flange → stem (red)
→ Shape is constant; only orientation changes over time
→ Use Surface With Edges to see the 36-segment mesh
```

**Option C — Tilt angle evolution**
```
Color by psiDeg
→ Uniform colour changes from 5° at start to 31° at rim contact
→ Stays constant at 31° once rim touches block
→ Directly shows the Euler top falling dynamics
```

**Option D — Trajectory post-processing**
```
Import top_data.csv into Python / Excel
Plot psiDeg vs time_s   → tilt grows slowly then plateaus at 31 deg
Plot OmegaRads vs time_s → exponential decay with kappa = 0.12
Plot phiDrads vs time_s  → precession diverges as Omega → OmCrit
Plot EkinJ vs time_s     → monotonic decay to zero
```

Press `Play` and set animation speed to `Slowest` for best visualisation.

---

## What to Look for in Results

### Gyroscopic Precession
During the SLEEPING phase the axis sweeps slowly around the vertical. The precession
rate `φ̇ ≈ mgL / (I₃·Ω)` increases visibly as spin decays — the top precesses
faster as it slows down. This is the classic gyroscopic behaviour.

### Moffatt-Style Singularity
As `Ω → Ω_crit`, the quadratic discriminant approaches zero and `φ̇` diverges.
The precession rate grows rapidly in the final seconds before rim contact,
mirroring the Moffatt finite-time singularity seen in the Euler disk.

### Hoop Stress Fading
The centrifugal hoop stress scales as `Ω²`. Starting at 51 kPa at the flange,
it drops to roughly 19 kPa at 6 seconds (when `Ω ≈ 121 rad/s`) and reaches
zero at rest. The Lamé profile (bright centre, dark rim) is clearly visible.

### Rim Contact Geometry
At `ψ = 31°` the flange rim touches the block exactly simultaneously with the tip.
This is geometrically exact — derived from `atan(H_cone / R_max)`, not a parameter.
After this, the top leans at a fixed angle with both contact points on the block surface.

---

## Common Errors and Fixes

| Error | Cause | Fix |
|---|---|---|
| Top penetrates block | Old code used zLiftTop offset breaking tip contact | Tip always at origin by construction: `P(s=0) = 0` always |
| Top floating above block | zLiftTop incorrectly applied | Removed entirely in v2 |
| psi stuck at initial value | Wrong sign in EOM bracket | Gravity term must be `+mgL·sin(ψ)` (positive = tilt away from vertical) |
| phiD underflow to 4e-323 | Circular dependency in quasi-static phiD formula | Use quadratic formula directly; prescribe Omega independently |
| No precession visible | phiD too small at psi=85° | Start at psi=5° where phiD=6.5 rad/s is clearly visible |
| Block red in ParaView | vtkBlockColors auto-coloring | Change coloring dropdown from `vtkBlockColors` to `Solid Color` |

---

## Reduced Order Modeling Notes

This simulation is a **Level 1 reduced order model** of the Euler symmetric top:

| Full system (4 ODEs) | This ROM (1 ODE + algebra) | Justification |
|---|---|---|
| ψ̈ = f(ψ, φ̇, Ω) | ψ̈ = EOM with slaved φ̇ | Retained — the quantity of interest |
| φ̈ = g(ψ, ψ̇, φ̇, Ω) | φ̇ = quadratic formula | **Slaved**: T_nutation/T_precession ≈ 0.001 |
| Ω̇ = −τ_friction/I₃ | Ω = Ω₀·exp(−κt) | **Prescribed**: back-reaction O(h/R)² ≈ 2% |
| contact point wander | contact fixed at origin | **Discarded**: top spins in place |

The ROM correctly captures: gyroscopic precession, spin decay, critical spin, rim contact geometry,
and the Moffatt-style singularity at `Ω → Ω_crit`. It does not capture: full nutation dynamics,
contact point migration, or exact friction-derived spin decay.

---

## Extending the Model

| Extension | What to change |
|---|---|
| Full nutation dynamics | Integrate φ̈ ODE instead of slaving; initialise with nutation amplitude |
| Elastic block deformation | Couple tip contact force to FEM substrate solve (as in ball bounce code) |
| Hertz tip contact | Add Hertz contact force F = K_tip · δ^(3/2) at tip; solve for tip indentation |
| Different top material | Update ρ, recompute mass/MoI integrals numerically |
| Multiple tops | Duplicate state variables; add collision detection between tips |
| Tippe top inversion | Modify profile to asymmetric (off-centre CM); add full 3D Euler equations |
| Wobble / nutation | Add `psiWobble = A·sin(ω_nut·t)·exp(−damp·t)` to output geometry |
| Precessing contact circle | Track `x_c(t) = R·cos(ψ)·cos(φ)`, `y_c(t) = R·cos(ψ)·sin(φ)` |

---

## Citation

If you use this simulation, please cite:

```bibtex
@software{mishra_2026_spinning_top,
  author    = {Mishra, A.},
  title     = {3D Spinning Top on Rectangular Block — Euler Symmetric Top with Two-Phase Contact},
  year      = {2026},
  publisher = {Zenodo},
  doi       = {10.5281/zenodo.20579442},
  url       = {https://doi.org/10.5281/zenodo.20579442}
}
```

Plain text citation:

> Mishra, A. (2026). *3D Spinning Top on Rectangular Block — Euler Symmetric Top with Two-Phase Contact*. Zenodo. https://doi.org/10.5281/zenodo.20579442

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.20579442.svg)](https://doi.org/10.5281/zenodo.20579442)

---

## Author

**akshansh11**
GitHub: https://github.com/akshansh11

---

## License

<p>
<a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/">
<img alt="Creative Commons Licence" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/88x31.png"/>
</a>
<br/>
This work is licensed under a
<a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/">Creative Commons Attribution-NonCommercial 4.0 International License</a>.
</p>

You are free to:

- **Share** — copy and redistribute the material in any medium or format
- **Adapt** — remix, transform, and build upon the material

Under the following terms:

- **Attribution** — You must give appropriate credit and provide a link to this repository
- **NonCommercial** — You may not use the material for commercial purposes

Copyright 2026 akshansh11. All rights reserved for commercial use.

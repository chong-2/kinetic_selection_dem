# kinetic_selection_dem

Compiled simulation software and accompanying data for the manuscript
**"Kinetic selection of magnetically directed particle monolayers in thickening coatings."**

The provided binary `kinetic_dem` is an overdamped, many-particle discrete-element
solver for ferromagnetic microparticles in a drying polymer film under a film-normal
magnetic field. It combines magnetic dipole–dipole interactions, lateral immersion
capillary attraction, soft-sphere contact, and a time-dependent Stokes mobility set by
the evaporating, thickening medium. A complete description of the model, governing
equations, and numerical scheme is given in **Supplementary Sections 6–8** of the
manuscript.

This repository provides the **compiled binary and input/output data** needed to
reproduce the simulation results in **Figure 4** (many-particle design chart) and
**Supplementary Section S7.4** (DEM validation against the low-viscosity experiment).

---

## 1. System requirements

The binary is **dynamically linked** and was built and tested on:

| | |
|---|---|
| OS | Rocky Linux 9.4 (RHEL 9 compatible) |
| Architecture | x86_64 |
| C library | glibc 2.34 |
| Kernel | 5.14.0 |
| Compiler (build) | GCC 11.4.1 |
| Build system | CMake 3.26.5 |
| Build/test machine | Intel Xeon Gold 6146 @ 3.20 GHz, 24 cores, 250 GiB RAM |

> **Compatibility note.** The binary requires a **Linux x86_64** system with
> **glibc ≥ 2.34** (e.g. RHEL / Rocky / AlmaLinux 9, Ubuntu 22.04+). It will **not**
> run on older glibc systems or non-x86_64 architectures (e.g. Apple Silicon).
> No non-standard hardware is required.

---

## 2. Installation

No installation or compilation is needed — the binary is precompiled. Make it
executable if necessary:

```bash
chmod +x kinetic_dem
```

Typical setup time on a normal desktop: **under one minute** (download + chmod).

---

## 3. Usage

The solver takes three input files, in this order:

```bash
./kinetic_dem [parameter_file] [particle_file] [boundingbox_file]
```

### Input files

**Parameter file** — simulation settings (time stepping, fluid properties,
contact, magnetization, evaporation). Key fields include:

| Field | Meaning |
|---|---|
| `save_dir` | output directory for the VTK files |
| `time_end`, `time_step`, `time_save` | total time, integration step, save interval (s) |
| `init_polymer_vf`, `init_fluid_viscosity` | initial polymer volume fraction and viscosity |
| `fluid_height`, `fluid_density`, `fluid_surfacetension` | film thickness and fluid properties |
| `contact_*` | Hertzian soft-sphere contact parameters |
| `magnetization_applied`, `magnetization_Ms`, `magnetization_chi` | applied field (T), saturation magnetization, susceptibility |
| `evaporation_rate` | linear film-thinning rate (m/s) |

**Particle file** — one particle per line, SI units (metres):

```
id    x            y            z            radius       density(kg/m^3)
000   3.631014e-04 2.046477e-04 1.750000e-06 1.750000e-06 3600
001   1.113342e-04 1.380442e-04 1.750000e-06 1.750000e-06 3600
...
```

**Bounding-box file** — periodic domain and spatial-grid settings:

```
domain_size = x_max, x_min, y_max, y_min, z_max, z_min   # metres
box_num     = nx, ny                                     # neighbour-search grid
```

### Output

VTK files are written to `save_dir`, one per saved step, containing each particle's
**id, position, and velocity**. They open directly in
[ParaView](https://www.paraview.org/). Saved-step spacing is set by `time_save`
(e.g. `0.1 s`). Filenames are time-stamped (`0sec.vtk`, `1sec.vtk`, … up to the
evaporation end time, which depends on the case).

> **Note.** `save_dir` in the provided parameter files is a relative path. Create the
> target directory (or adjust `save_dir`) before running, e.g. `mkdir -p results`.

---

## 4. Demo

Any single case under `fig4_data/` or `figS7_data/` serves as a self-contained demo.
For example:

```bash
cd fig4_data/Le_3.5
mkdir -p demo_out
# edit params/Case_9.txt so that save_dir = ./demo_out/   (or your path)
../../kinetic_dem params/Case_9.txt \
                  geometries/Case_9_particles.txt \
                  geometries/Case_9_boundarybox.txt
```

**Expected output:** a sequence of `*sec.vtk` files in `demo_out/`, showing the
particle field evolving from the initial configuration toward the arrested
microstructure as the film dries.

**Expected run time:** each case is a full production run, taking approximately
12–24 hours on the HPC node listed above (24-core Xeon). These are not
short desktop demos. Because the provided results/ folders already contain the
corresponding VTK output for every case, the expected output can be verified
without re-running — simply compare a freshly generated run against the
supplied VTK frames for the same parameter/geometry files.

---

## 5. Data and reproduction

### `fig4_data/` — Figure 4 (many-particle design chart)

Sweeps initial wet thickness `H_0` at three target lattice spacings, at fixed
`Bo_m = 0.226`, `Γ = 5.95`.

```
fig4_data/
├── Le_2.5/        # target spacing L_e = 2.5D
├── Le_3.5/        # target spacing L_e = 3.5D
└── Le_4.5/        # target spacing L_e = 4.5D
        ├── geometries/   Case_<k>_particles.txt, Case_<k>_boundarybox.txt
        ├── params/       Case_<k>.txt
        └── results/      Case_<k>/  -> VTK output
```

`Case_0 … Case_18` are the **19 initial wet thicknesses** `H_0/D ∈ [0.5, 2.0]`
(step `1/12`), with `Case_0 = 0.5D` and `Case_18 = 2.0D`.

### `figS7_data/` — Supplementary S7.4 (DEM validation)

Validation of the low-viscosity experiment, sweeping the thickening index `Γ`.

```
figS7_data/
├── geometries/   Case<r>_particles.txt, Case<r>_boundarybox.txt
├── params/       Gamma<g>_Case<r>.txt
└── results/
        ├── Gamma_2.5/   0 1 2 3 4   -> VTK output
        ├── Gamma_4.0/   0 1 2 3 4
        ├── Gamma_5.5/   0 1 2 3 4
        └── Gamma_7.0/   0 1 2 3 4
```

`Gamma1–Gamma4` in `params/` correspond to `Γ = 2.5, 4.0, 5.5, 7.0`; `Case0–Case4`
are five independent replicate coatings (results folders `0–4`).

### Reproducing a result

1. Pick a case and run the solver (Section 3) to generate VTK output.
2. The VTK frames provided in `results/` are time-stamped (`0sec.vtk`, `1sec.vtk`, …);
   the nearest-neighbour distance and bond-orientational order analyses in the
   manuscript are computed from these frames.

---

## 6. License

All materials in this repository (compiled software, demonstration examples, and data)
are proprietary. They are provided solely for academic evaluation and for reproducing
the results of the associated manuscript. Source code is withheld as it underlies
ongoing development for follow-up work. See [`LICENSE`](./LICENSE) for full terms.
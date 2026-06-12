# Beam_Envelope_Simulation

## Overview

This project is a beam envelope simulation of a real accelerator beamline at the **Mumbai University Accelerator Centre (MUAC)**. The simulation models charged particle transport through electrostatic and magnetic beamline elements using **first-order transfer matrix optics**.

The code tracks thousands of particles simultaneously and studies how the beam evolves through:

- Drift Spaces
- Einzel Lenses
- Electrostatic Sector Analyzer (ESA)
- Quadrupole Triplets
- Edge Focusing Elements
- Beam Apertures

The simulation provides information about:

- Beam Envelope Evolution
- Beam Transmission
- Beam Losses
- Beam Spot Diagrams
- Effect of Optical Elements on Beam Transport

---

# Physical Principle

Each particle is represented in phase space as:

### Horizontal Plane

$$
X=
\begin{bmatrix}
x\\
x'
\end{bmatrix}
$$

where:

- $x$ = horizontal position
- $x'$ = horizontal divergence angle

---

### Vertical Plane

$$
Y=
\begin{bmatrix}
y\\
y'
\end{bmatrix}
$$

where:

- $y$ = vertical position
- $y'$ = vertical divergence angle

The propagation through any optical element is given by:

$$
X_f = M_x X_i
$$

$$
Y_f = M_y Y_i
$$

where:

- $M_x$ = Horizontal Transfer Matrix
- $M_y$ = Vertical Transfer Matrix

---

# Particle Generation

The simulation begins by generating a Gaussian particle distribution.

```python
x  = np.random.normal(0, sigma_x, N)
xp = np.random.normal(0, sigma_xp, N)

y  = np.random.normal(0, sigma_y, N)
yp = np.random.normal(0, sigma_yp, N)
```

where:

- $N$ = Number of particles
- $\sigma_x$ = Beam Size
- $\sigma_{x'}$ = Beam Divergence

This creates a realistic beam distribution similar to experimental accelerator beams.

---

# Beam Representation

The horizontal beam is stored as:

```python
beam_x =
[
 [x1,x2,x3,...],
 [xp1,xp2,xp3,...]
]
```

The vertical beam is stored as:

```python
beam_y =
[
 [y1,y2,y3,...],
 [yp1,yp2,yp3,...]
]
```

Each column represents a single particle.

---

# Transfer Matrices

The simulation uses four primary transfer matrices.

---

## 1. Drift Matrix

A drift represents free propagation without any focusing.

$$
M_{drift}
=
\begin{bmatrix}
1 & L\\
0 & 1
\end{bmatrix}
$$

where:

- $L$ = Drift Length

The particle coordinates transform as:

$$
x_f = x_i + Lx'_i
$$

$$
x'_f = x'_i
$$

Effect:

- No focusing
- Beam expands due to divergence

---

## 2. Focusing Matrix

Used for:

- Einzel Lens
- Focusing Plane of Quadrupoles

$$
M_F
=
\begin{bmatrix}
\cos(\sqrt{k}L)
&
\dfrac{\sin(\sqrt{k}L)}{\sqrt{k}}
\\
-\sqrt{k}\sin(\sqrt{k}L)
&
\cos(\sqrt{k}L)
\end{bmatrix}
$$

where:

- $k$ = Focusing Strength

Effect:

- Converges particle trajectories
- Produces beam waist
- Reduces beam size

---

## 3. Defocusing Matrix

Used for:

- Defocusing Plane of Quadrupoles

$$
M_D
=
\begin{bmatrix}
\cosh(\sqrt{k}L)
&
\dfrac{\sinh(\sqrt{k}L)}{\sqrt{k}}
\\
\sqrt{k}\sinh(\sqrt{k}L)
&
\cosh(\sqrt{k}L)
\end{bmatrix}
$$

Effect:

- Diverges particle trajectories
- Increases beam size

---

## 4. Bending Matrix

Used to model the Electrostatic Sector Analyzer (ESA).

$$
M_{bend}
=
\begin{bmatrix}
\cos\left(\frac{L}{R}\right)
&
R\sin\left(\frac{L}{R}\right)
\\
-\frac{1}{R}\sin\left(\frac{L}{R}\right)
&
\cos\left(\frac{L}{R}\right)
\end{bmatrix}
$$

where:

- $R$ = Bend Radius
- $L$ = Arc Length

Effect:

- Changes beam direction
- Produces trajectory curvature
- Provides weak focusing in bend plane

---

# Beamline Components

## Drift

Implemented using:

$$
M_x = M_y = M_{drift}
$$

Code:

```python
M = drift(ds)
```

---

## Einzel Lens

Implemented using:

$$
M_x = M_y = M_F
$$

Code:

```python
M = focus(k, ds)
```

The Einzel Lens focuses equally in both planes.

---

## ESA Bend

Implemented using:

$$
M_x = M_{bend}
$$

$$
M_y = M_{drift}
$$

Code:

```python
Mx = bend(ds, R)
My = drift(ds)
```

The beam bends in the horizontal plane while drifting vertically.

---

## QF Quadrupole

Focusing Quadrupole.

$$
M_x = M_F
$$

$$
M_y = M_D
$$

Code:

```python
Mx = focus(k, ds)
My = defocus(k, ds)
```

Effect:

- Focuses x
- Defocuses y

---

## QD Quadrupole

Defocusing Quadrupole.

$$
M_x = M_D
$$

$$
M_y = M_F
$$

Code:

```python
Mx = defocus(k, ds)
My = focus(k, ds)
```

Effect:

- Defocuses x
- Focuses y

---

# Particle Tracking

Each beamline element is divided into small slices.

```python
nsteps = int(L/ds)
```

where:

- $L$ = Element Length
- $ds$ = Propagation Step Size

Typical:

```python
ds = 0.001
```

which corresponds to:

$$
ds = 1 \ mm
$$

For every step:

```python
beam_x = Mx @ beam_x
beam_y = My @ beam_y
```

All particles are propagated simultaneously using matrix multiplication.

---

# Aperture Scraping

The simulation includes physical apertures.

The radial distance of each particle from the beam axis is:

$$
r=
\sqrt{x^2+y^2}
$$

A particle survives if:

$$
r < r_{aperture}
$$

Code:

```python
mask = (r < aperture)

beam_x = beam_x[:,mask]
beam_y = beam_y[:,mask]
```

Particles outside the aperture are removed from the simulation.

---

# Beam Envelope Calculation

The beam envelope is calculated using the 95th percentile.

$$
x_{95}
=
P_{95}
\left(
|x|
\right)
$$

$$
y_{95}
=
P_{95}
\left(
|y|
\right)
$$

Code:

```python
np.percentile(
    np.abs(beam_x[0]),
    95
)
```

Meaning:

> 95% of all surviving particles lie inside the plotted envelope.

This removes sensitivity to a few extreme particles while preserving realistic beam behavior.

---

# Beam Transmission

Beam Transmission is defined as:

$$
T=
\frac{N_{surviving}}
     {N_{initial}}
$$

Code:

```python
transmission.append(
    beam_x.shape[1]/N
)
```

Examples:

| Transmission | Meaning |
|-------------|----------|
| 1.0 | 100% survive |
| 0.8 | 80% survive |
| 0.5 | 50% survive |
| 0.0 | Complete beam loss |

---

# Beam Spot Diagrams

The beam spot diagram is generated using:

```python
plt.plot(
    beam_x[0],
    beam_y[0],
    'r.',
    ms=0.3
)
```

These plots show the transverse beam profile:

$$
x \ vs \ y
$$

at selected locations in the beamline.

Applications:

- Beam Diagnostics
- Aperture Studies
- Beam Matching
- Beam Quality Analysis

---

# Simulation Outputs

The simulation generates:

## Beam Envelope

- Horizontal Envelope
- Vertical Envelope

as a function of beamline position.

---

## Beam Transmission

Plots:

$$
T(s)
$$

which indicates how many particles survive along the beamline.

---

## Beam Spot Diagrams

Particle distributions after:

- Drift Sections
- Einzel Lenses
- ESA
- QF Quadrupoles
- QD Quadrupoles
- Edge Focusing Elements

---

# Applications

This simulation can be used for:

- Accelerator Optics Design
- Beam Transport Studies
- Aperture Optimization
- Einzel Lens Optimization
- Quadrupole Tuning
- Transmission Studies
- Beam Matching
- ESA Beamline Analysis
- MUAC Beamline Development

---

# Author

Developed for:

**Mumbai University Accelerator Centre (MUAC)**

Implemented in **Python** using:

- Transfer Matrix Beam Dynamics
- Monte Carlo Particle Tracking
- NumPy
- Matplotlib

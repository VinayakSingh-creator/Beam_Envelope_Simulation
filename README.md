# Beam_Envelope_Simulation

## Overview

This project is a beam envelope simulation of a real accelerator beamline at the **Mumbai University Accelerator Centre (MUAC)**. The simulation models the transport of charged particle beams through electrostatic and magnetic beamline elements using first-order transfer matrix optics.

The code tracks thousands of particles simultaneously and studies how the beam evolves as it passes through:

- Drift spaces
- Einzel lenses
- Electrostatic bending sections
- Quadrupole triplets
- Edge focusing elements
- Beam apertures

The simulation provides a realistic representation of beam transport and transmission through the accelerator beamline.

---

# Objectives

The purpose of this simulation is to:

- Study beam transport through the accelerator.
- Understand beam focusing and defocusing.
- Investigate beam losses due to apertures.
- Optimize quadrupole strengths.
- Analyze beam transmission.
- Visualize beam profiles at different locations.
- Predict beam behavior before experimental implementation.

---

# Physical Principle

The simulation is based on the transfer matrix formalism of charged particle optics.

Each particle is represented by its transverse coordinates:

### Horizontal Plane

\[
X=
\begin{bmatrix}
x\\
x'
\end{bmatrix}
\]

where:

- \(x\) = horizontal displacement
- \(x'\) = horizontal divergence angle

### Vertical Plane

\[
Y=
\begin{bmatrix}
y\\
y'
\end{bmatrix}
\]

where:

- \(y\) = vertical displacement
- \(y'\) = vertical divergence angle

The propagation through every optical component is represented by:

\[
X_f=M_xX_i
\]

\[
Y_f=M_yY_i
\]

where:

- \(M_x\) = horizontal transfer matrix
- \(M_y\) = vertical transfer matrix

---

# Particle Generation

The simulation begins by generating a beam consisting of \(N\) particles.

Example:

```python
x  = np.random.normal(0, sigma_x, N)
xp = np.random.normal(0, sigma_xp, N)

y  = np.random.normal(0, sigma_y, N)
yp = np.random.normal(0, sigma_yp, N)
```

The particles follow Gaussian distributions similar to those observed experimentally.

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

Each column represents one particle.

For example:

```python
beam_x[:,0]
```

represents the first particle:

```python
[x1,
 xp1]
```

---

# Transfer Matrices Used

The simulation uses four primary transfer matrices.

---

## 1. Drift Matrix

A drift represents free propagation of particles.

\[
M_{drift}
=
\begin{bmatrix}
1 & L\\
0 & 1
\end{bmatrix}
\]

where:

- \(L\) = drift length

Effect:

\[
x_f=x_i+Lx'_i
\]

\[
x'_f=x'_i
\]

The particle continues in a straight line.

---

## 2. Focusing Matrix

Used for:

- Einzel lens
- Focusing quadrupole plane

\[
M_F
=
\begin{bmatrix}
\cos(\sqrt{k}L)
&
\frac{\sin(\sqrt{k}L)}{\sqrt{k}}
\\
-\sqrt{k}\sin(\sqrt{k}L)
&
\cos(\sqrt{k}L)
\end{bmatrix}
\]

where:

- \(k\) = focusing strength

Effect:

- Converges particle trajectories.
- Produces beam waists.
- Reduces beam size.

---

## 3. Defocusing Matrix

Used for:

- Defocusing quadrupole plane

\[
M_D
=
\begin{bmatrix}
\cosh(\sqrt{k}L)
&
\frac{\sinh(\sqrt{k}L)}{\sqrt{k}}
\\
\sqrt{k}\sinh(\sqrt{k}L)
&
\cosh(\sqrt{k}L)
\end{bmatrix}
\]

Effect:

- Diverges particle trajectories.
- Increases beam size.

---

## 4. Bending Matrix

Used for electrostatic bending sections.

\[
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
\]

where:

- \(R\) = bending radius

Effect:

- Changes beam trajectory.
- Produces weak focusing.
- Bends the beam through the analyzer section.

---

# Beamline Components

## Drift

A region with no focusing fields.

```python
Mx = drift(ds)
My = drift(ds)
```

Both planes behave identically.

---

## Einzel Lens

Electrostatic focusing element.

```python
Mx = focus(k, ds)
My = focus(k, ds)
```

Focuses in both planes simultaneously.

---

## ESA Bending Section

Electrostatic sector analyzer.

```python
Mx = bend(ds, R)
My = drift(ds)
```

The horizontal plane bends while the vertical plane drifts.

---

## QF Quadrupole

Focusing quadrupole.

```python
Mx = focus(k, ds)
My = defocus(k, ds)
```

Effect:

- Focuses x
- Defocuses y

---

## QD Quadrupole

Defocusing quadrupole.

```python
Mx = defocus(k, ds)
My = focus(k, ds)
```

Effect:

- Defocuses x
- Focuses y

---

## Edge Focusing

Thin lens approximation for bending magnet edge effects.

```python
Mx = thin_edge_x(f)
My = thin_edge_y(f)
```

Used to model fringe field focusing.

---

# Particle Tracking Algorithm

Each element is divided into small slices.

```python
nsteps = int(L/ds)
```

where:

- \(L\) = component length
- \(ds\) = propagation step size

Typical:

```python
ds = 0.001 m
```

For every slice:

```python
beam_x = Mx @ beam_x
beam_y = My @ beam_y
```

which propagates all particles simultaneously.

---

# Aperture Scraping

Real beamlines contain apertures and beam pipes.

To simulate losses:

\[
r=\sqrt{x^2+y^2}
\]

Particles survive only if:

\[
r<r_{aperture}
\]

Code:

```python
mask = (r < aperture)

beam_x = beam_x[:,mask]
beam_y = beam_y[:,mask]
```

Particles outside the aperture are removed from the simulation.

---

# Beam Envelope Calculation

The beam envelope is calculated using the 95th percentile:

```python
np.percentile(np.abs(beam_x[0]),95)
```

Meaning:

> 95% of all surviving particles lie inside the plotted envelope.

Advantages:

- Reduces sensitivity to outliers.
- Provides a realistic beam boundary.
- Produces smoother envelope curves.

The simulation stores:

```python
env
env_y
```

which represent:

- Horizontal envelope
- Vertical envelope

as functions of beamline position.

---

# Beam Transmission

Transmission is defined as:

\[
T=
\frac{N_{surviving}}
     {N_{initial}}
\]

Code:

```python
transmission.append(
    beam_x.shape[1]/N
)
```

Examples:

```text
1.0  → 100%
0.8  → 80%
0.5  → 50%
0.0  → 0%
```

This indicates how many particles survive through the beamline.

---

# Beam Spot Diagrams

Beam spot diagrams are generated using:

```python
plt.plot(
    beam_x[0],
    beam_y[0],
    'r.',
    ms=0.3
)
```

These plots show the transverse distribution of particles.

Applications:

- Beam diagnostics
- Aperture studies
- Beam quality analysis
- Matching studies

---

# Beam Profile Storage

The simulation can store beam profiles after every beamline component.

Example:

```python
beam_profiles.append(
(
    etype,
    beam_x[0].copy(),
    beam_y[0].copy()
)
)
```

This enables visualization of beam evolution after:

- Drift sections
- Einzel lenses
- Bending sections
- QF quadrupoles
- QD quadrupoles
- Edge lenses

---

# Simulation Outputs

The code generates:

## 1. Beam Envelope

Plots:

- Horizontal envelope
- Vertical envelope

vs beamline position.

---

## 2. Beam Transmission

Plots:

- Transmission
- Particle survival fraction

vs beamline position.

---

## 3. Beam Spot Diagrams

Plots:

- x-y particle distributions
- Beam profile after each component

---

# Applications

This simulation can be used for:

- Accelerator optics design
- Beam transport optimization
- Aperture optimization
- Quadrupole tuning
- Einzel lens optimization
- Transmission studies
- Beam matching studies
- ESA beam transport analysis
- MUAC beamline design and diagnostics

---

# Author

Developed for beam transport and optics studies at:

**Mumbai University Accelerator Centre (MUAC)**

Implemented in **Python** using **transfer matrix beam dynamics** and **Monte Carlo particle tracking** techniques.

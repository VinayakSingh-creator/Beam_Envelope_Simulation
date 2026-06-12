# Beam Envelope Simulation for an Electrostatic Accelerator

A Python-based beam dynamics simulation that models the transport of a carbon-ion beam through an electrostatic accelerator beamline using first-order transfer matrix optics.

---

## 📖 Project Overview

This project simulates the evolution of a charged particle beam through an accelerator lattice consisting of:

- Drift spaces
- Einzel lenses
- Electrostatic bending sections
- Quadrupole triplet
- Beam apertures
- Edge focusing elements

The simulation tracks thousands of particles and computes:

- Horizontal beam envelope
- Vertical beam envelope
- Beam transmission
- Beam divergence
- Beam loss
- Maximum surviving particle amplitudes

---

## ⚙️ Physical Parameters

| Parameter | Value |
|------------|---------|
| Particle Type | Carbon Ion |
| Beam Energy | 0.4 MeV |
| Source Aperture Radius | 1.5 mm |
| Number of Particles | 10,000 |
| Tracking Method | Matrix Optics |

---

## 🏗 Accelerator Components

The beamline contains:

1. Source Drift
2. Einzel Lens 1
3. ESA Section 1
4. Einzel Lens 2
5. First Bending Magnet
6. Q-Snout Lens
7. Quadrupole Triplet
8. Second Bending Magnet
9. ESA Section 2
10. Detector Region

Total Beamline Length ≈ 12 m

---

## 🧮 Physics Model

## Transfer Matrices

### Drift

```math
M=
\begin{bmatrix}
1 & L \\
0 & 1
\end{bmatrix}
```

### Focusing

```math
M=
\begin{bmatrix}
\cos(\sqrt{k}L) &
\dfrac{\sin(\sqrt{k}L)}{\sqrt{k}} \\
-\sqrt{k}\sin(\sqrt{k}L) &
\cos(\sqrt{k}L)
\end{bmatrix}
```

### Defocusing

```math
M=
\begin{bmatrix}
\cosh(\sqrt{k}L) &
\dfrac{\sinh(\sqrt{k}L)}{\sqrt{k}} \\
\sqrt{k}\sinh(\sqrt{k}L) &
\cosh(\sqrt{k}L)
\end{bmatrix}
```

### Electrostatic Bend

```math
M=
\begin{bmatrix}
\cos(\theta) &
R\sin(\theta) \\
-\dfrac{\sin(\theta)}{R} &
\cos(\theta)
\end{bmatrix}
```

where

```math
\theta=\frac{L}{R}
```


## 📊 Simulation Outputs

### Beam Envelope

Tracks the 95th percentile beam size in both transverse planes.

### Transmission

Calculates the percentage of particles surviving along the beamline.

### Beam Loss

Determines particle losses due to aperture restrictions.

### Divergence

Computes:

- Horizontal divergence
- Vertical divergence

### Maximum Surviving Particle

Tracks the largest particle amplitude that survives aperture scraping.

---

## 📈 Generated Plots

The code automatically generates:

- Horizontal & Vertical Beam Envelopes
- Beam Transmission
- Beam Loss Along Beamline
- Cumulative Beam Loss
- Beam Divergence
- Maximum Surviving Particle Position
- Final Beam Spot Distribution

---


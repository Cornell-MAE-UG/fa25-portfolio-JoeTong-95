---
layout: project
title: Torque wrench design project
description: Class project with Graphs
technologies: [Fusion360, python, Ansys FEM Simulation]
image: /assets/images/MAE3270(2).png
---
# Project Overview

As part of a class project for Mechanics of Materials, this project focuses on optimizing a torque wrench design. I aimed to maximize the strain gauge voltage output magnitude (achieved by maximizing the strain at the location of the gauges), while ensuring the structure survives the design safety factors for normal stress, fatigue, and fracture. 

I'll present the CAD model with dimensions, then talk through the material and geometry selection process,  simulation configuration and results, and append the code used to iterate the geometry and materials. 

---


To summarize the safety factors and performance of this design: 

Target for maximum normal stress is $X_0=4$, I achieved around $X_0 \approx 4.895$.

Target for fracture is $X_K=2$, I achieved around $X_K \approx 13$.

Target for fatigue is $X_S=1.5$ under fully reversed loading for $10^6$ cycles , I achieved around $X_S \approx 7$ for $10^7$ cycles.

Target for strain gauge sensitivity is $>1mV/V$, I achieved $1.352 mV/V$

With all results simulated for the target moment of $ 600in\cdot lb$ with an steel alloy (Maraging 300), which is within the design material domain. 

# CAD and Dimensions



<img src="{{ site.baseurl }}/assets/images/MAE3270(2).png" width="550">

<img src="{{ site.baseurl }}/assets/images/MAE3270(3).png" width="550">
























# Material Selection Process
We want the tool to be able to bend enough to ensure sufficient strain for improved sensitivity, and we don’t want it to yield. This means we can consider material selection by the index: 

$$
M = \frac{\sigma_u}{E}
$$

Where 
$M$ is the material index,
$\sigma_u$ is the yield stress,
$E$ is Young's modulus.  

<img src="{{ site.baseurl }}/assets/images/MAE3270(4).png" width="550">

This plot shows properties of Ti, Al, steel alloy via yield strength and Young’s module. Index line with slope of 1 shows the index $M$ for different materials. 

With the same material index, we see clearly that both Titanium alloy and steel alloy can be equally preferable. However, considering steel alloys typically clusters around higher Young’s modulus, giving higher stiffness while typically being lower cost, choosing from steel alloy makes sense. Aluminum alloy is not stiff enough and doesn’t have as high a module as we desire, so it’s not desirable. 
We select Maraging steel 300 and AISI 9225 as our primary targets for evaluation.  

<img src="{{ site.baseurl }}/assets/images/MAE3270(5).png" width="550">


Since we have a design point for fatigue, we would also prefer something that has higher fatigue strength. We clearly see that our selection of steel alloy is near the top of their family, confirming that the preliminary selection is likely valid. 


<img src="{{ site.baseurl }}/assets/images/MAE3270(6).png" width="550">


This plot shots candidate materials performance with base geometry using hand script appended below.

We see that using default geometry, aluminum has the best sensitivity, but is also quite weak, although it does pass the safety factor checks. Considering that geometry likely will be modified, this is a dangerous choice, and having overall big deflection is not a good idea if we can use local geometries to only design for larger strain at the gage location to result in a more rigid design in general. 
Titanium alloys are much more expensive and less strong in stress safety factor with comparable with steel. 

Steel alloys are more desirable, since they have moderate signals, and good strength for their lower prices. From the selections, it’s obvious that Maraging 300 has the best overall safety factor performance and the signal strength is on part with other steel alloys. 
Its detailed material properties are obtained from Granta. 

<img src="{{ site.baseurl }}/assets/images/MAE3270(7).png" width="550">




















# Geometry iteration Process
A python class that stores these geometries parameters was created and appended below. 
<img src="{{ site.baseurl }}/assets/images/MAE3270(25).png" width="550">

Using the class, I first changed how changing the second moment of area in the bending direction affects the design points. They are defined as: 

```python

# Geometry: M (Applied target moment), L, h, b, c
geom_baseline = Geometry(600.0, 16.0, 0.75, 0.5, 1.0)

# vary width second monent
geom_a1 = Geometry(600.0, 16.0, 0.5, 0.4, 1.0) 
geom_a2 = Geometry(600.0, 16.0, 0.7, 0.4, 1.0) 
geom_a3 = Geometry(600.0, 16.0, 0.9, 0.4, 1.0) 
geom_a4 = Geometry(600.0, 16.0, 1.1, 0.4, 1.0) 
geom_a5 = Geometry(600.0, 16.0, 1.3, 0.4, 1.0) 


```

<img src="{{ site.baseurl }}/assets/images/MAE3270(23).png" width="550">

<img src="{{ site.baseurl }}/assets/images/MAE3270(22).png" width="550">

Since the force load in this tool is mostly directional, I increase the dimension of h so the second moment of aera. I reduced the value of b and changed h until I see a cross-over for 1mv/V target sensitivity. 

Then, considering normal stress in FEA is known to be around 4 times higher than beam theory, and that target point safety factor is 4, so I need to look for around 16 safety factor for normal stress. Considering this with the sensitivity result, g2 looks like a good point for next round of evaluation. 


```python
# vary thickness b； Geometry: M, L, h, b, c
geom_b1 = Geometry(600.0, 16.0, 0.7, 0.1, 1.0) 
geom_b2 = Geometry(600.0, 16.0, 0.7, 0.2, 1.0) 
geom_b3 = Geometry(600.0, 16.0, 0.7, 0.3, 1.0) 
geom_b4 = Geometry(600.0, 16.0, 0.7, 0.4, 1.0) 
geom_b5 = Geometry(600.0, 16.0, 0.7, 0.5, 1.0) 
geom_b6 = Geometry(600.0, 16.0, 0.7, 0.6, 1.0) 
geom_b7 = Geometry(600.0, 16.0, 0.7, 0.7, 1.0) 

```


<img src="{{ site.baseurl }}/assets/images/MAE3270(1).png" width="550">

<img src="{{ site.baseurl }}/assets/images/MAE3270(24).png" width="550">

This is a much more linear relationship in safety factor. For an safety factor of ~16, we need to go with between b4-b5. This results in a sensitivity of around 0.6mv/v, and we need to optimize specific geometry to make it more than 1mV/V by changing the geometry at the gauge location. 
Following the result, the final CAD model uses 0.7'' for $h$ and 0.45'' for $b$.

This gives us fatigue safety factor of 7, and fracture safety factor of 13, which are much higher than the required 1.5 and 2, respectively. 







# Simulation Configurations 

<img src="{{ site.baseurl }}/assets/images/MAE3270(8).png" width="550">

A zero displacement boundary condition was placed on the 8 faces of the drive. This correctly simulates how the wrench is fixed when incerted in a screw cap. 


<img src="{{ site.baseurl }}/assets/images/MAE3270(9).png" width="550">
The force was placed at the very end of the wrench handle. This creates an overprediction of the responces, as a distributed force across a larger handle surface would result in less deformation. However, the moment it creates for the other end of the wrench is mostly the same. This choice is appropreate for an overpredicted, safe analysis. 




























# Analysis of Simulation Results

<img src="{{ site.baseurl }}/assets/images/MAE3270(13).png" width="550">
<img src="{{ site.baseurl }}/assets/images/MAE3270(14).png" width="550">

The indicated maximum is principle stress is likely an artifact due to how the modle was imported as two separate bodies. The actual maximum sress likely occures in the two indicated prob region, showing around 44 to 46 ksi.


<img src="{{ site.baseurl }}/assets/images/MAE3270(15).png" width="550">
<img src="{{ site.baseurl }}/assets/images/MAE3270(16).png" width="550">

The simulation result for maximum normal stress seemed reasonable in terms of the magnitude and the locations. Given the simulated maximum stresses (53525 psi), we find the safety factor is:

$$
X_0 = \frac{262000}{53525} \approx 4.895
$$



Which satisfies the requirement of $4.0$. Stresses everywhere else seems uniform and the magnitude are relatively low. 





<img src="{{ site.baseurl }}/assets/images/MAE3270(17).png" width="550">

We observe the maximum load point displacement is 0.21 inches, which is moderate consider we actively chose an material that has high yield stress to Young’s modulus index. 



<img src="{{ site.baseurl }}/assets/images/MAE3270(18).png" width="550">
<img src="{{ site.baseurl }}/assets/images/MAE3270(12).png" width="550">

This design results in an approximately 1352$\mu\epsilon$ deflection at the strain gauge location. 

The strain gauge of choice is an BF120-3AA with gauge factor $GF\approx 2$. We use 2 of these gauges, such that they are loaded in compression and tension to form a half bridge configuration when the wrench is used. This gives us an sensitivity of: 

$$
\frac{\mathrm{mV}}{\mathrm{V}}
= \frac{GF}{2} \left( \varepsilon_\mu \times 10^{-3} \right) = 1.352 \frac{mV}{V}
$$

Which passes the $1 \frac{mV}{V}$ design point. 



























# Strain Guage selection and installation check
I selected the modle BF120-3AA. its tehcnical documentation can be found here: <sup>[LINK](https://www.elecrow.com/download/Coding%20System%20of%20Strain%20Gauges-AGS-TECH%20Version.pdf?srsltid=AfmBOop0K1jrqhico4sunThEKNmjmEPvvwWc9BQia8rD8durZ6dHlYHB)</sup>

<img src="{{ site.baseurl }}/assets/images/MAE3270(21).png" width="550">

As shown above, there are plenty of space to mount these gauges in their correct orientation. 
























# Appendix: Material, Geometry selection and Safety Factor Verification

```python
"""
MAE 3270 Final Project
Torque Wrench Design and Analysis
"""

import numpy as np
from dataclasses import dataclass
import pandas as pd
from tabulate import tabulate 
import matplotlib.pyplot as plt

# =========================
# Data classes
# =========================


@dataclass
class Material: 
    "define material properties objects"
    name: str                   #Material Name
    E: float                    #Young's Modulus (psi)
    nu: float                   #Poisson's Ratio 
    su: float                   #Tensile Strength (psi)
    KIC: float                  #Fracture Toughness (psi sqrt(in))
    sfatigue: float             #Fatigue Strength at 10^6 cycles (psi)
    GF: float = 2.0             #Strain Gauge Factor (default 2.0)

@dataclass
class Geometry:
    "define geomatry properties object"
    M: float            #Max Torque (in-lbf) detaul 600 in-lbf
    L: float            #Length from drive to load point (inches)
    h: float            #Height (inches)
    b: float            #Width (inches)
    c: float            #Distance from center of drive to center of strain gauge (inches)



# =========================
# Module functions
# =========================

def compute_stress_strain_deflection(geom, mat, use_full_moment=False):
    """
    use full moment if ture to match manual numbers.
    """
    # section properties
    I = geom.b * geom.h**3 / 12.0       # in^4
    c_outer = geom.h / 2.0              # in

    # loads
    F = geom.M / geom.L                 # lbf
    if use_full_moment:
        M_g = geom.M                    # in-lbf
    else:
        M_g = F * (geom.L - geom.c)     # in-lbf

    # stress / strain / deflection
    sigma_max = M_g * c_outer / I              # psi
    eps_g = sigma_max / mat.E                 # strain
    eps_g_micro = eps_g * 1e6                 # microstrain
    delta = F * geom.L**3 / (3 * mat.E * I)   # in

    return sigma_max, eps_g, eps_g_micro, delta

def evaluate_design(geom, mat,
                    use_full_moment=False,
                    crack_depth=0.04, #inches as manual specifies
                    Y_K=1.12): #fractire geometry factor

    # beam stuff
    sigma_max, eps_g, eps_g_micro, delta = compute_stress_strain_deflection(
        geom, mat, use_full_moment=use_full_moment
    )

    # bridge output
    V_ratio = mat.GF * eps_g / 2.0
    mv_per_v = V_ratio * 1000.0

    # safety factors
    X0 = mat.su / sigma_max

    K = Y_K * sigma_max * np.sqrt(np.pi * crack_depth)
    XK = mat.KIC / K

    XS = mat.sfatigue / sigma_max

    # pass / fail check
    if (X0   < 4.0 or
        XK < 2.0 or
        XS  < 1.5):
        pass_fail = "SF-FAIL"
    else:
        pass_fail = "SF-PASS"

    return {
        "material": mat.name,
        "pass_fail": pass_fail,
        "sigma_max_ksi": sigma_max / 1e3,
        "epsilon_g_micro": eps_g_micro,
        "deflection_in": delta,
        "mv_per_v": mv_per_v,
        "X0_static": X0,
        "XK_fracture": XK,
        "XS_fatigue": XS,
    }

# visulaize results 

def plot_material_sweep(df):
    """
    Plot safety factors + mv_per_v for MATERIAL change with SAME geometry.
    """

    materials = df["material"]
    x = np.arange(len(materials))

    # Values
    X0 = df["X0_static"]
    XK = df["XK_fracture"]
    XS = df["XS_fatigue"]
    mv = df["mv_per_v"]

    # Scale mv_per_v for visual overlay
    mv_scaled = mv / mv.max() * max(X0.max(), XK.max(), XS.max())

    width = 0.2

    plt.figure(figsize=(12, 6))
    plt.bar(x - 1.5*width, X0, width=width, label="SF Static (X0)")
    plt.bar(x - 0.5*width, XK, width=width, label="SF Fracture (XK)")
    plt.bar(x + 0.5*width, XS, width=width, label="SF Fatigue (XS)")
    plt.bar(x + 1.5*width, mv_scaled, width=width, label="Output mV/V (scaled)")

    plt.xticks(x, materials, rotation=45, ha="right")
    plt.ylabel("Value (scaled)")
    plt.title("Material Comparison (Same Geometry)")
    plt.legend()
    plt.tight_layout()
    plt.show()

def plot_geometry_sweep_dual_axis(df):
    geoms = df["geometry"]

    # Data (no normalization)
    X0 = df["X0_static"]
    XK = df["XK_fracture"]
    XS = df["XS_fatigue"]
    mv = df["mv_per_v"]

    fig, ax1 = plt.subplots(figsize=(12, 6))

    # --- Left axis: safety factors ---
    ax1.plot(geoms, X0, marker="o", label="SF Static (X0)")
    ax1.plot(geoms, XK, marker="o", label="SF Fracture (XK)")
    ax1.plot(geoms, XS, marker="o", label="SF Fatigue (XS)")

    # SF design lines
    ax1.axhline(4.0, linestyle="--", linewidth=1, color="blue",  label="X0 = 4")
    ax1.axhline(2.0, linestyle="--", linewidth=1, color="orange", label="XK = 2")
    ax1.axhline(1.5, linestyle="--", linewidth=1, color="green", label="XS = 1.5")

    ax1.set_ylabel("Safety Factor")
    ax1.set_xlabel("Geometry")
    ax1.grid(True, alpha=0.3)

    # --- Right axis: mV/V ---
    ax2 = ax1.twinx()
    ax2.plot(geoms, mv, marker="s", linestyle="--", color="red", label="Output (mV/V)")
    ax2.set_ylabel("Output (mV/V)")

    # --- Combined legend ---
    lines1, labels1 = ax1.get_legend_handles_labels()
    lines2, labels2 = ax2.get_legend_handles_labels()
    ax1.legend(lines1 + lines2, labels1 + labels2, loc="upper left")

    plt.title("Geometry Sweep – Safety Factors and Output")
    fig.tight_layout()
    plt.show()



def print_results(results):

    print("\n")

    # ---- Pass/Fail Check ----
    if (results["X0_static"]   < 4.0 or
        results["XK_fracture"] < 2.0 or
        results["XS_fatigue"]  < 1.5):
        print("***** SF CHECK FAIL *****")
    else:
        print("***** SF CHECK PASS *****")

    # ---- Label (material or geometry) ----
    if "material" in results:
        print(f"Material = {results['material']}")
    if "geometry" in results:
        print(f"Geometry = {results['geometry']}")

    print("-" * 30)
    print("Stress and Deflection")
    print(f"deflection       = {results['deflection_in']:.3f} in")
    print(f"max stress       = {results['sigma_max_ksi']:.2f} ksi")

    print("Safety Factors")
    print(f"SF strength X0   = {results['X0_static']:.2f}")
    print(f"SF fracture XK   = {results['XK_fracture']:.2f}")
    print(f"SF fatigue XS    = {results['XS_fatigue']:.2f}")

    print("Strain Gauge Result")
    print(f"gauge strain     = {results['epsilon_g_micro']:.1f} microstrain")
    print(f"output           = {results['mv_per_v']:.2f} mV/V")

    print("-" * 30)
    print("\n")


# =========================
# Design Options
# =========================


# Material
                                                        #E        #nu      #su         #KIC      #sfatigue
mat_baseline = Material("BASELINE_M42 Steel", 32e6, 0.29, 370e3, 15e3, 115e3) #skip GF Def

#alloy steel                                                        
mat_9255 = Material("Steel - 9255"                  , 29.9e6    , 0.285 , 274e3     , 12.7e3    , 96.9e3)
mat_4140 = Material("Steel - 4140"                  , 30.22e6   , 0.285 , 86.3e3    , 69.2e3    , 43.1e3) #annealed
mat_300  = Material("Steel - Maraging 300"          , 26.5e6    , 0.31  , 271e3     , 68.3e3    , 118e3)

# Titanium
mat_6222 = Material("Titanium - 6Al-2Sn-4Zr-2Mo"    , 16e6      , 0.31  , 105e3     , 59.2e3    , 85.4e3)

# Aluminum
mat_7075 = Material("Aluminum - 7075-T77511"        , 9.72e6    , 0.323 , 82.2e3    , 24e3      , 45.2e3)


materials = [
    #mat_baseline,
    mat_9255,
    mat_4140,
    mat_300,
    mat_6222,
    mat_7075
]




# Geometry: M, L, h, b, c
geom_baseline = Geometry(600.0, 16.0, 0.75, 0.5, 1.0)

# vary width second monent
geom_a1 = Geometry(600.0, 16.0, 0.5, 0.4, 1.0) 
geom_a2 = Geometry(600.0, 16.0, 0.7, 0.4, 1.0) 
geom_a3 = Geometry(600.0, 16.0, 0.9, 0.4, 1.0) 
geom_a4 = Geometry(600.0, 16.0, 1.1, 0.4, 1.0) 
geom_a5 = Geometry(600.0, 16.0, 1.3, 0.4, 1.0) 

geom_a = [geom_baseline, geom_a1, geom_a2, geom_a3, geom_a4, geom_a5]


# vary thickness b； Geometry: M, L, h, b, c
geom_b1 = Geometry(600.0, 16.0, 0.7, 0.1, 1.0) 
geom_b2 = Geometry(600.0, 16.0, 0.7, 0.2, 1.0) 
geom_b3 = Geometry(600.0, 16.0, 0.7, 0.3, 1.0) 
geom_b4 = Geometry(600.0, 16.0, 0.7, 0.4, 1.0) 
geom_b5 = Geometry(600.0, 16.0, 0.7, 0.5, 1.0) 
geom_b6 = Geometry(600.0, 16.0, 0.7, 0.6, 1.0) 
geom_b7 = Geometry(600.0, 16.0, 0.7, 0.7, 1.0) 

geom_b = [geom_baseline, geom_b1, geom_b2, geom_b3, geom_b4, geom_b5, geom_b6, geom_b7]


# =========================
# Run Analysis
# =========================
# rows = []
# for m in materials:
#     result = evaluate_design(geom_baseline, m) #use baseline geometry, test material
#     rows.append(result)


rows = []
labels = ["baseline"] + [f"g{i}" for i in range(1, len(geom_b))]


for label, g in zip(labels, geom_b):
    res = evaluate_design(g, mat_300)
    # make 'geometry' the first key in the row
    row = {"geometry": label, **res}
    rows.append(row)

print(tabulate(rows, headers="keys", floatfmt=".3f", tablefmt="fancy_grid"))

df = pd.DataFrame(rows)
plot_geometry_sweep_dual_axis(df)

```

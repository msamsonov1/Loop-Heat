# LoopHeat: Two-Phase Spacecraft Thermal Architecture Auditor

[![NASA SBIR Phase I](https://img.shields.io/badge/NASA%20SBIR-Topic%20S1.02%20Phase%20I-0B3D91.svg)](https://sbir.nasa.gov/)
[![Contract Value](https://img.shields.io/badge/Contract%20Value-$225,000-10B981.svg)](#sbir-program-architecture)
[![TRL Maturation](https://img.shields.io/badge/TRL-3%20%E2%86%92%205-06B6D4.svg)](#technology-readiness-level-trl-pathway)
[![Live Simulation HUD](https://img.shields.io/badge/Live%20Simulation-GitHub%20Pages-8B5CF6.svg)](https://msamsonov1.github.io/Loop-Heat/)
[![License](https://img.shields.io/badge/License-Proprietary%20%2F%20NASA%20SBIR%20Data%20Rights-red.svg)](#ip--licensing)

> **Autonomous State-Space Inversion & Anti-Flooding Closed-Loop Controller for Deep-Space Loop Heat Pipes (LHPs) and Deployable Radiators.**

Developed by **RFWaveCode** for NASA SBIR 2026 Solicitation Topic **S1.02: Technologies for Deep Space Science & Cryogenic/Two-Phase Thermal Architecture**.

🔗 **Interactive Web Application & HUD:** [https://msamsonov1.github.io/Loop-Heat/](https://msamsonov1.github.io/Loop-Heat/)  
📬 **Primary Technical Point of Contact:** [support@rfwavecode.com](mailto:support@rfwavecode.com)

---

## Table of Contents
1. [Executive Summary](#executive-summary)
2. [Target Problem: The Capillary Dry-Out Cliff](#target-problem-the-capillary-dry-out-cliff)
3. [The Core Physical Insight](#the-core-physical-insight)
4. [Software Intervention Architecture](#software-intervention-architecture)
5. [Primary Diagnostic Axes: $P\text{-}E$ Phase-Space Diagram](#primary-diagnostic-axes-p-e-phase-space-diagram)
6. [Core Governing Mathematics](#core-governing-mathematics)
7. [System Architecture & HUD Overview](#system-architecture--hud-overview)
8. [NASA SBIR Phase I Allocation ($225,000)](#nasa-sbir-phase-i-allocation-225000)
9. [Civil & Commercial ARR Projections ($218.4M)](#civil--commercial-arr-projections-2184m)
10. [Local Quickstart & Deployment](#local-quickstart--deployment)
11. [NASA Core Flight System (cFS) Integration](#nasa-core-flight-system-cfs-integration)
12. [Contact & Support](#contact--support)

---

## Executive Summary

Deep-space exploration missions (e.g., outer planet orbiters to Uranus and Neptune, Europa surface landers, and Kilopower space nuclear fission systems) demand continuous multi-hundred-watt thermal transport across complex deployable radiator wings without mechanical pumps. 

**LoopHeat** is an embedded thermal architecture state observer and closed-loop auditor. It non-invasively tracks the thermodynamic operating point of two-phase Loop Heat Pipes (LHPs) on an active Pressure ($P$) vs. Heat/Energy ($E$) state diagram. By computing real-time meniscus stability derivatives ($\frac{dE}{dP}$), LoopHeat detects impending wick de-prime, capillary blowout, and compensation chamber vapor back-flooding **minutes before irreversible thermal runaway**, autonomously triggering downstream science throttling and secondary reservoir trim heaters.

---

## Target Problem: The Capillary Dry-Out Cliff

LHPs rely on sub-micron capillary forces inside a sintered metal primary wick (typically sintered titanium or nickel, $r_p \approx 0.8\text{--}1.2\,\mu\text{m}$) to drive anhydrous ammonia ($NH_3$) or propylene across tens of meters of tubing to radiation cold-plates. 

```
+-----------------------------------------------------------------------------------+
|                              CAPILLARY FAILURE MODES                              |
+-----------------------------------------------------------------------------------+
| 1. High Heat Flux Surge     -> Vapor bubble nucleation inside wick sub-layer      |
| 2. Radiator Eclipse Shock   -> Rapid subcooling loss at liquid return             |
| 3. NCG Accumulation         -> Non-condensable gas blocks condenser active area   |
|                                                                                   |
| RESULT: Loop Delta P exceeds Capillary Pumping Limit (Delta P_c,max)             |
|   --> Vapor punches through primary wick into compensation chamber                |
|   --> Irreversible Vapor Lock / Zero Mass Flow / Payload Burnout                  |
+-----------------------------------------------------------------------------------+
```

Incumbent architectures fly with conservative $300\%\text{--}400\%$ mass margins or rely solely on bulk temperature sensors. Because temperature rate-of-rise lags physical meniscus recession, legacy ground controllers receive alerts only after dry-out has already locked the loop.

---

## The Core Physical Insight

Healthy two-phase LHPs operate in a self-regulating hydrodynamic equilibrium where payload heat input ($E$) drives fluid vaporization, overcoming the loop friction and viscous pressure drops ($P$). 

* **Nominal Regime:** Heat transport scales smoothly with hydraulic pressure drop: $\frac{dE}{dP} > \kappa_{\text{nominal}}$.
* **The Dry-Out Bifurcation:** As the menisci approach the capillary limit of the pore throat radius:
  $$\Delta P_{c,\max} = \frac{2\sigma \cos\theta}{r_p}$$
  frictional losses spike non-linearly due to turbulent vapor return and micro-void drag. **The differential thermal lift collapses:**
  $$\lim_{P \to P_{\text{cap}}} \frac{dE}{dP} = 0$$
  Any marginal increase in heat flux produces an asymptotic pressure surge without extracting heat, representing the mathematical pre-cursor to wick dry-out.

---

## Software Intervention Architecture

LoopHeat continuously samples non-invasive external Platinum RTD arrays and sapphire piezoresistive pressure transducers at 25 Hz.

```
       +-----------------------------------------------------------------+
       |         Radiation-Tolerant Temperature & Pressure Sensors       |
       |             [T_evap, T_cc, T_cond, T_subcool, Delta P]          |
       +-----------------------------------------------------------------+
                                       |
                                       v
       +-----------------------------------------------------------------+
       |              LoopHeat Real-Time State Observer Inversion        |
       |           - Clausius-Clapeyron Saturation Curve Matching        |
       |           - Meniscus Capillary Margin Metric (% Safe)           |
       |           - Derivative Tracker: dE/dP Evaluation                |
       +-----------------------------------------------------------------+
                                       |
               +-----------------------+-----------------------+
               |                                               |
        Margin >= 35%                                   Margin < 20%
               |                                               |
               v                                               v
     [NOMINAL EQUILIBRIUM]                       [AUTONOMOUS INTERVENTION]
     - Passive state logging                     1. Throttle science payload
     - Telemetry packet streaming                   (15-40% cycle skip)
                                                 2. Fire secondary reservoir
                                                    heaters (18.5 W pulse)
                                                 3. Suppress vapor lock &
                                                    restore subcooling head
```

---

## Primary Diagnostic Axes: $P\text{-}E$ Phase-Space Diagram

The LoopHeat user interface and telemetry stream revolve around the **$P\text{-}E$ Phase-Space Auditor**:

| Diagnostic Axis | Parameter | Units | Description |
| :--- | :--- | :--- | :--- |
| **X-Axis** | **Hydraulic Pressure ($P$)** | $\text{kPa}$ | Total loop differential pressure across vapor line, condenser, wick, and liquid return ($\sum \Delta P_i$). |
| **Y-Axis** | **Heat / Energy ($E$)** | $\text{Watts}$ | Instantaneous thermal dissipation applied to the primary evaporator saddle from the science payload. |

### Visual Elements on the Phase Diagram
1. **Nominal Corridor:** The green/cyan operating envelope bounded by viscous laminar and turbulent limits.
2. **Capillary Limit Curve (Red Dashed Line):** Derived from maximum Laplace-Young meniscus retention. The area above and to the right represents the **Hazard Blow-Through Zone**.
3. **Live Operating Trajectory:** Trailing vector tracking the system state in real time. If the vector trends toward $\frac{dE}{dP} \to 0$, LoopHeat intercedes.

---

## Core Governing Mathematics

### 1. Laplace-Young Capillary Limit
$$\Delta P_{c,\max} = \frac{2\sigma \cos\theta}{r_p} \ge \Delta P_v + \Delta P_l + \Delta P_{\text{cond}} + \Delta P_w$$

### 2. Evaporator Two-Phase Energy Conservation
$$E = \dot{m} h_{fg} + Q_{\text{leak}} = \dot{m} h_{fg} + \frac{2\pi k_{\text{eff}} L_{\text{wick}}}{\ln(r_o/r_i)} (T_{\text{evap}} - T_{\text{cc}})$$

### 3. Non-Linear Pressure-Energy Inversion
$$\Delta P(E) = \frac{\mu_v E}{\rho_v h_{fg} K_v} + \frac{\beta_v E^2}{\rho_v h_{fg}^2} + \frac{\mu_l E}{\rho_l h_{fg} K_l}$$

### 4. Meniscus Depletion Index ($\Phi_{\text{margin}}$)
$$\Phi_{\text{margin}} = 1.0 - \frac{\sum_{i} \Delta P_i(E, T_{\text{sink}}, \chi_{\text{ncg}})}{\Delta P_{c,\max}(\sigma(T), r_p)}$$

---

## System Architecture & HUD Overview

The single-file web application (`index.html`) is structured into four primary interactive views:

```
+-----------------------------------------------------------------------------------+
|                        LOOPHEAT TELEMETRY DASHBOARD                               |
+-----------------------------------------------------------------------------------+
| [Live HUD]          [Physics & Math]     [$225k SBIR Plan]    [Civil ARR ($218M)] |
+-----------------------------------------------------------------------------------+
|  1. Live HUD (#viewSim)                                                           |
|     - Real-Time HTML5 Canvas: Evaporator, sintered wick, vapor line, & radiator.  |
|     - Scatter Chart.js: Dynamic P-E state-space with trailing trajectory.         |
|     - Telemetry Strip: Real-time sampling of Delta P, Margin, and Subcooling.     |
|     - Fault Injection: Science Surge (+250W), Sun Shift (+100K), Bubble Burp.    |
|                                                                                   |
|  2. Physics & Mathematical Model (#viewPhysics)                                   |
|     - MathJax v3 rendered cards detailing thermodynamic proofs & sensor schemas.  |
|                                                                                   |
|  3. $225,000 NASA SBIR Proposal (#viewProposal)                                   |
|     - S1.02 Solicitation mapping, DCAA-compliant budget, and 6-month milestones.  |
|     - Academic Subcontractor allocation: 30.0% ($67,500) for TVAC validation.     |
|                                                                                   |
|  4. Commercial & Civil ARR Breakdown (#viewARR)                                   |
|     - Market analysis for 5 civil sectors totaling $218.4M recurring ARR.         |
+-----------------------------------------------------------------------------------+
```

---

## NASA SBIR Phase I Allocation ($225,000)

| Cost Category | Scope & Justification | Amount | % Budget |
| :--- | :--- | :---: | :---: |
| **Direct Technical Labor** | Lead Spacecraft Thermal Architect & Embedded Flight Software Engineers (840 hours) | **$102,400** | 45.5% |
| **Indirect Overhead & Fringe** | DCAA-compliant provisional indirect cost pool (31.2% base) | **$31,950** | 14.2% |
| **University Subcontract (STTR)** | High-Vacuum Cryogenic Thermal Chamber (TVAC) testing & optical wick validation | **$67,500** | 30.0% |
| **Sensors & Prototyping** | Pt1000 DIN RTDs, sapphire differential transducers, sintered titanium test specimens | **$11,800** | 5.2% |
| **Negotiated Small Business Fee** | Standard FAR 15.404-4 commercialization fee (5.04%) | **$11,350** | 5.1% |
| **Total Firm-Fixed Price** | **NASA SBIR Phase I Solicitation Allocation** | **$225,000** | **100.0%** |

### 6-Month Phase I Milestone Roadmap
* **Month 1–2:** Nonlinear kinetic model formulation for $NH_3$ and propylene across 30 K to 220 K radiator sinks.
* **Month 3:** Embedded C/C++ state observer engine compiling indirect Clausius-Clapeyron inversion algorithms.
* **Month 4–5:** Subcontractor TVAC test campaigns in $10^{-6}\text{ Torr}$ vacuum chambers under 450W power steps.
* **Month 6:** Delivery of Phase II avionics transition package and NASA Core Flight System (cFS) application module.

---

## Civil & Commercial ARR Projections ($218.4M)

LoopHeat's passive two-phase monitoring engine scales into terrestrial and orbital commercial systems:

```
                                  COMMERCIAL ARR BREAKDOWN
+-----------------------------------+--------------------+------------------+---------------+
| Civil Target Sector               | Hardware Platform  | Pricing Model    | Annual ARR    |
+-----------------------------------+--------------------+------------------+---------------+
| 1. Hyperscale AI Data Centers     | 1.2 kW GPU Blades  | $180 / rack / mo | $90.7M        |
| 2. Commercial LEO Constellations  | Phased Array SAR   | $4,500 / sat / yr| $44.1M        |
| 3. Electric eVTOL & Aviation      | Inverter Coolers   | $3,200 / plane /yr| $36.8M       |
| 4. Grid Battery Storage (BESS)    | Megawatt Packs     | $1,800 / MWh / yr| $25.5M        |
| 5. Semiconductor EUV Lithography  | Cryo Collector     | $45,000 / tool/yr| $21.3M        |
+-----------------------------------+--------------------+------------------+---------------+
| TOTAL US CIVIL RECURRING ARR                           | 10x Multiple ARR | $218.4M       |
+-----------------------------------+--------------------+------------------+---------------+
```

---

## Local Quickstart & Deployment

The application is completely self-contained in a single, dependency-free file (`index.html`).

### 1. Clone the Repository
```bash
git clone https://github.com/msamsonov1/Loop-Heat.git
cd Loop-Heat
```

### 2. Run Locally
Because all libraries (Tailwind CSS, Chart.js, Lucide Icons, MathJax v3) are loaded via high-speed CDNs, no build step or `npm install` is required:

```bash
# Using Python 3 built-in HTTP server:
python -m http.server 8080

# Or open directly in your browser:
# On macOS:
open index.html

# On Linux:
xdg-open index.html
```

Visit `http://localhost:8080` in any modern web browser.

---

## NASA Core Flight System (cFS) Integration

The production version of the LoopHeat observer compiles into a native NASA cFS applet (`loopheat_app.c`):

```c
#include "cfe.h"
#include "loopheat_app.h"

/* Real-time 25Hz Cyclic Execution */
void LoopHeat_ProcessTelemetry(void)
{
    float P_measured = Read_Differential_Pressure();
    float E_heat     = Read_Evaporator_Thermal_Power();
    float P_cap_max  = Compute_Laplace_Young_Limit();

    /* Evaluate Meniscus Stability Gradient */
    float dE_dP = Calculate_Gradient(E_heat, P_measured);

    if (P_measured >= (0.80f * P_cap_max) || dE_dP < DEDP_CRITICAL_THRESHOLD)
    {
        /* Trigger Anti-Flooding Interlock */
        CFE_EVS_SendEvent(LOOPHEAT_WARN_EID, CFE_EVS_EventType_CRITICAL,
                          "Capillary limit imminent. Throttling payload to avert vapor lock.");
        
        Autonomous_Throttle_Payload(0.65f); // 35% thermal duty reduction
        Activate_CC_Trim_Heater(18.5f);     // 18.5W subcooling stabilization pulse
    }
}
```

---

## IP & Licensing

* **RFWaveCode:** Proprietary Software Architecture and Inversion Algorithms 2026.
* **NASA SBIR Data Rights:** Protected under FAR 52.227-20 for Phase I awardees. Commercial rights reserved for civil industrial deployments.

---

## Contact & Support

For licensing, technical documentation, or partnership inquiries regarding the **NASA SBIR Topic S1.02** flight transition:

* **Company:** RFWaveCode
* **Website:** [https://msamsonov1.github.io/Loop-Heat/](https://msamsonov1.github.io/Loop-Heat/)
* **Email:** [support@rfwavecode.com](mailto:support@rfwavecode.com)

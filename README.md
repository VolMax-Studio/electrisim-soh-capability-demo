# SOH-Aware Reactive Power Capability Demo
**Joint Collaboration: Electrisim (Adam Kierad) × VolMax Studio Lab (Ivan Nestorov)**

This repository demonstrates how battery degradation—specifically State of Health (SOH) and internal resistance ($R_{\text{int}}$) rise—physically constrains a BESS inverter's capability to deliver reactive power ($Q$) under grid-code compliance events (such as Q-V droop or voltage support).

---

## 1. Integration with Electrisim / pandapower

This prototype is designed to easily plug into the Electrisim/pandapower BESS model representation:
- The `reactive_capability` function computes the shrunk `q_avail_mvar` based on the battery's instantaneous state.
- In pandapower, this maps directly to the active limits of a static generator (`sgen`) or `storage` element:
  - `sgen.max_q_mvar = q_avail_mvar`
- You can run the existing Electrisim PQ module and grid-code compliance sweeps twice, swapping the SOH input to flip the compliance verdict at the Point of Connection (POC).

---

## 2. Verification Doctrine & Scope Boundaries

> [!IMPORTANT]
> **Representative Inverter Parameters**
> This repository is calibrated using representative specifications for a utility-scale BESS inverter class ($3.45\text{ MVA}$ apparent power rating, $550\text{ V}$ AC nominal line voltage). These parameters are illustrative and not tied to a specific verified manufacturer datasheet. Always replace the specifications in `soh_capability.py` with your specific project's inverter and aged battery datasheet values before drawing binding compliance conclusions.

> [!WARNING]
> **Out of Scope**
> This is a simplified, instantaneous single-stage electrical capability model. It is designed to capture the core modulation constraint ($R_{\text{int}} \to V_{\text{dc}} \to V_{\text{ac,max}} \to Q_{\text{avail}}$) on a seconds/minutes timescale. It does **not** model:
> - SOC energy depletion over time (multi-hour load flows).
> - Thermal derating of battery cells or inverter power electronics.
> - High-frequency inverter control-loop dynamics and transients.
> - Harmonic voltage/current limits at the POC.

---

## 3. How the Demo Flips

The compliance verdict flips under identical grid conditions depending solely on the integrity of the SOH input:

* At $P = 2.8\text{ MW}$ with a grid-code reactive requirement of $Q_{\text{req}} = 1.5\text{ Mvar}$ at the POC:
  - **Leakage-Inflated SOH** ($R_{\text{int}} = 0.025\,\Omega$, $95\%$ SOH): The inverter reports $1.61\text{ Mvar}$ is available. **Verdict: PASS (on paper).**
  - **Honest SOH (Aged Battery)** ($R_{\text{int}} = 0.045\,\Omega$, $80\%$ SOH): Due to the high $R_{\text{int}}$, the available DC voltage drops to $965.5\text{ V}$, limiting the AC modulation ceiling and shrinking the headroom to $1.15\text{ Mvar}$. **Verdict: FAIL (in the field).**

---

## 4. How to Run the Demo

Run the python capability sweep script:
```bash
python3 soh_capability.py
```
This will print the operating points sweep and output a detailed JSON breakdown of the operating point where the compliance verdict flips.

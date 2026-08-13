---
name: mat-qha-thermal-expansion
description: Calculate Quasi-Harmonic Approximation (QHA) thermal properties using MLIPs.
category: [materials]
---

# QHA Thermal Expansion Skill

This skill provides tools for calculating thermal expansion and temperature-dependent Gibbs energy using Machine Learning Interatomic Potentials (MLIPs).

## 1. Prerequisites

- The appropriate MLIP wrapper must be available (`MACEWrapper`, `MatGLWrapper`, or `FAIRCHEMWrapper`).
- `matcalc` must be installed in the relevant conda environment.

## 2. Choosing a Foundation Potential

QHA calculations require accurate lattice expansion and vibrational properties.

> [!IMPORTANT]
> - **Use OMAT or MatPES trained models**: These models (e.g., `MACE-OMAT-0-small`, `TensorNet-MatPES-r2SCAN`) are specifically optimized for forces and vibrational stability.
> - **Avoid MPtrj-trained models**: Models trained primarily on the `MPtrj` dataset (e.g., `CHGNet-MPtrj`) suffer from the "softening" problem, where the calculated phonon frequencies are significantly lower than DFT values.

Refer to the [foundation-potentials skill](../ml-foundation-potentials/SKILL.md) for more details.

## 3. Choosing the Volume Window

QHA fits the free energy against volume, so the sampled volume range is part of the
physics, not a cosmetic setting.

> [!IMPORTANT]
> **Sample close to equilibrium.** The quasi-harmonic approximation is an expansion
> about the equilibrium volume, and the fitted `F(V)` is only meaningful near its
> minimum. A scan reaching far-compressed or far-expanded volumes biases the fit and
> the phonons together. `matcalc`'s own `QHACalc` default spans **-14% to +16% in
> volume**; on BCC lithium that shifts the thermal expansion coefficient from
> 4.6e-5 to 5.1e-5 /K, about 15%, and widening further to a factor of two in volume
> roughly doubles it. `calculate_qha.py` therefore defaults to `--volume_window 0.10`
> (+/-10% in volume) rather than passing matcalc's default through.

> [!IMPORTANT]
> **But the window must still bracket the minimum at your highest temperature.** The
> lattice expands on heating, so a window adequate at 0 K can be too narrow at high
> T, and a minimiser that runs into the edge of the scan returns the edge rather than
> the minimum. Check that the equilibrium volume at your top temperature is interior
> to the sampled volumes, and widen with `--volume_window` if it is not.

## 4. Calculation Workflow

To calculate thermal expansion and temperature-dependent Gibbs energy, use `calculate_qha.py`.

```bash
conda activate matgl-agent
python .agents/skills/qha/scripts/calculate_qha.py \
    --structure path/to/relaxed_structure.cif \
    --model_type matgl \
    --eos vinet \
    --output_dir research/my_folder/qha
```

## 5. Output Files

- `qha_results.json`: Summary.
- `gibbs_temperature.dat`: Gibbs energy vs T.
- `thermal_expansion.dat`: Thermal expansion vs T.

## 6. Examples

See `examples/` for detailed usage scenarios.
---

**Author:** Bowen Deng
**Contact:** [GitHub @learningmatter-mit](https://github.com/learningmatter-mit)

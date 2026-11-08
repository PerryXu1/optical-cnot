# On-Chip MZI and Linear-Optical CNOT Gate

Silicon photonics design for the **HardHaQ 2026 Photonics Challenge**, extended into a simulated two-qubit quantum gate. This project won 2nd place in the challenge.

- **Part I:** a 50:50 directional coupler and a Mach–Zehnder interferometer (MZI), designed with a mode solver and 3D FDTD in Tidy3D.
- **Part II:** a probabilistic linear-optical CNOT gate built from the same coupler in PhotonForge and simulated with two photons.

## Results

| | Result |
|---|---|
| Coupler split at 1550 nm | 50.0 % |
| MZI extinction ratio at 1550 nm | 52.8 dB |
| CNOT process fidelity at 1550 nm | near unity |
| CNOT process fidelity, 1530–1565 nm | ≥ 95.6 % |
| CNOT post-selected success rate | 10 % (ideal 1/9 ≈ 11.1 %) |

## Part I: Directional coupler and MZI

The coupler uses two 0.50 µm silicon strip waveguides with a 0.20 µm gap. The design follows these steps:

1. **Mode solve.** Find the coupled supermodes of the two waveguides and estimate the coupling length for a 50:50 split.
2. **S-bend correction.** The waveguides also couple in the S-bends that bring them together and apart, so the straight coupling section is shortened to compensate.
3. **3D FDTD.** Simulate the full coupler in Tidy3D and make a small length adjustment to reach exactly 50:50. The final coupling length is 14.88 µm.
4. **MZI.** Combine two simulated couplers and two arms as a circuit, and optimize the arm-length difference (9.53 µm) so that all the light leaves through the bright port at 1550 nm.

The design meets all challenge rules: a split of 50 ± 1 % at 1550 nm, a gap of at least 100 nm, and a bend radius of at least 5 µm.

## Part II: Probabilistic CNOT gate

The gate follows the post-selected scheme of Ralph *et al.* [1], demonstrated in bulk optics by O'Brien *et al.* [2] and on chip by Politi *et al.* [3].

### Encoding

Each qubit is a single photon in one of two waveguides (dual-rail encoding): the control qubit uses `C0`/`C1` and the target uses `T0`/`T1`. Two extra waveguides, `VC` and `VT`, start empty and collect light that is removed from the qubit modes.

### Circuit

The gate is built from the identity CNOT = (I ⊗ H) · CZ · (I ⊗ H):

| MZI | Waveguides | Bar fraction R | Role |
|---|---|---|---|
| H1, H2 | T0, T1 | 1/2 | Hadamard gates on the target |
| CZ | C1, T0 | 1/3 | Controlled-Z through two-photon interference |
| Ac | VC, C0 | 1/3 | Balances the loss on C0 |
| At | T1, VT | 1/3 | Balances the loss on T1 |

Each MZI is two copies of the Part I FDTD coupler with a thermo-optic heater on one arm, so its split can be tuned after fabrication. Eight more heaters on the qubit waveguides cancel the phases added by couplers that are not perfectly 50:50.

### How the CZ works

When the control photon is in `C1` and the target photon is in `T0`, both photons reach the R = 1/3 MZI together. The two paths that leave one photon in each output, "both stay" and "both swap", interfere as in the Hong–Ou–Mandel effect [4]. At R = 1/3 the swap path outweighs the stay path, so the combined amplitude has the same size as when the photons pass separately but the opposite sign. This conditional sign flip is the controlled-Z. The two Hadamards then turn it into a bit flip on the target.

### Post-selection

The gate works only when exactly one photon leaves in the control pair and one in the target pair. This happens with probability 1/9; all other events are discarded.

### Simulation

1. **Layout.** The chip is a PhotonForge component with 6 waveguides, 5 tunable MZIs and 8 phase heaters. Couplers use the Part I FDTD S-matrix, and waveguides and heaters use PhotonForge's analytic waveguide model with the indices from the Part I mode solve.
2. **Heater calibration.** Heater voltages for R = 1/2 and R = 1/3 are solved from the coupler's FDTD amplitudes and checked by simulating each MZI. The phase heaters are then set from the simulated MZI matrices.
3. **Chip S-matrix.** PhotonForge computes the 6×6 transfer matrix of the full chip at each wavelength.
4. **Two-photon amplitudes.** For indistinguishable photons, the amplitude for each pair of output waveguides is the permanent of a 2×2 sub-block of the transfer matrix. Keeping only the outputs with one photon per qubit gives the post-selected gate, which is compared with an ideal CNOT.
5. **Virtual coincidence detector.** A Monte Carlo simulation sends 20,000 photon pairs per input and keeps only the coincidences with one photon per qubit.

Part II reuses the cached FDTD results from Part I, so it needs no new 3D simulations.

## Usage

1. Install the dependencies (developed with PhotonForge 2.0.0 and Tidy3D 2.12.2):

   ```bash
   pip install photonforge tidy3d numpy matplotlib
   ```

2. Set up a Flexcompute account and Tidy3D API key, which Part I's FDTD simulation needs.
3. Open `MZI_On_Chip_CNOT.ipynb` and run all cells from top to bottom. Part I must run before Part II.

## Limitations

- Heater parameters are assumed: 5 V for a π phase shift, 100 µm heater length and 2 dB/cm waveguide loss.
- The chip is simulated at circuit level. Coupling between neighboring waveguides outside the couplers, thermal crosstalk between heaters and fabrication variation are not modeled.
- Photons are assumed to be perfectly indistinguishable, and detectors are assumed ideal (100 % efficiency, no dark counts).
- Heaters are calibrated at 1550 nm and kept fixed across the band, which is why the fidelity drops toward 1530 and 1565 nm.

## Future work

- Broadband couplers (MMI or adiabatic) to keep the gate accurate across the band
- Models for thermal crosstalk, fabrication variation, photon distinguishability and detector efficiency
- On-chip photon-pair sources and a heralded (rather than post-selected) gate for scaling to more qubits

## References

1. T. C. Ralph, N. K. Langford, T. B. Bell and A. G. White, "Linear optical controlled-NOT gate in the coincidence basis," *Phys. Rev. A* **65**, 062324 (2002).
2. J. L. O'Brien, G. J. Pryde, A. G. White, T. C. Ralph and D. Branning, "Demonstration of an all-optical quantum controlled-NOT gate," *Nature* **426**, 264–267 (2003).
3. A. Politi, M. J. Cryan, J. G. Rarity, S. Yu and J. L. O'Brien, "Silica-on-silicon waveguide quantum circuits," *Science* **320**, 646–649 (2008).
4. C. K. Hong, Z. Y. Ou and L. Mandel, "Measurement of subpicosecond time intervals between two photons by interference," *Phys. Rev. Lett.* **59**, 2044–2046 (1987).
5. N. C. Harris *et al.*, "Efficient, compact and low loss thermo-optic phase shifter in silicon," *Opt. Express* **22**, 10487–10493 (2014).

## Authors

Perry Xu and Jeffrey Hu, University of Toronto

# Dataset: `RC_dataset_python.mat`

Simulated thermal-noise power spectral densities of a one-pole RC circuit (Simulink/Simscape, sample time
Ts = 1 µs, resistor tolerance 1 %, capacitor tolerance 5 %, temperature-dependent component values).

## Design grid (full factorial, 2880 circuits)

| Quantity | Values | Count |
|----------|--------|-------|
| R (Ω) | 100, 200, ..., 2000 | 20 |
| C (F) | 1e-9, 1e-8, 1e-7, 2.2e-7, 1e-6, 1e-5, 1e-4, 2.2e-4, 1e-3 | 9 |
| T (K) | 250, 260, ..., 400 | 16 |

Each circuit has one PSD with 1025 frequency bins from 0 to 500 kHz (bin spacing 488.28125 Hz).

## File format

MATLAB v5 `.mat` file. All variables are column vectors of length 2,952,000 = 2880 circuits × 1025 bins in
"long" format: rows `1025*i ... 1025*i + 1024` belong to circuit `i`.

| Variable | Type | Unit | Description |
|----------|------|------|-------------|
| `f` | float64 | Hz | frequency of the bin |
| `PSD` | float64 | V²/Hz | one-sided noise PSD (Welch estimate) |
| `R` | uint16 | Ω | nominal resistance |
| `C` | float64 | F | nominal capacitance |
| `R_eff` | float64 | Ω | effective resistance (tolerance and temperature applied) |
| `C_eff` | float64 | F | effective capacitance (tolerance and temperature applied) |
| `T` | uint16 | K | temperature |

## Loading in Python

```python
import numpy as np, scipy.io as sio

m = sio.loadmat("RC_dataset_python.mat", squeeze_me=True)
F = 1025
f = m["f"][:F]                                  # frequency axis (same for all circuits)
PSD = m["PSD"].reshape(-1, F)                   # (2880, 1025)
R, C, T = (m[k][::F] for k in ("R", "C", "T"))  # one value per circuit
R_eff, C_eff = m["R_eff"][::F], m["C_eff"][::F]
```

## License

Creative Commons Attribution 4.0 International (CC BY 4.0), see `../LICENSE-DATA`.

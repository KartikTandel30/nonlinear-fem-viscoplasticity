# Reproducibility and execution

## Environment

The current standalone `PVL.py` imports **NumPy** and **Matplotlib**. A Python 3 environment containing compatible versions of these packages is sufficient to *attempt* execution. Exact versions used for the archived plots were not captured in the source repository, so the root `requirements.txt` lists direct dependencies **without claiming version reproducibility**.

On Windows PowerShell:

```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
python PVL.py
```

On Git Bash / Linux / macOS:

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
python PVL.py
```

If PowerShell prevents script activation, use `\.venv\Scripts\python.exe -m pip install -r requirements.txt` and `\.venv\Scripts\python.exe PVL.py` without activating the environment.

## Current script behavior

- Model and material parameters are defined at the beginning of `PVL.py`, not through CLI flags or a separate configuration file.
- Both ends are fixed, and the applied force is prescribed at the segment junction.
- The code attempts 4,200 load/time increments over 0.01 s with up to 10 Newton iterations per increment.
- It displays a 2 × 2 Matplotlib figure for the two stress–strain responses, force–displacement response and displacement–time response.
- It does **not** automatically export run metadata, data CSV files or its final plot to disk.
- No exact package versions or deterministic benchmark regression tests are supplied yet.

## Before reporting numerical values

Record the Git commit SHA (`git rev-parse HEAD`), `python --version`, `python -m pip freeze`, force/time schedule, number of elements, constitutive constants, error tolerances, and whether the Newton iteration converged at each increment. Use a separately saved output directory or commit when producing new verification figures, and compare the elastic junction displacement to the analytical reference in `VERIFICATION.md`.

Do **not** mistake the project's existing figures for proof of constitutive-model verification; see [verification and known limitations](VERIFICATION.md).

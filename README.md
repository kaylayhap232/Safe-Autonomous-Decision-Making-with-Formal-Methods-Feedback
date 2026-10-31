# Paper baseline

A small runnable test of the NuSMV controller examples in Appendix D of Yang et al., *Fine-Tuning Language Models Using Formal Methods Feedback*.

## Windows

1. Extract the ZIP completely. Keep the notebook and NuSMV folder together.
2. Install Python 3.10 or newer if needed (https://www.python.org/downloads/).
3. Open a terminal inside this folder and run:

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe -m jupyterlab paper_baseline_completed.ipynb
```

After setup, double-click `start_windows.bat` to reopen it. In JupyterLab choose **Run > Run All Cells** with the Python 3 kernel. The first setup downloads Jupyter packages and needs internet. Model checking needs no API key or internet.

If Jupyter is already installed, open this notebook with it from this folder; the virtual environment and launcher are optional.

## Mac or Linux

The bundled checker is Windows x86-64 only. Install the appropriate NuSMV binary from https://nusmv.fbk.eu/downloads.html and put it on PATH, or set `NUSMV_PATH` to the executable. Then:

```sh
python3 -m venv .venv
.venv/bin/python -m pip install -r requirements.txt
.venv/bin/python -m jupyterlab paper_baseline_completed.ipynb
```

Mac/Linux execution has not been tested for this bundle.

## Files

- `paper_baseline_completed.ipynb`: repaired examples, checker runner, and results display.
- `paper_code_only.ipynb`: unchanged paper listings for comparison.
- `NuSMV/`: portable Windows checker; keep all its contents together.
- `requirements.txt`: notebook interface dependencies.
- `THIRD_PARTY.md`: sources.

## Expected results

The Windows test completed all six code cells and all 16 property checks using NuSMV 2.7.1: **5 PASS, 11 FAIL**. FAIL is a counterexample, not a failed installation. Timing and missing environment assumptions explain several failures; see the notebook notes. Do not alter the model just to make every property pass.

This is a repaired Appendix D example, not a reproduction of the full paper experiment. Changes are listed in the notebook. The original controller branches remain intact; Python integration and extra next-action properties were added. The saved notebook has cleared outputs so recipients produce their own results.

If the checker is not found, launch Jupyter from this extracted folder. If Python is not found, install Python or replace `python` with your system's Python command.

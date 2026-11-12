# Driving decisions with formal feedback

This project compares Granite 3.1 MoE 3B and LLaMA 3.2 3B on driving scenarios. Both models run locally through Ollama. NuSMV checks their proposed actions, and mistakes from the development cases become examples in later prompts.

The question is whether those examples help each model make better decisions on new scenarios. Feedback changes the prompt, not the model weights.

## Run the comparison

Install Python 3.10 or newer and [Ollama](https://ollama.com/). Keep Ollama running, then open PowerShell in this folder. Download the two models once:

```powershell
ollama pull granite3.1-moe:3b
ollama pull llama3.2:3b
```

Check the setup without calling either model:

```powershell
python compare_llms.py --check-only
```

Start with the six-scenario pilot:

```powershell
python compare_llms.py --run --stage starter
```

Run the full experiment:

```powershell
python compare_llms.py --run --stage full
```

The script uses Python's standard library, so Jupyter and the notebook requirements are not needed for this experiment. No API key or paid API is used. Model downloads need internet; the experiment runs on your computer. Runtime depends on your hardware.

On Windows, keep the bundled `NuSMV/` folder beside the script. On Mac or Linux, install NuSMV and pass `--nusmv /path/to/NuSMV`. Those platforms have not been tested here.

## What gets tested

Each scenario starts with a stopped vehicle planning to go straight, turn left, or turn right. The model chooses `PROCEED`, `STOP`, or `REQUEST_INFO`. Cases include pedestrians, blocked paths, conflicting traffic, traffic-control permission, and missing or contradictory sensor data.

The full experiment has 15 development cases and 35 held-out test cases. Each test case runs three times under each condition:

- **Baseline:** the scenario and response instructions.
- **Rules:** the same information plus the driving rules.
- **Feedback:** the rules plus that model's corrected development mistakes.

Feedback is collected separately for each model and frozen before testing. Test answers never become feedback. The six-case pilot uses cases from the development pool, leaving the full test set out of the pilot.

There are 105 test decisions per model and condition, or 660 model calls including development. Use `--trials 1` for a shorter full run. The default `--stage both` runs the pilot and full experiment in sequence with fresh feedback for each stage.

These scenarios were written for this project. Different descriptions can share the same encoded state, so held-out descriptions do not necessarily mean new state combinations.

## Formal verification

For each valid proposed action, the script checks a NuSMV model against 11 LTL properties: eight safety checks and three action-selection checks. Identical state/action checks reuse the checker result. Invalid model responses are recorded as invalid, not treated as verified decisions.

The model holds the observations and proposed action fixed. It checks that decision against the encoded rules; it does not simulate a complete journey or prove that a real vehicle is safe. Staying stopped can satisfy the safety checks while still being the wrong action for a clear road.

## Read the results

Each run creates a timestamped folder under `llm_results/`. The completed starter and full runs from November 12, 2026 are included here. New runs stay local unless added explicitly.

- `summary.csv`: metrics for each model and condition.
- `feedback_improvement.csv`: feedback compared with rules alone.
- `model_comparison.csv`: Granite minus LLaMA for each condition.
- `test_results.csv`: individual test decisions and reference answers.
- `feedback.json`: the development corrections given to each model.
- `decisions.jsonl` and `responses.jsonl`: prompts, responses, and verification records.
- `verification/`: generated `.smv` files and NuSMV logs, including counterexamples.
- `config.json`, `scenarios.csv`, and `test_order.json`: the settings, cases, and evaluation order.

Rates are decimals: `0.70` means 70%. Accuracy measures the expected action. Unsafe rate counts proceeding when the reference policy requires stopping or more information. Safe-proceed rate measures how often the model proceeds when it should. Uncertainty recognition measures correct requests for information. Invalid rate counts unusable responses.

Formal-pass rate requires all 11 properties to pass. Safety compliance covers the eight safety properties; invalid responses count as failures. Consistency measures how often a scenario receives the same valid action across its repetitions. Compare all these measures: a model that never moves can avoid unsafe movement while making poor decisions.

## Troubleshooting

If Ollama cannot be reached, open the Ollama app and run `ollama list`. The default models must appear as `granite3.1-moe:3b` and `llama3.2:3b`. If NuSMV cannot be found, check that `NuSMV/bin/NuSMV.exe` is beside the script's folder structure, or pass its executable path with `--nusmv`.

## Original paper baseline

The notebooks below are the earlier Appendix D baseline. The driving comparison is a separate project extension, not a reproduction of the paper's training experiment. The original notebook instructions are preserved below.

---

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

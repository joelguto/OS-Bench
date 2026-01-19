# OSBench / OSWorld-SFT Assessment Overview

This document summarizes the provided assessment PDF (`OsWorld RL Assessment.pdf`) and explains the task, deliverables, setup, and terminology for newcomers. Source: [Assessment PDF](file:///Users/joelasiago/Downloads/OS%20Bench/OsWorld%20RL%20Assessment.pdf).

## What the Assessment Is About
- You must complete an OSBench assessment using the OSWorld-SFT framework.
- Core task to implement (as both a Task JSON and an SFT trajectory): open GIMP, convert `photo.png` to black and white (grayscale), then export it to the Desktop as `gray_photo.png`.
- You need to demonstrate the full workflow: environment setup, repo exploration, task definition, VM execution, and producing the required artifacts.

## Required Deliverables
- **Task JSON**: Formal, machine-readable description of the GIMP grayscale task following the OSWorld Task JSON schema.
- **SFT Trajectory**: A recorded step-by-step execution trace showing how the task is performed in the VM (used for supervised fine-tuning).
- Both artifacts must follow the formats described in the assessment guide.

## High-Level Workflow
1. Set up the environment (clone repo, install Python 3.10, create venv with `uv`, install requirements including the patched `borb`).
2. Ensure VMware is installed and usable from the terminal (`vmware`, `vmrun` on PATH), accept the EULA, and download the provided VM image.
3. In the VM, perform the task: use GIMP to grayscale `photo.png` and export `gray_photo.png` to the Desktop.
4. Capture the Task JSON and the SFT trajectory that reflect the exact steps and outputs.
5. Validate the task execution using the evaluator guidance in the repo.

## Environment Setup (macOS focus from guide)
- Clone: `git clone https://github.com/asad-a1-tr/OSWorld-SFT.git && cd OSWorld-SFT`
- Python: use **Python 3.10** (Homebrew: `brew install python@3.10 && brew link python@3.10`)
- Virtual env: `uv venv .venv --python=python3.10 && source .venv/bin/activate`
- Dependencies: `uv pip install -r requirements.txt`
  - Note: `borb==3.0.2` on PyPI is broken; install from GitHub `git+https://github.com/jorisschellekens/borb.git@master` (already reflected in the guide’s requirements).
- Verify install: `python -c "import desktop_env; print('OSWorld-SFT ready')"`

## VMware Setup (from guide)
- Install VMware Fusion 13.6.2 on macOS (requires Broadcom account).
- Open once to accept EULA and allow module downloads; ensure `vmware` and `vmrun` are on PATH.
- Download and start the VM image: `python Turing_tooling/run_manual.py --provider_name vmware` (downloads ~11.3 GB into `vmware_vm_data` and launches the VM).

## Hosting Task Files (Hugging Face Note)
- To avoid Google Drive download limits, host task files on Hugging Face Datasets.
- Task JSON can reference Hugging Face URLs directly for downloads in the VM.

## Evaluator Notes (quick orientation)
- Evaluators check outputs (files, command outputs, accessibility trees, etc.) and return scores 0.0–1.0.
- Common result types: `vm_file`, `vm_command_line`, `cloud_file`, `accessibility_tree`.
- Example evaluator functions include `exact_match`, `compare_table`, `compare_pdf`, etc. (full list in repo).

## Key Terms for Beginners
- **OSWorld-SFT**: A framework for creating and replaying desktop automation tasks inside a VM, producing data for supervised fine-tuning.
- **OSBench**: The assessment/benchmark context you’re completing using OSWorld-SFT.
- **Task JSON**: Structured JSON that defines the task (initial file downloads, expected outputs, and evaluator configuration).
- **SFT trajectory**: The recorded sequence of actions and observations taken to complete the task (used to train or evaluate models).
- **Evaluator**: A function that scores task completion by comparing produced results to expected outputs or rules.
- **VMware Fusion**: The virtualization software used to run the provided desktop VM where tasks are executed.
- **uv**: A fast Python dependency/venv manager used to create and manage the virtual environment.
- **borb**: A Python PDF library; the PyPI release has a packaging issue, so the guide installs it from GitHub.
- **GIMP**: An image editing application used here to convert `photo.png` to grayscale and export `gray_photo.png`.

## What to Do Next
- Confirm Python 3.10 and venv setup with `uv`.
- Install dependencies (including patched `borb`) and verify with the import test.
- Ensure VMware Fusion works from the terminal and download the VM image.
- Plan the Task JSON and the SFT trajectory steps for the GIMP grayscale conversion.
- Run and validate the task inside the VM, then save the JSON and trajectory as deliverables.

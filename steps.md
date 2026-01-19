# OSWorld-SFT Assessment Steps (from OsWorld RL Assessment.pdf)

Use the exact flow below to set up the environment, prepare the VM, and produce the required artifacts for the assessment.

## 1) Understand the required task and deliverables
1. Snapshot to use: `gimp`.
2. Instruction to fulfill: open `photo.png`, convert it to black and white (grayscale), and export the grayscale image to the Desktop as `gray_photo.png`.
3. Final deliverables (both are mandatory):
   - Task JSON representing the instruction in OSWorld-SFT format.
   - SFT trajectory capturing the full task execution flow.

## 2) Get the codebase
1. Clone the repository:
   ```bash
   git clone https://github.com/asad-a1-tr/OSWorld-SFT.git
   cd OSWorld-SFT
   ```

## 3) Use Python 3.10 exactly
1. Check current Python:
   ```bash
   python3 --version
   ```
2. If Python 3.10 is missing on macOS, install and link:
   ```bash
   brew install python@3.10
   brew link python@3.10
   ```
3. Verify Python 3.10 availability:
   ```bash
   python3.10 --version
   ```
   (Torch wheels are available for Python 3.10; Python 3.13 on macOS ARM is not supported by Torch yet.)

## 4) Create an isolated virtual environment with uv
1. From the repo root, create the venv pinned to Python 3.10:
   ```bash
   uv venv .venv --python=python3.10
   ```
2. Activate it:
   ```bash
   source .venv/bin/activate
   ```
3. Confirm the interpreter:
   ```bash
   python --version
   ```

## 5) Handle the borb packaging bug
1. `borb==3.0.2` on PyPI is corrupted (duplicate file entries). Install from GitHub instead.
2. Either replace the `borb` line in `requirements.txt` with:
   ```
   git+https://github.com/jorisschellekens/borb.git@master
   ```
   or install it directly after activation:
   ```bash
   uv pip install git+https://github.com/jorisschellekens/borb.git@master
   ```

## 6) Install dependencies
1. From the repo root with the venv active:
   ```bash
   uv pip install -r requirements.txt
   ```
   (Installs Torch-compatible wheel, Playwright via setup hook, and other loosened requirements.)

## 7) Verify installation
1. Run the import check:
   ```bash
   python -c "import desktop_env; print('OSWorld-SFT ready')"
   ```
   Expect the output: `OSWorld-SFT ready`.

## 8) Install VMware (desktop VM provider)
### macOS (Fusion)
1. Create/log in to a Broadcom account (VMware distribution).
2. Download VMware Fusion Player 13.6.2 for macOS (Broadcom portal or the provided Google Drive link).
3. Install from the DMG, drag Fusion to Applications, then launch once to grant permissions.

### Windows (Workstation)
1. Download VMware Workstation 17.6.3 installer.
2. Run the `.exe`, follow on-screen steps, and reboot if prompted.

## 9) Accept VMware EULA and ensure CLI availability
1. Launch VMware at least once to accept the EULA and allow module downloads.
2. Confirm `vmware` and `vmrun` are in `PATH`; if not, add them and reboot.
3. You do not need to subscribe to the user experience program or auto-update on start.

## 10) Download and start the VM image
1. From the repo root after requirements are installed:
   ```bash
   python Turing_tooling/run_manual.py --provider_name vmware
   ```
2. This downloads ~11.3GB into `vmware_vm_data/`. If successful, the VM starts locally.

## 11) Execute and record the task
1. In the VM snapshot `gimp`, open `photo.png`.
2. Convert the image to grayscale/black-and-white.
3. Export the grayscale result to the Desktop as `gray_photo.png`.
4. Record the complete SFT trajectory for this execution.

## 12) Prepare the submission artifacts
1. Create the Task JSON that encodes the instruction and any needed file downloads per OSWorld-SFT schema.
2. Ensure the SFT trajectory is complete and consistent with the executed steps.
3. Validate both artifacts against the format guidance in the assessment.

## 13) (Optional but recommended) Host task files on Hugging Face
1. Rationale: avoids Google Drive download limits by using Hugging Face Datasets (or ModelScope) for stable, versioned hosting.
2. Create a Hugging Face account, then create a Dataset repo.
3. Upload required task assets (e.g., initial files) and reference their HTTPS URLs in task configs (e.g., `config.download.files.url`).
4. Use absolute URLs in task JSON/evaluator fields so the VM can fetch assets reliably.

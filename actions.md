# OSBench / OSWorld-SFT — **actions.md (exhaustive runbook)**
This file is intentionally **long** and **non-summarized**: it is a “do-everything” runbook derived from `instructions.md` (and intended to mirror the contents of `OsWorld RL Assessment.pdf`, which is described as the same guideline), with **every detail preserved as concrete actions, checks, paths, commands, flags, structures, warnings, and notes**.

If you are doing the OSBench assessment, you should be able to follow this file **linearly** from top to bottom and produce the deliverables exactly as required.

---

## Assessment task you must implement (do not change)

- **Snapshot**: `gimp`
- **Instruction** (the exact task):
  - Convert the image `photo.png` to black and white (grayscale), then export the grayscale image to the Desktop as `gray_photo.png`.

---

## Final deliverables (what must exist at the end)

You must submit **two** artifacts, each following the OSWorld-SFT formats:

- **Task JSON**: a JSON task definition that fully specifies:
  - VM snapshot / environment
  - pre-task setup (`config`) such as download + launch actions
  - evaluator definition (how success is checked)
  - metadata fields required by schema and by the assessment instructions
- **SFT trajectory**: a recorded “golden” demonstration of completing the task in the VM using manual step entry (PyAutoGUI-style actions) that outputs:
  - a trajectory log (`trajectory.jsonl`)
  - screenshots for each step (`step_N_before.png`)
  - accessibility trees (`step_N_before.xml`)
  - evaluation output file(s) such as `result.txt` (and/or `evaluation_score.txt` depending on packaging)
  - logs (runtime/manual log files)
  - optional recording (`recording.mp4`) if produced

---

## Repo setup — OSWorld-SFT local installation

### Clone the repository (if you don’t already have it)

Run in a shell:

```bash
git clone https://github.com/asad-a1-tr/OSWorld-SFT.git
cd OSWorld-SFT
```

### Ensure correct Python version

- **Requirement**: Python **== 3.10**
- **Reason note from instructions**:
  - System default may be older (e.g., 3.9).
  - Torch does not provide wheels for Python 3.13 on macOS ARM (so you must use 3.10).

Check what you have:

```bash
python3 --version
```

Example expected:

```text
Python 3.10.x
```

If on macOS and you need Python 3.10:

```bash
brew install python@3.10
brew link python@3.10
python3.10 --version
```

### Create a virtual environment (the guide uses `uv`)

They used `uv` as a fast dependency resolver / venv manager.

Create venv:

```bash
uv venv .venv --python=python3.10
```

Activate:

```bash
source .venv/bin/activate
```

Verify:

```bash
python --version
```

Expected:

```text
Python 3.10.x
```

---

## Dependency issues & fixes (do these if applicable)

### Torch note

- Torch wheels are available for Python 3.10 → no special manual fix needed (as long as you’re on 3.10).

### BOR(B) packaging bug (IMPORTANT)

- `borb==3.0.2` on PyPI is described as “corrupted” (duplicate file entries).
- Fix: install directly from GitHub.

Do **one** of the following:

- **Option A (edit requirements)**: replace the `borb` pin in `requirements.txt` with:

```text
git+https://github.com/jorisschellekens/borb.git@master
```

- **Option B (manual install)**:

```bash
uv pip install git+https://github.com/jorisschellekens/borb.git@master
```

### Updated requirements note (from the guide)

The instructions mention they “cleaned up requirements.txt to loosen strict pins and add missing dependencies”.
An excerpt of a “working set” mentioned includes:

```text
numpy>=1.26.0
Pillow>=10.0.0
gymnasium>=0.29.0
requests>=2.31.0
torch>=2.2,<2.6
transformers>=4.38.0
...
google-cloud-compute
google-cloud-storage
git+https://github.com/jorisschellekens/borb.git@master
```

Note: the guide says the “full list” is maintained in the repo.

---

## Install dependencies

From repo root:

```bash
uv pip install -r requirements.txt
```

The guide claims this installs:

- all standard dependencies
- torch compatible wheel for Python 3.10
- Playwright (auto-installed via setup.py post-install hook)
- BOR(B) from GitHub (if configured as above)

---

## Verify installation (smoke test)

Run:

```bash
python -c "import desktop_env; print('OSWorld-SFT ready ✅')"
```

Expected output:

```text
OSWorld-SFT ready ✅
```

---

## VM setup — OSWorld-SFT Local Installation (VMware)

### Why VMware (from the guide)

OSWorld-SFT tasks run inside a virtual desktop environment. VMware isolates the agent and ensures reproducibility. Scripts handle download/config.

### Installing VMware on macOS (Fusion)

1) **Create/Use a Broadcom account**
   - VMware is distributed by Broadcom.
   - Use Broadcom Support Portal, create free account, log in.

2) **Download VMware Fusion 13.6.2**
   - Guide mentions a “VMware Fusion Free Downloads” page.
   - Also mentions a Google Drive DMG link (as an alternative installer source):
     - `https://drive.google.com/file/d/1lZupdvHR8pepLiNHXE90Ku-EcdPX89mv/view?usp=sharing`

3) **Install**
   - Open `.dmg`
   - Drag VMware Fusion to Applications
   - Open it and grant permissions (System Preferences → Security & Privacy → Allow)

4) **Verify**
   - Launch VMware Fusion
   - You should see “Create a New Virtual Machine”

### Installing VMware on Windows (Workstation)

1) Download VMware Workstation 17.6.3 (direct link mentioned in guide)
2) Install by running the `.exe`, follow prompts, reboot if prompted
3) Verify: open VMware Workstation → you should see the “Home” screen

### CRITICAL: Start VMware once and accept EULA / install missing modules

- You must start VMware at least once before continuing:
  - allow missing modules download
  - accept EULA
  - you do **not** need to subscribe to user experience program
  - you do **not** need to enable “check for updates on start”

Start VMware from terminal:

```bash
vmware
```

### CRITICAL: `vmware` and `vmrun` must be accessible

- The commands `vmware` and `vmrun` must be searchable in PATH and usable from prompt.
- If not accessible: add to PATH and reboot.

---

## Download the VM image / ISO and start VM (per guide)

After cloning repo and installing requirements:

```bash
python Turing_tooling/run_manual.py --provider_name vmware
```

Notes from guide:

- This downloads ~11.3GB under the folder `vmware_vm_data`.
  - The guide’s formatting may show this as split text like `vmware vm _ _ data`, but it refers to the same `vmware_vm_data` directory in the repo.
- If everything is fine, it will start the VM locally.
- If you have done these steps, proceed with task creation.

---

## PDF copy of the instructions (OsWorld RL Assessment.pdf) — how to cross-check it (when needed)

The PDF is described as the same guideline. If you want to verify/cross-check wording from the PDF (or extract text from it), do one of the following on your machine (outside the VM):

- **Option A (macOS / Linux)**: use `pdftotext` if available:

```bash
pdftotext "OsWorld RL Assessment.pdf" - | less
```

- **Option B (Python)**: use `pdfminer.six` (install into the same Python environment you’re using for tooling, ideally Python 3.10 as required by OSWorld-SFT):

```bash
python -m pip install pdfminer.six
python -c "from pdfminer.high_level import extract_text; print(extract_text('OsWorld RL Assessment.pdf')[:5000])"
```

Keep the extracted text alongside your notes to ensure the runbook steps you follow match the official wording.

---

## File hosting guidance (Hugging Face) — use this if your task needs file downloads

### Why Hugging Face (from the guide)

- Older OSWorld versions used Google Drive for task files.
- Google Drive has download limits; links can get blocked.
- Hugging Face Datasets (or ModelScope) provides stable versioned hosting.
- OSWorld-SFT tasks can fetch from Hugging Face URLs in `config` and evaluator fields.

### How to structure download config in task JSON

Example (shape only; you must substitute your real URLs/paths):

```json
"config": [
  {
    "type": "download",
    "parameters": {
      "files": [
        {
          "url": "https://huggingface.co/datasets/<user>/<repo>/resolve/main/Initial_file.png",
          "path": "/home/user/Desktop/photo.png"
        }
      ]
    }
  }
]
```

### Create Hugging Face account

1) Go to `https://huggingface.co/join`
2) Sign up (guide suggests gmail)
3) Verify email
4) Your profile page becomes:
   - `https://huggingface.co/<your-username>`

### Create a Dataset repository

1) Create New Dataset
2) Fill:
   - Owner: your username or org
   - Dataset name: e.g. `osworld_tasks_files`
   - Visibility: **Public** (required)
   - License: default
3) Create Dataset
4) Repo URL format:
   - `https://huggingface.co/datasets/<your-username>/<your-repo>`

### Upload files

Via web UI:

1) Dataset repo → “Files and Versions”
2) “Add file” → “Upload from computer”
3) Drag & drop your files (.docx, .xlsx, .pdf, .png, etc.)
4) Commit changes

### Get file download links

- Use `resolve/main/<filename>` URLs.
- Example:
  - `https://huggingface.co/datasets/<your-username>/<your-repo>/resolve/main/budget.xlsx`

### Best practices (from guide)

- Keep initial files and golden files in same repo
- Use clear filenames with suffixes:
  - `Initial_file.png`
  - `Golden_file.png`

---

## OSWorld Task JSON creation (Phase A) — what the JSON is and what it must include

### What the JSON is used for (exact responsibilities described)

The JSON tells the system:

1) What files to download and where to put them in the VM (`download` config)
2) How to open the application with the file (`launch` config)
3) What the LLM should accomplish (`instruction` field)

System flow described:

- JSON sets up VM environment automatically.
- Instruction is sent to LLM agent.
- Agent interacts with VM via screenshots and actions to complete task.

### Key parameters called out

- `config` array:
  - automates VM setup (downloads, launches apps) before AI starts
- `instruction`:
  - the actual task description
- `evaluator`:
  - defines how to check if completed successfully
- `snapshot`:
  - pre-configured VM environment to use

### Mandatory JSON file contents (explicitly labeled “disqualification if missing”)

The guide states: “Failure to include these fields will result in disqualification from evaluation”.

Fields listed:

- **`id`**: unique UUID string
- **`snapshot`**: VM environment to use (for this assessment you must set: `gimp`)
- **`instruction`**: task description (must match assessment instruction)
- **`source`**: reference URL or attribution (guide uses “Turing” as example)
- **`trajectory`**: directory path where execution logs are stored (guide says mostly `"trajectories/"`)
- **`config`**: setup steps (download, launch, activate window, etc.)
- **`related_apps`**: list of apps involved (for this assessment: include `gimp`)
- **`evaluator`**: how to check success (choose from evaluator documentation; incorrect choice can yield wrong score)
- **`proxy`**: boolean
- **`fixed_ip`**: boolean
- **`possibility_of_env_change`**: risk level, `"low"` or `"high"`
- **`model_pass_rate`**:
  - expected success rate; guide instructs to include **only** `"claude-4-sonnet-20250514"`
- **`annotator_hints`**: step-by-step hints for humans
- **`knowledge_points`**: skills/concepts tested
- **`coverage`**: brief description of what task covers

### Example JSON in the guide (domain: VS Code)

The guide includes a full example for VS Code; keep it as a reference for structure (even though your snapshot is `gimp`). The example is reproduced here in the same shape/wording as shown:

```json
{
  "id": "a1b2c3d4-e5f6-7890-ab12-cdef34567890",
  "snapshot": "vscode",
  "instruction": "Open VS Code, open **User Settings (JSON)** (not the read-only Default Settings) and set \"editor.fontSize\" to 16 and write , after it. Then verify the User settings.json contains \"\\\"editor.fontSize\\\": 16\".",
  "source": "custom_task_vscode_settings_check",
  "config": [
    {
      "type": "launch",
      "parameters": { "command": ["code"] }
    },
    {
      "type": "activate_window",
      "parameters": { "window_name": "Visual Studio Code" }
    }
  ],
  "trajectory": "trajectories/",
  "related_apps": ["vscode"],
  "evaluator": {
    "func": "is_extension_installed",
    "result": {
      "type": "vm_command_line",
      "command": [
        "sh",
        "-c",
        "cat ~/.config/Code/User/settings.json 2>/dev/null || cat ~/Library/Application\\ Support/Code/User/settings.json 2>/dev/null || echo '{}'"
      ]
    },
    "expected": {
      "type": "rule",
      "rules": { "type": "contain", "expected": "\"editor.fontSize\": 16" }
    }
  },
  "proxy": false,
  "fixed_ip": false,
  "possibility_of_env_change": "low",
  "model_pass_rate": {
    "claude-4-sonnet-20250514": 0.875
  },
  "annotator_hints": [
    "Step 1: Launch Visual Studio Code",
    "Step 2: Open User Settings (JSON)",
    "Step 3: Add or update the setting \"editor.fontSize\": 16",
    "Step 4: Save the changes and close settings.json"
  ],
  "knowledge_points": [
    "VS Code configuration",
    "JSON-based settings editing",
    "Verifying applied settings"
  ],
  "coverage": "Basic VS Code customization"
}
```

Immediately after this example, the guide warns: “The evaluator function should be chosen correctly, or else it will affect and cause wrong evaluator result”.

- has `config` with `launch` and `activate_window`
- has `evaluator` with `func`, `result`, `expected`
- includes `proxy`, `fixed_ip`, `possibility_of_env_change`, `model_pass_rate`, `annotator_hints`, `knowledge_points`, `coverage`

---

## Where to put your task JSON (paths the guide uses)

### Validation staging (“Deliverable” folder)

After creating the JSON file:

- Create a folder named `Deliverable` (guide says: “place it in a new folder named Deliverable within the directory”).
- The validation command assumes the file name equals `<task_id>.json`.

Validate:

```bash
python Turing_tooling/validation_script.py Deliverable <task_id>
```

The guide shows an additional line right above the command:

- `--checks json`

Expected success output (guide says it is concise):

```text
✅ JSON structure validation: PASSED
```

On failure:

```text
❌ JSON validation error: [specific error message]
```

Additional validation note:

- Script uses `osworld_updated_schema.json` schema file to validate.
- You can use that schema file to create your JSON manually or with an LLM.

### SFT run placement (evaluation_examples)

For SFT creation, guide says to store task JSON under:

```text
evaluation_examples/examples/<domain>/<task_id>.json
```

Example given:

```text
evaluation_examples/examples/vs_code/Turing2ba02ad5-d008-4f9e-a07f-729aae3c531b.json
```

For your assessment, choose the correct `<domain>` folder consistent with your repo’s structure (commonly `gimp` or similar, depending on how OSWorld-SFT organizes domains).

---

## SFT creation — Solution generation (golden demonstration)

### Objective (explicit)

- Create the best possible “golden” solution that completes task exactly as expected.
- Record the exact sequence of GUI actions (clicks, keystrokes).
- Package trajectory into SFT folder alongside task JSON.
- This demonstration is used for fine-tuning later.

### Environment setup checklist (per guide)

- **Reset VM to clean snapshot** (e.g., `init_state`) before starting.
- For retries:
  - ALWAYS revert the VM before re-running to avoid side-effects.

### Run the manual SFT tool (run_manual.py)

Command shown in guide:

```bash
python Turing_tooling/run_manual.py \
  --provider_name vmware \
  --observation_type screenshot_a11y_tree \
  --task_file evaluation_examples/manual_task.json \
  --log_level INFO \
  --result_dir SFT
```

Important details:

- `--provider_name vmware`: uses VMware backend
- `--observation_type screenshot_a11y_tree`: collects screenshot + accessibility tree per step
- `--task_file`: full path to task JSON
- `--result_dir SFT`: output folder

Timing note:

- VM start takes ~1–2 minutes depending on machine.

### Enter actions one by one (interactive)

You will be prompted:

```text
Enter actions one by one. Type 'done' to finish the task.
```

- Each action you input is logged and executed in the VM.
- When finished, type `done`.

### Use PyAutoGUI-style commands (guide examples)

Examples included:

```python
pg.click(960, 540)
pg.hotkey("ctrl", "shift", "s")
pg.typewrite("jsonformatter.json")
pg.press("enter")
```

### Understanding state from screenshot + XML

For each step, the tool generates:

- `step_0_before.png`
- `step_0_before.xml`

Use XML to find actionable elements.

Guide example shows:

- `cp:screencoord="(70, 64)"`
- `cp:size="(36, 25)"`

To click center:

```python
x = 70 + 36 / 2
y = 64 + 25 / 2
pg.click(x, y)
```

### Common SFT challenges + workarounds (guide list)

1) **No coordinates in XML**
   - Ubuntu sidebar or floating elements may not return `screencoord`.
   - Workaround:
     - Use external coordinate tool (guide provides):
       - `https://www.programminghead.com/Projects/find-coordinates-of-image-online.htm`
     - Load screenshot, locate approximate center, use that click location.

2) **Stuck in the middle of a task**
   - Try a different valid path (Save As menu vs shortcut).
   - Ask an LLM for alternative valid approaches (guide explicitly says this).

4) **Validator not triggering**
   - Ensure evaluator block has correct JSON keys.
   - Example shown (shape):

```json
"result": {
  "type": "vm_file",
  "path": "/home/user/Desktop/jsonformatter.json",
  "dest": "jsonformatter.json"
}
```

### Finish and save (what happens after `done`)

When you type `done`, the tool will:

- run evaluator function
- write `result.txt` with `1.0` or `0.0`
- save actions into:
  - `SFT/<domain>/<task_id>/trajectory.jsonl`
  - `SFT/<domain>/<task_id>/step_*.xml`
  - `SFT/<domain>/<task_id>/step_*.png`

Guide example of folder structure:

```text
SFT/
└── vs_code/
    └── 2ba02ad5-d008-4f9e-a07f-729aae3c531b/
        ├── trajectory.jsonl
        ├── result.txt
        ├── step_0_before.xml
        ├── step_0_before.png
        ├── ...
```

### Final tips (explicitly listed)

- Use keyboard shortcuts for consistent cross-resolution support.
- Use `pg.hotkey("ctrl", "shift", "s")` for Save As.
- Use `pg.typewrite()` and `pg.press()` instead of clicking on text fields.
- Preview every screenshot carefully before each step.
- Retry if evaluation fails even after “seemingly correct steps”.

### Colab notebook note (explicit)

- Script produces a formatted Colab with logs.
- Script provides only a skeleton notebook.
- You must convert it into Assistant/User conversation form, including “thought process” and response as if an LLM generated them.

---

## Final packaging & submission (exact structure described)

### Directory name must be the task UUID

Create directory named:

```text
<task_id>
```

Example in guide:

- task id: `e0eedf6d-9dc7-11f0-a98b-17892f803206`
- directory name: `e0eedf6d-9dc7-11f0-a98b-17892f803206`

### Inside this directory, include

The guide’s structure description (keep exact intent):

```text
<task_id>/
├── <task_id>.json
├── SFT/
├── Colab/
│   └── osw.manual_task.<timestamp>.ipynb
└── Trajectory and Screenshot/
    ├── trajectory.jsonl
    ├── evaluation_score.txt
    ├── recording.mp4 (Optional)
    ├── manual-<timestamp>.log
    ├── step_N_before.png
    ├── step_N_before.xml
```

Checklist items explicitly mentioned:

- Files are not zipped inside subfolders.
- Verify presence of all `result.txt` and runtime logs (guide mentions `runtime.log`).
- Follow folder naming structure exactly.

Pro tips repeated:

- Reset VM snapshot before each run (guide says “done automatically during SFT runs”, but also instructs manual reset earlier).
- Evaluator correctness is critical; invalid or missing `result.txt` can cause rejection.
- “Annotator trajectories are critical for final acceptance” and guide says: “Build realistic mistakes!” (include this note even if you do not apply it to golden trajectory).

---

## OSWorld Evaluators documentation (as included in the guide)

Evaluators are functions that assess whether a task completed successfully by comparing actual results with expected outcomes.

They are organized into modules (the guide lists and explains many).

### 1) General evaluators (`general.py`)

- **`exact_match(result, rules)`**
  - Purpose: exact string match
  - Parameters:
    - `result`: actual result string
    - `rules`: dict with `"expected"`
  - Returns: `1.0` if exact match else `0.0`
  - Example:

```json
{
  "func": "exact_match",
  "result": { "type": "vm_command_line", "command": ["echo", "hello"] },
  "expected": { "expected": "hello" }
}
```

- **`fuzzy_match(result, rules)`**
  - Purpose: fuzzy string match (Levenshtein distance)
  - Returns: similarity ratio `0.0` to `1.0`
  - Example:

```json
{
  "func": "fuzzy_match",
  "result": { "type": "vm_command_line", "command": ["cat", "file.txt"] },
  "expected": { "expected": "expected content" }
}
```

- **`check_include_exclude(result, rules)`**
  - Purpose: contains required strings; excludes forbidden strings
  - Rules: `"include"` and `"exclude"` lists
  - Returns: `1.0` if all includes are present and none of excludes appear, else `0.0`
  - Example:

```json
{
  "func": "check_include_exclude",
  "result": { "type": "vm_file", "path": "/home/user/output.txt" },
  "expected": { "include": ["success", "completed"], "exclude": ["error", "failed"] }
}
```

- **`check_csv(result, rules)`**
  - Purpose: validate CSV content
  - Rules: `"expect"` and `"unexpect"` lists of record patterns

- **`check_json(result, rules)`**
  - Purpose: validate JSON file content
  - Example shape includes `"key"` path arrays and `"method"` such as `eq`/`exists`

- **`check_accessibility_tree(result, rules, osname)`**
  - Purpose: validate accessibility tree structure
  - `osname`: `"ubuntu"`, `"windows"`, `"macos"`
  - Example:

```json
{
  "func": "check_accessibility_tree",
  "result": { "type": "accessibility_tree" },
  "expected": [
    { "selectors": ["button"], "text": "Save", "exact": true }
  ]
}
```

- **`file_contains(file_path, config)`**
  - Purpose: check file contains specified strings

- **`compare_terminal_and_txt(txt_file_path, terminal_output)`**
  - Purpose: compare terminal output with expected text file

- **`compare_python_pure_text(py_file_path, gold_file_path)`**
  - Purpose: compare Python files ignoring metadata/formatting

### 2) Table evaluators (`table.py`)

- **`compare_table(result, expected, **options)`**
  - Purpose: Excel/CSV comparison
  - Supported rule types (as listed): `sheet_name`, `sheet_data`, `sheet_print`, `sheet_fuzzy`, `sparkline`, `chart`, `style`, `freeze`, `zoom`, `data_validation`, `row_props`, `col_props`, `filter`, `pivot_table`, `check_cell`
  - Returns: `1.0` if all rules pass else `0.0`

- **`compare_csv(result, expected, **options)`**
  - Purpose: compare CSVs

- **`compare_conference_city_in_order(actual_city_list_path, expected_city)`**
  - Purpose: validate ordering

### 3) Chrome evaluators (`chrome.py`)

- **`is_expected_tabs(open_tabs, rule)`** (type: URL matching)
- **`is_expected_active_tab(active_tab_info, rule)`**
- **`is_expected_bookmarks(bookmarks, rule)`** (types include bookmark bar folders/websites/liked authors)
- **`compare_pdfs(pdf1_path, pdf2_path)`** (text similarity)
- **`compare_htmls(html_path1, html_path2, **options)`**
- **`is_cookie_deleted(cookie_data, rule)`**
- **`is_shortcut_on_desktop(shortcuts, rule)`**

### 4) VS Code evaluators (`vscode.py`)

- `check_json_keybindings(actual, expected)`
- `check_json_settings(actual, expected)`
- `compare_text_file(actual, expected, **options)`
- `check_json_settings(actual, expected)` (guide also shows an underscore-variant name in text; keep awareness of naming differences)
- `is_extension_installed(actual, rules)` (example uses `code --list-extensions`)
- `check_python_file_by_test_suite(actual_files, test_file, **options)`
- `check_python_file_by_gold_file(...)` (guide says “stub / not implemented yet”)

### 5) Document evaluators (`docs.py`)

- `compare_docx_files(file1, file2, **options)`
- `contains_page_break(docx_file, rules)`
- `find_default_font(config_file_path, rules)`

### 6) GIMP evaluators (`gimp.py`) — relevant to this assessment

Functions listed in the guide include:

- `compare_image_list(pred_img_path_list, gold_img_path_list)` (exact pixel match)
- `check_file_exists(directory, filename)`
- `increase_saturation(image1_path, image2_path)`
- `decrease_brightness(image1_path, image2_path)`
- `compare_triangle_positions(image1, image2)`
- `check_brightness_decrease_and_structure_sim(src_path, tgt_path)`
- `check_saturation_increase_and_structure_sim(src_path, tgt_path)`
- `check_file_exists_and_structure_sim(src_path, tgt_path)`
- `check_triangle_position(tgt_path)`
- `check_structure_sim(src_path, tgt_path)`
- `check_structure_sim_resized(src_path, tgt_path)`
- `check_contrast_increase_and_structure_sim(src_path, tgt_path)`
- `check_config_status(actual_config_path, rule)`
- `check_image_size(src_path, rule)`
- `check_palette_and_structure_sim(src_path, tgt_path)`
- `check_textbox_on_leftside(src_path)`
- `check_image_mirror(src_path, tgt_path)`
- `check_green_background(src_path, tgt_path)`
- `check_sharper(src_path, tgt_path)`
- `check_image_file_size(src_path, rule)`

### 7) VLC evaluators (`vlc.py`)

Functions listed include:

- `is_vlc_playing(actual_status_path, rule)`
- `is_vlc_recordings_folder(actual_config_path, rule)`
- `is_vlc_fullscreen(actual_window_size, screen_size)`
- `compare_images(image1_path, image2_path, **options)` (SSIM)
- `compare_audios(audio_path_1, audio_path_2)` (MFCC/DTW)
- `compare_videos(video_path1, video_path2, **options)`
- `check_qt_bgcone(actual_config_path, rule)`
- `check_qt_max_volume(actual_config_path, rule)`
- `check_qt_minimal_view(actual_config_path, rule)`
- `check_qt_slider_colours(actual_config_path, rule)`
- `check_global_key_play_pause(actual_config_path, rule)`
- `check_one_instance_when_started_from_file(actual_config_path, rule)`
- `check_libre_locale(config_file, rules)`

### 8) Slides evaluators (`slides.py`)

- `compare_pptx_files(file1, file2, **options)`
- `check_slide_numbers_color(pptx_file, expected_color)`
- `check_presenter_console_disable(config_file_path)`
- `check_image_stretch_and_center(modified_ppt, original_ppt)`

### 9) Thunderbird evaluators (`thunderbird.py`)

- `check_thunderbird_prefs(result, rule)`
- `check_thunderbird_filter(result, rules)`
- `check_thunderbird_folder(result, reference, **kwargs)`

### 10) PDF evaluators (`pdf.py`)

- `check_pdf_pages(pdf_file, rules)`
- `extract_answers_from_pdf(pdf_file)`

### 11) Basic OS evaluators (`basic_os.py`)

- `check_gnome_favorite_apps(apps_str, rule)`
- `is_utc_0(timedatectl_output)`
- `check_text_enlarged(scaling_factor_str)`
- `check_moved_jpgs(directory_list, rule)`
- `is_in_vm_clickboard(config, terminal_output)` (note: guide spells “clickboard”)

### 12) Other evaluators (`others.py`)

- `compare_epub(result, expected)`
- `check_mp3_meta(result, meta)`

### Special evaluator

- `infeasible()` marks task infeasible; always failure.

### Evaluator usage in JSON configuration (shape rules)

Guide states evaluators appear in the `evaluator` field, e.g.:

```json
{
  "evaluator": {
    "func": "exact_match",
    "result": {
      "type": "vm_command_line",
      "command": ["echo", "hello world"]
    },
    "expected": {
      "expected": "hello world"
    }
  }
}
```

For more complex evaluators (example `compare_table`), the guide shows `result` and `expected` file objects and `options.rules`.

### Result types (as listed)

- `vm_file`: file on virtual machine
- `vm_command_line`: output of a command
- `cloud_file`: file from cloud storage
- `accessibility_tree`: accessibility XML
- `open_tabs`: Chrome open tabs
- `active_tab`: active tab info

### Expected types (as listed)

- `rule`: rule-based expectations
- `cloud_file`: expected file from cloud
- `vm_file`: expected file on VM

### Notes (as listed)

- evaluators return float in `[0.0, 1.0]` where `1.0` indicates success
- some evaluators support additional options
- file paths should be absolute
- some evaluators require libraries (see repo README)

---

## PyAutoGUI mouse controls (pg) — full cheat list included in the guide

```python
# --- Movement ---
pg.moveTo(x, y, duration=1)        # Move to absolute screen position
pg.moveRel(dx, dy, duration=1)     # Move relative to current position
pg.position()                      # Get current mouse coordinates
pg.size()                          # Get screen resolution

# --- Clicking ---
pg.click(x, y)                     # Left click (optionally at x,y)
pg.doubleClick(x, y)               # Double left click
pg.rightClick(x, y)                # Right click
pg.middleClick(x, y)               # Middle click
pg.tripleClick(x, y)               # Triple click

# --- Press & Release ---
pg.mouseDown(x, y, button='left')  # Press (hold down) mouse button
pg.mouseUp(x, y, button='left')    # Release mouse button
pg.click(button='right')           # Explicit button argument

# --- Dragging ---
pg.dragTo(x, y, duration=1, button='left')    # Drag to absolute coords
pg.dragRel(dx, dy, duration=1, button='left') # Drag relative to pos
pg.drag(xOffset, yOffset, duration=1)         # Alias for dragRel()

# --- Scrolling ---
pg.scroll(500)                     # Vertical scroll (+up / -down)
pg.vscroll(-200)                   # Vertical scroll (explicit)
pg.hscroll(200)                    # Horizontal scroll
```

---

## PyAutoGUI keyboard controls (pg) — full cheat list included in the guide

```python
# --- Typing ---
pg.write("Hello World", interval=0.1)  # Type text with delay
pg.typewrite("Hello")                 # Alias for write()

# --- Single Key Press ---
pg.press("enter")                     # Press one key
pg.press(["left", "backspace", "enter"])  # Press sequence of keys

# --- Hold & Release ---
pg.keyDown("shift")                   # Hold down a key
pg.keyUp("shift")                     # Release key

# --- Combos ---
pg.hotkey("ctrl", "c")                # Combo (copy)
pg.hotkey("ctrl", "shift", "esc")     # Multi-key combo

# --- Arrow Keys ---
pg.press("up")
pg.press("down")
pg.press("left")
pg.press("right")

# --- Navigation Keys ---
pg.press("tab")
pg.press("esc")
pg.press("home")
pg.press("end")
pg.press("pageup")
pg.press("pagedown")

# --- Editing Keys ---
pg.press("backspace")
pg.press("delete")
pg.press("insert")
pg.press("space")
pg.press("enter")

# --- Function Keys ---
pg.press("f1")
pg.press("f5")
pg.press("f12")

# --- Modifier Keys ---
pg.press("shift")
pg.press("ctrl")
pg.press("alt")
pg.press("win")
pg.press("command")                  # macOS

# --- Lock Keys ---
pg.press("capslock")
pg.press("numlock")
pg.press("scrolllock")
```

---

## Task-specific action plan for THIS assessment (GIMP grayscale export)

This section is still “non-summarized”: it spells out all required choices you must encode in the JSON and in the manual trajectory, while staying consistent with the guide’s required schema/fields and evaluator options.

### A) Prepare task files (you need at least the input file; you may also need a golden file)

- Ensure `photo.png` is available for download or already present in the VM snapshot.
  - If you are responsible for providing `photo.png`, host it (Hugging Face recommended in guide) and download it to:
    - **VM path**: `/home/user/Desktop/photo.png`

- Decide if your evaluator will require a golden file:
  - If using **pixel-exact** comparison (`compare_image_list`), you must provide a **golden grayscale image** with identical export settings, and download it somewhere accessible to evaluator.
  - If using existence/structure similarity evaluators, you may only need to ensure output exists and is grayscale-like by structure checks (note: the guide does not define a grayscale-specific evaluator; you must choose among what exists in `gimp.py`).

### B) Create a UUID (task id)

- Generate a UUID (format: `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`).
- This UUID must be used consistently:
  - JSON `id`
  - JSON filename `<task_id>.json`
  - directory name for final submission `<task_id>/`
  - SFT output folder `SFT/<domain>/<task_id>/...`

### C) Create Task JSON with required fields

Minimum required by guide (repeat here because it is disqualifying if missing):

- `id`, `snapshot`, `instruction`, `source`, `trajectory`, `config`, `related_apps`, `evaluator`, `proxy`, `fixed_ip`, `possibility_of_env_change`, `model_pass_rate`, `annotator_hints`, `knowledge_points`, `coverage`

Required values for this assessment:

- `snapshot`: **must be `gimp`**
- `instruction`: must match assessment instruction exactly (convert `photo.png` to black and white, export as `gray_photo.png` on Desktop)
- `related_apps`: include `gimp`

`config` for this task typically includes (use the guide’s `launch` + `activate_window` pattern):

- `download` (if you need to place `photo.png` on Desktop)
- `launch` GIMP (command depends on snapshot; commonly `gimp` on Linux)
- `activate_window` for the GIMP window title (must match what the VM shows)

### D) Validate Task JSON using validation script

- Put `<task_id>.json` into `Deliverable/`.
- Run:

```bash
python Turing_tooling/validation_script.py Deliverable <task_id>
```

- Confirm you see the “PASSED” message.

### E) Record the golden SFT trajectory

Use run_manual tool with:

- provider: `vmware`
- observation type: `screenshot_a11y_tree`
- task file: your JSON path
- result dir: `SFT`

Then, inside the interactive session, perform the full GUI workflow in GIMP to:

- Open `photo.png`
- Convert to grayscale / black-and-white
- Export as `gray_photo.png` to Desktop

Important implementation note:

- The guide strongly encourages keyboard shortcuts for reliability.
- For export, you may need “Export As…” (often `Ctrl+Shift+E` in GIMP) or menu navigation; use whichever is stable in the snapshot and confirmed by screenshots.
- Every click/keypress must be entered as `pg.*` actions.
- After finishing, type `done` and confirm evaluator passes.

### F) Package final submission folder exactly as required

- Create folder named `<task_id>/`
- Put `<task_id>.json` at top-level
- Copy SFT outputs and Colab outputs into the structure shown in “Final packaging & submission”.

---

## End of actions.md


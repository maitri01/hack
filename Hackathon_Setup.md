# Hackathon Setup (Paris)

## Hackathon Environment Guide

This guide explains how to set up and use the shared environment on JURECA.

**Important: Always update the project ID before sharing the scripts and this guide.** For example, `training2666`.

The scripts currently use `training2666`. Replace this with the correct project ID in **both scripts** and in the commands below if your assigned project is different.

### Setup scripts

- [hackathon_setup.sh](https://huggingface.co/datasets/SprintML/hackathon/resolve/main/hackathon_setup.sh)
- [teammate.sh](https://huggingface.co/datasets/SprintML/hackathon/resolve/main/teammate.sh)

Instructions for downloading these files are provided below.

## 1. Decide roles and execution order

Before starting, decide who will be the **Team Lead** and who will be the **Teammates**.

**The order is important:**

1. **Team Lead runs first:** Configures and runs `hackathon_setup.sh` to create the shared environment.
2. **Teammates run afterward:** Once the team lead finishes successfully, every remaining member individually configures and runs `teammate.sh` from their own JURECA account.

The team lead runs only `hackathon_setup.sh`. Every other member runs only `teammate.sh`.

**Do not run both scripts from the same account.**

## 2. Team Lead setup (run first)

### Step 1: Download the scripts

Log in to JURECA using your own Jülich username.

Download the scripts into a directory in your home folder:

```bash
mkdir -p ~/hackathon-scripts
cd ~/hackathon-scripts

wget https://huggingface.co/datasets/SprintML/hackathon/resolve/main/hackathon_setup.sh -O hackathon_setup.sh

wget https://huggingface.co/datasets/SprintML/hackathon/resolve/main/teammate.sh -O teammate.sh
```

You do not need to manually create your folder in scratch. The setup script creates it automatically.

### Step 2: Edit the configuration

Open `hackathon_setup.sh` and update the configuration:

```bash
TEAM_LEAD="<your-juelich-username>"

TEAMMATE_1="<teammate1-username>"
TEAMMATE_2="<teammate2-username>"
TEAMMATE_3="<teammate3-username>"

TEAM_FOLDER="<your-team-name>"
```

- `TEAM_LEAD`: Your Jülich username.
- `TEAMMATE_1`, `TEAMMATE_2`, `TEAMMATE_3`: The Jülich usernames of your teammates.
- Leave unused teammate entries as `""`.
- `TEAM_FOLDER`: The name of the shared team folder. All teammates must use exactly the same name.

For example, a team with one lead and two teammates should leave `TEAMMATE_3=""`.

**Do not leave `TEAM_LEAD` or `TEAM_FOLDER` empty. At least one teammate must be specified.**

### Step 3: Run the setup

Only the team lead runs:

```bash
source hackathon_setup.sh
```

Allow the setup to finish successfully before teammates proceed.

### What this script does

- Activates the assigned JURECA project.
- Creates the team lead's folder in `/p/scratch/training2625/`.
- Creates the shared team folder inside it.
- Sets ACL permissions for the team lead and listed teammates.
- Downloads all task datasets from Hugging Face.
- Creates per-task Python environments (`.venv`).
- Configures shared uv and Hugging Face cache directories.

The shared folder will be located at:

```text
/p/scratch/training2625/<team-lead-username>/<team-name>/
```

**Once the team lead's setup is complete, all remaining team members can proceed to Section 3.**

## 3. Teammate setup (run after Team Lead)

Every teammate must complete these steps individually using their own JURECA account.

**Do not run this script before the team lead finishes Section 2.**

### Step 1: Download the teammate script

```bash
mkdir -p ~/hackathon-scripts
cd ~/hackathon-scripts

wget https://huggingface.co/datasets/SprintML/hackathon/resolve/main/teammate.sh -O teammate.sh
```

### Step 2: Edit the configuration

Open `teammate.sh` and fill in:

```bash
TEAM_LEAD="<team-lead-username>"
TEAM_FOLDER="<team-name>"
```

- `TEAM_LEAD`: The Jülich username of your team lead.
- `TEAM_FOLDER`: Exactly the same team name used by the lead in `hackathon_setup.sh`.

Do not enter your own username as `TEAM_LEAD` unless you are actually the team lead.

### Step 3: Run the script

```bash
source teammate.sh
```

Every teammate must run this separately.

### What this script does

- Activates the assigned JURECA project.
- Verifies access to the shared team folder.
- Configures shared uv and Hugging Face cache paths.
- Installs `uv` for the current user if missing.
- Saves the environment configuration in the user's `.bashrc`.

**Teammates do not need to download the datasets or create the environments again.** These are shared with the team.

## 4. Working on tasks

All team members work inside the same shared team folder.

Go to a task directory:

```bash
cd /p/scratch/training2625/<team-lead>/<team-folder>/<dataset-name>
```

Activate the task's Python environment:

```bash
source .venv/bin/activate
```

Run your Python code:

```bash
python main.py
```

Alternatively, if the task's uv project is configured:

```bash
uv run python main.py
```

Use the actual filename instead of `main.py` if your script has a different name.

## 5. Every new session

When starting a new JURECA session, activate the project:

```bash
jutil env activate -p training2625
```

The shared cache configuration is written to `.bashrc` during setup.

If the configuration is not loaded, run:

```bash
source ~/.bashrc
```

Then navigate to your task directory and activate its environment if needed.

You do not need to rerun the initial setup scripts for every session.

## 6. Folder structure

Example folder structure after setup:

```text
/p/scratch/training2625/
└── <team-lead-username>/
    └── <team-folder>/
        ├── MGI/
        │   ├── .venv/
        │   ├── requirements.txt
        │   └── output/
        ├── FL_Data_Reconstruction/
        │   ├── .venv/
        │   ├── requirements.txt
        │   └── output/
        ├── watermark_localization/
        │   ├── .venv/
        │   ├── requirements.txt
        │   └── output/
        ├── .uv/
        └── .cache/
```

The task folders contain the downloaded datasets, code, and any other provided files.

All team members with the required permissions can work inside the shared folder.

## 7. Adding a new teammate later

If your team has already completed setup and another member joins later, the team lead must grant access to the new member.

**Do not ask everyone to rerun their scripts.**

The team lead can update permissions for the existing folder using:

```bash
TEAM_LEAD="<team-lead-username>"
NEW_MEMBER="<new-member-username>"

TEAM_LEAD_DIR="/p/scratch/training2625/$TEAM_LEAD"

# Grant access to existing files and folders
setfacl -R -m "u:$NEW_MEMBER:rwx" "$TEAM_LEAD_DIR"

# Grant default access to future files and folders
find "$TEAM_LEAD_DIR" -type d \
    -exec setfacl -d -m "u:$NEW_MEMBER:rwx" {} +
```

This provides the new teammate with full access to the existing directory tree and default permissions on future files. It should be run only by the team lead and only when such access is intended.

After the permissions have been added, the new teammate follows **Section 3** and runs:

```bash
source teammate.sh
```

Existing teammates do not need to repeat their setup.

## 8. Rules

- Do not change shared folder permissions manually unless you are the team lead managing access.
- Do not delete shared folders or datasets.
- Do not recreate `.venv` unless necessary.
- Avoid installing unnecessary or conflicting packages into shared environments.
- Keep task-related work inside the shared team folder.
- Store generated outputs and logs in the appropriate task's `output/` directory.
- Coordinate changes to shared code and environments with your teammates.

## 9. Submitting GPU jobs

To run your code on a GPU, create a Slurm job script called `main.sh` inside your task directory.

Example:

```bash
#!/bin/bash
#SBATCH --account=training2625
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --gres=gpu:1
#SBATCH --reservation=cispa-hack
#SBATCH --cpus-per-task=8
#SBATCH --partition=dc-gpu
#SBATCH --output=output/%j.out
#SBATCH --error=output/%j.err

# Go to the task directory
cd /p/scratch/training2625/<team-lead>/<team-folder>/<dataset-name>

# Activate the task environment
source .venv/bin/activate

# Run your Python code
srun python code.py

echo "done!"
```

Replace the placeholders with your actual folder names and Python filename.

The example Slurm account, partition, reservation, and GPU settings must match the resources assigned for the hackathon.

Make sure the task's `output/` directory exists before submitting.

Submit the job:

```bash
sbatch main.sh
```

Check submitted jobs:

```bash
squeue -u $USER
```

The job logs will be stored in the task's `output/` folder.

## 10. Common issues

### Cannot access the shared folder

Check that:

- You entered the correct `TEAM_LEAD` and `TEAM_FOLDER`.
- The team lead completed the initial setup.
- Your Jülich username was included in the team lead's permissions.

If you joined after the initial setup, ask the team lead to follow Section 7.

### uv not found

Install uv using:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
source ~/.local/bin/env
```

### Python

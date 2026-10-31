# WSL + Ubuntu + Miniforge setup (Windows)

Linux environment running inside Windows, no VM or dual boot. Needed when a project
(conda envs, Snakemake, bash scripts) assumes Linux.

## 1. Install WSL with Ubuntu

PowerShell **as Administrator**:

```powershell
wsl --install -d Ubuntu
```

Restart the PC. Then open **Ubuntu** from the Start menu once and set a Linux username + password.

Check:

```powershell
wsl -l -v
# NAME      STATE     VERSION
# Ubuntu    Running   2
```

> **Gotchas**
> - Having WSL enabled is not enough. Without a distribution there is no Linux terminal.
> - The Ubuntu ISO from ubuntu.com is the full OS, **not** for WSL. Use the command above or the Microsoft Store app.
> - Explorer's "Linux" folder stays empty until Ubuntu is installed **and** started once.
> - If install fails: enable virtualization in BIOS, update Windows.

## 2. Install Miniforge (inside Ubuntu)

`$(uname)` and `$(uname -m)` fill in your OS and CPU architecture automatically. To see what they resolve to:

```bash
uname         # Linux
uname -m      # x86_64  (most PCs; ARM machines show aarch64)
echo "Miniforge3-$(uname)-$(uname -m).sh"   # → Miniforge3-Linux-x86_64.sh
```

Download and run:

```bash
wget "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh"
bash Miniforge3-$(uname)-$(uname -m).sh
```

Enter through the license, `yes`, accept default path, `yes` to initialize conda.
If it asks for the shell name: `bash` (check with `echo $SHELL`).

## 3. Make `conda` available

If `conda: command not found` after install:

```bash
~/miniforge3/bin/conda init bash
source ~/.bashrc
conda --version        # prompt should now start with (base)
```

Still not found?
- `ls ~/miniforge3/bin/conda` → missing = installer didn't finish, rerun it
- close the terminal completely and open a new one
- make sure you're in **Ubuntu**, not Git Bash / PowerShell

## 4. Build the project env

```bash
conda env create -f environment.yml
conda activate envname
```

Share envs as a file, never the env folder itself:

```bash
conda env export --from-history > environment.yml
```

## 5. VS Code in WSL

1. Install extension **WSL** (Microsoft)
2. `Ctrl+Shift+P` → **WSL: Connect to WSL** → open folder
3. Bottom left shows `WSL: Ubuntu`; terminal is now Ubuntu
4. `Ctrl+Shift+P` → **Python: Select Interpreter** → the conda env

Shortcut from the Ubuntu terminal:

```bash
cd ~/projects/repo
code .
```

> **Gotcha:** "bash" in VS Code's terminal dropdown is usually **Git Bash**, not WSL.
> Test: `conda env list` there shows the Windows envs.

## Useful

| Task | Command |
|---|---|
| Open current Linux folder in Explorer | `explorer.exe .` |
| Linux files from Windows | `\\wsl$\Ubuntu\home` |
| List / remove envs | `conda env list` / `conda env remove -n envname` |
| Clean conda cache | `conda clean --all` |
| Run WSL command from PowerShell | `wsl -e bash -lc "conda activate envname && python script.py"` |

- Keep repos in the Linux filesystem (`~/projects/`), not `/mnt/c/`, which is much slower.
- Windows and WSL have **separate** conda installs; envs in one are invisible to the other.

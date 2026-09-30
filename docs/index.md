---
layout: default
title: Launching Jupyter on HPC
---

# Launching Jupyter Notebook on HPC

Two ways to run Jupyter on an HPC compute node. Method 1 gives you full control from the command line. Method 2 is a web portal that handles everything for you.

> **Currently using TACC systems** (Vista, Lonestar6, Stampede3, Frontera). The concepts apply to any HPC system with Slurm and module environments.

**Prerequisites:** An HPC account with MFA set up. If you don't have one, see the [MSF Getting Started guide](https://morehouse-supercomputing.github.io/mscf-getting-started/).

---

# Method 1: From the Terminal (SSH + idev)

You SSH into the HPC system, request a compute node, launch Jupyter, and tunnel it back to your laptop's browser.

---

## Step 1: SSH into the System

Open a terminal on your laptop:

```bash
ssh your_username@vista.tacc.utexas.edu
```

Enter your password and MFA token when prompted.

> Nothing shows on screen when you type your password. That's normal. Just type and press Enter.

**Windows users:** Use PowerShell or [PuTTY](https://www.putty.org/).

> **Other systems:** Replace `vista` with `ls6` (Lonestar6), `stampede3`, or `frontera` depending on your allocation.

---

## Step 2: Request a Compute Node

You're on the **login node**. Do not run heavy code here. Request a dedicated compute node using `idev`:

```bash
idev -p gh-dev -N 1 -n 1 -t 01:00:00 -A YOUR_ALLOCATION -r RESERVATION_NAME
```

> **No reservation?** If you weren't given one, just leave the `-r RESERVATION_NAME` off the end and run the rest of the command as-is.

| Flag | Meaning |
|------|---------|
| `-p gh-dev` | Queue name (varies by system — see table below) |
| `-N 1` | 1 node |
| `-n 1` | 1 task |
| `-t 01:00:00` | Time limit (1 hour) |
| `-A YOUR_ALLOCATION` | Your project allocation code |
| `-r RESERVATION_NAME` | *(optional)* A reservation you've been given |

> **Don't know your allocation name?** On the login node, run `/usr/local/etc/taccinfo` to see your projects and allocation codes.

> **Using a reservation?** If the facility gave you a reservation (for a workshop or class), add `-r` with its name. Use a lowercase `-r`, and make sure `-p` matches the queue the reservation was created on:
>
> ```bash
> idev -p gh -N 1 -n 1 -t 01:00:00 -A YOUR_ALLOCATION -r RESERVATION_NAME
> ```
>
> If the reservation is on the `gh` queue but you request `gh-dev` (or the reverse), the job fails with `Requested node configuration is not available`. When in doubt, ask whoever gave you the reservation which queue it's on.

Wait for the node to be assigned. Your prompt will change to something like:

```
c642-081[gh](360)$
```

That node name (`c642-081` in this example) is important. Write it down.

> **Common queues by system:**
>
> | System | CPU Queue | GPU Queue |
> |--------|-----------|-----------|
> | Vista | `gg-dev` | `gh-dev` |
> | Stampede3 | `skx-dev` | `gpu-a100-dev` |
> | Lonestar6 | `development` | `gpu-a100-dev` |

---

## Step 3: Load Modules and Start Jupyter

Once on the compute node:

```bash
module load gcc/13.2.0
module load python3
jupyter notebook --ip=0.0.0.0 --no-browser
```

Jupyter will print output including a URL like:

```
http://c642-081.vista.tacc.utexas.edu:8888/tree?token=abc123def456...
```

**Copy the full token** (the part after `token=`). You'll need it.

> **If `jupyter` is not found:** Try `module spider jupyter` to find the right module, or use `pip install --user jupyter` to install it.

---

## Step 4: Create an SSH Tunnel

**Keep the first terminal running.** Open a **new terminal window** on your laptop and run:

```bash
ssh -N -L 8888:NODE_NAME:8888 your_username@vista.tacc.utexas.edu
```

Replace `NODE_NAME` with your actual compute node (e.g., `c642-081`).

You'll need to enter your password and MFA token again. After that, the terminal will appear to hang with no output. That's correct -- it's forwarding the connection in the background.

> **What this command does:** It creates a secure tunnel from port 8888 on your laptop to port 8888 on the compute node, going through the login node. This lets your browser talk to Jupyter running on the compute node.

---

## Step 5: Open Jupyter in Your Browser

Open your browser and go to:

```
http://localhost:8888
```

If prompted for a token, paste the token from Step 3.

You should see JupyterLab or the Jupyter file browser. You're now running Python on an HPC compute node.

---

## Step 6: When You're Done

1. **Save your work** (Ctrl+S in Jupyter)
2. Close the browser tab
3. In the first terminal, press **Ctrl+C** to stop Jupyter
4. Type `exit` to leave the compute node
5. Close the SSH tunnel terminal (Ctrl+C)

> Your files in `$WORK` persist between sessions. You won't lose anything when the job ends.

---

# Method 2: Analysis Portal (Web Browser)

Everything happens in your browser. No SSH, no tunnels.

---

## Step 1: Log into the Portal

Go to [tap.tacc.utexas.edu](https://tap.tacc.utexas.edu) and log in with your username, password, and MFA token.

---

## Step 2: Submit a Jupyter Job

Fill in the form on the left side of the dashboard:

| Field | What to Enter |
|-------|--------------|
| **System** | Your system (e.g., `Vista`, `Stampede3`) |
| **Application** | `Jupyter notebook` |
| **Project** | Your allocation code |
| **Queue** | A dev queue (e.g., `gh-dev` on Vista, `skx-dev` on Stampede3) |
| **Nodes** | `1` |
| **Tasks** | `1` |
| **Job Name** | Anything descriptive |
| **Time Limit** | `02:00:00` (2 hours max on dev queues) |
| **Reservation** | Leave blank, *unless* you were given a reservation name (for a workshop or class) — then enter it here |
| **VNC Resolution** | Leave blank |

Click **Submit**.

---

## Step 3: Connect

Your job appears under **Current Jobs** on the right side. Wait for it to switch from **Pending** to **Running**, then click **Connect**.

JupyterLab opens in your browser. You're on a compute node. Done.

---

## Step 4: When You're Done

Click **End** next to your job on the dashboard, or just close the tab and let the timer run out.

---

# Working with Files

## Where to Put Your Code

| Location | Size | Backed Up? | Purged? | Use For |
|----------|------|-----------|---------|---------|
| `$HOME` | ~10 GB | Yes | No | Config files, scripts |
| `$WORK` | ~1 TB | No | No | Code, repos, datasets |
| `$SCRATCH` | Unlimited | No | Yes (10 days) | Job output, temp files |

**Put your repos and data in `$WORK`.** It has plenty of space and doesn't get purged.

```bash
cd $WORK
git clone https://github.com/your-repo.git
```

## Installing Python Packages

The system provides Python via modules, but you can install additional packages with `--user`:

```bash
pip install --user package_name
```

These install to `$HOME/.local` and persist between sessions.

---

# Troubleshooting

**"idev is taking forever"**
- The queue might be full. Try a shorter time: `-t 00:30:00`
- Check system status on the portal -- if utilization is 100%, wait or try a different queue

**"SSH tunnel gives 'Name or service not known'"**
- You typed the node name wrong. Check your idev terminal for the actual node (e.g., `c642-081`)

**"localhost:8888 won't load"**
- Make sure the SSH tunnel terminal is still running
- Make sure Jupyter is still running in the first terminal
- Try `localhost:8888/tree` instead of just `localhost:8888`

**"I can't find my files in Jupyter"**
- Jupyter opens in whatever directory you launched it from
- Navigate to `$WORK` using the file browser, or restart Jupyter from `$WORK`:
  ```bash
  cd $WORK
  jupyter notebook --ip=0.0.0.0 --no-browser
  ```

**"Module not found" in a notebook cell**
- Run `!module load python3` in a cell, or load modules before starting Jupyter

**"My session died"**
- You hit the time limit. Resubmit with a longer `-t` value
- Save frequently -- `$WORK` files survive between sessions

---

# Quick Reference

| Task | Command |
|------|---------|
| SSH in | `ssh user@vista.tacc.utexas.edu` |
| Request a compute node | `idev -p gh-dev -N 1 -n 1 -t 01:00:00 -A ALLOCATION` |
| Request a node in a reservation | `idev -p QUEUE -N 1 -n 1 -t 01:00:00 -A ALLOCATION -r RESERVATION` |
| Load Python | `module load gcc/13.2.0 && module load python3` |
| Start Jupyter | `jupyter notebook --ip=0.0.0.0 --no-browser` |
| SSH tunnel (second terminal) | `ssh -N -L 8888:NODE:8888 user@vista.tacc.utexas.edu` |
| Open in browser | `http://localhost:8888` |
| Analysis Portal | [tap.tacc.utexas.edu](https://tap.tacc.utexas.edu) |
| Check your node name | Look at your terminal prompt or run `hostname` |
| End idev session | `exit` |

---

# Getting Help

- **System Documentation:** [docs.tacc.utexas.edu](https://docs.tacc.utexas.edu/)
- **Support Ticket:** [portal.tacc.utexas.edu/tacc-consulting](https://portal.tacc.utexas.edu/tacc-consulting)
- **MSF Questions:** [ashley.scruse@morehouse.edu](mailto:ashley.scruse@morehouse.edu)
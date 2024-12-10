# CMU_Babel_Cluster_Setup

## Introduction

[Babel](https://hpc.lti.cs.cmu.edu/wiki/index.php?title=Main_Page)
is a high-performance computing cluster at
[Carnegie Mellon University](https://www.cmu.edu/).
This tutorial aims to
help users onboard the cluster and start running jobs with minimal efforts.

## Disclaimer

Due to its minimalistic nature, the content in this tutorial
does not necessarily reflect state-of-the-art empirical best practices.
Moreover, some content may become outdated.
However, feel free to send a GitHub Issue or pull request
if you'd like to update it!
(For example, feel free to suggest some useful commands.)

Some pages referenced in this tutorial require CMU login.
Therefore, this tutorial is the most suitable for CMU-affiliated readers.

Since we are not the admins of this cluster,
we can neither grant you access to the cluster,
nor help you debug.
Please reach out to the admins of this cluster if you need such help.

## Login

```commandline
ssh <andrew_id>@babel.lti.cs.cmu.edu
```
where `<andrew_id>` should be replaced by your Andrew ID.

More info:
[Connecting to the Cluster](https://hpc.lti.cs.cmu.edu/wiki/index.php?title=Connecting_to_the_Cluster)

## Virtual environment
Different projects may have conflicting package dependencies.
For example, some codes require `python=3.9` whereas others require `python=3.10`. 

Use 
[Miniconda](https://docs.anaconda.com/miniconda/)
to manage virtual environments.
In your home directory, 
run the following to install Miniconda
```commandline
mkdir -p ~/miniconda3
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda3/miniconda.sh
bash ~/miniconda3/miniconda.sh -b -u -p ~/miniconda3
rm ~/miniconda3/miniconda.sh
```
Then run
```commandline
~/miniconda3/bin/conda init bash
~/miniconda3/bin/conda init zsh
```
For these to take effect,
log out of the server, and log in again.

Now you can create a virtual environment:
```commandline
conda create -n <virtual_env_name> python=3.10
```
where `<virtual_env_name>` should be replaced by
a string representing the name of the virtual environment.
Feel free to use a different Python version based on your codes and their dependencies.

## Install packages

To avoid conflicts in package dependencies (mentioned in the above section),
I recommend always activating a virtual environment before installing packages.

```commandline
conda activate <virtual_env_name>
```
Then, list your set of package requirements in a file `requirements.txt`.
If you work on machine learning, language models, or related areas,
feel free to use my [requirements.txt](./requirements.txt).
Finally, install these packages by running
```commandline
pip install -r requirements.txt
```

## Submit jobs to queue

First, write your codes e.g.
[try_gpu.py](./try_gpu.py).

Then create a job script file `myjob.sh` with the following content:
```commandline
#!/bin/bash
#SBATCH --job-name=myjob
#SBATCH --output=myjob.out
#SBATCH --error=myjob.err
#SBATCH --partition=general
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=16G
#SBATCH --time=1:00:00

# Your job commands go here
cd ~/CMU_Babel_Cluster_Setup/  # or project directory
conda init
conda activate <virtual_env_name>
echo "Hello, World!"
python try_gpu.py
```
If you need GPU, add the following line below the above `#SBATCH` commands:
```commandline
#SBATCH --gres=gpu:A6000:1
```
where `A6000` can be replaced by another GPU type.

Then, to submit the job, use the following commands:
```commandline
# NB: In my experience, activating the virtual environment 
# both in `myjob.sh` and in the below command seems to be the most robust.
# Feel free to suggest a simpler recipe that reduces duplication. 
conda activate <virtual_env_name>
sbatch myjob.sh
```
Monitor output and error logs at `myjob.out` and `myjob.err`.

You should be good to go!

More information:
[Slurm](https://hpc.lti.cs.cmu.edu/wiki/index.php?title=Slurm).

## Appendix: Useful commands

### Show job information

```
scontrol show job -d 461721
```

```
squeue --start -j <pid>
```

### Ask for a GPU

```
srun -p debug  --time=2:00:00 --gres gpu:L40:1 -c1 --wait 600000 --mem=18g  --pty $SHELL
```
```
srun -p debug --time=2:00:00 --gres=gpu:A6000:1 -c2 --mem=25g --wait 600000 --pty $SHELL
```
- I think `c2` here refers to the number of cpus that you can ask 
-  `-p` here is for partition.
### Ask for a CPU
```
srun -p cpu --time=10:00:00 -c2 --wait 600000 --mem=10g --pty $SHELL
```

### GPU list
```
python /opt/cluster_tools/babel_contrib/tir_tool/gpu.py 
```
 
### Check max number of GPUs you can request for a specific job type or partition


#### **1. Use `sinfo` to Check Partition Information**
The `sinfo` command displays information about available partitions, including the maximum number of GPUs.

```bash
sinfo -o "%P %G %D %C"
```

- **`%P`**: Partition name.
- **`%G`**: GPUs available in the partition.
- **`%D`**: Number of nodes in the partition.
- **`%C`**: CPU/core counts and their states.

Example output:
```
PARTITION  GPUs   NODES  CPUS(A/I/O/T)
gpu-part   4      10     320/80/0/400
```

Here, the `gpu-part` partition supports up to 4 GPUs per node.

---

#### **2. Check Available Resources with `scontrol show partition`**
Use the `scontrol` command to get detailed information about a specific partition:

```bash
scontrol show partition=<partition_name>
```

Look for the `TRES` (Trackable Resources) field, which shows the maximum number of GPUs available for the partition.

Example:
```
PartitionName=gpu-part
   TotalNodes=10
   DefaultTime=01:00:00
   MaxTime=7-00:00:00
   TRES=gres/gpu:4
```

In this example, the `TRES=gres/gpu:4` indicates a maximum of 4 GPUs per node.

---

#### **3. Use `scontrol show node` for Per-Node Information**
If you want to know how many GPUs are available on each node:

```bash
scontrol show node
```

Search for the `Gres` (Generic Resources) field, which lists GPU availability. Example:
```
NodeName=node01
   Gres=gpu:4
   CPUs=32
   State=IDLE
```

This means the node `node01` has 4 GPUs available.

---

#### **4. Check Job Submission Limits**
Some clusters enforce job-specific limits on GPUs. Check your SLURM configuration or ask your system administrator for policies like:
- Maximum GPUs per user.
- Maximum GPUs per job.
- Limits on specific partitions.

---

#### **5. Submit a Test Job**
If you're unsure about GPU limits, submit a test job requesting the maximum possible GPUs:

```bash
sbatch --gres=gpu:<number> --time=00:10:00 --partition=<partition_name> test_script.sh
```

- Replace `<number>` with the number of GPUs you want to test.
- If your request exceeds the available resources, the job will fail, and the error message will indicate the limit.

---

### Example Workflow:
1. List partitions and GPU availability:
   ```bash
   sinfo -o "%P %G %D %C"
   ```

2. Get detailed information for a specific partition:
   ```bash
   scontrol show partition=gpu-part
   ```

3. Check per-node GPU availability:
   ```bash
   scontrol show node
   ```

4. Verify with the cluster administrator if necessary.



## Appendix: Useful info

### Partitions

General Partitions:
* debug: For testing and debugging, max queue time 2 hours, max 2 GPU, max 1 node.
* general: Default for standard jobs, max time 48 hours, max 8 GPUs, max 2 nodes.
* long Partition: For longer jobs, max time 7 days, max 8 GPUs, max 2 nodes.
* cpu: For CPU-intensive jobs, contains only nodes with older GPUs, max time 48 hours, max 1 node.

Dedicated Group Partitions:
* elysium: For unrestricted usage on dedicated nodes. Pre-approval required. Access limited to approved users/projects.
* admin: For administrative tasks, restricted to administrators.
* high priority partitions for specific nodes:
* inst: high priority queue for members of dmorten_group
* locus: high priority queue for members of locus group
* r3lit : high priority queue for members of mdiab_group
* shire: high priority queue for members of shire group


## Contributors

[Yuchen Li](https://yuchenli01.github.io/)

[Tanya Marwah](https://tm157.github.io/)

We welcome and appreciate your contribution!

In particular,
please suggest more to be added to 
[Appendix: Useful commands](##-Appendix-Useful-commands)
:) 

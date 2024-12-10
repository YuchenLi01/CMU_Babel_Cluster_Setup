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

You can follow this page:
[Slurm](https://hpc.lti.cs.cmu.edu/wiki/index.php?title=Slurm).

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

## Contributors

[Yuchen Li](https://yuchenli01.github.io/)

We welcome and appreciate your contribution!

We plan to add a "useful commands" section.
please suggest some useful commands :) 

# Cluster IT Context — Matrix (UAH HPC Cluster)

Embedded context document mirrored from akd-ext `akd_ext/agents/closed_loop_cm1/context/cluster_it.md`. Used by the Capability & Feasibility Mapper for cluster-fit assessment in Step 5 Part B.

---

Matrix is a Linux cluster designed for single and multi-threaded modeling jobs. It uses the SLURM queuing system. It is IMPORTANT that you do not execute code in the same manner you would on a traditional Linux system. Please read this page thoroughly. System paths are set using the modules software.

## Structure

- OS: Redhat Enterprise 7.9
- 2PB Disk
- Supermicro Xeon Blade Servers
- TBD Nodes
- TBD Processor Cores
- Infiniband high-performance I/O

## How To Use The System

Do NOT run code interactively on Matrix. You must use the SLURM queueing system to run all jobs (even cron jobs). The system is optimized for non-interactive CPU intensive applications. It is recommended that you run graphical applications on another machine.

Review documentation about the directory structure and disk quotas on the system. See the modules Documentation page for information on application usage.

Modify your `.tcshrc.matrix` file if you are using a TCSH shell, or your `.bashrc.matrix` file if you are using the BASH shell. These files are to be located in your top user directory in `/rhome`. Matrix uses a path management utility to configure paths for all of programs used on the system. DO NOT manually set paths as you may have with other Unix machines.

## SLURM

### Where to Begin

- Create a SLURM shell script to call your executable (example scripts in the Matrix documentation).
- Any shell script must start either with `#!/bin/csh` or `#!/bin/bash` depending on your shell. Bash users need to make sure to use `#!/bin/bash` instead of `#!/bin/sh`.
- Use `sbatch <script>` to submit.
- Output is placed in a log file in your working directory: `slurm-<jobid>.out`.
- Errors are placed in `slurm-<jobid>.err`.

### Commands

- `sbatch <slurm script>` — Submit a job for execution
- `scancel <job id>` — Kill a running job
- `squeue -u <username>` — Show queue information about your jobs
- `scontrol show job <job id>` — Show detailed information about a running job
- `sproc` — Show system CPU usage
- `sinfo` — Show info about available queues
- `sacct sstat --format=JobID,JobName,AveRss,MaxRSS,Elapsed -j <jobid>` — Show statistics on a completed job
- `--format=JobID,AveRss,MaxRSS -a -j <jobid>` — Show statistics on a running job

### Queues

Matrix has 3 queues with functionality outlined below. Maximum interactive sessions are limited to 3.

| Name | Max Jobs | Max Procs | Max Total Memory | Max Memory per Node | Notes |
| --- | --- | --- | --- | --- | --- |
| `shared` | 20 | 64 | 400G | 100G | Use for single and low priority multi processor jobs. Nodes are shared with other users. |
| `standard` | 30 | 256 | 1000G | 100G | Nodes are exclusive to users but are subject to pre-emption by the special queue. |
| `special` | 5 | QOS | QOS | 100G | Processors are allocated to special projects and limits are based on a QOS parameter. |

### QOS

The `special` queue sets limits on resources using a Quality of Service (QOS) type. It is assigned to users based on the type of research they are running on the cluster. By default, users do not have access to this partition. Check with your PI or IT for availability.

### Pre-emption

Pre-emption suspends a running job temporarily so that a higher priority process can execute. After that job is finished, the pre-empted job resumes from the point it stopped. On Matrix, jobs running in the `special` queue have the potential to pre-empt `standard` jobs if additional processors are not available.

### GPUs

GPUs are available on select nodes with the `shared` queue. 4 NVIDIA A100 Tensor GPUs (`module load cuda`).

### Scheduling jobs at regular intervals

Configure your SLURM script as usual. Use the command `crontab -e` to edit your configuration file. Make sure the first two lines of the crontab file include the following:

```
BASH_ENV="$HOME/.bashrc"
SHELL="/bin/bash"
```

Add: `min hr day mon weekday sbatch /path/script >/dev/null 2>&1`.

Use `*` for "every" interval (e.g., every hour, every minute).

### Crontab Fields

```
# +---------------- minute (0 - 59)
# | +------------- hour (0 - 23)
# | | +---------- day of month (1 - 31)
# | | | +------- month (1 - 12)
# | | | | +---- day of week (0 - 7) (Sunday=0 or 7)
# | | | | |
# * * * * * command to be executed
```

Example crontab format for executing both CSH and BASH SLURM scripts:

```
BASH_ENV="$HOME/.bashrc"
SHELL=/bin/bash
MAILTO=user@nsstc.uah.edu
30 * * * * sbatch /rstor/scottp/test.csh > /dev/null 2>&1
30 12,13 * * * sbatch /rhome/scottp/test2.bash > /dev/null 2>&1
```

The first job runs every 30 minutes. The second one runs at 12:30pm and 1:30pm every day.

## Applications

Type `module avail` to see a list of available packages.

### Python

Python V2 and V3 are both available with many standard packages by loading the relevant module.

```
module load python/v2
module load python/v3
```

To list installed Python packages (after loading module): `conda list`.

To install additional/custom packages, create your own Python environment as follows:

```
module load python/v3
conda create -n py3 python=3
conda activate py3
conda install <your packages>
```

To activate a custom Python environment the next time you log in or run a SLURM script:

```
module load python/v3
conda activate py3
```

### IDL

When running IDL on Matrix, be sure to use the queuing system. Do not run it interactively. Use a system other than Matrix if you need an interactive graphical display. If you are generating graphical output that would normally output to the screen, you must first run `Xvfb` within the SLURM script to simulate a connection to a display.

### MATLAB

To run a job on Matrix, write a MATLAB script file (`.m`) and then call the script from within a SLURM script. Include `-singleCompThread -nodisplay -nosplash -nojvm` directives with the MATLAB executable to indicate that your code will not be running interactively.

## Interactive Commands

Code should not be interactively executed on Matrix but for certain high CPU commands that must be run on the interactive node, use the `scmd` command to offload the function to a compute node. Note that `scmd` will only run on a single processor. Examples:

```
scmd tar czf test.tgz testdir
scmd ftp ftp.uah.edu
scmd scp testdir aqua:/testdir
scmd du
scmd gzip testfile
scmd cp /rstor/testuser/test /rtmp/test
```

## Caveats

### Using more Processors is not always faster

Many commonly run multi-processor models do not always scale as you might expect. For example, you might expect that if you ran a model with 32 processors instead of 16, it should run twice as fast. Depending on how the code was written, this may or may not be the case. In fact, sometimes it may even run slower with more processors, due to the communications overhead. The recommendation is to make test runs with a variety of processor sizes and determine what works best. It is generally a waste to just assume the maximum processor size as it ties up resources and may not be faster than a much smaller set. Also, if your job is not multi-threaded (e.g., designed to run on multiple processors), assigning it more than 1 processor will not make it run faster.

### IEEE 754 Floating-Point Arithmetic

By default, the PGI compiler does not fully adhere to the IEEE 754 standard for floating-point arithmetic. This could mean that small round-off errors might occur as a trade-off for speed optimizations. Model results may vary as a function of the number of processors chosen and/or CPU type. If your model or code requires very high precision floating-point arithmetic, use the `-Kieee` compiler option to assure compliance.

### Illegal Instruction PGI Compiler Error

When running code compiled with the Portland Group Compiler, your executable crashes with an "Illegal Instruction" error. Recompile adding `-tp=p7` as a compiler option. Note that this only applies to PGI compilers (e.g., `pgf90`, `pgf77`, `pgcc`, `pgc++`).

### Running a Multi-threaded non MPICH job

When running a job that does not support MPICH but is multi-threaded, it will not span more than one node so the maximum number of processor cores that can be used is 16. Your code must support multi-threading for it to use more than 1 processor core. It is recommended that you run a test job with 1 thread and then again with 2 or more threads comparing the run times.

## References

- SLURM Documentation: https://slurm.schedmd.com/documentation.html
- Portland Group Documentation: https://www.pgroup.com/resources/docs.htm
- MPICH Guides: https://www.mpich.org/documentation/guides/
- NVIDIA CUDA Toolkit: https://docs.nvidia.com/cuda

<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/3/3a/Slurm_logo.svg/2000px-Slurm_logo.svg.png" width=25%>

This semester we learned a bit about the SLURM scheduler on the HPCC.  Since everyone is new to this system (including the instructor) I have put together this summary document as a reference.

Note the examples below are what we have learned works on the HPCC.  However, I also think that all of the most common choices can be reduced to ```--ntasks``` (```-n```) and ```--ntasks-per-core``` (```-c```).  These are much easier to remember. I tend to think of them as ``-n`` nodes and ```-c``` cores. 

### Shared Memory
```bash
#SBATCH -n 1  #ntasks
#SBATCH -c 32 #cpus-per-task
```

### Shared Network
```bash
#SBATCH -n 32 #ntasks
#SBATCH -c 1  #cpus-per-task
```

### Hybrid 
```bash
#SBATCH -n 2  #ntasks
#SBATCH -c 16 #cpus-per-task
```

This new syntax is included below but has not been fully tested. There may be a few errors. Please contact colbrydi@msu.edu if you find something weird.

- [Shared Memory Jobs](#OpenMP)
- [Shared Network Jobs](#MPI)
- [Accelerators](#CUDA)
- [Hybrid Jobs](#Hybrid)
- [Job Arrays](#Arrays)

----
<a name="OpenMP"></a>
## Shared Memory Jobs (OpenMP)

The following submission script requests 12 tasks on a single node. 

```bash
#!/bin/bash
#SBATCH -N 1
#SBATCH --ntasks-per-node 32 
#SBATCH --time 01:00:00
#SBATCH --mem 4gb

module load powertools #needed for js shortcut

export OMP_NUM_THREADS=${SLURM_CPUS_PER_TASK} # I think this is wrong, Need to check

# Output this file
cat $0

# Show Environment variables
env | grep SLURM

time srun ./your_openmp_program

js $SLURM_JOB_ID
```

The above has been tested and seems to work. However, based on what we have learned when figuring out the hybrid code, the following syntax may be a better choice. However, this has not yet been fully tested:

```bash
#!/bin/bash
#SBATCH -n 1  #ntasks
#SBATCH -c 32 #cpus-per-task
#SBATCH --time 01:00:00
#SBATCH --mem 4gb

...
```

----
<a name="MPI"></a>
## Shared Network Jobs (MPI)

The following syntax is really good for a shared network job. 
```bash
#!/bin/bash
#SBATCH --ntasks=7 #same as -n
#SBATCH --time 04:00:00
#SBATCH --mem 42gb


module load powertools #needed for js shortcut

# Output this file
cat $0

# Show Environment variables
env | grep SLURM

time srun ./your_mpi_program

js $SLURM_JOB_ID

```

The above has been tested and seems to work. However, based on what we have learned when figuring out the hybrid code, the following syntax may be a better choice. However, this has not yet been fully tested:

```bash
#!/bin/bash
#SBATCH -n 7 #ntasks
#SBATCH -c 1 #cpus-per-task
#SBATCH --time 01:00:00
#SBATCH --mem 4gb

...
```

----
<a name="CUDA"></a>
## Accelerators (GPUs)

The following requests one GPU using the default node allocation which is 1 task on 1 node with 1 CPU.

```bash
#!/bin/bash
#SBATCH -gres gpu:1
#SBATCH --time 01:00:00
#SBATCH --mem 8gb

module load powertools #needed for js shortcut
module load cuda

# Output this file
cat $0

# Show Environment variables
env | grep SLURM

time srun ./your_cuda_program

js $SLURM_JOB_ID
```

----
<a name="Hybrid"></a>
## Hybrid Jobs (MPI / OpenMP)

Here is how we combine MPI and OpenMP jobs.

```bash
#!/bin/bash
#SBATCH -n 10  #ntasks
#SBATCH -c 7   #cpus-per-task
#SBATCH --time=3:00:00

module load powertools #needed for js shortcut

# Set OMP_NUM_THREADS to the same value as -c
# with a fallback in case it isn't set.
# SLURM_CPUS_PER_TASK is set to the value of -c, but only if -c is explicitly set
if [ -n ${SLURM_CPUS_PER_TASK} ]
then
  omp_threads=${SLURM_CPUS_PER_TASK}
else
  omp_threads=1
fi
export OMP_NUM_THREADS=${omp_threads}

time srun ./your-mpi-openmp-program 


js $SLURM_JOB_ID
 ```

----
<a name="Arrays"></a>
## Job Arrays

In this example, each job in the array defaults to one CPU.  This can be changed by combining the array option with the syntax above.
 

```bash
#!/bin/bash -login
#SBATCH --time=02:00:00
#SBATCH --mem=2GB
#SBATCH --array=1-10

module load powertools #needed for js shortcut

time ./array_program < data.${SLURM_JOBID} > output.${SLURM_JOBID}

js ${SLURM_JOBID}

```


Written by Dr. Dirk Colbry, Michigan State University
<a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/">Creative Commons Attribution-NonCommercial 4.0 International License</a>.

----

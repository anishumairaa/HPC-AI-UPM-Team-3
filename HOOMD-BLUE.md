# Base Code
This project is based on https://github.com/hpcac/2024-APAC-HPC-AI 

```
#!/bin/bash
#PBS -P 50000022
#PBS -l walltime=00:00:60
#PBS -j oe
#PBS -M 393958790@qq.com
#PBS -m abe
##-map-by ppr:$((2*${NCPUS})):node \
##-bind-to hwthread -use-hwthread-cpus \
##-report-bindings \

date
module purge
module load openmpi/4.1.2-hpe
module load libfabric/1.11.0.4.125

cmd="time mpirun \
-mca opal_common_ucx_opal_mem_hooks 1 \
-wdir ${HOME}/scratch/workdir/hoomd \
-output-filename ${HOME}/run/output/${PBS_JOBNAME}.${PBS_JOBID} \
-oversubscribe \
-map-by ppr:$((1*${NCPUS})):node \
-bind-to core \
-x PYTHONPATH=${HOME}/scratch/workdir/hoomd/build/hoomd-openmpi-4.1.2-hpe:${HOME}/scratch/workdir/hoomd/hoomd-benchmarks \
${HOME}/scratch/workdir/hoomd/hoomd.py312/bin/python \
-m hoomd_benchmarks.md_pair_wca \
--device CPU -v \
-N ${N} --repeat ${repeat} \
--warmup_steps ${warmup_steps} --benchmark_steps ${benchmark_steps}"

echo ${cmd}

exec ${cmd}
date
```

# Modifications to the code

# Reference Results
## Performance Metrics
- Steps per second: This measures how fast the system can simulate particle movements over time
- Execution time (s): This shows how long the task takes to complete
- Speedup: This measures how much faster a task completes when multiple processors are used compared to using a single processor
- Efficiency: This measures how effectively the processors are being used in parallel

# Value initialization
| Number of nodes | Number of cores used | Warmup/Benchmark   | Walltime Requested | Memory Requested |
|------------------|----------------------|--------------------|--------------------|------------------|
| 1 x 2            | 48 x 1              | 40,000/80,000      | 10 mins            | 48GB             |
| 2 x 2            | 48 x 2              | 40,000/80,000      | 10 mins            | 96GB             |
| 4 x 2            | 48 x 4              | 40,000/80,000      | 10 mins            | 192GB            |
| 8 x 2            | 48 x 8              | 40,000/80,000      | 10 mins            | 384GB            |
| 16 x 2           | 48 x 16             | 10,000/160,000     | 10 mins            | 768GB            |
| 32 x 2           | 48 x 32             | 10,000/320,000     | 10 mins            | 1536GB           |

## Results
| Number of nodes | Number of cores used | Total Cores | Memory requested (GB) | Walltime Used | Memory Used (GB) | Steps per second |
|------------------|----------------------|-------------|------------------------|---------------|------------------|------------------|
| 1 x 2            | 48 x 1              | 48          | 48                     | 0:55          | 21.54            | 423              |
| 2 x 2            | 48 x 2              | 96          | 96                     | 0:27          | 31.68            | 1058             |
| 4 x 2            | 48 x 4              | 192         | 192                    | 0:18          | 62.11            | 2142             |
| 8 x 2            | 48 x 8              | 384         | 384                    | 0:15          | 123.58           | 3431             |
| 16 x 2           | 48 x 16             | 768         | 768                    | 0:14          | 249.8            | 4831             |
| 32 x 2           | 48 x 32             | 1536        | 1536                   | 0:14          | 494.79           | 6431             |

## Result Analysis 
The steps per second increases substantially with a higher number of nodes:
  - 1 node, 48 cores: 423 steps per seconds
  - 32 nodes, 1536 cores: 6431 steps per second

The execution time decreases as more nodes were added:
  - 1 node, 48 cores: 55 seconds
  - 32 nodes, 1536 cores: 14 seconds

The simulation speed increases with the total number of cores:
  - 1 node, 48 cores: 1.00s
  - 32 nodes, 1536 cores: 3.93s

The system becomes less efficient as more cores are added:
  - 1 node, 48 cores: 0.02
  - 32 nodes, 1536 cores: 0.003

In summary, while scaling up the number of nodes and cores yields substantial performance gains in speed and execution time, the efficiency per core decreases, suggesting that beyond a certain point, adding more resources may provide limited benefits compared to the cost in system resources.

improvements  
advantages of your codes  
instructions for result reproduction

# Configuration Instructions
Step 1: Setting Up the Environment
```
module purge
module load ${HOME}/hpcx-v2.20-gcc-mlnx_ofed-redhat8-cuda12-x86_64/modulefiles/hpcx-ompi
```
Purpose: Purging clears the environment, and loading hpcx-ompi includes MPI, GCC, and network support libraries needed for HPC applications.

Step 2: Constructing and Running the MPI Command
```
hosts=$(sort -u ${PBS_NODEFILE} | paste -sd ',')
cmd="time mpirun \
    -host ${hosts} \
    -wdir ${HOME}/scratch/workdir/hoomd \
    -output-filename ${HOME}/run/output/${PBS_JOBNAME}.${PBS_JOBID} \
    -map-by ppr:$((1*${NCPUS})):node \
    -oversubscribe -use-hwthread-cpus \
    -x PYTHONPATH=${HOME}/scratch/workdir/hoomd/build/hoomd-openmpi4.1.5:${HOME}/scratch/workdir/hoomd/hoomd-benchmarks \
    ${HOME}/scratch/workdir/hoomd/build/hoomd_executable \
    --device CPU -v \
    -N ${N} --repeat ${repeat} \
    --warmup_steps ${warmup_steps} --benchmark_steps ${benchmark_steps}"
```
- host ${hosts}: Lists the nodes available for the job.
- wdir: Sets the working directory.
- output-filename: Directs output to a unique file.
- map-by ppr:$((1*${NCPUS})):node: Maps one process per core.
- oversubscribe -use-hwthread-cpus: Allows using more processes than physical cores.

Step 3: Execute the command
```
echo ${cmd}   # Prints the command for verification
exec ${cmd}   # Executes the MPI job
```

# Test Methods
1. Scalability Tests: Run the script with varying numbers of nodes and processes to test scalability. For example, change NCPUS or ngpus and track the performance impact.
2. Warm-up Testing: Ensure that the warm-up steps are long enough to allow the system to reach stable performance, particularly in simulations with complex initialization.
3. Cross-validation: Run the same job with different configurations and compare the results. This can identify issues caused by specific parameters.
4. Benchmarking: regularly test different system configurations by varying the number of nodes, GPUs, and CPUs.

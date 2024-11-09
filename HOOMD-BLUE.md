# Base Code
This project is based on https://github.com/hpcac/2024-APAC-HPC-AI  
GCC is being utilized as the compiler for this project

## PBS Script
```
#!/bin/bash
#PBS -j oe  
#PBS -M 216990@student.upm.edu.my,214928@student.upm.edu.my,215541@student.upm.edu.my,215014@student.upm.edu.my,214511@student.upm.edu.my
#PBS -m abe
#PBS -P zd64
#PBS -l ngpus=0
#PBS -l walltime=00:00::60
#PBS -l other=hyperthread
#-report-bindings \

module purge
module load ${HOME}/hpcx-v2.20-gcc-mlnx_ofed-redhat8-cuda12-x86_64/modulefiles/hpcx-ompi

hosts=$(sort -u ${PBS_NODEFILE} | paste -sd ',')

cmd="time mpirun \
    -host ${hosts} \
    -wdir ${HOME}/scratch/workdir/hoomd \
    -output-filename ${HOME}/run/output/${PBS_JOBNAME}.${PBS_JOBID} \
    -map-by ppr:$((1*${NCPUS})):node \
    -oversubscribe -use-hwthread-cpus \
    -x PYTHONPATH=${HOME}/scratch/workdir/hoomd/build/hoomd-openmpi4.1.5:${HOME}/scratch/workdir/hoomd/hoomd-benchmarks \
    ${HOME}/scratch/workdir/hoomd/hoomd.py312/bin/python \
    -m hoomd_benchmarks.md_pair_wca \
    --device CPU -v \
    -N ${N} --repeat ${repeat} \
    --warmup_steps ${warmup_steps} --benchmark_steps ${benchmark_steps}"

echo ${cmd}

exec ${cmd}

```

# Modifications to the code 
The number of nodes, walltime configuration,warmup steps and benchmark steps were adjusted to optimize performance, allowing for a comparative analysis of the results to identify the most efficient configuration

## Our code Vs Base code

```
cd ${HOME}/run

nodes=8 walltime=00:00:200 \
warmup_steps=40000 benchmark_steps=80000 repeat=1 N=200000 \
bash -c \
'qsub -V \
-l walltime=${walltime},select=${nodes}:ncpus=$((128*1)):mem=$((128*2))gb \
-N hoomd.nodes${nodes}.WS${warmup_steps}.BS${benchmark_steps}.N${N} \
hoomd.sh'
```

We increase the value of the walltime used,warmup steps and benchmark steps by using different number of nodes for each job and compare their optimization

```
nodes=32 walltime=00:10:00 \
warmup_steps=10000 benchmark_steps=8000 repeat=1 N=200000 \
bash -c \
'qsub -V \
-l walltime=${walltime},ncpus=$((48*nodes)),mem=$((48*nodes*1))gb \
-N hoomd.nodes${nodes}.WS${warmup_steps}.BS${benchmark_steps} \
hoomd.sh'

```
```
cat hoomd.nodes32.WS10000.BS8000.o126506599
```



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

# Improvements
This project uses parallel computing to simulate particle movements with High-Performance Computing (HPC) for greater efficiency. Key improvements include:

## 1. Enhanced Performance with Increased Nodes and Cores
- **Speed and Execution Time:** Increasing nodes and cores leads to faster steps per second and reduced execution time. This shows effective parallel processing, allowing more tasks to complete quickly.
  
- **Trade-off in Efficiency:** While speed increases with more cores, individual core efficiency decreases due to the coordination required among processors. This is a common trade-off in parallel computing.

## 2. Improved Scalability
- The system supports up to 32 nodes, managing larger workloads effectively. However, efficiency slightly declines at high core counts due to increased communication overhead among cores.

## 3. Optimized Resource Allocation and Memory Management
- Memory was allocated based on node and core requirements, scaling from 48 GB to 1536 GB to handle large-scale tasks smoothly. This ensures the system can operate reliably for intensive simulations.

## 4. Resource Optimization
- Balances speed and resource use for optimal performance, achieving a good trade-off between performance gains and resource efficiency.


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
4. Benchmarking: Regularly test different system configurations by varying the number of nodes, GPUs, and CPUs.

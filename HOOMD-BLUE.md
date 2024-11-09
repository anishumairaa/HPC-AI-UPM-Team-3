# Base Code
This project is based on https://github.com/hpcac/2024-APAC-HPC-AI 
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
Step 1: Load Required Modules
---
module purge
module load ${HOME}/hpcx-v2.20-gcc-mlnx_ofed-redhat8-cuda12-x86_64/modulefiles/hpcx-ompi

# Test Methods

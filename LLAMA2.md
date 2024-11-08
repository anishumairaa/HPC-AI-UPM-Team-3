# LLAMA2 Base Code
We are using Aspire2A as our cluster.  
This project is based on [https://github.com/hpcac/2024-APAC-HPC-AI ](https://github.com/hpcac/2024-APAC-HPC-AI/blob/main/3_2_LitGPT_Llama2_Application_Notes_ASPIRE-2A.md)   
We are copying baseline code from that github link, and change pbs initialization such as project code and email.  
The copied `llama.sh` with this content:  
```
#!/bin/bash
#PBS -P 50000032
#PBS -l walltime=00:01:00
#PBS -j oe
#PBS -M 214928@student.upm.edu.my,216638@student.upm.edu.my
#PBS -m abe

date
module purge
module load openmpi/4.1.2-hpe
module load libfabric/1.11.0.4.125

#env
cat $PBS_NODEFILE
#hosts=$(sort -u ${PBS_NODEFILE} | paste -sd ',')
#-host ${hosts} -np 8 \

nvidia-smi

cmd="mpirun \
-wdir ${HOME}/scratch/workdir/llama \
-output-filename ${HOME}/run/output/${PBS_JOBNAME}.${PBS_JOBID} \
-map-by ppr:4:node -oversubscribe \
-report-bindings \
-x NCCL_DEBUG=INFO \
-x NCCL_IB_DISABLE=1 \
-mca pml ^ucx \
-x NCCL_NET_GDR_LEVEL=0 \
${HOME}/scratch/workdir/llama/litgpt.py312/bin/litgpt \
finetune_full \
${HOME}/scratch/workdir/llama/model/litgpt/meta-llama/Llama-2-7b-hf \
--out_dir ${HOME}/scratch/workdir/llama/out/finetune/full \
--data JSON --data.json_path ${HOME}/scratch/workdir/llama/dataset/alpaca1024 \
--config ${HOME}/scratch/workdir/llama/full.yaml \
--eval.final_validation=false \
--train.epochs=1 \
--devices=4 --num_nodes=2 \
--train.max_steps=${max_steps} \
--train.global_batch_size=${global_batch_size} \
--train.micro_batch_size=${micro_batch_size}"

echo ${cmd}

exec ${cmd}

#EleutherAI/pythia-70m \
#--train.max_steps=1 \
#--devices=4 --num_nodes=2"
date
```

# Modifications to the code
In the github link given, we modified this part [3_2_LitGPT_Llama2_Application_Notes_ASPIRE-2A create-pbs-bash-script ](https://github.com/hpcac/2024-APAC-HPC-AI/blob/main/3_2_LitGPT_Llama2_Application_Notes_ASPIRE-2A.md#create-pbs-bash-script)  
Our tuned-script file `${HOME}/run/tuningllama.sh` with this content:  
```
#!/bin/bash
#PBS -P 50000032
#PBS -l walltime=00:01:00
#PBS -j oe
#PBS -M 214928@student.upm.edu.my,216638@student.upm.edu.my
#PBS -m abe

date
module purge
module load openmpi/4.1.2-hpe
module load libfabric/1.11.0.4.125

env
cat $PBS_NODEFILE
	
export NCCL_DEBUG=INFO
export NCCL_IB_DISABLE=1
export NCCL_NET_GDR_LEVEL=0
export NCCL_SHM_DISABLE=1

nvidia-smi

cmd="mpirun \
-wdir ${HOME}/scratch/workdir/llama \
-output-filename ${HOME}/run/output/${PBS_JOBNAME}.${PBS_JOBID} \
-map-by ppr:4:node -oversubscribe \
-report-bindings \
-x mpirun -mca btl ^ucx \
-mca coll_hcoll_enable 1 -mca coll_basic_priority 10 \
${HOME}/scratch/workdir/llama/litgpt.py312/bin/litgpt \
finetune_full \
${HOME}/scratch/workdir/llama/model/litgpt/meta-llama/Llama-2-7b-hf \
--data JSON --data.json_path ${HOME}/scratch/workdir/llama/dataset/alpaca1024 \
--out_dir ${HOME}/scratch/workdir/llama/out/finetune/full
--config ${HOME}/scratch/workdir/llama/full.yaml \
--eval.final_validation=false \
--train.epochs=1 \
--devices=4 --num_nodes=2 \
--train.max_steps=${max_steps} \
--train.global_batch_size=${global_batch_size} \
--train.micro_batch_size=${micro_batch_size}"

echo ${cmd}

exec ${cmd}
date
```
## Our code vs base code
```
diff tuningllama.sh llama.sh
13c13
< env
---
> #env
15,19c15,16
<
< export NCCL_DEBUG=INFO
< export NCCL_IB_DISABLE=1
< export NCCL_NET_GDR_LEVEL=0
< export NCCL_SHM_DISABLE=1
---
> #hosts=$(sort -u ${PBS_NODEFILE} | paste -sd ',')
> #-host ${hosts} -np 8 \
28,29c25,28
< -x mpirun -mca btl ^ucx \
< -mca coll_hcoll_enable 1 -mca coll_basic_priority 10 \
---
> -x NCCL_DEBUG=INFO \
> -x NCCL_IB_DISABLE=1 \
> -mca pml ^ucx \
> -x NCCL_NET_GDR_LEVEL=0 \
32a32
> --out_dir ${HOME}/scratch/workdir/llama/out/finetune/full \
34d33
< --out_dir ${HOME}/scratch/workdir/llama/out/finetune/full
45a45,48
>
> #EleutherAI/pythia-70m \
> #--train.max_steps=1 \
> #--devices=4 --num_nodes=2"
```
We added and modified few lines in `tuningllama.sh`, that are:
```
-x mpirun -mca btl ^ucx \
-mca coll_hcoll_enable 1 -mca coll_basic_priority 10 \
```
```
export NCCL_DEBUG=INFO
export NCCL_IB_DISABLE=1
export NCCL_NET_GDR_LEVEL=0
export NCCL_SHM_DISABLE=1
```
and deleted:
```
-x NCCL_DEBUG=INFO \
-x NCCL_NET_GDR_LEVEL=0 \
-x NCCL_IB_DISABLE=1 \
```
## Submit jobs
To submit jobs, we run command in [submit_job_llama.txt ](https://github.com/anishumairaa/HPC-AI-UPM-Team-3/blob/main/script_job_output_logs/submit_job_llama.txt)  

# Reference Results
## Performance metrics
Training time (s): This measures how long it takes to complete the training.  
Our goal is to lower the training time as much as we can.  
## Workload profile
- Workload: Llama-2-7b finetune-full
- Max Seq Length: 512
- Number of Epochs: 1
- Dataset

| Supercomputer	| NSCC SG Aspire-2A iterations	|
|:--------------|:------------------------------|
| Dataset	| Alpaca1024/train.json	        |


## Value initialization
These values are constant throughout the performance improvement process.
| Num. of nodes | Num. of GPUs | Num. of CPUs | Num. of Epochs | Global Batch Size | Micro Batch Size | Max Steps |
|:--------------|:-------------|:-------------|:---------------|:------------------|:-----------------|:----------|
| 2             | 8            | 128          | 1              | 128               | 32               | 20        |

## Results and advantages
### Baseline
| Num. of nodes | Num. of GPUs | Num. of CPUs | Memory Requested | Average Training Time |
|:--------------|:-------------|:-------------|:-----------------|:----------------------|
| 2             | 8            | 128          | 17.73            | 41.57s                |  

This script:
- uses OpenMPI version 4.1.2
- uses Libfabric
- uses `mpirun` to do MPI job
- maps 4 processes per node
- oversubscribes which allows running more MPI processes than there are physical cores available
- uses MCA (Modular Component Architecture) parameters for optimizing MPI job
- disables Infiniband support in NCCL
- excludes the UCX layer in MPI
- disables GPU Direct RDMA

### Improved script
| Num. of nodes | Num. of GPUs | Num. of CPUs | Memory Requested | Average Training Time |
|:--------------|:-------------|:-------------|:-----------------|:----------------------|
| 2             | 8            | 128          | 17.73            | 28.09s                |  

Our script:
- uses exact configurations as baseline script, except that,
- `mpirun` command is using export which makes these variables available globally to all processes
- disables shared memory communication `NCCL_SHM_DISABLE=1` that will reduce conflicts or contention during processes' communication
- enables High-Performance Collectives (HCOLL) `coll_hcoll_enable 1`
- lowers the priority of the basic collective module `coll_basic_priority 10` to ensure HCOLL is used preferentially if available  

Although our script has improved slightly in the training speed, but our script is focused on improving inter-node communication performance and stability where shared memory access might cause bottlenecks or instability. In HPC environments where data transfer between GPUs needs efficiency, which can increase training speed. Thus, it is important to concentrated on communication settings in this job.

## Output file
Our output file for `tuningllama.sh` is in [llama.nodes2.GBS128.MBS32.o8613326](https://github.com/anishumairaa/HPC-AI-UPM-Team-3/blob/main/script_job_output_logs/llama.nodes2.GBS128.MBS32.o8613326) 

# Configuration Instructions

# Test Methods

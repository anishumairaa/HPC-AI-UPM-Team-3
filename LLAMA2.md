# Base Code
Our LLAMA2 is using this as base code  
We are using ASPIRE2A as our cluster  
This project is based on [https://github.com/hpcac/2024-APAC-HPC-AI ](https://github.com/hpcac/2024-APAC-HPC-AI/blob/main/3_2_LitGPT_Llama2_Application_Notes_ASPIRE-2A.md)   

# Modifications to the code
In the github link given, we modified this part [3_2_LitGPT_Llama2_Application_Notes_ASPIRE-2A create-pbs-bash-script ](https://github.com/hpcac/2024-APAC-HPC-AI/blob/main/3_2_LitGPT_Llama2_Application_Notes_ASPIRE-2A.md#create-pbs-bash-script)  
Our script file `${HOME}/run/llama.sh` with this content:  
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
We added and modified few lines in `llama.sh`, that are:
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
| Num. of nodes | Num. of GPUs | Num. of CPUs | Num. of Epochs | Global Batch Size | Micro Batch Size | Max Steps |
|:--------------|:-------------|:-------------|:---------------|:------------------|:-----------------|:----------|
| 2             | 8            | 128          | 1              | 128               | 32               | 20        |

## Results
### Baseline
| Num. of nodes | Num. of GPUs | Num. of CPUs | Memory Requested | Average Training Time |
|:--------------|:-------------|:-------------|:-----------------|:----------------------|
| 2             | 8            | 128          | 17.73            | 41.57s                |


### Improved script
| Num. of nodes | Num. of GPUs | Num. of CPUs | Memory Requested | Average Training Time |
|:--------------|:-------------|:-------------|:-----------------|:----------------------|
| 2             | 8            | 128          | 17.73            | 28.09s                |

computing/training/throughput performances  
improvements  
advantages of your codes  
instructions for result reproduction

# Configuration Instructions

# Test Methods

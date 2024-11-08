
# Base Code

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
# Execute the job

```
nodes=32 walltime=00:10:00 \
warmup_steps=10000 benchmark_steps=8000 repeat=1 N=200000 \
bash -c \
'qsub -V \
-l walltime=${walltime},ncpus=$((96*nodes)),mem=$((48*nodes*1))gb \
-N hoomd.nodes${nodes}.WS${warmup_steps}.BS${benchmark_steps} \
hoomd.sh'
```

# Output

```
time mpirun -host gadi-cpu-clx-1906.gadi.nci.org.au -wdir /home/552/sp1115/scratch/workdir/hoomd -output-filename /home/552/sp1115/run/output/hoomd.nodes1.WS10000.BS8000.128060868.gadi-pbs -map-by ppr:48:node -oversubscribe -use-hwthread-cpus -x PYTHONPATH=/home/552/sp1115/scratch/workdir/hoomd/build/hoomd-openmpi4.1.5:/home/552/sp1115/scratch/workdir/hoomd/hoomd-benchmarks /home/552/sp1115/scratch/workdir/hoomd/hoomd.py312/bin/python -m hoomd_benchmarks.md_pair_wca --device CPU -v -N 200000 --repeat 1 --warmup_steps 10000 --benchmark_steps 8000
Using existing initial_configuration_cache/hard_sphere_200000_1.0_3.gsd
notice(2): Using domain decomposition: n_x = 3 n_y = 4 n_z = 4.
Running MDPairWCA benchmark
.. warming up for 10000 steps
.. running for 8000 steps 1 time(s)
.. 427.3252261191434 time steps per second
427.3252261191434
2016.74user 148.87system 0:48.10elapsed 4502%CPU (0avgtext+0avgdata 552868maxresident)k
0inputs+16840outputs (507major+11782588minor)pagefaults 0swaps

======================================================================================
                  Resource Usage on 2024-11-03 12:08:43:
   Job Id:             128060868.gadi-pbs
   Project:            zd64
   Exit Status:        0
   Service Units:      1.33
   NCPUs Requested:    48                     NCPUs Used: 48              
                                           CPU Time Used: 00:36:06        
   Memory Requested:   48.0GB                Memory Used: 21.48GB         
   Walltime requested: 00:10:00            Walltime Used: 00:00:50        
   JobFS requested:    100.0MB                JobFS used: 0B              
======================================================================================
```




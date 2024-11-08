
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
# Submit the job

```
nodes=32 walltime=00:10:00 \
warmup_steps=10000 benchmark_steps=8000 repeat=1 N=200000 \
bash -c \
'qsub -V \
-l walltime=${walltime},ncpus=$((96*nodes)),mem=$((48*nodes*1))gb \
-N hoomd.nodes${nodes}.WS${warmup_steps}.BS${benchmark_steps} \
hoomd.sh'
```
# Execute Output

```
cat hoomd.nodes32.WS10000.BS8000.o126506599
```

# Output

```
time mpirun -host gadi-cpu-clx-2341.gadi.nci.org.au,gadi-cpu-clx-2342.gadi.nci.org.au,gadi-cpu-clx-2343.gadi.nci.org.au,gadi-cpu-clx-2344.gadi.nci.org.au,gadi-cpu-clx-2347.gadi.nci.org.au,gadi-cpu-clx-2348.gadi.nci.org.au,gadi-cpu-clx-2349.gadi.nci.org.au,gadi-cpu-clx-2350.gadi.nci.org.au,gadi-cpu-clx-2351.gadi.nci.org.au,gadi-cpu-clx-2352.gadi.nci.org.au,gadi-cpu-clx-2353.gadi.nci.org.au,gadi-cpu-clx-2354.gadi.nci.org.au,gadi-cpu-clx-2355.gadi.nci.org.au,gadi-cpu-clx-2356.gadi.nci.org.au,gadi-cpu-clx-2357.gadi.nci.org.au,gadi-cpu-clx-2358.gadi.nci.org.au,gadi-cpu-clx-2359.gadi.nci.org.au,gadi-cpu-clx-2360.gadi.nci.org.au,gadi-cpu-clx-2361.gadi.nci.org.au,gadi-cpu-clx-2362.gadi.nci.org.au,gadi-cpu-clx-2365.gadi.nci.org.au,gadi-cpu-clx-2366.gadi.nci.org.au,gadi-cpu-clx-2367.gadi.nci.org.au,gadi-cpu-clx-2368.gadi.nci.org.au,gadi-cpu-clx-2369.gadi.nci.org.au,gadi-cpu-clx-2370.gadi.nci.org.au,gadi-cpu-clx-2371.gadi.nci.org.au,gadi-cpu-clx-2372.gadi.nci.org.au,gadi-cpu-clx-2373.gadi.nci.org.au,gadi-cpu-clx-2374.gadi.nci.org.au,gadi-cpu-clx-2375.gadi.nci.org.au,gadi-cpu-clx-2376.gadi.nci.org.au -wdir /home/552/sp1115/scratch/workdir/hoomd -output-filename /home/552/sp1115/run/output/hoomd.nodes32.WS10000.BS8000.128061191.gadi-pbs -map-by ppr:48:node -oversubscribe -use-hwthread-cpus -x PYTHONPATH=/home/552/sp1115/scratch/workdir/hoomd/build/hoomd-openmpi4.1.5:/home/552/sp1115/scratch/workdir/hoomd/hoomd-benchmarks /home/552/sp1115/scratch/workdir/hoomd/hoomd.py312/bin/python -m hoomd_benchmarks.md_pair_wca --device CPU -v -N 200000 --repeat 1 --warmup_steps 10000 --benchmark_steps 8000
Using existing initial_configuration_cache/hard_sphere_200000_1.0_3.gsd
notice(2): Using domain decomposition: n_x = 8 n_y = 12 n_z = 16.
Running MDPairWCA benchmark
.. warming up for 10000 steps
.. running for 8000 steps 1 time(s)
.. 6568.257745002172 time steps per second
6568.257745002172
175.89user 93.14system 0:11.65elapsed 2308%CPU (0avgtext+0avgdata 452796maxresident)k
0inputs+16824outputs (430major+3813746minor)pagefaults 0swaps

======================================================================================
                  Resource Usage on 2024-11-03 12:55:36:
   Job Id:             128061191.gadi-pbs
   Project:            zd64
   Exit Status:        0
   Service Units:      11.95
   NCPUs Requested:    1536                   NCPUs Used: 1536            
                                           CPU Time Used: 02:32:59        
   Memory Requested:   1.5TB                 Memory Used: 493.55GB        
   Walltime requested: 00:10:00            Walltime Used: 00:00:14        
   JobFS requested:    3.12GB                 JobFS used: 0B              
======================================================================================
```
# Result




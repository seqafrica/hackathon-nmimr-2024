# Slurm Demo

Here are some commands to familiarize you with `slurm`.


## HPC Setup

This is the setup of the Noguchi HPC:

![HPC Diagram](hpc-diagram.jpg)

We have:
 * Login nodes `node101 (10.27.7.7)` and `node103 (10.27.7.9)`
 * Worker nodes `node1..4`
 * Controller node (but you won't notice) `node3`

All nodes share the `/hpc` and `/home` directories.

When working on the cluster, remember to:
 * Always enter through a **login node**
 * Don't run (heavy) jobs on a login node
 * First thing is always to `kinit` to authenticate

Then, the core commands are:
 * `srun` for immediate commands
 * `salloc` to enter an interactive session
 * `sbatch` to asynchronously run batch jobs


## Using `srun`



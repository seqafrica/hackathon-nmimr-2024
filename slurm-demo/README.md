# Slurm Demo

Here are some commands to familiarize you with `slurm`.


## HPC Setup

This is the setup of the Noguchi HPC:

![HPC Diagram](hpc-diagram.jpg)

We have:

 * Login nodes `node101 (10.27.7.7)` and `node103 (10.27.7.9)` (@TODO@: ask Razak to add to UG domain)
 * Worker nodes `node1..4` (`.noguchi.hpc` -> internal DNS domain, known within the cluster only)
 * Controller node (but you won't notice) `node3`

All nodes share the `/hpc` and `/home` directories.

When working on the cluster, remember to **always**:

 * Enter through a **login node**: (@TODO@: allow only `sysadmins` group outside login on `node{1..4}`)
 * Not run (heavy) tasks on a login node: it is small, and you share it with everyone
 * Start with `kinit` to authenticate (or check with `klist` first)

Then, the core commands are:

 * `srun` for immediate commands
 * `salloc` to enter an interactive session
 * `sbatch` to asynchronously run batch jobs

For the example sessions below, login on an entry node of your choice, then `kinit`

    $ ssh zwets@10.27.7.9   # zwets is my username
    ...

    # We are on node103, authenticate
    [@node103:~] $ kinit
    Password for zwets@NOGUCHI.HPC: 

    # Recommended to work in a screen or tmux session
    [@node103:~] $ screen    # to later recover your session: screen -DRRU

Note that Slurm has very complete `man` pages, e.g. for an overview:

    [@node103:~] $ man slurm
    ...

    # And of course commands have help as well
    srun --help

We are good to go!


## Using `srun`

`srun` is for invoking an immediate command, except it will execute on a worker node:

    # Run the hostname command locally on the login node
    [@node103:~] $ hostname
    node103

    # Same but run it on the HPC
    [@node103:~] $ srun hostname
    node1

> **From here I will leave out the `[@node103:~] $` prompt for easier cut & paste**

If you go through Slurm's getting started, this is what they show first:

    srun -n4 hostname
    node1
    node1
    node1
    node1


## Using `salloc`

You could 

## Using `sbatch`


### Job Arrays


### Workflow Support



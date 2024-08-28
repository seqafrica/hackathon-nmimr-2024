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

If you go through Slurm's getting started, this is what they show first:

    [@node103:~] $ srun -n 4 hostname
    node1
    node1
    node1
    node1

In Slurm terminology this one **job** executes 4 **tasks**.  All tasks (in this
case) run on `node1`.

Then they show you this feature:

    [@node103:~] $ srun -N 4 hostname
    node3
    node1
    node2
    node4

So the `-N4` param requests one task per **node**, whereas this:

    [@node103:~] $ srun -n 4 -N 2-3 hostname
    node3
    node1
    node1
    node2

Requests four tasks to run, using at least 2 and at most 3 nodes.

> You may now wonder why anyone would want to run the same thing
> four times (in parallel).  Very good point, we will get to that.

First, two more enlightening examples:

    [@node103:~] $ srun -N 5 hostname
    srun: Requested partition configuration not available now
    srun: job 51 queued and waiting for resources
    ^C

Aha! Slurm replies that right now, there are not a mininum of 5 nodes.

How about asking for more tasks than any single node has CPUs (72):

    [@node103:~] $ srun -n 100 hostname | sort | uniq -c
    72 node1
    28 node2

OK, so Slurm will spread the tasks over the CPUs across the nodes!


## Using `salloc`

@TODO@

## Using `sbatch`

@TODO@

### Job Arrays

@TODO@

### Workflow Support

@TODO@


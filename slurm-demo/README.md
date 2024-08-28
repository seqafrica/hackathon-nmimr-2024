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
 * Always claim the _number of CPUs_ you will use, and usually also _how much memory_

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

`srun` is for invoking an immediate command, just like you would in a shell,
except it will execute on a worker node:

    # Run the hostname command locally on the login node
    [@node103:~] $ hostname
    node103

    # Same but make Slurm run it on the HPC
    [@node103:~] $ srun hostname
    node1

If you go through Slurm's getting started, this is what they show first:

    [@node103:~] $ srun -n 4 hostname
    node1
    node1
    node1
    node1

In Slurm terminology this is one **job** executing 4 parallel **tasks**.
All tasks (in this case) run on `node1`.

Another option to demonstrate how Slurm works:

    [@node103:~] $ srun -N 4 hostname
    node3
    node1
    node2
    node4

The `-N` param requests the minimum number of **node** to run the task on.
Combining this, we can request 4 tasks to run on two or three nodes:

    [@node103:~] $ srun -n 4 -N 2-3 hostname
    node3
    node1
    node1
    node2

The `-N` here specifies the minimum and maximum number of nodes to use.
Add the `-l` option to see which output came from which task:

    [@node103:~] $ srun -l -n 4 -N 2-3 hostname
    3: node3
    0: node1
    1: node1
    2: node2

> You may now wonder why anyone would want to do the exact same thing
> four times (in parallel).  Very good point, we will get to that.

First, two more enlightening examples:

    [@node103:~] $ srun -N 5 hostname
    srun: Requested partition configuration not available now
    srun: job 51 queued and waiting for resources
    ^C

Aha! Slurm replies that right now, there are not a mininum of 5 nodes
available, and it will block until they are.  (Press Ctrl-C to abort.)

How about asking for more tasks than any single node has CPUs (ours
have 72):

    [@node103:~] $ srun -n 100 hostname | sort | uniq -c
    72 node1
    28 node2

OK, so Slurm will spread the tasks over the CPUs across the nodes,
assigning a CPU per task.

### Why run the exact same thing four times?

Two answers:

 * Some software can make use of this through **MPI**, but needs to be
   specifically programmed for it.  It is particularly useful for tasks
   with lots of computation on relatively small amounts of data.

   In bioinformatics, this is rare; I am aware of only PhyML that can
   make use of MPI when bootstrapping (so you could use all 296 cores
   of the cluster at once.

 * Slurm passes to each task a set of environment variables, among which
   `SLURM_PROCID` and `SLURM_LOCALID` that give the sequence number of
   the task:

        [@node103:~] $ srun -l -n 2 env | grep SLURM_PROCID
        0: SLURM_PROCID=0
        1: SLURM_PROCID=1

   So the processes could use this to pick e.g. a specific input file from
   a list and work on that.

> **What is difference between a CPU and a core**
>
> @TODO@ explain with `hwloc` and `lstopo`.
> For now, remember that a CPU is the lowest level (and we have 72 per node)
> but colloquially we often call that a core.

### Always request CPUs

Whenever you use `srun` (or `salloc` or `sbatch`), always specify the
number of CPUs you are requesting from Slurm:

    [@node103:~] $ srun -c 4 flye -t 4 ...
    ...

The `-c 4` requests 4 CPUs, and you tell Flye to use 4 threads.  This will
likely give you optimal performance.  In fact, this ...

    [@node103:~] $ srun -c 1 flye -t 4 ...
    ...

... may result in Slurm terminating your job for going over its allocation.

### (Almost) always also request memory

The default setting on the HPC is to request 4GB memory for every core you
request.  So this Flye command:

    [@node103:~] $ srun -c 4 --mem=32G flye -t 4 ...
    ...

will run with 32GB of memory rather than the default 4x4 = 16GB.

**Be nice to others and request what you need** (else their jobs will be
waiting for resources to become available).

### (Future) request GPUs

When the HPC gets GPUs, use `-G 1` to request use of a (single) GPU.
     

## Using `salloc`

@TODO@

## Using `sbatch`

@TODO@

### Job Arrays

@TODO@

### Workflow Support

@TODO@


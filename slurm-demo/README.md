# Slurm Demo

Here are some commands to familiarize you with `slurm`.


## HPC Setup

The Noguchi HPC currently has:

 * 2 Login nodes `node101 (10.27.7.7)` and `node103 (10.27.7.9)`
 * 4 Worker nodes `node1..4` (`.noguchi.hpc` -> DNS domain internal to the cluster)
 * 1 Controller node (but you won't notice): `node3`

All nodes share the `/hpc` and `/home` directories.

We have a single Slurm **partition** (= queue to submit work to), called `global`.

> **Note** the [Noguchi HPC repo](https://github.com/seqafrica/noguchi-hpc)
> (private to Noguchi team members) has detailed documentation of its setup.

### General Guidance

When working on the HPC, remember to **always**:

 * Enter through a login node
 * Start with `kinit` to authenticate (or check with `klist` first)
 * Not run heavy tasks on a login node: it is small and shared with everyone
 * Always claim the _number of CPUs_ you will need, usually also _how much memory_,
   and preferably also the _time limit_ of your job or session.

### Core Commands

Then, the core commands are:

 * `srun` for immediate commands
 * `salloc` to enter an interactive session
 * `sbatch` to asynchronously run batch jobs

### Logging In

For the example session that follows, login on the entry node of your choice:

    $ ssh {yourusername}@10.27.7.9   # or .7 for node101
    ...

    # We are on node103, authenticate to Kerberos
    [@node103:~] $ kinit
    Password for zwets@NOGUCHI.HPC: 

    # Recommended to work in a screen or tmux session
    [@node103:~] $ screen    # to later recover your session: screen -DRRU

> **Sideline** to get that special coloured prompt, I use LiquidPrompt.
> Use `liquidprompt_activate` to add it in your `~/.bashrc`.  Turn on
> and off with `liquidprompt_{on,off}`.

### Getting Help

Find Slurm's usage docs here: <https://slurm.schedmd.com/documentation.html>

Slurm has comprehensive `man` pages, e.g. for an overview:

    [@node103:~] $ man slurm
    ...

The SEE ALSO section has pointers to all other `man` pages.  All commands
also have built-in help as expected:

    [@node103:~] $ srun --help

We are good to go!


## Using `srun`

`srun` is for invoking an immediate command, just like you would in a shell,
except it will execute on a worker node:

    # Run the hostname command locally on the login node
    [@node103:~] $ hostname
    node103

    # Now make Slurm run it on the HPC
    [@node103:~] $ srun hostname
    node1

If you walk through Slurm's getting started, this is what they show first:

    [@node103:~] $ srun -n 4 hostname
    node1
    node1
    node1
    node1

In Slurm terminology this is one **job** executing 4 parallel **tasks**.
All tasks (in this case) have been executed on `node1`.

Another option to demonstrate what Slurm can do:

    [@node103:~] $ srun -N 4 hostname
    node3
    node1
    node2
    node4

The `-N` param requests the minimum number of **nodes** to run the task on.
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

> Slurm has many ways to influence the way tasks are distributes over
> nodes, sockets, cores, and CPUs.  We can safely ignore these for now.
>
> **What is difference between a CPU and a core?**
>
> In Slurm, a _CPU_ is the lowest level, and on our hardware is generally
> a hyperthread in a _core_, which is the physical package that sits in
> a _socket_ with a bunch of other cores.
>
> If you `srun lstopo` on the cluster, you will see that our nodes have
> two "packages" (= sockets), each holding 18 cores, each of which has
> two CPUs, for a total of 72 CPUs (often colloquially called "cores")
> per node.

#### Why run the exact same thing four times?

Two answers:

 * Some software can make use of this through **MPI** (`man mpirun`),
   if it is be specifically programmed for this.  This is particularly
   useful for tasks with lots of computation on small amounts of data.

   In bioinformatics, this is rare; we generally deal with lots of data
   going through relatively little computation.  I am aware of only PhyML
   that can use MPI when bootstrapping (so you could use all 296 cores of
   the cluster at once, and do 1000 bootstraps in a quarter of the time).

 * Slurm passes to each task a set of environment variables, among which
   `SLURM_PROCID` and `SLURM_LOCALID`, that give the sequence number of
   the task:

        [@node103:~] $ srun -l -n 2 env | grep SLURM_PROCID
        0: SLURM_PROCID=0
        1: SLURM_PROCID=1

   This means that each instance of the process could use the value of
   this variable to pick e.g. the input it should work on from a list.
   (This will come back later with `--array` jobs.)


## Guidelines for `srun`, `salloc` and `sbatch`

Before moving on to `salloc` and `sbatch`, here is how to request
resources (CPU, memory, time) for jobs.

### Request CPUs: `-c`

Whenever you use `srun` (or `salloc` or `sbatch`), always specify the
number of CPUs you are requesting:

    [@node103:~] $ srun -c 4 flye -t 4 ...
    ...

The `-c 4` requests 4 CPUs, and you tell Flye to use 4 threads.  This will
likely give you optimal performance.  In fact, this ...

    [@node103:~] $ srun -c 1 flye -t 4 ...
    ...

... may result in Slurm terminating your job for going over its allocation,
whereas this:

    [@node103:~] $ srun -c 4 flye -t 1 ...
    ...

... would be wasting valuable resources, as other jobs could have used the
cores you have reserved.

### Request memory: `--mem`

The default setting on the HPC is to request 4GB memory for every CPU you
request.  So this Flye command:

    [@node103:~] $ srun -c 4 --mem=32G flye -t 4 ...
    ...

will run with 32GB of memory, rather than the default 4x4 = 16GB.

**Be nice to others and request no more than you need**.

### Limit run time: `-t`

By default, jobs can run for an indefinite time.  Specify your expected run
time to Slurm schedule resources, and to clean up runaway jobs:

    # The -t for srun is time (minutes), for flye it is threads
    [@node103:~] $ srun -t 60 -c 4 --mem=32G flye -t 4 ...
    
This requests 60 minutes of "wall clock time".

### Request GPUs

When the HPC gets GPUs, use `-G 1` to request use of a (single) GPU.
     

## Using `salloc`

Use `salloc` to create an interactive session:

    [@node103:~] $ salloc
    salloc: Granted job allocation 64
    [@node103:~]└2 $ 

The `└2` in my prompt indicates we are in a nested shell.  Note that
we are still on `node103`, however we have reserved an allocation:

    [@node103:~]└2 $ sinfo
    PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
    global*      up   infinite      1    mix node1
    global*      up   infinite      3   idle node[2-4]

    [@node103:~]└2 $ squeue
    JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
        64    global interact    zwets  R       3:45      1 node1

We can now execute `srun` commands that will _inherit_ our settings from
`salloc`.  To demonstrate, let's exit the `salloc` session:

    [@node103:~]└2 $ exit
    salloc: Relinquishing job allocation 64

Now ask for half an hour, 8 CPUs, 32GB memory (and `-n4` just for the demo):

    [@node103:~] $ salloc -t 30 -c 8 --mem 32G -n 4
    salloc: Granted job allocation 65
    [@node103:~]└2 $ srun hostname
    node1
    node1
    node1
    node1

This to prove that `srun` without options 'inherits' `salloc` settings.

You can see this in our (local) environment:

    [@node103:~]└2 $ env | fgrep SLURM
    SLURM_JOB_NUM_NODES=1
    SLURM_JOB_NODELIST=node1
    SLURM_NTASKS=4              <- The -n4 we specified
    SLURM_TASKS_PER_NODE=4
    SLURM_JOB_CPUS_PER_NODE=32  <- Four tasks of 8 CPUs each
    SLURM_TRES_PER_TASK=cpu:8
    SLURM_CPUS_PER_TASK=8
    SLURM_MEM_PER_NODE=32768    <- Slurm default memory unit is MB
    ...

You can background `srun` jobs like any other program:

    [@node103:~]└2 $ srun somecommand &
    [1] 17995
    [@node103:~]└2 $ do other things
    ...
    [1]+  Done                    somecommand

But remember to remain within your resource allocation.

#### Why use `salloc` if we still need to `srun`?

With `srun`, the resource request is made for every separate invocation,
with `salloc`, you own the reservation for the duration of the session.

It is **important** therefore to exit your `salloc`, especially when you
have a `screen/tmux` session, and/or set a time limit with `-t minutes`.


## Using `sbatch`

The `sbatch` command is for submitting _asynchronous_ jobs.

A job file is just any shell script.  For instance create `my.job`:

    #!/bin/sh
    hostname
    sleep 15s

You may `chmod +x my.job` but this is not necessary.  Then submit it:

    # This job needs just one CPU and 1MB memory
    [@node103:~] $ sbatch -c 1 --mem 1 my.job
    Submitted batch job 70

We can check the job's status with `squeue`:

    [@node103:~] $ squeue
      JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
         70    global   my.job    zwets  R       0:02      1 node1

Batch jobs write their stdout and stderr to `slurm-${JOBID}.out`.
This can be configured with `-o` and `-e` (and `-i` to redirect input).

    [@node103:~] $ cat slurm-70.out
    node1
    
Rather than pass command-line arguments to `sbatch`, you can put these in the
job, like so:

    #!/bin/bash
    #SBATCH -c 1 --mem 1 -n 4 -N 2
    #SBATCH ... more options
    hostname
    sleep 15s

Now we see:

    [@node103:~] $ sbatch
    Submitted batch job 73

    [@node103:~] $ sinfo
    PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
    global*      up   infinite      2    mix node[1-2]
    global*      up   infinite      2   idle node[3-4]

    [@node103:~] $ squeue
      JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
         73    global  my2.job    zwets  R       0:10      2 node[1-2]

Once done:

    [@node103:~] $ cat slurm-73.out
    node1
    
> **TODO** Hmm, that's weird.  I would expect to see `node1` _four times_.
> My suspicion is that the four tasks overwrite each other - need to check.

#### Batching: Job Arrays

Slurm does not have 'native' support for running `sbatch` over e.g. all
the files in a directory.  However, it has the concept **batch arrays**:

    [@node103:~] $ sbatch -a 1-10 my.job

Or specify this in the job:

    #!/bin/sh
    #SBATCH --array 1-10 --mem 1 -c 1
    hostname
   
This looks a lot like `-n 4`, except that we are not requesting _parallel_
tasks.  The array jobs execute, as if we had done:

    for I in {1..10}; do
        sbatch ...
    done

... except that array jobs are much lighter on the resource management,
and hence recommended for "horizontal batches" (across inputs).

Within each instance of the job, Slurm sets a number of environment vars:

    # Note the convenient --wrap option to generate an on-the-fly script
    [@node103:~] $ sbatch -a 1-4 --wrap="env | grep -E '(ARRAY|JOB_ID)'"
    Submitted batch job 187

Each array task writes to its own output file:

    [@node103:~] $ ls slurm-187*
    slurm-187_1.out  slurm-187_2.out  slurm-187_3.out  slurm-187_4.out

    [@node103:~] $ cat slurm-187_2.out
    SLURM_ARRAY_JOB_ID=187      <- The "parent" job ID for the array
    SLURM_JOB_ID=189            <- The job ID of this task (in Slurm)
    SLURM_ARRAY_TASK_ID=2       <- Unique index of this task in the array
    SLURM_ARRAY_TASK_MIN=1
    SLURM_ARRAY_TASK_MAX=4
    SLURM_ARRAY_TASK_COUNT=4
    SLURM_ARRAY_TASK_STEP=1     <- Huh, what is a STEP?

About steps: it is possible inside batch scripts, to use `srun`:

    #!/bin/sh
    #SBATCH --array 1-10 --mem 1 -c 1
    hostname
    srun date
    srun hostname
   
In this script, the `srun date` is step 1, and `srun hostname` step 2.

> If (as we typically do in bioinformatics) you want to run a batch script
> over a directory (or list) of files, then this maps naturally on arrays.
>
> However, unlike in HTCondor, in Slurm you can't simply do:
> `queue F matching *.fastq` or `queue IN,OUT from inputs.tsv`.
>
> In the [slurm-goodies repo](https://github.com/seqafrica/slurm-goodies),
> we develop some "wrapper" scripts to make this possible.


## Workflow Support

Slurm has the `-d`, `--dependency` option to make jobs wait on each other's
completion, but this is quite low level.  There is no native workflow support
(like HTCondor's DAGs).

Solutions are discussed in our [slurm-goodies repo](https://github.com/seqafrica/slurm-goodies).
[MyQueue](https://myqueue.readthedocs.io/en/latest/) looks promising.


## Container Support

See the `--container` argument and the Slurm documentation at
<https://slurm.schedmd.com/containers.html>.  @TBD@


# Nelle's Parallel Pipeline (Continued)

Last week we timed parallel and serial implementations of `goostats`. We noticed a significant difference between the two. This difference becomes even more pronounced when we use HPC resources for bigger problems. Recall in our example that Nelle had fifteen data files she was working with. In reality, a scientist would actually be dealing with much larger data sets.

Nelle informs you that she has the complete data set stored in the `/opt` folder. Let's see how many data files she has.

```bash
$ ls /opt/examples/north-pacific-gyre/2017-02-23/ | wc -l
1521
```

> Recall our times from last week. How much time do you expect it will take to complete the serial version of `goostats`? How much time to you expect it will take to complete the parallel version of `goostats`?

Nelle has shown an excellent example of best practices when it comes to data management. In order to keep files organized and make it easy to reproduce our process, it is a good idea to keep data, scripts, and results each in their own locations. If data and results are going to take a lot of space, it may also be a good idea to keep them in `/scratch` instead of your `/home` directory.

We want to run `goostats` on all 1,521 files. That means we will have 1,521 output files as well! The data is already stored in `/opt`, and with this many files, it may be a good idea to have `goostats` put the results files in our `/scratch` directory. Since scripts do not take very much space, we will place our script and the timing results in our `/home` directory.

```bash
$ cd
$ mkdir /scratch/your-username/nelles-results
$ mkdir nelle-parallel
$ cd nelle-parallel/
$ cp /scratch/your-username/data-shell/north-pacific-gyre/2012-07-03/do-stats.pbs .
$ cp /scratch/your-username/data-shell/north-pacific-gyre/2012-07-03/do-stats-parallel.pbs .
$ cp /scratch/your-username/data-shell/north-pacific-gyre/2012-07-03/goostats .
$ ls
do-stats-parallel.pbs  do-stats.pbs  goostats
```

## Using Variables

We now have our project folder setup. Let's begin by practicing using `goostats` to process some of Nelle's data. Remember that `goostats` takes two arguments, first the input file, and second, the output file.

```bash
$ bash goostats /opt/examples/north-pacific-gyre/2017-02-23/NENE01729A.txt /scratch/your-username/nelles-results/stat-NENE01729A.txt
$ ls /scratch/your-username/nelles-results/
stat-NENE01729A.txt
```

This technique is a bit messy. Fortunately, we can use a variable to hold the first part of the file path. That way we only need to type the file name.

```bash
$ export DATA='/opt/examples/north-pacific-gyre/2017-02-23'
$ echo $DATA
/opt/examples/north-pacific-gyre/2017-02-23
```

Notice that we can use `echo` to check what is inside of our variable. We can now save ourselves some typing and use `goostats` as follows:

```bash
bash goostats $DATA/NENE01729B.txt /scratch/pdoehle/nelles-results/stat-NENE01729B.txt
$ ls /scratch/your-username/nelles-results/
stat-NENE01729A.txt  stat-NENE01729B.txt
```

> Create another variable named `RESULTS` that contains the path to the results folder. Use both of the variables with `goostats` again.

```bash
$ export RESULTS='/scratch/your-username/nelles-results'
$ echo $RESULTS 
/scratch/your-username/nelles-results
$ bash goostats $DATA/NENE01736A.txt $RESULTS/stat-NENE01736A.txt
$ ls $RESULTS 
stat-NENE01729A.txt  stat-NENE01729B.txt  stat-NENE01736A.txt
```

Let's modify both the serial and parallel scripts now that we know how to use variables. We'll start with `do-stats-parallel.pbs`. Begin by changing the wall time to accommodate more files.

```bash
#PBS -l walltime=1:00:00
```

Add the variables to the script after the line `cd $PBS_O_WORKDIR`.

```bash
export DATA='/opt/examples/north-pacific-gyre/2017-02-23'
export RESULTS='/scratch/your-username/nelles-results'
```

Now modify the `parallel` command so that it can see the new locations that your stored in the variables.

```bash
time parallel "bash goostats $DATA/{} $RESULTS/stats-{}" ::: $(ls $DATA)
```

Notice that we changed around the list of things that we passed to `parallel`. Instead of just using `*[AB].txt`, we are now using the expression `$(ls $DATA)` in its place. This is because we still want to pass the list of files ending in 'A' or 'B' to parallel, but since the script is being executed in our home directory, we use `ls` with the `$DATA` variable to tell `parallel` that the files are actually in a different place than where the script it running. The expression `$(STUFF-INSIDE-HERE)` tells the shell to execute what is inside of the parentheses first before executing any other part of the command.

The `do-stats-parallel.pbs` script should look like the following:

```bash
$ cat do-stats-parallel.pbs
#!/bin/bash
#PBS -q express
#PBS -l nodes=1:ppn=12
#PBS -l walltime=1:00:00
#PBS -j oe

cd $PBS_O_WORKDIR
# variables
export DATA='/opt/examples/north-pacific-gyre/2017-02-23'
export RESULTS='/scratch/pdoehle/nelles-results'

module load gnu-parallel

time parallel "bash goostats $DATA/{} $RESULTS/stats-{}" ::: $(ls $DATA)
```

Now let's modify `do-stats.pbs`. Begin by changing the walltime so that the job can run longer.

```bash
#PBS -l walltime=1:00:00
```

Add the variables after `cd $PBS_O_WORKDIR`.

```bash
# variables
export DATA=/opt/examples/north-pacific-gyre/2017-02-23
export RESULTS=/scratch/your-username/nelles-results
```

Also modify the loop so that it has a timer and uses the new variables.

```bash
time for datafile in $(ls $DATA)
do
  bash goostats $DATA/$datafile $RESULTS/stat-$datafile
done
```

Let's have a look at `do-stats.pbs`:

```bash
$ cat do-stats.pbs
#!/bin/bash
#PBS -q express
#PBS -l nodes=1:ppn=1
#PBS -l walltime=1:00:00
#PBS -j oe

cd $PBS_O_WORKDIR

# varaibles
export DATA=/opt/examples/north-pacific-gyre/2017-02-23
export RESULTS=/scratch/pdoehle/nelles-results

time for datafile in $(ls $DATA)
do
  bash goostats $DATA/$datafile $RESULTS/stat-$datafile
done
```

## More Nodes

We've used twelve cores on one node, but what if we used twenty-four cores on two nodes? Let's make one more script that submits our job to two nodes.

```bash
$ cp do-stats-parallel.pbs more-nodes.pbs
```

Open `more-nodes.pbs` and begin by changing the number of nodes that we request from the scheduler.

```bash
#PBS -l nodes=2:ppn=12
```

We also need to add some flags to `parallel` so that it knows to look for more than one node.

```bash
time parallel -j $(nproc) --sshloginfile $PBS_NODEFILE --workdir $PBS_O_WORKDIR "bash goostats $DATA/{} $RESULTS/stats-{}" ::: $(ls $DATA)
```

`-j` explicitly tells `parallel` how many cores are available. The program `nproc` can be run in the shell. It returns the number of cores in the computer it is running on. `--sshloginfile $PBS_NODEFILE` tells `parallel` all the nodes and their addresses that have been made available to it. Finally, `--workdir $PBS_O_WORKDIR` tells `parallel` the directory from where you are running the job.

Here is what `more-nodes.pbs` should look like.

```bash
$ cat more-nodes.pbs 
#!/bin/bash
#PBS -q express
#PBS -l nodes=2:ppn=12
#PBS -l walltime=1:00:00
#PBS -j oe

cd $PBS_O_WORKDIR
export DATA='/opt/examples/north-pacific-gyre/2017-02-23'
export RESULTS='/scratch/pdoehle/nelles-results'

module load gnu-parallel

time parallel -j $(nproc) --sshloginfile $PBS_NODEFILE --workdir $PBS_O_WORKDIR "bash goostats $DATA/{} $RESULTS/stats-{}" ::: $(ls $DATA)
```

## Submitting the Jobs

We're now ready to submit both serial and parallel jobs.

> What kind of a difference do you expect in speed of execution between the three jobs?

```bash
$ qsub do-stats.pbs
741880.mgmt1
$ qsub do-stats-parallel.pbs
741881.mgmt1
$ qsub more-nodes.pbs 
741886.mgmt1
```

> How do we come back later and check on the results?

Remember you can use the following commands to check the status of your jobs:
```bash
$ showq | grep yourusername
```

```bash
$ qpeek jobidno
```

# Not So Fast Nelle: Taking Advantage of Parallelism
We have certainly helped Nelle save a lot of time at this point, but our script can be run on her personal computer, and Nelle has access to high performance computing resources. While it's better than clicking 'OK' so many times, we have written a job that runs on one core, in serial. That means that each data file needs to wait for the file ahead of it to be completed in the loop before it gets processed. None of the files depend on each other so we can spread the work across many processor cores and get done faster.  How much faster do you think it can be? 

## GNU Parallel

Let's begin by cleaning up after our last script.

```bash
$ rm stats-*
```

In order to run our job in parallel, we are going to use the program GNU parallel (https://www.gnu.org/software/parallel/).  This program is useful for running lots and lots of smaller programs at the same time on multiple cores. In order to use `parallel`, we must first load its module.

```bash
$ module load gnu-parallel
```

Modules are a common tool for managing software on high performance clusters. This loads the right software so that we can use the command.

`parallel` takes a list of commands or files and passes each one out to available cores. In our case, we will pass the list of files ending in ‘A’ or ‘B’, and we will tell `parallel` that we want it to run `bash goostats filename stats-filename` on each one. `parallel` then diligently passes each task out to available cores. The command is as follows:

```bash
$ parallel “bash goostats {} stat-{}” ::: *[AB].txt
```

The first word in the command is `parallel`, followed by what we we want it to do on each core in quotations. Notice that `parallel` uses the special symbol `{}` to indicate: “whatever the next thing in the list is that you give me.” `parallel` also uses the special symbol `:::` which works similar to a pipe. It passes the list of things to `parallel`. In our case, we have a list of files that we give to `parallel` and it runs `goostats` on each one. This is similar to our loop except multiple files can be run at the same time. Notice the added bonus that the entire command fits on only one line.

Before we start a bunch of jobs on several cores, it would be nice to test out our command first, just like we did using `echo` in the loop. Fortunately, `gnu-parallel` has a flag for this. Let’s run it on the login node to see how it works.

```bash
$ parallel --dry-run “bash goostats {} stat-{}” ::: *[AB].txt
bash goostats NENE01729A.txt stat-NENE01729A.txt
bash goostats NENE01729B.txt stat-NENE01729B.txt
bash goostats NENE01736A.txt stat-NENE01736A.txt
bash goostats NENE01751A.txt stat-NENE01751A.txt
bash goostats NENE01751B.txt stat-NENE01751B.txt
bash goostats NENE01812A.txt stat-NENE01812A.txt
bash goostats NENE01843A.txt stat-NENE01843A.txt
bash goostats NENE01843B.txt stat-NENE01843B.txt
bash goostats NENE01978A.txt stat-NENE01978A.txt
bash goostats NENE01978B.txt stat-NENE01978B.txt
bash goostats NENE02018B.txt stat-NENE02018B.txt
bash goostats NENE02040A.txt stat-NENE02040A.txt
bash goostats NENE02040B.txt stat-NENE02040B.txt
bash goostats NENE02043A.txt stat-NENE02043A.txt
bash goostats NENE02043B.txt stat-NENE02043B.txt
```

## Writing a Parallel Script

Just like using `echo` with the loop, we now have a good idea what will run when we call `parallel`. It looks like we setup our command correctly. We are now ready to create our submit script. Let’s begin by copying our current submit script and using it to create a new parallel submit script.

```bash
$ cp do-stats.pbs do-stats-parallel.pbs
```

Open it in nano and delete the line we don’t need anymore: `bash do-stats-.sh *[AB].txt`.

In order to use the program `parallel`, we must first load the correct module. Add the following line after `cd $PBS_O_WORKDIR`:

```bash
module load gnu-parallel
```

We can now use the program `parallel` in our script. One of the great features of `parallel` is that every time a core finishes a job, `parallel` hands it a new one. To see a picture of how this works, click [here](https://www.biostars.org/p/63816/ "Parallel's Work Distribution"). Instead of arbitrarily splitting up the work, it hands each core a new job the moment it finishes its current task. Recall that each compute node on cowboy has two CPUs with six cores each. That means we have twelve cores to play with for on each node.

> How can we change our script so that we request one compute node and twelve cores on Cowboy?

Find the following line in `do-stats-parallel.pbs`:

```bash
#PBS -l nodes=1:ppn=1
```

Now change it to request one node with twelve cores.

```bash
#PBS -l nodes=1:ppn=12
```

Let’s add the `parallel` command to the end of the script with a timer.

```bash
$ time parallel “bash goostats {} stat-{}” ::: *[AB].txt
```

Let’s see what `do-stats-parallel.pbs` looks like now:

```bash
$ cat do-stats.pbs
#!/bin/bash
#PBS -q express
#PBS -l nodes=1:ppn=12
#PBS -l walltime=10:00
#PBS -j oe
cd $PBS_O_WORKDIR

module load gnu-parallel

time parallel  "bash goostats {} stats-{}" ::: *[AB].txt
```

Before submitting our script, it would be nice to know how many files we are working with. That way our timer has relevance. Let's figure out how many files we have.

> Can you use the commands we have learned so far to figure out how many files we will have?

```bash
$ ls *[AB].txt
NENE01729A.txt  NENE01736A.txt  NENE01751B.txt  NENE01843A.txt  NENE01978A.txt  NENE02018B.txt  NENE02040B.txt  NENE02043B.txt
NENE01729B.txt  NENE01751A.txt  NENE01812A.txt  NENE01843B.txt  NENE01978B.txt  NENE02040A.txt  NENE02043A.txt
$ ls *[AB].txt | wc -l
15
```

## Submitting the Job

Now we’re ready to submit the job.

```bash
$ qsub do-stat-parallel.pbs
730683.mgmt1
```

> How do we check on the status of our job?

```bash
$ showq | grep your-username
```

Once your job has finished, you should have the same results as when you ran the loop:

```bash
$ ls
do-stat-parallel.pbs     NENE01729B.txt  NENE01812A.txt  NENE01978A.txt  NENE02040B.txt  stats-NENE01729A.txt  stats-NENE01751B.txt  stats-NENE01978A.txt  stats-NENE02040B.txt
goodiff         NENE01736A.txt  NENE01843A.txt  NENE01978B.txt  NENE02040Z.txt  stats-NENE01729B.txt  stats-NENE01812A.txt  stats-NENE01978B.txt  stats-NENE02043A.txt
goostats        NENE01751A.txt  NENE01843B.txt  NENE02018B.txt  NENE02043A.txt  stats-NENE01736A.txt  stats-NENE01843A.txt  stats-NENE02018B.txt  stats-NENE02043B.txt
NENE01729A.txt  NENE01751B.txt  NENE01971Z.txt  NENE02040A.txt  NENE02043B.txt  stats-NENE01751A.txt  stats-NENE01843B.txt  stats-NENE02040A.txt
```

> How long did it take?

```bash
$ cat do-stat-parallel.pbs.o730683
```

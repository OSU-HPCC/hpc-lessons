# Nelle's Pipeline (In Class Exercises)
---

Nelle Nemo, a marine biologist, has just returned from a six-month survey of the [North Pacific Gyre](http://en.wikipedia.org/wiki/North_Pacific_Gyre "North Pacific Gyre"), where she has been sampling gelatinous marine life in the [Great Pacific Garbage Patch](http://en.wikipedia.org/wiki/Great_Pacific_Garbage_Patch "Great Pacific Garbage Patch"). She has 1520 samples in all, and now needs to:

1. Run each sample through an assay machine that will measure the relative abundance of 300 different proteins. The machine’s output for a single sample is a file with one line for each protein.
2. Calculate statistics for each of the proteins separately using a program her supervisor wrote called goostat.
3. Compare the statistics for each protein with corresponding statistics for each other protein using a program one of the other graduate students wrote called goodiff.
4. Write up results. Her supervisor would really like her to do this by the end of the month so that her paper can appear in an upcoming special issue of Aquatic Goo Letters.

It takes about half an hour for the assay machine to process each sample. The good news is that it only takes two minutes to set each one up. Since her lab has eight assay machines that she can use in parallel, this step will “only” take about two weeks.

The bad news is that if she has to run goostat and goodiff by hand, she’ll have to enter filenames and click “OK” 46,370 times (1520 runs of goostat, plus 300\*299/2 (half of 300 times 299) runs of goodiff). At 30 seconds each, that will take more than two weeks. Not only would she miss her paper deadline, the chances of her typing all of those commands right are practically zero.

In this lesson, we will explore what she should do instead. Nelle has asked you to use your expertise as a command line expert to help her speed up her research. We will use what we have learned in class thus far to work in groups and help speed up Nelle's process. We will also write a script for her so that she can easily repeat the process with less work next time.

# Setup
---

To start with, let's download Nelle's files and have a look around. Note: the `wget` link is posted in the [etherpad](http://tiger.hpc.okstate.edu/sites/etherpad/p/2017-spring-math2910 "Etherpad").

```bash
$ cd
$ wget http://swcarpentry.github.io/shell-novice/data/shell-novice-data.zip
--2017-02-07 09:03:05--  http://swcarpentry.github.io/shell-novice/data/shell-novice-data.zip
Resolving swcarpentry.github.io... 151.101.44.133
Connecting to swcarpentry.github.io|151.101.44.133|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 168355 (164K) [application/zip]
Saving to: “shell-novice-data.zip”

100%[===============================================================================================================================================>] 168,355     --.-K/s   in 0.07s   

2017-02-07 09:03:05 (2.20 MB/s) - “shell-novice-data.zip” saved [168355/168355]

$ unzip shell-novice-data.zip 
Archive:  shell-novice-data.zip
   creating: data-shell/
  inflating: data-shell/.bash_profile  
   creating: data-shell/creatures/
  inflating: data-shell/creatures/basilisk.dat  
  inflating: data-shell/creatures/unicorn.dat  
               
                 .
                 .
                 .

 extracting: data-shell/writing/thesis/empty-draft.md  
   creating: data-shell/writing/tools/
  inflating: data-shell/writing/tools/format  
   creating: data-shell/writing/tools/old/
 extracting: data-shell/writing/tools/old/oldtool  
 extracting: data-shell/writing/tools/stats  

$ cd data-shell/
$ ls
creatures  data  Desktop  molecules  north-pacific-gyre  notes.txt  pizza.cfg  solar.pdf  writing
```

# Nelle's Pipeline
---

>Use your commands for navigating files and directories to look around and see what kind of files Nelle has. What is the most interesting one to you? What files are relevant to us helping Nelle with make her research faster?

```bash
$ cd ~/data-shell
$ tree .
.
├── creatures
│   ├── basilisk.dat
│   └── unicorn.dat
├── data
│   ├── amino-acids.txt
│   ├── animals.txt
│   ├── elements
│   │   ├── Ac.xml
│   │   ├── Ag.xml
│   │   ├── Al.xml
│   │   ├── Am.xml
│   │   ├── Ar.xml
│   │   ├── As.xml
│   │   ├── At.xml
│   │   ├── Au.xml
│   │   ├── Ba.xml
│   │   ├── Be.xml
│   │   ├── Bi.xml
│   │   ├── Bk.xml
│   │   ├── Br.xml
│   │   ├── B.xml
│   │   ├── Ca.xml
│   │   ├── Cd.xml
│   │   ├── Ce.xml
│   │   ├── Cf.xml
│   │   ├── Cl.xml
│   │   ├── Cm.xml
│   │   ├── Co.xml
│   │   ├── Cr.xml
│   │   ├── Cs.xml
│   │   ├── Cu.xml
│   │   ├── C.xml
│   │   ├── Dy.xml
│   │   ├── Er.xml
│   │   ├── Es.xml
│   │   ├── Eu.xml
│   │   ├── Fe.xml
│   │   ├── Fm.xml
│   │   ├── Fr.xml
│   │   ├── F.xml
│   │   ├── Ga.xml
│   │   ├── Gd.xml
│   │   ├── Ge.xml
│   │   ├── He.xml
│   │   ├── Hf.xml
│   │   ├── Hg.xml
│   │   ├── Ho.xml
│   │   ├── H.xml
│   │   ├── In.xml
│   │   ├── Ir.xml
│   │   ├── I.xml
│   │   ├── Kr.xml
│   │   ├── K.xml
│   │   ├── La.xml
│   │   ├── Li.xml
│   │   ├── Lr.xml
│   │   ├── Lu.xml
│   │   ├── Md.xml
│   │   ├── Mg.xml
│   │   ├── Mn.xml
│   │   ├── Mo.xml
│   │   ├── Na.xml
│   │   ├── Nb.xml
│   │   ├── Nd.xml
│   │   ├── Ne.xml
│   │   ├── Ni.xml
│   │   ├── No.xml
│   │   ├── Np.xml
│   │   ├── N.xml
│   │   ├── Os.xml
│   │   ├── O.xml
│   │   ├── Pa.xml
│   │   ├── Pb.xml
│   │   ├── Pd.xml
│   │   ├── Pm.xml
│   │   ├── Po.xml
│   │   ├── Pr.xml
│   │   ├── Pt.xml
│   │   ├── Pu.xml
│   │   ├── P.xml
│   │   ├── Ra.xml
│   │   ├── Rb.xml
│   │   ├── Re.xml
│   │   ├── Rh.xml
│   │   ├── Rn.xml
│   │   ├── Ru.xml
│   │   ├── Sb.xml
│   │   ├── Sc.xml
│   │   ├── Se.xml
│   │   ├── Si.xml
│   │   ├── Sm.xml
│   │   ├── Sn.xml
│   │   ├── Sr.xml
│   │   ├── S.xml
│   │   ├── Ta.xml
│   │   ├── Tb.xml
│   │   ├── Tc.xml
│   │   ├── Te.xml
│   │   ├── Th.xml
│   │   ├── Ti.xml
│   │   ├── Tl.xml
│   │   ├── Tm.xml
│   │   ├── U.xml
│   │   ├── V.xml
│   │   ├── W.xml
│   │   ├── Xe.xml
│   │   ├── Yb.xml
│   │   ├── Y.xml
│   │   ├── Zn.xml
│   │   └── Zr.xml
│   ├── morse.txt
│   ├── pdb
│   │   ├── aldrin.pdb
│   │   ├── ammonia.pdb
│   │   ├── ascorbic-acid.pdb
│   │   ├── benzaldehyde.pdb
│   │   ├── camphene.pdb
│   │   ├── cholesterol.pdb
│   │   ├── cinnamaldehyde.pdb
│   │   ├── citronellal.pdb
│   │   ├── codeine.pdb
│   │   ├── cubane.pdb
│   │   ├── cyclobutane.pdb
│   │   ├── cyclohexanol.pdb
│   │   ├── cyclopropane.pdb
│   │   ├── ethane.pdb
│   │   ├── ethanol.pdb
│   │   ├── ethylcyclohexane.pdb
│   │   ├── glycol.pdb
│   │   ├── heme.pdb
│   │   ├── lactic-acid.pdb
│   │   ├── lactose.pdb
│   │   ├── lanoxin.pdb
│   │   ├── lsd.pdb
│   │   ├── maltose.pdb
│   │   ├── menthol.pdb
│   │   ├── methane.pdb
│   │   ├── methanol.pdb
│   │   ├── mint.pdb
│   │   ├── morphine.pdb
│   │   ├── mustard.pdb
│   │   ├── nerol.pdb
│   │   ├── norethindrone.pdb
│   │   ├── octane.pdb
│   │   ├── pentane.pdb
│   │   ├── piperine.pdb
│   │   ├── propane.pdb
│   │   ├── pyridoxal.pdb
│   │   ├── quinine.pdb
│   │   ├── strychnine.pdb
│   │   ├── styrene.pdb
│   │   ├── sucrose.pdb
│   │   ├── testosterone.pdb
│   │   ├── thiamine.pdb
│   │   ├── tnt.pdb
│   │   ├── tuberin.pdb
│   │   ├── tyrian-purple.pdb
│   │   ├── vanillin.pdb
│   │   ├── vinyl-chloride.pdb
│   │   └── vitamin-a.pdb
│   ├── planets.txt
│   ├── salmon.txt
│   └── sunspot.txt
├── Desktop
├── molecules
│   ├── cubane.pdb
│   ├── ethane.pdb
│   ├── methane.pdb
│   ├── octane.pdb
│   ├── pentane.pdb
│   └── propane.pdb
├── north-pacific-gyre
│   └── 2012-07-03
│       ├── goodiff
│       ├── goostats
│       ├── NENE01729A.txt
│       ├── NENE01729B.txt
│       ├── NENE01736A.txt
│       ├── NENE01751A.txt
│       ├── NENE01751B.txt
│       ├── NENE01812A.txt
│       ├── NENE01843A.txt
│       ├── NENE01843B.txt
│       ├── NENE01971Z.txt
│       ├── NENE01978A.txt
│       ├── NENE01978B.txt
│       ├── NENE02018B.txt
│       ├── NENE02040A.txt
│       ├── NENE02040B.txt
│       ├── NENE02040Z.txt
│       ├── NENE02043A.txt
│       └── NENE02043B.txt
├── notes.txt
├── pizza.cfg
├── solar.pdf
└── writing
    ├── data
    │   ├── one.txt
    │   └── two.txt
    ├── haiku.txt
    ├── old
    ├── thesis
    │   └── empty-draft.md
    └── tools
        ├── format
        ├── old
        │   └── oldtool
        └── stats

14 directories, 194 files
```

Looks like the files that are relevant to Nelle's research are in the `north-pacific-gyre` folder.

> Can you use a command we have discussed to search for the `north-pacific-gyre` folder?

```bash
$ find . -name 'north-pacific-gyre'
./north-pacific-gyre
```

As we discussed above, Nelle has run her samples through the assay machines and created 1520 files in the `north-pacific-gyre/2012-07-03` directory. Each file has been saved as a text file. If there were no mistakes with the assay machine (almost never happens in real life), each text file will be 300 lines long.

> Assuming there really are 1520 text files in `north-pacific-gyre`, can you use pipes to create a workflow that checks to make sure we don't have files that are too long or too short? Hint: we have several commands that may be useful. They include: `wc`, `sort`, `head`, and `tail`.

```bash
$ cd north-pacific-gyre/2012-07-03/
$ ls *.txt
NENE01729A.txt  NENE01736A.txt  NENE01751B.txt  NENE01843A.txt  NENE01971Z.txt  NENE01978B.txt  NENE02040A.txt  NENE02040Z.txt  NENE02043B.txt
NENE01729B.txt  NENE01751A.txt  NENE01812A.txt  NENE01843B.txt  NENE01978A.txt  NENE02018B.txt  NENE02040B.txt  NENE02043A.txt
$ wc -l *.txt | sort -n | head -n 5
  240 NENE02018B.txt
  300 NENE01729A.txt
  300 NENE01729B.txt
  300 NENE01736A.txt
  300 NENE01751A.txt
$ wc -l *.txt | sort -n | tail -n 5
  300 NENE02040B.txt
  300 NENE02040Z.txt
  300 NENE02043A.txt
  300 NENE02043B.txt
 5040 total
```

Using the idea of 'Pipes and Filters,' we see that there was one file that was too short. When Nelle goes back and checks the file, she sees that she did the assay at 8:00 on a Monday morning. Somebody was probably using the machine on the weekend, and she forgot to reset it. You also notice that, when you were looking for files that were too long, one of the files ended with a 'Z':

```bash
$ wc -l *.txt | sort -n | tail -n 5
  300 NENE02040B.txt
  300 NENE02040Z.txt
  300 NENE02043A.txt
  300 NENE02043B.txt
 5040 total
```

When you ask Nelle about it, she tells you that all files should end with either 'A' or 'B', but that her lab uses the convention of ending files with a 'Z' to indicate samples with missing information.

> Nelle needs to know all the files that end with 'Z' so that she can determine which files are missing information. Can you find all the files that end with 'Z'?

```bash
$ ls *Z.txt
NENE01971Z.txt  NENE02040Z.txt
```

With this information in hand, Nelle checks her laptop and finds that there is no depth recorded for either of these two samples. You suggest using `rm` to delete these files, but she informs you that there is other analysis that can be done with these files for other projects. This means you will need to remember that for now, you only want to use files that end in 'A' or 'B'. One strategy for doing this is to use the wildcard expression `*[AB].txt`. Let's see how that works.

```bash
$ ls *[AB].txt
NENE01729A.txt  NENE01736A.txt  NENE01751B.txt  NENE01843A.txt  NENE01978A.txt  NENE02018B.txt  NENE02040B.txt  NENE02043B.txt
NENE01729B.txt  NENE01751A.txt  NENE01812A.txt  NENE01843B.txt  NENE01978B.txt  NENE02040A.txt  NENE02043A.txt
```

# Processing Files
---

Nelle is now ready to begin processing her files. Recall that Nelle wants to use the `goostats` program to analyze each file. The `goostats` program is a bash script that takes two arguments. First, you tell it the name of the file that you want it to analyze and then you tell it the name of the output file that you want it to write the results to. For example, if you were to use `goostats` to analyze `NENE01729A.txt` and write the results to a file called `stat-NENE01729A.txt`, you would type the following:

```bash
$ bash goostats NENE01729A.txt stat-NENE01729A.txt
```

> Show Nelle how to build up a loop that will analyze all of the files. Make sure you use `echo` and demonstrate how one can build up a loop step by step until it you are sure that it will behave correctly.

```bash
$ for datafile in *[AB].txt
> do
> echo $datafile
> done
NENE01729A.txt
NENE01729B.txt
NENE01736A.txt
NENE01751A.txt
NENE01751B.txt
NENE01812A.txt
NENE01843A.txt
NENE01843B.txt
NENE01978A.txt
NENE01978B.txt
NENE02018B.txt
NENE02040A.txt
NENE02040B.txt
NENE02043A.txt
NENE02043B.txt

$ for datafile in *[AB].txt; do echo $datafile stat-$datafile; done
NENE01729A.txt stat-NENE01729A.txt
NENE01729B.txt stat-NENE01729B.txt
NENE01736A.txt stat-NENE01736A.txt
NENE01751A.txt stat-NENE01751A.txt
NENE01751B.txt stat-NENE01751B.txt
NENE01812A.txt stat-NENE01812A.txt
NENE01843A.txt stat-NENE01843A.txt
NENE01843B.txt stat-NENE01843B.txt
NENE01978A.txt stat-NENE01978A.txt
NENE01978B.txt stat-NENE01978B.txt
NENE02018B.txt stat-NENE02018B.txt
NENE02040A.txt stat-NENE02040A.txt
NENE02040B.txt stat-NENE02040B.txt
NENE02043A.txt stat-NENE02043A.txt
NENE02043B.txt stat-NENE02043B.txt

$ for datafile in *[AB].txt; do echo bash goostats $datafile stat-$datafile; done
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
## Nelle's Pipeline: Creating a Script

An off-hand comment from her supervisor has made Nelle realize that
she should have provided a couple of extra parameters to `goostats` when she processed her files.
This might have been a disaster if she had done all the analysis by hand,
but thanks to `for` loops,
it will only take a couple of hours to re-do.

But experience has taught her that if something needs to be done twice,
it will probably need to be done a third or fourth time as well.
She runs the editor and writes the following:

```
# Calculate reduced stats for data files at J = 100 c/bp.
for datafile in "$@"
do
    echo $datafile
    bash goostats -J 100 -r $datafile stats-$datafile
done
```

(The parameters `-J 100` and `-r` are the ones her supervisor said she should have used.)
She saves this in a file called `do-stats.sh`
so that she can now re-do the first stage of her analysis by submitting this to the cluster:

> Create a job submission script named `do-stats.pbs` that will run the script we just made:

```bash
#!/bin/bash
#PBS -q express
#PBS -l nodes=1:ppn=1
#PBS -l walltime=10:00
#PBS -j oe
cd $PBS_O_WORKDIR

bash do-stats.sh *[AB].txt
```

And submit the job:

```
$ qsub do-stats.pbs
```

Want to see what's going on with your job? Use the `qpeek` or `showq` commands     

```
$ qpeek jobidnumber
$ showq | grep yourusername
```
    
Once your job completes, you can now view the contents of your `2012-07-03`directory and you'll notice a few changes!


Good work! Look at all the time we are going to save Nelle by using the command line instead of having to constantly click 'OK'. Next we'll see how she can speed things up even more by making use of multiple processors. 


# Lesson Objectives

* Discuss Unix's 'small pieces, loosely joined' philosophy.
* Students will be able to use pipes to combine exisiting programs and create more complex work flows.
* Students will be able to automate repetitive tasks using loops.
* Students will be able to use `grep` and `find` to find things.

___

Now that we've seen how Cowboy's directories are set up and have learned a few basic commands, we will now start working on a real-world HPC Bioinformatics problem involving genome assembly:
* [Bioinformatics](http://hpcc.okstate.edu/bioinformatics) is a field that is seeing rapid and growing use of HPC resources. Bioinformatics analyzes increasingly larger and more complex sets of biological data, i.e. genomics, cancer and antibiotics research.    

Our bioinformatics exercise involves looking at mutated genotypes, gene sequencing, and genome assembly:
* **Genotype:** the DNA sequence of the genetic makeup of a cell/organism/individual
     * Genotypes are one of three factors that determine phenotype (a specific characteristic of that cell/organism/individual - i.e. the petal color in a plant)
     * DNA mutations that are acquired versus inherited are not part of the individual's genotype - scientists often then look at the "genotype" of the mutated cell 
* **Gene sequencing:** the process of determining the order of the four bases (adenine, guanine, cytosine, and thymine) in a strand of DNA
* **Genome assembly:** the genome sequence produced after chromosomes have been fragmented, those fragments sequenced, and those sequences put back together.     

#### Goals:
* Use **pipes & filters** to determine the amount of reads in a sequence file
* Use **loops** to test smaller components of your job across multiple files before you run it
* Use **'grep'** and **'find'** to search for patterns to determine the PHRED score
____

## Let's get started:
The first thing we need to do is change directories (`cd`) from our home directory to mcbios directory that we put in our scratch directory last week.

```bash
$ cd /scratch/username/mcbios
```

* Note we're using the absolute path this time to do this.

Remember: you can use `pwd` to confirm you are there: 
```bash
$ pwd
/scratch/username/mcbios
```

Now that we're in our mcbios directory, we can type `ls` (and add the -F option) to _list_ the contents of that directory and distinguish what are directories and what are files:
```bash
$ ls -F
abyss/  data/  results/  soap/  velvet/
```

For today, we want to work with files in the `data/` directory. Before we move into that directory, let's take a peek at what's in it. 

```bash
$ tree data/
```
![alt text](https://github.com/HPC-classes/class-materials/blob/master/lesson_plans/img/tree_data.png)

You can see the `data/` directory contains five subdirectories each containing some .fastq files and .fasta files. 

Before we move forward, let’s change into the 'group1' directory within the 'data' directory with `cd` and the _relative path_. 

```bash
$ cd data/group1/
```

___

### Bioinformatics Pipeline: Checking Files
Let's look at the contents of a .fastq file to get a better idea of what kind of data we're working with. 

Last week we used the 'cat' command to view the contents of some .txt files we created. While the 'cat' command is useful, it has the disadvantage that it always dumps the whole file onto your screen. For longer files, we want to use the command 'less':

```bash
$ less PE-350.1.fastq
```

The 'less' command allows you to view the contents of a large file without having it all load at once. This displays a screenful of the file, and then stops. 
* The ':' at the bottom of the screen indicates there is more content. 
* You can go forward one screenful by pressing the spacebar, or back one by pressing b. 
* Press q to quit. 

We can also get a peek at how the file is structured by using the 'head' command.
* You can specify an amount of lines to look at with the 'head' command by using a dash (-) and a number after. I.e. `head -10 PE-350.1.fastq` would give you the first 10 lines of the PE-350.1.fastq file.

```bash
$ head PE-350.1.fastq
  @DRR001841.41/1
AAAAGAATGGAAATCTATGTTTTTATTATTACAAGTTTTGAAGATTGCCAAAGAAATCAAGAATTTCGTGAGATTGAAAGTCATCGGGTC
+
CCCCCCBBCCCCCCCCCCCCCCCBBBBBCCCCCCCCCCCCCCCCCCCCCCCCCCCB@CCCCCCCCCCCBCCCCACCCCABA>@CCAB6<B
@DRR001841.45/1
ACGACTTGGTACATTGTCCCTCAATAACTTTATTTGAATCGATCCCCACGGAAGTGCGGTCATTCTACGAAGACGAAAAGTCTGGCCTAA
+
CCCCCCCCCCCCCCCCDCCCCCCCCCCCCCCCCC@BCCCCCC=CCCCCCCABCB>CCBCBCABBCCCCCAACDCCBAC<C=C<CAAC5B=
@DRR001841.58/1
GTAAGGTTTGCAGGTGATAGGACAAGTCAATTCAATTTTACCACCTTCTATTTGTAGATCGTCTTCGTCTGCGGGGTTTTGCAGATCCGG
```

Four consecutive lines of a fastq define one sequence "read". The output is structured as follows:   
* @DRR001841.41/1 (**Line 1:**): READID
* AAAAGAATGGAAATCTAT ... (**Line 2:**): A sequence of base pairs
* + (**Line 3:**): READ-ID proceded by a `+` sign 
* CCCCCCBBCCCCCCCCCCCC ... ( **Line 4:**): Quality score identifiers 

Quality identifiers (or characters) for each base will be the same length as the sequence in line 2. We will learn more about this later.

___

#### What does all this mean for our bioinformatics researcher? 
Well, first we can confirm how many reads are in the PE-350 library for each file. We can do this by determining the number of lines in the .fastq files and dividing that by 4 (the number of lines in each read).
* 'wc' stands for word count and on it's own will give you the word count, line count and character count of a file. 
* the '-l' flag attached to it tells it to only give you the line count. Type `wc --help` to see more flags you can use with the 'wc' command.

```bash
$ wc -l PE-350.1.fastq
  381536 PE-350.1.fastq
```
Then we can divide 381536 by 4 right?  If it doesn't divide by 4 evenly then something is wrong with the fastq file.


#### What if we wanted to figure out the number of reads in all the .fastq files in the group1 directory? One way we can start to do this is to use a wildcard.

```bash
$ wc -l *.fastq
  381536 PE-350.1.fastq
  381536 PE-350.2.fastq
  763072 total
```
This gives us the amount of lines in each file and then we can still easily divide by 4 in our heads to get our number of reads.

> ##### Wildcards

> \* is a wildcard. It matches zero or more characters, so \*.fastq matches PE-350.1.fastq, PE-350-2.fastq, and every file that ends with ‘.fastq’. On the other hand, \*2.fastq only matches PE-350.2.fastq, because the ‘2’ after the \* only matches filenames that end in 2.fastq.

> ? is also a wildcard, but it only matches a single character. This means that PE-350.?.fastq would match both PE-350.1.fastq and PE-350.2.fastq files (and any other file with a single digit number between PE-350 and fastq), but would not match a file named PE-350.25.fastq. 

We can go back to our `data` directory and even take a look at ALL the line counts for ALL the .fastq files in ALL the group directories.

```bash
$ cd ..
$ pwd
/scratch/username/mcbios/data
$ wc -l group*/*.fastq
   381536 group1/PE-350.1.fastq
   381536 group1/PE-350.2.fastq
   380484 group2/PE-350.1.fastq
   380484 group2/PE-350.2.fastq
   296396 group3/PE-350.1.fastq
   296396 group3/PE-350.2.fastq
   491052 group4/PE-350.1.fastq
   491052 group4/PE-350.2.fastq
   440868 group5/PE-350.1.fastq
   440868 group5/PE-350.2.fastq
```

As you can see, as we look at more files, it's going to be harder to divide by 4 in our head each time.


### But...we could have the command line do all the arithmetic for us:

##### Pipes
The vertical bar, |, between the two commands is called a pipe. It tells the shell that we want to use the output of the command on the left as the input to the command on the right. 
```bash
$ expr $(cat group1/PE-350.1.fastq | wc -l) / 4
  95384
```

This is exactly like a mathematician nesting functions like log(3x) and saying “the log of three times x”. In our case, the calculation is “the expression of (word count by line of the concatenated contents of PE-350.1.fastq) divided by 4”.  (Note that this is integer division.)

When the shell executes this command, the first thing it does is run whatever is inside the $(). It then replaces the $() expression with that command’s output. 
* The output of 'cat' would be all the contents of the PE-350.1.fastq file. 
* Then the pipe would do a word count by line (wc -l) on the file. 
* Then outside of the $() we provide the integer arithmetic function of dividing by 4.
* Finally, the result of this is sent back to the "expr" command and it expresses the result of 95384 reads.


> Pipes & Filters: This simple idea is why Unix has been so successful. Instead of creating enormous programs that try to do many different things, Unix programmers focus on creating lots of simple tools that each do one job well, and that work well with each other. This programming model is called “pipes and filters”. We’ve already seen pipes; a filter is a program like wc or sort that transforms a stream of input into a stream of output. Almost all of the standard Unix tools can work this way: unless told to do otherwise, they read from standard input, do something with what they’ve read, and write to standard output.

___

## Loops

___


Now that we've seen that we can use "pipes and filters" to string multiple commands together to make more complex work flows, using wildcards we should be able to run jobs across multiple files and see how many reads are in EACH file, correct? Let's try:

```bash
$ expr $(cat group*/*.fastq | wc -l) / 4
995168
```

We get one big number as the output to our arithmetic problem INSTEAD of outputs for each file. Why do you think this is?

We are trying to run 'cat' on multiple files, so it concatenates all contents of all files together and does a word count on the total number of lines in all the files. So how do we solve this problem?

### Enter Loops

Loops allow us to execute commands repetitively and are good for running commands over multiple files. Similar to "wildcards" and "TAB completion", loops also allow us to reduce the amount of typing we do, therefore reducing the amount of typos! 

> QUICK RECAP: We tried earlier to look at the line counts of a few files and divide by 4 to determine the amount of reads in each file. While it was easy to do file by file for a few files, we soon discovered that the more files we work with (and Bioinformatics researchers work with **A LOT** of files), it didn't make sense to keep doing it this way.

> We figured out how we could have the computer do the arithmetic for us, but then our 'cat' command combined the contents of all the files together to do the word count on. This didn't help us much.

```bash
$ for filename in group*/*.fastq
> do
>   expr $(cat $filename | wc -l) / 4
> done
95384
95384
95121
95121
74099
74099
122763
122763
110217
110217
```

Yes! This worked. We get the number of reads in each file listed separately instead of all together. But what if we want to know what filename corresponds to each number of reads? Press the up-arrow to pull up the loop. You'll see it has the same content of our loop we just created, but is formatted a little differently on the command line. Using the cursor arrows, navigate over to add in another command (echo $filename) to execute over the files.
* Why did we put 'filename' and '$filename' in our loop??? In the first line of the loop we establish a name for our variable, which in this case, we've named it filename. A variable stands in place for any number of files that fit your established parameters (i.e. group*/*.fastq)
* In the third line, where we we started typing our commands, we used the $ to call the variable --> $filename --> and the command runs through through each item that matches in the list. 

```bash
$ for filename in group*/*.fastq; do echo $filename; expr $(cat $filename | wc -l) / 4; done
group1/PE-350.1.fastq
995168
group1/PE-350.2.fastq
995168
group2/PE-350.1.fastq
995168
group2/PE-350.2.fastq
995168
group3/PE-350.1.fastq
995168
group3/PE-350.2.fastq
995168
group4/PE-350.1.fastq
995168
group4/PE-350.2.fastq
995168
group5/PE-350.1.fastq
995168
group5/PE-350.2.fastq
995168
```

That is a little easier to read. But we can even add another command in this loop to make it even MORE easy to read. Notice as you add commands, you will add a semicolon (;) before or after each new command depending on where you placed it in your loop. There is NOT a semicolon in between "do" and the first command in a loop.

```bash
$ for filename in group*/*.fastq; do echo $filename; expr $(cat $filename | wc -l) / 4; echo " "; done
group1/PE-350.1.fastq
995168

group1/PE-350.2.fastq
995168

group2/PE-350.1.fastq
995168

group2/PE-350.2.fastq
995168

group3/PE-350.1.fastq
995168

group3/PE-350.2.fastq
995168

group4/PE-350.1.fastq
995168

group4/PE-350.2.fastq
995168

group5/PE-350.1.fastq
995168

group5/PE-350.2.fastq
995168
```

Obviously if you're running a loop on thousands of files, you may not want to include all these commands (it would take up a lot of space on your screen), but it's useful to know that you can add commands to loops and test each command along the way before adding another one. This process can be useful when testing on a few files before you run on thousands!

___

## Finding Things - using "grep" and "find"

If you remember from the beginning of the lesson, we said we were going to come back to the fourth line in a read - the quality identifier line. As a reminder, recal the first few lines of the fastq file.

```bash
$ head group1/PE-350.1.fastq 
@DRR001841.41/1
AAAAGAATGGAAATCTATGTTTTTATTATTACAAGTTTTGAAGATTGCCAAAGAAATCAAGAATTTCGTGAGATTGAAAGTCATCGGGTC
+
CCCCCCBBCCCCCCCCCCCCCCCBBBBBCCCCCCCCCCCCCCCCCCCCCCCCCCCB@CCCCCCCCCCCBCCCCACCCCABA>@CCAB6<B
@DRR001841.45/1
ACGACTTGGTACATTGTCCCTCAATAACTTTATTTGAATCGATCCCCACGGAAGTGCGGTCATTCTACGAAGACGAAAAGTCTGGCCTAA
+
CCCCCCCCCCCCCCCCDCCCCCCCCCCCCCCCCC@BCCCCCC=CCCCCCCABCB>CCBCBCABBCCCCCAACDCCBAC<C=C<CAAC5B=
@DRR001841.58/1
GTAAGGTTTGCAGGTGATAGGACAAGTCAATTCAATTTTACCACCTTCTATTTGTAGATCGTCTTCGTCTGCGGGGTTTTGCAGATCCGG
```

The fourth line in any given read is a list of quality scores. There is a score for each base pair (called [PHRED scores](https://en.wikipedia.org/wiki/Phred_quality_score)) and it uses a special encoding system in order to save memory. There are two PHRED encoding systems: PHRED+33, and PHRED+64. The following key shows how they are encoded:

```bash
...............................IIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIII......................
SSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSSS....................................................
!"#$%&'()*+,-./0123456789:;<=>?@ABCDEFGHIJKLMNOPQRSTUVWXYZ[\]^_`abcdefghijklmnopqrstuvwxyz{|}~
|                   |          |                   |                  |                     |
33                  53         64                  84                 104                   126
```

*Notice that the counting starts with 33 at the first character: `!`. `"` is 32, `#` is 33, and so on.* 

1. For each base pair in the fastq file, we can look up the quality score on this table and read the associated number. 
2. We then subtract from this number based on which system we are using. 
3. If we are using PHRED+33, we subtract 33, and if we are using PHRED+64, we subtract 64. 
4. This process produces our final PHRED score. 
5. We can then use the following table to look up the accuracy of the sequencer for that base pair:

Phred Quality Score | Probability of Incorrect Base Pair | Base Call Accuracy |
--- | --- | --- 
10 | 1 in 10 | 90% |
20 | 1 in 100 | 99% |
30 | 1 in 1000 | 99.9% |
40 | 1 in 10000 | 99.99% |
50 | 1 in 100000 | 99.999% |

The PHRED+33 system is common with the Illuminia PE-350 library, but sometimes the PHRED+64 is used. We vneed to be aware of whether we are using PHRED+33 or PHRED+64. 

Looking at our 'cipher' again, notice that PHRED+33 (symbols with `S` over them) take up the first part of the 'cipher', while PHRED+64 (symbols with `I` over them) take up the second part of the 'cipher'. There are a small number of symbols shared by both. 

In order to determine which system we are using, we can search through the quality scores in the `.fastq` files for symbols that only belong to one system of the other.

The command for searching file content is `grep`. Let's search `PE-350.1.fastq` for the `>` character to see if we are using PHRED+33.

```bash
$ cd group1
$ grep '>' PE-350.1.fastq
```

Wow! That's a lot of information. That's because grep returns every line in the file where it found `>`. Since this is not a quality score used by PHRED+64, it's safe to say we are using PHRED+33. Let's make our search output more human-friendly by using **pipes and filters** to count the number of lines where '>' appears instead of printing out each line.

```bash
$ grep '>' PE-350.1.fastq | wc -l
23781
```

We piped all that output to word count (wc), which counted the number of lines for us. Just to be safe, let's make sure there are no scores that belong **only** to the PHRED+64 system in our file by using a symbol located only under an `I` on the cipher.

```bash
grep 'L' PE-350.1.fastq | wc -l
0
```

`grep` is a powerful search tool for searching the content of files. `find`, on the other hand, allows us to search through our file system for different files. Recall that earlier, we are working with several different groups of data.

```bash
$ cd ..
$ ls
group1  group2  group3  group4  group5
```

We can search through all of the groups in order to find all of the `.fastq` files.

```bash
$ find . -name '*.fastq'
./group2/PE-350.1.fastq
./group2/PE-350.2.fastq
./group1/PE-350.1.fastq
./group1/PE-350.2.fastq
./group3/PE-350.1.fastq
./group3/PE-350.2.fastq
./group5/PE-350.1.fastq
./group5/PE-350.2.fastq
./group4/PE-350.1.fastq
./group4/PE-350.2.fastq
```

We can even combine the two search commands in order to look for a certain base pair sequence across all of our data. Let's see how many times the sequence `AAAAGAATGGAAA` appears across all of the fastq files.

```bash
$ grep 'AAAAGAATGGAAA' $(find . -name *.fastq) | wc -l 
126
```

This will 1) find all the files with \*.fastq, 2) look through those files for the string "AAAAGAATGGAAA", and 3) provide a word count for all the lines containing that string. 

__

## FASTQC EXERCISE

Now that we've 1) confirmed the number of reads in our .fastq files and 2) checked the accuracy probabilities of our base pairs using the PHRED score, **how good is the sequencing run?**

We can use an application called FastQC Let's start by doing a fastqc exercise for one file: 
```bash
$ pwd
/scratch/<username>/mcbios/data
$ cd group1
$ ls
PE-350.1.fastq  PE-350.2.fastq  ref.fasta
$ module load fastqc
```

To learn more about FastQC type:
```bash
$ fastqc -h
```

The description toward the top of the output reads:
```bash
DESCRIPTION

    FastQC reads a set of sequence files and produces from each one a quality
    control report consisting of a number of different modules, each one of 
    which will help to identify a different potential type of problem in your
    data.
    
    If no files to process are specified on the command line then the program
    will start as an interactive graphical application.  If files are provided
    on the command line then the program will run with no user interaction
    required.  In this mode it is suitable for inclusion into a standardised
    analysis pipeline.
```

For the purposes of the exercise, we are going to include a filename. 
```bash
$ fastqc PE-350.1.fastq
Started analysis of PE-350.1.fastq
Approx 5% complete for PE-350.1.fastq
Approx 10% complete for PE-350.1.fastq
Approx 15% complete for PE-350.1.fastq
Approx 20% complete for PE-350.1.fastq
Approx 25% complete for PE-350.1.fastq
Approx 30% complete for PE-350.1.fastq
Approx 35% complete for PE-350.1.fastq
Approx 40% complete for PE-350.1.fastq
Approx 45% complete for PE-350.1.fastq
Approx 50% complete for PE-350.1.fastq
Approx 55% complete for PE-350.1.fastq
Approx 60% complete for PE-350.1.fastq
Approx 65% complete for PE-350.1.fastq
Approx 70% complete for PE-350.1.fastq
Approx 75% complete for PE-350.1.fastq
Approx 80% complete for PE-350.1.fastq
Approx 85% complete for PE-350.1.fastq
Approx 90% complete for PE-350.1.fastq
Approx 95% complete for PE-350.1.fastq
Analysis complete for PE-350.1.fastq
```

You will get an .html output file in your group1 directory
```bash
$ ls group1
PE-350.1.fastq        PE-350.1_fastqc.zip  ref.fasta
PE-350.1_fastqc.html  PE-350.2.fastq
```
___
# Last Step!!!!

#### Now let's show you how to move that .html off Cowboy onto your own computer because you can't view it on Cowboy.

1. Go to the [HPCC File Transfer Page](https://hpcc.okstate.edu/content/uploading-and-downloading-files-0)
2. Choose the file transfer application that is right for you. 
* In class, we are going to use WinSCP, but if you use FileZilla or CyberDuck, you can follow the instructions on the [HPCC File Transfer Page](https://hpcc.okstate.edu/content/uploading-and-downloading-files-0).

#### Once you move the .html file off of Cowboy onto your Desktop, you can open the file and view the webpage with your quality control results!!

To learn more about what your results mean, you can refer to this [Introduction to FastQC PDF](https://biof-edu.colorado.edu/videos/dowell-short-read-class/day-4/fastqc-manual)

Or watch this <a href="https://www.youtube.com/watch?v=GnWSXwQeJ_U
" target="_blank"><img src="http://img.youtube.com/vi/GnWSXwQeJ_U/0.jpg" 
alt="FastQC Youtube Video" width="240" height="180" border="10" /></a>



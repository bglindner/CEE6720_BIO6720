# CEE/BIO 6720: Laboratory 1
## Exercises on Average Nucleotide Identity (ANI) 

## Preparation (these must be done before class):
1.	Log in to LinkedIn Learning (using your Gatech credentials) and complete the “Unix Essential Training” course by Kevin Skoglund (https://www.linkedin.com/learning/unix-essential-training/). Budget about 1-2 hours for this and feel free to skip Chapter 6 for now (or any other sections, based on your familiarity with Unix). 
2.	Much of this course involves remote computing on a cluster with a Unix-based operating system. Please review PACE’s materials on the Instructional Cluster Environment (ICE) where we will be completing most of our laboratory exercises this semester. Please do so even if you are already familiar with other PACE clusters; ICE has some unique features and different computational resources compared to the others (i.e., Phoenix, Hive).
3.	Configure a working terminal on your system. MacOS users have a native terminal. Windows users can quickly obtain a standalone and lightweight terminal via MobaXterm (https://mobaxterm.mobatek.net/). 
4.	Log in to ICE.
5.	Create a Github user account.

## Purpose: 
To introduce students to computing in a Unix environment, PACE’s ICE cluster, and the concept of average nucleotide identity (ANI) as the mean of a distribution of sequence similarities between two genomes. 

Note: This laboratory assignment will contain step-by-step instructions to assist you in completing it. Future assignments will contain less verbose instructions. Kostas and the course TA are always available for questions should you run into issues. 

## Outline: 
This exercise is intended to be completed in two class sessions with the following goals:

•	Review Unix and remote computing basics.

•	Run the BLAST algorithm on the whole genome sequences of the two *Shewanella* strains. 

•	Understand the structure of BLAST tabular outputs and filter alignments appropriately.

•	Manually compute and visualize ANI between the two *Shewanella* strains from BLAST outputs. 

•	Practice using a dedicated tool, fastANI, to compute ANI automatically. 

•	We will scale up our ANI analysis using fastANI to cluster a set of genomes into species.

•	Discuss how ANI can be used to view recent horizontal gene transfer events.

# Instructions:

## **Step 1:** 
Review Unix and remote computing basics.
Please respond to the following before beginning the exercise and include your responses in your final assignment. One to two sentences of response will suffice for each.
1.	What command does your user use to log into ICE?
2.	What is the difference between login nodes and compute nodes?
3.	Should you compute on a login node? Why or why not?
4.	How does a user request time on a compute node?
5.  Define "memory", "nodes", and "cores" in the context of remote computing.
6.	What is a job scheduler? What type does ICE use?
7.  What is "scratch storage" and how long do files persist there?
8.  What command cancels a job on a compute node?
9.  Should I ask the course TA or PACE's helpdesk for software assistance during this course?

## **Step 2:** 
Log into ICE and configure your workspace.
1.	On your local machine, log on to PACE’s ICE cluster. `ssh username@login-ice.pace.gatech.edu`
2.  Ensure you have access to the course's shared directory. If when you call `ls` you only see `scratch/` then add our shared directory as a new symlink with `ln`: `ln -s /storage/ice-shared/cee6720 shared`
3.  Find the data for today's exercise in `shared/lab01/` and copy it into your `scratch` directory using `cp`
4.  You should have two sets of data, one folder containing *Shewanella* data (`shewy_data`) and the other containing 10 genomes (`10genomes/`)

## **Step 3:** 
Run BLAST both ways i.e., genes of OS185 => genome of OS195 and then genes of OS195 => genome of OS185. 
1. Request an interactive job from the scheduler. Adjust the following as you see fit: `salloc -N 1 --ntasks-per-node=2 --mem=32G -t 1:00:00`
2. Review core software modules available with `module avail` and load the most recent version of `blast-plus` with `module load modulename/version-number`. Note: Advance `module avail` with `return` line-by-line and press `q` if you would like to exit `module avail` prematurely. 
3. Verify you have access to `blastn` with `blastn -h`. If not, reload the BLAST+ module. 
4. Query the genes of one strain against the genome of the other and then repeat in the other direction. Do so with `blastn`: `blastn -subject genome.fna -query genes.fna -out output.tsv -outfmt 6`
5. Ensure you have two separate tabular outputs, one for each direction (genes1 => genome2, genes2 => genome1).

## **Step 4:**
Filter and visualize the results of both BLAST runs.
1. Import both tabular outputs as dataframes into the workspace of your choice (e.g., `R`, `jupyter`, Excel, etc.). NEVER copy and paste data from the command line to elsewhere! See the command `scp`, which when called from your local workspace offers a quick solution: `scp username@login-ice.pace.gatech.edu:remote_path/to/file.txt local_path/to/file.txt`. See also dedicated apps like, Globus, MobaXterm, VS Code, etc. 
2. Ensure that each query **gene** aligns at most to one section of the subject **genome**. In instances of multiple alignments, select the best scoring alignment and remove any secondary alignments. You may script this or do it manually, as you prefer. 
3. Prepare histograms for each output and compute their mean as ANI.
4. Be sure to include these histograms in your final report.

## **Step 5:**
Use `FastANI` to automatically generate ANI values.
1. Create an environment and install `FastANI` with `conda` or `mamba` (there will be prompts after some of these calls so don't run this as a single block of code):
```
module load anconda3
conda create -n fastANI
conda activate fastANI
conda install -c bioconda fastANI
```
2. Use `FastANI` to produce ANI values for OS185:OS195 and OS195:OS185. Pay attention to the tool's documentation -- input to `FastANI` are genomes not genes.
3. Be sure to include a comparison between the ANI values found manually and by `FastANI` in your final report. Did you observe any differences? If so, why might that be?

## **Step 6:**
Scaling up: examining ANI values for a collection of genomes.
1. In the data for session 2, you will find 10 genomes (`genome01.fna`,...`genome10.fna`). Using `FastANI` compute the ANI between all genomes. 
2. Using these ANI values, prepare a similarity matrix. Pay attention to the tool's usage guidelines, there is an optional output parameter that will make this much easier.
3. Cluster the rows and/or columns of your simiarlity matrix and visualize as you would like.
4. Be sure to include a visualization of this similarity matrix in your final report. Also, use this visualization to hypothesize which genomes belong to the same species and/or genus.

# Assignment Reporting:
Email Kostas a lab report. Briefly respond to the following conclusion questions at the end of your report:

1. What are the differences between a FASTA file containing a complete genome sequence and a FASTA file containing just gene sequences?
2. Please compare and contrast the concept of percent identity of a sequence alignment with the ANI metric.
3. Both *Shewanella* strains have been observed to carry plasmids. Did the FASTA files for OS185 and OS195 contain only chromosomal DNA or were these plasmids included? Should or should we not include extra-chromosomal elements in ANI calculations?
4. Say you accidentally and permanently deleted the genomes provided to you for use in this exercise (perhaps through uncautious use of `rm -r`) and that they have also been removed from the `shared` directory. How would you obtain them independently?
5. What are the column names of a BLAST tabular produced by `-outfmt 6`? 
6. Approximately, what is the lowest ANI that `FastANI` will report? Why do you think the tool has a lower limit?

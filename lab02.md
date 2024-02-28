# CEE/BIO 6720: Laboratory 2
## Comparative metagenomics with short reads (part 1)


## Purpose: 

In this exercise, we'll be comparing the composition of metagenomic datasets across nucleotide, ecological, and taxonomic diversities. Six metagenomic datasets recovered from timeseries monitoring of a freshwaster lake have been provided, which were captured during the winter and summer of 2010, 2012, and 2014. 

Short read datasets have dominated metagenomic studies for almost two decades. A substantial amount of information can be extracted from short read datasets without the need to assemble (or provide longer readers from other platforms). The goal of this exercise is to acquaint users with simple worklows utilizing only short reads with the aim to provide a short overview of easy methods for basic steps involved in comparative metagenomics:

1. Assessing alpha diversity (`nonpareil`)
2. Assessing beta diversity (`simka`)
3. Taxonomic profiling with both 16S fragments (`vsearch` + `silva`) and *k*-mers (`kraken2`)

These exercises will also acquaint users with `sbatch` jobs and utilizing shared databases for database-dependent tools. 

## Outline: 

•	Step 0: Set up your workspace and retrieve software.

•	Step 1: Trim raw reads with `fastp`

•	Step 2: Assess and visualize alpha diversity with `nonpareil`

•	Step 3: Assess and visualize beta diversity with `simka`

•	Step 4: Create taxonomic profiles for all samples based on 16S sequence fragments (`vsearch` and `silva`) and/or all short reads (`kraken2`)

•	Compare samples between years (2010, 2012, 2014) and seasons (summer and winter)

# Instructions:

## **Step 0: Workspace preparation** 

1.  Install `micromamba`:
```
curl -Ls https://micro.mamba.pm/api/micromamba/osx-64/latest | tar -xvj bin/micromamba
~/bin/micromamba shell init -s bash -p ~/micromamba
source ~/.bashrc
# ensure you have it working:
micromamba -h
```
2.  Once you have a working deployment of `micromamba`, let's update its config file. Open `nano ~/.condarc` and copy+paste the following: 
```
channels:
  - conda-forge
  - bioconda
  - defaults
channel_priority: flexible
```
3.  Copy `shared/lab02` to your `scratch` directory (`cp -r ~/shared/lab02 ~/scratch/`)
4.  Create an environment with the software we need:
```
# if your command line can't recognize micromamba, try: source ~/.bashrc
micromamba create -p ~/scratch/lab02/env
micromamba activate ~/scratch/lab02/env
micromamba install fastp kraken2 nonpareil simka vsearch
```
5.  You'll want to clean up after installing all of those packges
```
micromamba clean --all -y
```

## **Step 1: Read trimming** 
Now that we've set up our working directory and installed the needed software, let's start with our first step: read trimming. This is a quality control step designed to ensure that the sequences (reads) we use in subsequent steps have high quality base calls, are devoid of any adapter sequences, etc.
1.  Take a look inside the `sbatch` script provided for the first step (`step01.sbatch`). If you've made an exact copy of the `lab02` data from `shared` and installed the software in an environment therein called `env`, you should be able to get right to work.
2.  Run `step01.sbatch`. See usage guides inside the script itself (`cat step01.sbatch`) . You'll need to launch this step 6 times, one for each metagenomic read pair.
3.  Monitor the job IDs you are given. You can examine your account's running jobs with: `squeue -u username`
4.  Pay attention to any failed jobs -- clean them up as you need to and re-launch jobs.
5.  Ensure you have all of the expected outputs: trimmed reads (forward and reverse), `.html`, and `.json`.
6.  You should download the `.html` outputs for each set of paired reads and open a few locally (with Chrome or Firefox) for a nice visual summary of each metagenome's quality.

## **Step 2: Assessing alpha diversity** 
With our short reads successfully trimmed, let's assess nucleotide diversity with `nonpareil`. We'll run the tool and then take its outputs into `R` to quickly produce a visual and summary dataframe.
1.  As you did for step 1, run `step02.sbatch` according to its usage instructions.
2.  While your jobs run, prepare a `manifest.tsv` dataframe for your samples which we'll use with `nonpareil` below. It should have this format:
```
files	labels	colors
path/to/sample1.npo	sample1	color1
path/to/sample2.npo	sample2	color1
path/to/sample3.npo	sample3	color2
path/to/sample4.npo	sample4	color2
```
Note: be careful copy+pasting this -- it's better to re-create it yourself so that you know the whitespace is properly formatted as tabs. Keep the header names the same. You can use this opporunity to pick colors for your samples, to color them either individually or by group (i.e. season or year).

3.  Once your nonpareil jobs are finished, we can read our information in to `R`:
```
# make sure your environment is active. If needed: micromamba activate ./env

R

library("Nonpareil")
getwd()
# you should be in your ~/scratch/lab02 directory. If not, consider setwd() or exit R and cd there

### generate plots
manifest = read.table(file="manifest.tsv",header=TRUE)
files = as.character(manifest$files)
labels= as.character(manifest$labels)
colors= as.character(manifest$colors)
curves= Nonpareil.set(files,colors,labels)

### extract summary info and write to CSV
info = summary.Nonpareil.Set(curves)
write.csv(info, "nonpareil_results.csv", row.names=FALSE)

### write plot to PDF
pdf("nonpareil_curves.pdf", width=12, height=6)
Nonpareil.set(files,colors,labels,plot.opts=list(plot.diversity=FALSE))
dev.off()
```
4.  Transfer both `nonpareil_results.csv` and `nonpareil_curves.pdf` to your local workspace. 

## **Step 3: Assessing beta diversity**
Assessing beta diversity

1.  Similarly to `nonpareil`, we want to prepare a table for use by `simka`. In this case, we need this table before running `simka`, so let's prepare it like so (note the lack of headers and use only spaces for whitespace here):
```
sampleA: path/to/sampleA.1.fastq ; path/to/sampleA.2.fastq
sampleB: path/to/sampleB.1.fastq ; path/to/sampleB.2.fastq
sampleC: path/to/sampleC.1.fastq ; path/to/sampleC.2.fastq
```
2.  Similarly to steps 1 and 2, launch `simka` with its own batch script (`step03.sbatch`). This will process the entire dataset as a batch, so you only need to launch this once.
3.  Collect PCA plots from the output folder and transfer them to your local workspace. 

## **Step 4: Profiling taxonomic composition**
Taxonomic profiling.

Under construction

## Discussion

In your laboratory report, please provide:
   
•	both the `nonpareil` summary table and visualized curves.
  
•	the Bray-Curtis PCA produced by `simka`
    
•	a taxonomic profile for the entire dataset based on 16S data or all short reads.

Please respond to the following at the end of your report:

1.  List the names of the tools we used in this exercise and a link to their documentation.
2.  Briefly: What is the method `nonpareil` uses to estimate alpha diversity (and a metagenomic samples coverage of it)? What is the method `simka` uses to estimate beta diversity?
3.  Researchers involved in this study would like to ensure future sequencing runs capture >=75% of the expected nucleotide diversity for both summer and winter samples. What sequencing effort (in GB) would you recommend to accomplish this? Does seasonality have any impact on your suggestions?
4.  Regardless of which tool(s) you opted to run, what is the difference between the approaches we used with `vsearch` and `kraken2` for generating taxonomic profiles?
5.  Do you observe any trends between the winter and summer samples? What about across years? Please describe your answer and highlight visualizations to support your conclusions, as necessary.  


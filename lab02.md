# CEE/BIO 6720: Laboratory 2
## Comparative metagenomics with short reads

## Preparation (these must be done before class on Feb 26):
1.  Review the meaning of alpha and beta diversity. 
2.  Briefly familiarize yourself with the documentation and usage guidelines for the following tools/databases:
* fastp
* nonpareil
* simka
* vsearch
* kraken2

## Purpose: 
Short read datasets have dominated metagenomic studies for almost two decades. A substantial amount of information can be extracted from short read datasets without the need to assemble (or provide longer readers from other platforms). The goal of this exercise is to acquaint users with basic comparative metagenomic worklows utilizing only short reads. The aim is to review
1. Alpha diversity (`nonpareil`)
2. Beta diversity (`simka`)
3. Taxonomic profiling with both 16S fragments (`vsearch` + `silva`) and k-mers (`kraken2`)

Six metagenomic datasets recovered from timeseries monitoring of a freshwaster lake have been provided, which were captured during the winter and summer of 2010, 2012, and 2014. We will assess high level differences between the compositions of these samples. 

## Outline: 

•	Copy raw metagenomic reads from `shared/lab02`
•	Step 1: Trim raw reads with `fastp`
•	Step 2: Assess and visualize alpha diversity with `nonpareil`
•	Step 3: Assess and visualize beta diversity with `simka`
•	Step 4: Create taxonomic profiles for all samples based on 16S sequence fragments (`vsearch` and `silva`) or all short reads (`kraken2`)
•	Compare samples between years (2010, 2012, 2014) and seasons (summer and winter)

# Instructions:

## **Step 0:** 
Set up your workspace:
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
3.  Copy `shared/lab02` to your `scratch` directory
4.  Create an environment with the software we need:
```
# if your command line can't recognize micromamba, try: source ~/.bashrc
micromamba create -p ~/scratch/lab02/env
micromamba activate ~/scratch/lab02/env
micromamba install fastp kraken2 nonpareil simka vsearch
```

...

## **Step 1:** 
Now that we've set up our working directory and installed the needed software, let's start with our first step: read trimming. This is a quality control step designed to ensure that the sequences (reads) we use in subsequent steps have high quality base calls, are devoid of any adapter sequences, etc.
1.  Configure your copy of the `sbatch` script `00_qc_fastp.sbatch`
2.  Launch `fastp` from your `sbatch` script providing the forward and reverse raw reads. You can pass in variables to `sbatch` calls using a comma-delimited list: `sbatch --export var1=string,var2=string script.sbatch`
3.  Monitor the job IDs you are given for completion. You can examine your account's running jobs with: `squeue -u username`
4.  Pay attention to any failed jobs -- clean them up as you need to and re-launch jobs.
5.  Ensure you have all of the expected outputs: trimmed reads (forward and reverse), `.html`, and `.json`.

## **Step 2:** 
With our short reads successfully trimmed, let's assess nucleotide diversity with `nonpareil`. We'll run the tool and then take its outputs into `R` to quickly produce a visual and summary dataframe.
1.  Configure your copy of the `sbatch script` 01_a-div_nonpareil.sbatch`
2.  Launch and monitor jobs as you did for step 1. `nonpareil` will output several files, which we will need the `.npo` file specifically. 
3.  While your jobs run, prepare a `manifest.tsv` dataframe for your samples which we'll use with `nonpareil` below. It should have this format:
```
files	labels	colors
path/to/sample1.npo	sample1	color1
path/to/sample2.npo	sample2	color1
path/to/sample3.npo	sample3	color2
path/to/sample4.npo	sample4	color2
```
Note: be careful copy+pasting this -- it's better to re-create it yourself so that you know the whitespace is properly formatted as tabs. Keep the header names the same.
4.  Once your nonpareil jobs are finished, we can read our information in to `R`:
```
module load R
R
# make sure your working directory is where you think it is!
getwd()
#
manifest = read.table(file="manifest.tsv",header=TRUE)
files = as.character(manifest$files)
labels= as.character(manifest$labels)
colors= as.character(manifest$colors)
library("Nonpareil")
curves = Nonpareil.set(files,colors,labels,plot.opts=list(plot.diversity=False))
pdf(curves, width=8, height=6)
info = summary.Nonpareil.Set(curves)
write.csv(info, "nonpareil_results.csv", row.names=FALSE)
```


## **Step 3:**
Assessing beta diversity
Filter and visualize the results of both BLAST runs.
1.  Configure your copy of the `sbatch script` 01_b-div_simka.sbatch`
2.  Prepare a text file listing the input files and their sample IDs for use by `simka`
3.  Launch `simka` with your `sbatch` script providing the input file prepared in step 2.
4.  Collect Bray-Curtis PCoA visuals from the output folder.

## **Step 5:**
Extract and classify 16S sequences 
1. 

## **Step 6:**

1. In `10genomes`, you will unsurprisingly find 10 genomes (`genome01.fna`,...`genome10.fna`). Using `FastANI` compute ANI values for each pair. You can do this individually or provide the tool a list of genomes (i.e., run `FastANI` all vs all). 
2. Using these ANI values, construct a similarity matrix. Pay attention to the tool's documentation, there is an optional output parameter that could make this much easier.
3. Cluster the rows and/or columns of your simiarlity matrix and visualize as you would like. Do you observe any clustering patterns? 
4. Be sure to include a visualization of this similarity matrix in your final report. Also, use this visualization to hypothesize which genomes belong to the same species and/or genus.

# Assignment Reporting:
Email Kostas a lab report. Briefly respond to the following conclusion questions at the end of your report:

1. 
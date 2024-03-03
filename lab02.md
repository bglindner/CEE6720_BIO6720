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
curl -Ls https://micro.mamba.pm/api/micromamba/linux-64/latest | tar -xvj bin/micromamba
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

### write plot to PDF (this will be saved to your workdir as "Rplots.pdf")
Nonpareil.set(files,colors,labels,plot.opts=list(plot.diversity=FALSE))

```
4.  Transfer both `nonpareil_results.csv` and `nonpareil_curves.pdf` to your local workspace. 

## **Step 3: Assessing beta diversity**
We also want to examine trends in beta diversity between all samples. We'll prepare an input file for batch processing of all our samples at once (since we're examining beta diversity, this makes sense), hand it off to `simka` which will estimate distances between samples according to several ecological indices and do some dimensionality reductions for us to visualize it. 

1.  Similarly to `nonpareil`, we want to prepare a table for use by `simka`. In this case, we need this table before running `simka`, so let's prepare it like so (note the lack of headers and use only spaces for whitespace here):
```
sampleA: path/to/sampleA.1.fastq ; path/to/sampleA.2.fastq
sampleB: path/to/sampleB.1.fastq ; path/to/sampleB.2.fastq
sampleC: path/to/sampleC.1.fastq ; path/to/sampleC.2.fastq
```
2.  Similarly to steps 1 and 2, launch `simka` with its own batch script (`step03.sbatch`). This will process the entire dataset as a batch, so you only need to launch this once.
3.  Collect PCA plots from the output folder and transfer them to your local workspace. 

## **Step 4: Profiling taxonomic composition**
We would like to create taxonomic profiles for our samples as part of this last step. Below, we will create taxonomic profiles from either 16S fragments or all short reads. Our approach using all the short reads involves the tool `kraken2` which uses a large whole genome database whereas our 16S approach uses `vsearch` to find reads containing 16S sequences by comparing reads to the `SILVA` (SSU) database (akin to "closed or reference-based OTU picking"). Both of these approaches generate tables which you will need to manage in order to generate visuals.

1. Review the associated scripts ( `step04_vsearch.sbatch` or `step04_kraken2.sbatch`). Both workflows will generate output files as either a matrix (`vsearch`; in `mothur` format) or a long format dataframe (`kraken2`).

2. Once all your `vsearch` jobs have finished, you can examine the slurm log to get information on the run. Next, combine the outputs into a single file:
```
head -1 04_taxonomic_profiles/2010-summer.mothur > 04_taxonomic_profiles/all.mothur
for file in 04_taxonomic_profiles/*.mothur; do tail -n 1 ${file} >> 04_taxonomic_profiles/all.mothur; done 
```
2. (cont.) You have now created a single table representing the OTUs as columns and the samples as rows. From here, you can use the `SILVA_taxonomy_legend.tsv` provided to you in order to ascertain the taxonomy of OTUs. We recommend reading `all.mothur` into into your preferred data visualization platform, adding the taxonomy data from the legend provided and removing columns with all zeroes -- these are OTUs represented in the SILVA database but that were not identified in any of the samples.

3. Once all your `kraken2` jobs have finished, you can examine the slurm log to get information on the run. The workflow uses `kraken2` to do profiling and then cleans up the resulting relative abundances with `bracken`. You should have results for species, genus, class, and phylum -- try to prepare visuals for at least genus and class. You should find the following useful in visualizing your results: https://github.com/rotheconrad/Kraken-Bracken-plot

4. Do your best to prepare visuals summarizing the taxonomic profiles using the results from both tools. Be prepared to compare and contrast their results at the same taxonomic rank (e.g., `vsearch` class to `kraken2` class, etc.)


# Results

1.  List the names of the tools we used in this exercise and a link to their documentation.
2.  Report the number of reads in each sample before and after running `fastp`.
3.  What values did you find for Nonpareil Diversity for each sample?
4.  How many 16S reads did `vsearch` find for each sample?
5.  Across the entire dataset, how many total OTUs were detected at least once?
6.  On average, how many OTUs were detected in the summer samples? Winter samples?
7.  What percentage of the short reads was `kraken2` able to identify in each sample?
8.  What was the single most abundant species at each time point according to `kraken2`?

Additionally, please provide the following in your lab report:
   
•	both the `nonpareil` summary table and visualized curves.
  
•	the "Abundance" Bray-Curtis PCA produced by `simka` (and any other visual results from the tool which you liked)
    
•	a taxonomic profile for the entire dataset based on 16S data or all short reads.

# Discussion

Please respond to the following at the end of your report -- feel free to reference your visualizations and the results you reported above to assist you:

1.  What is the method `nonpareil` uses to estimate alpha diversity (and a metagenomic samples coverage of it)? What is the method `simka` uses to estimate beta diversity?
2.  Researchers involved in this study would like to ensure future sequencing runs capture >=75% of the expected nucleotide diversity for both summer and winter samples. What sequencing effort (in GB) would you recommend to accomplish this? Does seasonality have any impact on your suggestions?
3.  Briefly compare and contrast our two approaches for taxonomic classification. What are the strengths or weaknesses of each. Would you expect differences in their results? If so, from what?
4.  A reviewer asks you whether a recently discovered freshwater microbial species is present in your dataset and at what abundance. The new species genome was reported after you submitted your manuscript. Do you think these methods would have detected it? Explain why or why not.
5.  Do you observe any trends between the winter and summer samples? What about across years? Please describe your answer and highlight visualizations to support your conclusions, as necessary.  


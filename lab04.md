# CEE/BIO 6720: Laboratory 4
## Manual annotation and phylogenetics

## Purpose: 

In this exercise we will learn to build a (16S) phylogenetic tree and how to identify and functionally annotate the genes encoded on an assembled contig. Finally, we will learn how to assess horizontal gene transfer of genes based on phylogenetic trees.


## Outline: 

•	Step 1: Build phylogenetic trees with 16S rRNA gene sequences.

•	Step 2: Predict genes on fosmid sequence.

•	Step 3: Annotate predicted genes from the fosmid sequence.

•	Step 4: Find whole genomes matching each of the given 16S rRNA gene sequences.

•	Step 5: mcr tree construction.


# Instructions:

## **Step 0: preparation** 

1.  The steps in this exercise can be completed with a lightweight interactive job. No `.sbatch` scripts are provided.

## **Step 1: 16S tree building** 
1.  Copy the 16S rRNA sequences provided in `~/shared/lab04`
2.  Use SILVA's online `sequence search` webtool to classify each of the 10 16S sequences provided (https://www.arb-silva.de/aligner/).
3.  Find the genus (and species, if possible) associated with each sequence.
4.  Construct phylogenetic trees using the provided 16S sequences either with SILVA's online treebuilding tool (see link in step 2), `phylip`, MEGA (on your local workspace), or any other better tree building tool of your choice (e.g., `IQ-TREE`, `FastTree`).

NOTE: If you elect to use `phylip`, you can store the directory containing its executables as a variable like so:

`phylip="/storage/ice-shared/cee6720/00_software/phylip/phylip-3.697/exe"`

And then you can call them:
```
${phylip}/dnadist
${phylip}/dnaml
```
You are not required to use `phylip` in this exercise but it is provided to you since we have not introduced a tree building tool hitherto.

5.  Gather one tree for each of the following methods with a minimum of 100 bootstraps each: distance, parsimony, and maximum like. Visualize these trees with labels for the underlying method and the bootstrap support for each node.

## **Step 2: Fosmid gene prediction** 
1.  Copy the fosmid sequence prodiced in `~/shared/lab04`
2.  Obtain `pyrodigal`: 
```
micromamba clean --all -y
micromamba create -n pyrodigal
micromamba activate pyrodigal
micromamba install -c bioconda pyrodigal
```
3.  Predict genes for the fosmid sequence using `pyrodigal`.

## **Step 3: Fosmid gene annotation** 
1.  Using the tool of your choice, annotate the predicted genes of the fosmid sequence. If doing so with webtools, consider UniProt or NCBI as discussed in class. If using the command line, consider `bakta` from the previous exercise which will automate all steps -- you can examine its tabular output (`.tsv`) to find a good summary of this information.
2.  Report all annotations. Most importantly, identify a metal reducing operon (*mcr*), report its location in the sequence and the operon's length.

## **Step 4: Whole genome collection** 
1.  For each 16S rRNA gene provided, find 1 whole genome sequence ("complete" level not "contig" or "scaffold") on NCBI that carries a matching 16S rRNA gene (100% identity).
2.  Download these 10 whole genomes and be sure to track which genome belongs to which 16S sequence.
3.  Examine which of these 10 genomes also contain the *mcr* operon using a method of your choice. 

## **Step 5: mcr gene tree** 
1.  Extract each of the *mcr* operons from each genome encoding it and build a gene tree using the method of your choice.


# Results

1.  This exercise is more open-ended than previous ones. Please summarize here what methods you used to accomplish each step. 
2.  For each 16S sequence you classified in step 1, provide a summary of the ecological or biotechnological importance of that genus or species.
3.  Compare and contrast the 3 16S rRNA gene trees you constructed. Do their topologies differ? Did you note differences in computational time for their construct? Do you observe any patterns in bootstrap values?
4.  Summarize your annotation of the fosmid sequence. How many coding sequences do you predict? What are their functions? Did your annotation efforts uncover an pseudogenes on the fosmid sequence?
5.  For the 10 whole genome representatives you selected as representatives for each of the 10 16S rRNA sequences, which encoded *mcr*? Report the amino acid identity of these *mcr* operons to the `mcr` operon encoded on the fosmid.
6.  Compare the tree you constructed in step 5 to the tree of your choice from step 1. How might these results inform researchers about the occurence of horizontal gene transfer of the *mcr* operon?

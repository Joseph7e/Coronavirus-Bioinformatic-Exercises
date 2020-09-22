# Coronavirus Bioinformatic Exercises


## Investigating the origin of the novel 2019 coronavirus, COVID-19


To start this analysis we will download some reference covid-19 genomes that were originally isolated and sequenced at the beginning of the outbreak. One was isolated from Wuhan and the other from Florida. These two genomes will be considered our 'unknown' novel isolates. We will combine these 'unknown' sequences with a set of 'known' coronavirus genomes sequenced in the past. These 'known' sequences include coronaviruses from diffrent genera (Figure 1) that were isolated form various hosts, including civets, bats, humans, and more. 

We will use various methods to investigate which 'known' sequences are closest (phylogenetically) to our novel covid-19 sequences.

<img src="https://cmr.asm.org/content/cmr/28/2/465/F7.large.jpg" width="520">
source:https://cmr.asm.org/content/28/2/465/F7


The following manuscript discusses the origin of the novel 2019 coronavirus and includes NCBI accessions for various coronaviruses genomes. This dataset includes a good variety of coronaviruses and will be used as our starting point. 

Li, C., Yang, Y., & Ren, L. (2020). Genetic evolution analysis of 2019 novel coronavirus and coronavirus from other species. Infection, Genetics and Evolution, 104285.

## Samples
The list of genome accessions can be found in the manuscripts supplementary table. 

https://ars.els-cdn.com/content/image/1-s2.0-S1567134820301167-mmc1.pdf


I converted it a tab delimited text file and uploaded it to this github repo.


## Copy starting data and setup working directory

The starting data can be copied from a shared directory on the server. Each genome is found in an individual FASTA file with filenames and FASTA headers uniquely identified in the sample sheet. Be sure to review the FASTA file format.

The 'unknown' COVID-19 sequences include 'UNKNOWN' as part of their filename and sequence header.

```bash
mkdir ~/coronavirus-analysis
cd ~/coronavirus-analysis
cp -r /home/genome/joseph7e/coronavirus_genomes/ ./

# view the first few lines of each FASTA file
head coronavirus_genomes/*.fasta

```

## Optional step 1: Download other coronavirus genomes from NCBI. These can be more 'unknown' covid-19 genomes or 'known' sequences from various hosts.

The following page from NCBI includes a table to ~60 Coronaviridae genome assemblies. These all include isolation sources and download links. 


https://www.ncbi.nlm.nih.gov/genomes/GenomesGroup.cgi?taxid=11118


To retrieve the sequence, click on the NCBI accession number (formatted like "NC_XXXXXXXX") to bring up the sequence page. On the upper right is a drop down menu called "Send to:" Change "Choose Destination" to "File", change the format from "GenBank" to "FASTA(text)", and Create. You'll need to transfer the file to RON. Alternatively, instead of using the "Send to" function, you can change the format on the upper right of the page from "GenBank" to "FASTA(text)" and copy the sequence to your clipboard. You can then simply paste the sequence into a text editor (i.e. nano) on the server and save the file.

Be sure to make the filename and FASTA header for the sequence something meaningful and include it in the "coronavirus_genomes/” directory we created above.

## Optional step 2: **de novo** assembly of a covid-19 genome.

Here I will show you how to download data from the Sequence read archive. I simply searched for covid-19 in the SRA https://www.ncbi.nlm.nih.gov/sra and skimmed through for an Illumina paired-end dataset. 

The first match was an Illumina NovaSeq paired-end dataset from Sept. 19th 2020.

https://www.ncbi.nlm.nih.gov/sra/ERX4536357[accn]

Once you find a dataset, you'll want to take note of the 'Run' accession, in this case its ERR4604210.

Read the help menu to determine what the options that I use do.

```bash
cd ~/coronavirus-analysis
mkdir new_reads
cd new_reads

# Download the data to RON, we can specify the number of reads ("spots") we want to download.
fastq-dump --help
fastq-dump --gzip --split-files -X 1000000 ERR4604210
```

After you download the data you can follow the same bacterial assembly workflow as before. Just be sure to be mindful of potential options you may want to change in each program. For example, use 'Virus' as the kingdom for PROKKA.

** If there is interest, I can go through an download a bunch of datasets, create a sample sheet, and include them in a shared directory on RON **


## Combine all the data into a single FASTA file. 

```bash
cd ~/coronavirus-analysis
cat coronavirus_genomes/* > coronas.fasta

# examine the new file
less -S coronas.fasta

```

## Average Nucleotide Identity (ANI)

The first analysis we'll run calculates the average nucleotide identity. This will give us pairwise statistics showing the % similarity of two genomes.
The program fastANI calculates ANI between two or more genomes by using kmer frequencies and the MinHASH algorithm.
https://github.com/ParBLiSS/FastANI

```bash
fastANI --help

# run a one v one comparison
fastANI -q coronavirus_genomes/NC_045512-UNKNOWN.fasta -r coronavirus_genomes/AY515512.fasta -o NC_045512_v_AY515512.txt --fragLen 200

# gather a list of all the genomes
readlink -f coronavirus_genomes/* > all_genomes.txt

# run one vs all
fastANI -q coronavirus_genomes/NC_045512-UNKNOWN.fasta --rl all_genomes.txt -o NC_045512_v_all.txt --fragLen 200

```

Results:
We are simply looking to see which 'known' sequence has the highest ANI score for each 'unknown' sequence.

## reciprocal best BLAST
BLAST is a very common bioinformatic tool. It's the primary program behind a lot of bioinformatics software. See the genome assembly workflow for a description.

In this instance we will BLAST all the coronavirus genomes against themselves, both the query and the subject are the same FASTA file. The output is a tab delimited file showing the top closest matching sequences for each genome sequence.

```bash
blastn -query coronas.fasta -subject coronas.fasta -outfmt 6 -out self-blast-format6.txt
```

## Whole genome alignment

```bash
# align data using mafft, this can take some time!
mafft --thread 18 coronas.fasta > mafft-coronas.aln
```

## Phylogenetic inference

```bash
#construct phylogeny using fasttree
fasttree -nt mafft-coronas.aln > fasttree-coronas.tre

# construct phylogeny using raxml
raxmlHPC -m GTRCAT --bootstop-perms=100 -T 48 -s mafft-coronas.aln -n raxml-tree.tre -p 7
```


## View and edit the phylogenetic tree with the Iterative Tree of Life (IToL).
https://itol.embl.de/


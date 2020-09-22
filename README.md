# Coronavirus Bioinformatic Exercises


## Investigating the origin of the novel 2019 coronavirus, COVID-19


To start this analysis we will download some reference covid-19 genomes that were originally isolated and sequenced at the beginning of the outbreak. One was isolated from Wuhan and the other from Florida. These two genomes will be considered our 'unknown' novel isolates. We will combine these 'unknown' sequences with a set of 'known' coronavirus genomes sequenced in the past. These 'known' sequences include coronaviruses from diffrent genera (Figure 1) that were isolated form various hosts, including civets, bats, humans, and more. 

We will use various methods to investigate which 'known' sequences are closest (phylogenetically) to our novel covid-19 sequences.

<img src="https://cmr.asm.org/content/cmr/28/2/465/F7.large.jpg" width="520">


The following manuscript discusses the origin of the novel 2019 coronavirus and includes NCBI accessions for various coronaviruses genomes. This dataset includes a good variety of coronaviruses and will be used as our starting point. 
https://www.sciencedirect.com/science/article/pii/S1567134820301167?casa_token=UcRnVCfmY1QAAAAA:eJDSlVlUjLLm_xs5ZCFhhSgoLDhEbvNsB7wqjt8e7iN3EjvFRZL74jiKyN09BEjYPEhlz7PO#t0005


## Samples
The list of genome accessions can be found in this supplementary table. https://ars.els-cdn.com/content/image/1-s2.0-S1567134820301167-mmc1.pdf


I converted it a tab deliminated text file and uploaded it to this github repo.


## Copy starting data and setup working directory

```bash
mkdir ~/coronavirus-analysis
cd ~/coronavirus-analysis
cp -r /home/genome/joseph7e/coronavirus_genomes/ ./

# combine genomes into a single fasta file
cat coronavirus_genomes/* > coronas.fasta

```

## Optional step 1: Download other coronavirus genomes from NCBI. These can be more 'unknown' covid-19 genomes or 'known' sequences from various hosts.


## Optional step 2: **de novo** assembly of a covid-19 genome.

## Average Nucleotide Identity (ANI)

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
## reciprical best BLAST
Query and Subject are the exact same. 

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

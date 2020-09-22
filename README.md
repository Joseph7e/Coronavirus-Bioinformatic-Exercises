# Coronavirus Bioinformatic Exercises

## Optional assembly of a coronavirus genome

In the next set of exercises we will download one of the covid-19 genomes 
Data on NCBI ...




## Investigating the origin of the novel 2019 coronavirus, COVID-19

This manuscript discusses the origin of the novel 2019 coronavirus. 
https://www.sciencedirect.com/science/article/pii/S1567134820301167?casa_token=UcRnVCfmY1QAAAAA:eJDSlVlUjLLm_xs5ZCFhhSgoLDhEbvNsB7wqjt8e7iN3EjvFRZL74jiKyN09BEjYPEhlz7PO#t0005

I am going to use the same starting genome sequences but use my own methods to determine phylogeny etc.


<img src="https://cmr.asm.org/content/cmr/28/2/465/F7.large.jpg" width="520">



## Samples
The list of genome accessions can be found in this supplementary table. I converted it a tab deliminated text file and uploaded it to ron.
https://ars.els-cdn.com/content/image/1-s2.0-S1567134820301167-mmc1.pdf

## Copy starting data and setup working directory

```bash
mkdir ~/coronavirus-analysis
cd ~/coronavirus-analysis
cp -r /home/genome/joseph7e/coronavirus_genomes/ ./

# combine genomes into a single fasta file
cat coronavirus_genomes/* > coronas.fasta

```

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

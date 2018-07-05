# MDIBL-T3-WGS-Comparative

## Our Starting data

```bash
ls /home/maineBK/shared_data/filtered_genome_assemblies/
```


Organism       |  Our Data      | NCBI-Assemblies | NCBI-Genomes
-------------  | -------------- | -------------   | ------------
Herbaspirillium| 4              | 34    | 12
Rhizobium      | 1              | 309    | 62
Rothia         | 1              | 76    | 5
Kocuria        | 1              | 57   | 13
Streptomyces   | 13             | 1,185    | 338

## What we will be doing

We will be using **Orthofinder** for our main comparative genomic analysis. The manual is great and documents how it works very well. To run the program we will need some genomes to compare, specifically we will need proteins files.

Orhtofinder Manual: https://github.com/davidemms/OrthoFinder

The program takes a set of protein sequences for each species and runs pair-wise comarisons to identify orhtologous groups. For each orthogroup a gene tree is calculated and in the end an overall species tree is computed. To get any sort of meaningful phylogenetic tree we need to be sure to include **at least four different genome datasets**. Ideally we would run this program with all of the avaialble sequences on NCBI. As you can imagine, a pairwise comparison with 1,185 Streptomyces genomes will take a lone time (days). We will therfore run it with a reduced set. Next we will determine what genomes we want to download and go over the best ways to retrieve them from NCBI.


## Set up working directories
```bash
cd ~/mdibl-t3-WGS/
mkdir orthofinder-analysis
cd orthofinder-analysis/
mkdir genbank_downloads
cd genbank_downloads/
```

## Locate Reference Data on NCBI

FAQs about genome download from NCBI: https://www.ncbi.nlm.nih.gov/genome/doc/ftpfaq/#GBorRS

Whatever method you use be sure to grab an outgroup, or don't thats your call.

### Method 1: Download genomes based on 16S BLAST results

We ran a NCBI blast during the genome assembly tutorial. This BLAST should have given you the closest match against the nt database. Chances are it 'hit' well to many genomes. Choose the top hit to a full genome and follow the links to retriev the download link of the genome FAA from the ftp site.


### Method 2: Download all refseq genomes for your genus

link to NCBI prokaryote tables: https://www.ncbi.nlm.nih.gov/genome/browse#!/prokaryotes/
link to genome reports FTP: ftp://ftp.ncbi.nlm.nih.gov/genomes/GENOME_REPORTS/

* Download the genome report file for all all of prokaryotes

We will download the file directly to the server, there is no need to download it to your computer. Right lick on the link and copy the link address. This is a big file so we will filter it a bit first before opening it with tabview.

```bash
# download the file
wget "ftp://ftp.ncbi.nlm.nih.gov/genomes/GENOME_REPORTS/prokaryotes.txt"
# grep for species in question and view.
grep -i "Streptomyces" prokaryotes.txt | tabview -
```

This file has a lot of useful information. For now we really care about column 21 which is the downloa link for the genome on the ftp site. Copy that link and paste it into a browser to see the files. We will then download the FAA files to the server.

```bash
# download the link
wget "ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/000/009/765/GCA_000009765.2_ASM976v2/*.faa.gz"
# repeat until we have all the genomes we want (I waould write a script to do this"
wget "ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/000/091/305/GCA_000091305.1_ASM9130v1".faa.gz"
# unzip the files ( required for orthofinder)
gunzip *.gz
```


## Set up orthofinder directory

```bash
# move to analysis folder
cd ../orthofinder-analysis
# create a soft link to the FAA files we just downloaded
ln -s ../genbank_downloads/*.faa ./
# create a soft link to the FAA fles from our PROKKA analysis
ln -s /home/maineBK/shared_data/filtered_genome_assemblies/prokka_faas/*.faa ./
```

## Count the number of proteins in all the starting files
Think about what these numbers tell us right off the bat.

```bash
grep -c '>' *.faa
```

## Run Orthofinder2

The input to the program is a directory containing a FAA file for each species.

```bash
# view the manual
orthofinder2 --help
# run the program
nohup time orthofinder2 -t 16 -a 16 -S diamond -f ./ &
```

## Examine the output files

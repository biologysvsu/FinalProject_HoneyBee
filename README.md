## Aggression-associated gene transcription levels of African honey bee compared to European honey bee
### detailed workflow
### download files
1. request resources
```bash
interact -t 4:00:00 --ntasks-per-node=48
```
2. go to shared + make folder
```bash
cd /ocean/projects/bio260081p/shared
mkdir -p bee_transcriptome
cd bee_transcriptome
```
4. make file containing srr accession numbers
```bash
vi srr_accessions.txt
```
5. type i to enter insert mode, then copy + paste so they appear one per line:
```bash
SRR10030210
SRR12918257
SRR31156059
SRR989674
```
6. Esc  :wq  enter
7. check file is not empty
```bash
cat srr_accessions.txt
```
8. load conda + activate sra-tools
```bash
module load anaconda3
conda create -n sra-tools -c conda-forge -c bioconda sra-tools -y
conda activate sra-tools
``` 
9. make script to download srr files
```bash
vi download_sra.sh
```
10. type i, then enter following lines
```bash
#!/bin/bash
mkdir -p sra-data

while read -r SRR; do
    prefetch "$SRR" -O "$PWD/$SRR"
    fasterq-dump "$SRR" -e 8 --split-files -O sra-data/
done < srr_accessions.txt
```
11. Esc  :wq  enter
12. note: error > exit HPC > run using
```bash
bash download_sra.sh &
```
### get reference genome (fasta) and annotation file (GTF)
1. download fasta file, unzip, rename it reference.fasta
```bash
wget
gunzip
mv reference.fasta
```
2. download GTF file, unzip, rename it annotation.gtf
```bash
wget
gunzip
mv annotation.gtf
```
### star prep: make star genome index (3hr 16cpus RAM 50GB)
1. only if interact has stopped, re-request memory resources
```bash
interact -t 4:00:00 --ntasks-per-node=48
```
2. load star and create directory
```bash
module load STAR
mkdir -p star_index
```
3. generate star_index
```bash
STAR --runMode genomeGenerate
--runThreadN 48
--genomeDir star_index
--genomeFastaFiles reference.fasta
--sjdbGTFfile annotation.gtf
--sjdbOverhang 100
```
3. should have: star_index, reference.fasta, annotation.gtf
### align reads using star
1. European honey bee
a. make directory for European honey bee
```bash
mkdir eurobee
cd eurobee
```
b. align reads for 1st rep
```bash
STAR --genomeDir ../star_index \
     --readFilesIn ../sra-data/SRR10030210_1.fastq \
     --outFileNamePrefix eurobee_rep1_ \
     --runThreadN 32 \
     --outSAMtype BAM SortedByCoordinate
```
c. align reads for 2nd rep
```bash
STAR --genomeDir ../star_index \
     --readFilesIn ../sra-data/SRR12918257_1.fastq \
     --outFileNamePrefix eurobee_rep2_ \
     --runThreadN 32 \
     --outSAMtype BAM SortedByCoordinate
```
2. African honey bee
a. exit the eurobee directory
```bash
cd ..
```
b. make directory for African honey bee
```bash
mkdir afribee
cd afribee
```
c. align reads for 1st rep
```bash
STAR --genomeDir ../star_index \
     --readFilesIn ../sra-data/SRR31156059_1.fastq \
     --outFileNamePrefix afribee_rep1_ \
     --runThreadN 32 \
     --outSAMtype BAM SortedByCoordinate
```
d. align reads for 2nd rep
```bash
STAR --genomeDir ../star_index \
     --readFilesIn ../sra-data/SRR989674_1.fastq \
     --outFileNamePrefix afribee_rep2_ \
     --runThreadN 32 \
     --outSAMtype BAM SortedByCoordinate
```
3. should have: sorted BAM file and alignment log file
### quantify gene expression 



6. load conda + make environment
```bash
module load anaconda3/2024.10-1
conda create --name bioinfo-env python=3.9 -y
conda activate bioinfo-env 
conda install -c bioconda star subread fastqc samtools -y
conda deactivate
```

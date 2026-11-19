## Aggression-associated gene expression in African and European honey bees
### We are using publicly available RNA sequencing (RNA-seq) data to compare the expression of selected aggression-associated genes in African and European honey bee samples. We first download the sequencing reads, then use STAR to map them to a common honey bee reference genome. The resulting alignments will let us count the reads assigned to each gene. Those counts are the starting point for comparing RNA abundance between the two groups.
### Download the RNA-seq files
1. Request computing resources. This starts an interactive session on the cluster so we have computing resources to process the sequencing files.
```bash
interact -t 4:00:00 --ntasks-per-node=48
```
2. Go to the shared folder. Then create a new folder and go into it. The bee_transcriptome folder gives everyone in the group a common place to access the sequencing files and analysis outputs.
```bash
cd /ocean/projects/bio260081p/shared
mkdir -p bee_transcriptome
cd bee_transcriptome
```
3. Make a list of the sequencing runs. Each SRR accession identifies one sequencing run in NCBI’s Sequence Read Archive (SRA). We save one accession per line so the download script can process the runs one at a time.
```bash
vi srr_accessions.txt
```
4. Press i to enter insert mode, then paste the accessions on separate lines.
```bash
SRR10030210
SRR12918257
SRR31156059
SRR989674
```
5. Press Esc, type :wq, and press Enter to save and exit.
6. Display the file to confirm all four accessions are present.
```bash
cat srr_accessions.txt
```
7. Set up SRA Toolkit. This creates and activates a Conda environment with the tools we need to download the sequencing runs and convert them to FASTQ files.
```bash
module load anaconda3
conda create -n sra-tools -c conda-forge -c bioconda sra-tools -y
conda activate sra-tools
``` 
8. Write the download script. For each accession, prefetch retrieves the sequencing run, and fasterq-dump converts it into FASTQ files in sra-data/. FASTQ files store each read’s sequence and base quality scores. For paired-end runs, --split-files saves the two mates in separate files.
```bash
vi download_sra.sh
```
9. Press i, then paste the script.
```bash
#!/bin/bash
mkdir -p sra-data

while read -r SRR; do
    prefetch "$SRR" -O "$PWD/$SRR"
    fasterq-dump "$SRR" -e 8 --split-files -O sra-data/
done < srr_accessions.txt
```
10. Press Esc, type :wq then press Enter. 
11. Run the script from the project folder with the sra-tools environment active. The & starts it in the background of the current shell. Start it after you’re in the cluster session you’ll use for the download. Note: error > exit HPC > run using
```bash
bash download_sra.sh &
```
### Download the reference genome (fasta) and gene annotation (GTF)
1. Download the reference genome FASTA. This file contains the honey bee genome sequence. Using the same reference for both groups gives STAR a common sequence to map all four runs against. Decompress the downloaded file and name it reference.fasta
```bash
wget
gunzip
mv reference.fasta
```
2. Download the gene annotation GTF. The GTF tells us where genes and exons are located in the reference genome. Use an annotation from the same genome assembly as the FASTA so the coordinates match. Decompress the file and name it annotation.gtf 
```bash
wget
gunzip
mv annotation.gtf
```
### Build the STAR genome index (3hr 16cpus RAM 50GB)
1. Request another interactive session if the first one ended. Indexing uses cluster resources. If your first session is still active, continue in that session. If you start a new one, return to the bee_transcriptome project folder before running STAR.
```bash
interact -t 4:00:00 --ntasks-per-node=48
```
2. Load STAR and create the index folder. The star_index directory will hold the index files that STAR uses for all four alignments.
```bash
module load STAR
mkdir -p star_index
```
3. Generate the genome index. STAR turns the reference sequence into a searchable index so it can quickly find where reads map. It also uses the GTF to identify known splice junctions, which helps it align reads that span two exons.
```bash
STAR --runMode genomeGenerate
--runThreadN 48
--genomeDir star_index
--genomeFastaFiles reference.fasta
--sjdbGTFfile annotation.gtf
--sjdbOverhang 100
```
4. After this step, the project folder should contain reference.fasta, annotation.gtf, and star_index/. The outline lists 16 CPUs and 50 GB RAM for indexing, while this command uses 48 threads. Confirm the actual allocation before running it and set --runThreadN to a value within the CPUs available. The alignment commands below also need an allocation that supports their 32 threads.
### Align reads using STAR
## STAR maps each RNA-seq run to the same indexed reference genome. Each command gives its outputs a run-specific prefix and requests a BAM file sorted by genomic position. This BAM records where the reads aligned and will be used for gene counting. Check the FASTQ inputs before running STAR. The commands below pass only the _1.fastq file for each run. If a run is paired-end, pass both its _1.fastq and _2.fastq files after --readFilesIn so STAR can align the mates together. If a run is single-end, use the actual filename produced by fasterq-dump.
1. European honey bee
a. Create a folder for the European honey bee alignments. Run this from bee_transcriptome. From inside eurobee, ../ points back to the project folder where the index and FASTQ files are stored.
```bash
mkdir eurobee
cd eurobee
```
b. Align the first European run. Map SRR10030210 to the reference and use eurobee_rep1_ to identify its output files.
```bash
STAR --genomeDir ../star_index \
     --readFilesIn ../sra-data/SRR10030210_1.fastq \
     --outFileNamePrefix eurobee_rep1_ \
     --runThreadN 32 \
     --outSAMtype BAM SortedByCoordinate
```
c. Align the second European run. Use the same index and settings for SRR12918257. Give this run its own prefix, eurobee_rep2_, so the results can be matched to the right sample.
```bash
STAR --genomeDir ../star_index \
     --readFilesIn ../sra-data/SRR12918257_1.fastq \
     --outFileNamePrefix eurobee_rep2_ \
     --runThreadN 32 \
     --outSAMtype BAM SortedByCoordinate
```
2. African honey bee
a. Return to the project folder
```bash
cd ..
```
b. Create a folder for the African honey bee alignments. This puts afribee alongside eurobee, so both groups can access the same index and FASTQ.
```bash
mkdir afribee
cd afribee
```
c. Align the first African run. Map SRR31156059 to the same reference used for the European runs and label its outputs with afribee_rep1_ 
```bash
STAR --genomeDir ../star_index \
     --readFilesIn ../sra-data/SRR31156059_1.fastq \
     --outFileNamePrefix afribee_rep1_ \
     --runThreadN 32 \
     --outSAMtype BAM SortedByCoordinate
```
d. Align the second African run. Map SRR989674 using the same settings and label its outputs with afribee_rep2_ 
```bash
STAR --genomeDir ../star_index \
     --readFilesIn ../sra-data/SRR989674_1.fastq \
     --outFileNamePrefix afribee_rep2_ \
     --runThreadN 32 \
     --outSAMtype BAM SortedByCoordinate
```
3. Check the alignment outputs. Each run should produce a sorted BAM ending in Aligned.sortedByCoord.out.bam and an alignment summary ending in Log.final.out. The summary reports how many reads STAR processed and what fraction mapped to the reference. The sorted BAM is the input for the next stage.
### Quantify gene expression 



1. Set up the software for gene counting and quality checks. This block creates the bioinfo-env Conda environment and installs the analysis tools. FastQC summarizes read quality, STAR performs alignment, and SAMtools helps inspect and manage BAM files. Subread provides featureCounts, which uses the GTF to assign aligned reads to genes.
```bash
module load anaconda3/2024.10-1
conda create --name bioinfo-env python=3.9 -y
conda activate bioinfo-env 
conda install -c bioconda star subread fastqc samtools -y
conda deactivate
```
### We use featureCounts to assign reads from the four sorted BAM files to genes using annotation.gtf. The output gives us a table of raw counts for each gene in each sample. These counts can then be normalized to account for differences in sequencing depth before we compare the selected aggression-associated genes between the two groups.

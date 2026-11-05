### Aggression-associated gene transcription levels of African honey bee compared to European honey bee
## detailed workflow
1. request resources
```bash
interact -t 4:00:00 --ntasks-per-node=48
```
2. go to shared + make folder
```bash
mkdir -p transcriptome
cd transcriptome
```
4. make file containing srr accession numbers
```bash
vi bee_srr_accessions.txt
```
type i to enter insert mode, then copy + paste:
```bash
SRR10030210
SRR12918257
SRR31156059
SRR989674
```
so they appear one at a time 
5. star index symlink
```bash
ln -s /ocean/projects/bio260081p/shared/tutorial-data/tutorial_3_data/star_index .
```
6. load conda + make environment
```bash
module load anaconda3/2024.10-1
```
```bash
conda create --name bioinfo-env python=3.9 -y
```
```bash
conda activate bioinfo-env 
```
```bash
conda install -c bioconda star subread fastqc samtools -y
```
```bash
conda deactivate
```
7. download fastq files

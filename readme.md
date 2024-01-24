# Biol217 Practice session Day-1

What we learned so far?

1. Basic Linux
2. Bioinformatics basic understanding
3. Linux commands

- copy from one folder to another:
  
  Block of code:
```sh
cp source destination
```

inline code:
this is the command `cp`



# Task: Practice how to add links and images to the md file

# day 2

quality control of raw reads
make file / file.txt / file.sh

```sh
touch anvio_slurm
```

Quality Control of raw reads
```sh
#!/bin/bash
#SBASH --nodes=1
#SBATCH --cpus-per-task=4
#SBATCH --mem=10G
#SBATCH --time=5:00:00
#SBATCH--job-name=fastqc
#SBATCH--error=fastqc.err
#SBATCH --partition=base
#SBATCH --reservation=biol217

module load gcc12-env/12.1.0
module load miniconda3/4.12.0
conda activate anvio-8

cd /work_beegfs/sunam232/Metagenomics/0_raw_reads
for i in *fastq.gz; do fastqc $i -o ../1_fastqc/; done
```

`sbatch anvio_slurm` for sending the task to the HPC

`squeue` for all running tasks

`squeue -u sunam232` for my running tasks

```sh
scp sunam232@caucluster.rz.uni-kiel.de:/work_beegfs/sunam232/Metagenomics/1_fastqc/*.html .
```
kopiert von Link in atuelles Verzeichnis (neues Terminal benötigt)

```sh
fastp -i BGR_130305_mapped_R1.fastq.gz -I BGR_130305_mapped_R2.fastq.gz -R fastp_report -o BGR_130305_mapped_R1_clean.fastq.gz -O BGR_130305_mapped_R2_clean.fastq.gz -t 6 -q 20
```

```sh
fastp -i BGR_130527_mapped_R1.fastq.gz -I BGR_130527_mapped_R2.fastq.gz -R fastp_report -o BGR_130527_mapped_R1_clean.fastq.gz -O BGR_130527_mapped_R2_clean.fastq.gz -t 6 -q 20
```
```sh
fastp -i BGR_130708_mapped_R1.fastq.gz -I BGR_130708_mapped_R2.fastq.gz -R fastp_report -o BGR_130527_mapped_R1_clean.fastq.gz -O BGR_130708_mapped_R2_clean.fastq.gz -t 6 -q 20
```


> `--html` creates an .html report file in html format\
>`-i` input file name\
>`-I` R2 input file name\
>`-R` report title, here ‘_report’ is added to each file\
>`-o` output_folder/R1.fastq.gz output file\
>`-O` output_folder/R2.fastq.gz output file\
>`-t` trim tail 1, default is 0, here 6 bases are trimmed\
>`-q` 20 reads with a phred score of <=20 are trimmed

```sh
cd /work_beegfs/sunam232/Metagenomics/2_fastp
megahit -1 BGR_130305_mapped_clean_R1.fastq.gz -1 BGR_130527_mapped_clean_R1.fastq.gz -1 BGR_130527_mapped_clean_R1.fastq.gz -2 BGR_130305_mapped_clean_R2.fastq.gz -2 BGR_130527_mapped_clean_R2.fastq.gz -2 BGR_130527_mapped_clean_R2.fastq.gz --min-contig-len 1000 --presets meta-large -m 0.85 -o ../3_coassembly -t 12
```

```
-1 path to R1 file
-2 path to R2 file, for paired end readings only
-o path to output folder
```

# day 3

checking if assembly worked

`grep -c ">" final.contigs.fa` 

57414

`scp sunam232@caucluster.rz.uni-kiel.de:/work_beegfs/sunam232/Metagenomics/3_coassembly/final.contigs.fastg .`

## Quality Assessment of Assemblies

```sh
cd /work_beegfs/sunam232/Metagenomics/3_coassembly
metaquast -t 6 -o /work_beegfs/sunam232/Metagenomics/3_metaquast_out/final.contigs.fastg -m 1000 final.contigs.fa
```

### What is your N50 value? 
N50 = 2963
### Why is this value relevant?

### How many contigs are assembled?
57414

### What is the total length of the contigs?

145675865

## Genomes Binning

```sh
cd /work_beegfs/sunam232/Metagenomics/3_coassembly
anvi-script-reformat-fasta final.contigs.fa -o /work_beegfs/sunam232/Metagenomics/3_1_binning/contigs.anvio.fa --min-len 1000 --simplify-names --report-file binning_conversion.txt
```

## Mapping

```sh
cd /work_beegfs/sunam232/Metagenomics/3_1_binning 
module load bowtie2
bowtie2-build contigs.anvio.fa contigs.anvio.fa.index
bowtie2 --very-fast -x contigs.anvio.fa.index -1 /work_beegfs/sunam232/Metagenomics/2_fastp/BGR_130305_mapped_clean_R1.fastq.gz -2 /work_beegfs/sunam232/Metagenomics/2_fastp/BGR_130305_mapped_clean_R2.fastq.gz -S 130305.sam

bowtie2-build contigs.anvio.fa contigs.anvio.fa.index
bowtie2 --very-fast -x contigs.anvio.fa.index -1 /work_beegfs/sunam232/Metagenomics/2_fastp/BGR_130527_mapped_clean_R1.fastq.gz -2 /work_beegfs/sunam232/Metagenomics/2_fastp/BGR_130527_mapped_clean_R2.fastq.gz -S 130527.sam

bowtie2-build contigs.anvio.fa contigs.anvio.fa.index
bowtie2 --very-fast -x contigs.anvio.fa.index -1 /work_beegfs/sunam232/Metagenomics/2_fastp/BGR_130708_mapped_clean_R1.fastq.gz -2 /work_beegfs/sunam232/Metagenomics/2_fastp/BGR_130708_mapped_clean_R2.fastq.gz -S 130708.sam

```

```sh
module load samtools
samtools view -bS 130305.sam > BGR_130305.bam
samtools view -bS 130527.sam > BGR_130527.bam
samtools view -bS 130708.sam > BGR_130708.bam
```

### contigs data preparation
```sh
anvi-gen-contigs-database -f contigs.anvio.fa -o ../5_anvio_profiles/contigs.db -n 'biol217'

anvi-run-hmms -c contigs.db --num-threads 4
```

## binning with anvio


```sh
cd /work_beegfs/sunam232/Metagenomics/3_1_binning 

for i in *.bam; do anvi-init-bam $i -o "$i".sorted.bam; done


anvi-profile -i BGR_130305.sorted.bam -c contigs.db --output-dir ../5_anvio_profiles

anvi-profile -i BGR_130527.sorted.bam -c contigs.db --output-dir ../5_anvio_profiles

anvi-profile -i BGR_130708.sorted.bam -c contigs.db --output-dir ../5_anvio_profiles

```


```sh
anvi-merge /PATH/TO/SAMPLE1/? /PATH/TO/SAMPLE2/? /PATH/TO/SAMPLE3/? -o ? -c ? --enforce-hierarchical-clustering
```


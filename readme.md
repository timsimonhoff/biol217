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
fastp -i BGR_130305_mapped_R1.fastq.gz -I BGR_130305_mapped_R2.fastq.gz -R fastp_report -o *R1_clean.fastq.gz -O *R2_clean.fastq.gz -t 6 -q 20


fastp -i BGR_130527_mapped_R1.fastq.gz -I BGR_130527_mapped_R2.fastq.gz -R fastp_report -o *R1_clean.fastq.gz -O *R2_clean.fastq.gz -t 6 -q 20

fastp -i BGR_130708_mapped_R1.fastq.gz -I BGR_130708_mapped_R2.fastq.gz -R fastp_report -o *R1_clean.fastq.gz -O *R2_clean.fastq.gz -t 6 -q 20
```


> `--html` creates an .html report file in html format\
>`-i` input file name\
>`-I` R2 input file name\
>`-R` report title, here ‘_report’ is added to each file\
>`-o` output_folder/R1.fastq.gz output file\
>`-O` output_folder/R2.fastq.gz output file\
>`-t` trim tail 1, default is 0, here 6 bases are trimmed\
>`-q` 20 reads with a phred score of <=20 are trimmed
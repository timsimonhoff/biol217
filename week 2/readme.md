# Day 6
## Genomics

### quality control of short reads
in script

1.1 fastqc
```sh
mkdir -p $WORK/genomics/1_short_reads_qc/1_fastqc_raw
for i in $WORK/genomics/0_raw_reads/short_reads/*.gz; do fastqc $i -o $WORK/genomics/1_short_reads_qc/1_fastqc_raw
 -t 32; done


##1.2 fastp
mkdir -p $WORK/genomics/1_short_reads_qc/2_cleaned_reads
fastp -i $WORK/genomics/0_raw_reads/short_reads/241155E_R1.fastq.gz \
 -I $WORK/genomics/0_raw_reads/short_reads/241155E_R2.fastq.gz \
 -R $WORK/genomics/1_short_reads_qc/2_cleaned_reads/fastp_report \
 -h $WORK/genomics/1_short_reads_qc/2_cleaned_reads/report.html \
 -o $WORK/genomics/1_short_reads_qc/2_cleaned_reads/241155E_R1_clean.fastq.gz \
 -O $WORK/genomics/1_short_reads_qc/2_cleaned_reads/241155E_R2_clean.fastq.gz -t 6 -q 25

##1.3 fastqc cleaned
mkdir -p $WORK/genomics/1_short_reads_qc/3_fastqc_clean
for i in $WORK/genomics/1_short_reads_qc/2_cleaned_reads/*.gz; do fastqc $i -o $WORK/genomics/1_short_reads_qc/3_fastqc_clean -t 32; done


jobinfo
```

before trimming:
	241155E_R1: 1639549
    241155E_R2: 1639549
after trimming: 
    241155E_R1: 1613392
    241155E_R2: 1613392
    quality improved






How good is the quality of genome?



Why did we use Hybrid assembler?
    combine the best of long and short reads


What is the difference between short and long reads?
    short reads 150-300 bp, long rerads up to 10,000 bp
    
Did we use Single or Paired end reads? Why?
    paired end reads, to increase the reliability of the reads and minimize errors


Write down about the classification of genome we have used here
    drftg
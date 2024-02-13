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
completeness: 98.88          Contamination:  0.19 
high quality, nearly complete
N50 über 50% in einem contig

Why did we use Hybrid assembler?
    combine the best of long and short reads


What is the difference between short and long reads?
    short reads 150-300 bp, long rerads up to 10,000 bp
    
Did we use Single or Paired end reads? Why?
    paired end reads, to increase the reliability of the reads and minimize errors


Write down about the classification of genome we have used here
Taxonomy: d__Bacteria;p__Bacteroidota;c__Bacteroidia;o__Bacteroidales;f__Bacteroidaceae;g__Bacteroides;s__Bacteroides sp002491635


# Day 7

```sh
anvi-display-contigs-stats $WORK/pangenomics/02_anvio_pangenome/V_jascida_genomes/*db
```

## Create external genomes file

```sh
anvi-script-gen-genomes-file --input-dir $WORK/pangenomics/02_anvio_pangenome/V_jascida_genomes/ -o external-genomes.txt
```

## Investigate contamination

```sh
anvi-estimate-genome-completeness -e external-genomes.txt
```

## 7. Visualise contigs for refinement
```sh
anvi-profile -c V_jascida_52.db --sample-name V_jascida_52 --output-dir V_jascida_52 --blank
```

```sh
anvi-interactive -c V_jascida_52.db -p V_jascida_52/PROFILE.db
```

```sh
ssh -L 8060:localhost:8080 sunam226@caucluster.rz.uni-kiel.de
ssh -L 8080:localhost:8080 n171
```

```sh
anvi-split -p V_jascida_52/PROFILE.db -c V_jascida_52.db -C default -o V_jascida_52_SPLIT

# Here are the files you created
#V_jascida_52_SPLIT/V_jascida_52_CLEAN/CONTIGS.db

sed 's/V_jascida_52.db/V_jascida_52_SPLIT\/V_jascida_52_CLEAN\/CONTIGS.db/g' external-genomes.txt > external-genomes-final.txt
```

## Estimate completeness of split vs. unsplit genome:
```sh
anvi-estimate-genome-completeness -e external-genomes.txt
anvi-estimate-genome-completeness -e external-genomes-final.txt
```


## Compute pangenome
```sh
anvi-gen-genomes-storage -e external-genomes-final.txt -o V_jascida-GENOMES.db
anvi-pan-genome -g V_jascida-GENOMES.db --project-name V_jascida --num-threads 4     
```

## Displaying pangenome

```sh
anvi-display-pan -p V_jascida/V_jascida-PAN.db -g V_jascida-GENOMES.db
```



# Own Transcriptomics

```sh
grabseqs -t 4 -m SRP081251
```

```sh
module load fastqc
fastqc -t 4 -o fastqc_output *.fastq.gz
```

for i in *.fastq.gz; do fastp -i $i -o ${i}_cleaned.fastq.gz -h ../qc_reports/${i}_fastp.html -j ${i}_fastp.json -w 4 -q 20 -z 4; done

# day 8

log2 - downregulated in mt\
RS03790: -1.9, yes\
RS06645: -1.71, ?\
RS00155: -1.58, mostly yes\
RS02310: -1.3, yes\
RS00765: -1.23, yes\

log2 + upregulated in mt\
RS00150: 2.06, yes \
RS04770: 1.2, yes\
RS05370: 1.16, yes\
RS07870: 1.16, yes\
RS05335: 1.15, yes



tnoar : total number of aligned reads


# Fotos hinzufügen

![igb](/resources/igb_RS00765.png)

![igb](/resources/igb_RS05335.png)


# csrA:
start codon:
ATG

stop codon:
TAA

length:
61 aa

SD:-7 (last nucleotide)

AGGAG

upstream gene: 
alaS (oder )

translated?:
yes, a lot of coverage from Riboprofiling

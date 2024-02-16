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


 # 2 Long read cleaning-----------------------------------------------------
 echo "---------long reads cleaning started---------"
 micromamba activate 02_long_reads_qc

 ## 2.1 Nanoplot raw
 cd $WORK/genomics/0_raw_reads/long_reads/
 mkdir -p $WORK/genomics/2_long_reads_qc/1_nanoplot_raw
 NanoPlot --fastq $WORK/genomics/0_raw_reads/long_reads/*.gz \
  -o $WORK/genomics/2_long_reads_qc/1_nanoplot_raw -t 12 \
  --maxlength 40000 --minlength 1000 --plots kde --format png \
  --N50 --dpi 300 --store --raw --tsv_stats --info_in_report

 ## 2.2 Filtlong
 mkdir -p $WORK/genomics/2_long_reads_qc/2_cleaned_reads
 filtlong --min_length 1000 --keep_percent 90 $WORK/genomics/0_raw_reads/long_reads/*.gz | gzip > $WORK/genomics/2_long_reads_qc/2_cleaned_reads/241155E_cleaned_filtlong.fastq.gz

 ## 2.3 Nanoplot cleaned
 cd $WORK/genomics/2_long_reads_qc/2_cleaned_reads
 mkdir -p $WORK/genomics/2_long_reads_qc/3_nanoplot_cleaned
 NanoPlot --fastq $WORK/genomics/2_long_reads_qc/2_cleaned_reads/*.gz \
  -o $WORK/genomics/2_long_reads_qc/3_nanoplot_cleaned -t 12 \
  --maxlength 40000 --minlength 1000 --plots kde --format png \
  --N50 --dpi 300 --store --raw --tsv_stats --info_in_report

 micromamba deactivate
 echo "---------long reads cleaning completed Successfully---------"

# 3 Assembly (1 hour)-----------------------------------------------------------
echo "---------Unicycler Assembly pipeline started---------"
micromamba activate 03_unicycler
cd $WORK/genomics
mkdir -p $WORK/genomics/3_hybrid_assembly
unicycler -1 $WORK/genomics/1_short_reads_qc/2_cleaned_reads/241155E_R1_clean.fastq.gz -2 $WORK/genomics/1_short_reads_qc/2_cleaned_reads/241155E_R2_clean.fastq.gz -l $WORK/genomics/2_long_reads_qc/2_cleaned_reads/241155E_cleaned_filtlong.fastq.gz -o $WORK/genomics/3_hybrid_assembly/ -t 32
micromamba deactivate
echo "---------Unicycler Assembly pipeline Completed Successfully---------"

# 4 Assembly quality-----------------------------------------------------------
echo "---------Assembly Quality Check Started---------"

## 4.1 Quast (5 minutes)
micromamba activate 04_checkm_quast
cd $WORK/genomics/3_hybrid_assembly
mkdir -p $WORK/genomics/3_hybrid_assembly/quast
quast.py $WORK/genomics/3_hybrid_assembly/assembly.fasta --circos -L --conserved-genes-finding --rna-finding \
 --glimmer --use-all-alignments --report-all-metrics -o $WORK/genomics/3_hybrid_assembly/quast -t 32
micromamba deactivate

## 4.2 CheckM
micromamba activate 04_checkm_quast
cd $WORK/genomics/3_hybrid_assembly
mkdir -p $WORK/genomics/3_hybrid_assembly/checkm
checkm lineage_wf $WORK/genomics/3_hybrid_assembly/ $WORK/genomics/3_hybrid_assembly/checkm -x fasta --tab_table --file $WORK/genomics/3_hybrid_assembly/checkm/checkm_results -r -t 32
checkm tree_qa $WORK/genomics/3_hybrid_assembly/checkm
checkm qa $WORK/genomics/3_hybrid_assembly/checkm/lineage.ms $WORK/genomics/3_hybrid_assembly/checkm/ -o 1 > $WORK/genomics/3_hybrid_assembly/checkm/Final_table_01.csv
checkm qa $WORK/genomics/3_hybrid_assembly/checkm/lineage.ms $WORK/genomics/3_hybrid_assembly/checkm/ -o 2 > $WORK/genomics/3_hybrid_assembly/checkm/final_table_checkm.csv
micromamba deactivate

# 4.3 Checkm2
# (can not work, maybe due to insufficient memory usage)
micromamba activate 05_checkm2
cd $WORK/genomics/3_hybrid_assembly
mkdir -p $WORK/genomics/3_hybrid_assembly/checkm2
checkm2 predict --threads 32 --input $WORK/genomics/3_hybrid_assembly/* --output-directory $WORK/genomics/3_hybrid_assembly/checkm2 
micromamba deactivate
echo "---------Assembly Quality Check Completed Successfully---------"

# 5 Annotate-----------------------------------------------------------
echo "---------Prokka Genome Annotation Started---------"

micromamba activate 06_prokka
cd $WORK/genomics/3_hybrid_assembly
# Prokka creates the output dir on its own
prokka $WORK/genomics/3_hybrid_assembly/assembly.fasta --outdir $WORK/genomics/4_annotated_genome --kingdom Bacteria --addgenes --cpus 32
micromamba deactivate
echo "---------Prokka Genome Annotation Completed Successfully---------"


# 6 Classification-----------------------------------------------------------
echo "---------GTDB Classification Started---------"
# (can not work, maybe due to insufficient memory usage increase the ram in bash script)
micromamba activate 07_gtdbtk
conda env config vars set GTDBTK_DATA_PATH="$WORK/Databases/GTDBTK_day6";
micromamba activate 07_gtdbtk
cd $WORK/genomics/4_annotated_genome
mkdir -p $WORK/genomics/5_gtdb_classification
echo "---------GTDB Classification will run now---------"
gtdbtk classify_wf --cpus 12 --genome_dir $WORK/genomics/4_annotated_genome/ --out_dir $WORK/genomics/5_gtdb_classification --extension .fna 
# reduce cpu and increase the ram in bash script in order to have best performance
micromamba deactivate
echo "---------GTDB Classification Completed Successfully---------"

# 7 multiqc-----------------------------------------------------------
echo "---------Multiqc Started---------"
micromamba activate 01_short_reads_qc
multiqc -d $WORK/genomics/ -o $WORK/genomics/6_multiqc
micromamba deactivate
echo "---------Multiqc Completed Successfully---------"


module purge
jobinfo

```

reads before trimming:

	241155E_R1: 1639549
    241155E_R2: 1639549

reads after trimming: 

    241155E_R1: 1613392
    241155E_R2: 1613392
   -> quality improved, phred score lower

NanoPlot:
reads before trimming:
    15963
    quality: 10.4
reads after trimming:
    12446
    quality: 11.4



How good is the quality of genome?

    completeness: 98.88          Contamination:  0.19 
    high quality, nearly complete
    N50 über 50% in einem contig

Why did we use Hybrid assembler?

    combine the best of long and short reads


What is the difference between short and long reads?

    short reads 150-300 bp, long reads up to 10,000 bp
    
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

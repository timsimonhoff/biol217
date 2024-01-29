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
>`-t` trim tail 1, default is 0, here 6 biol217_2024!!stq.gz -1 BGR_130527_mapped_clean_R1.fastq.gz -1 BGR_130527_mapped_clean_R1.fastq.gz -2 BGR_130305_mapped_clean_R2.fastq.gz -2 BGR_130527_mapped_clean_R2.fastq.gz -2 BGR_130527_mapped_clean_R2.fastq.gz --min-contig-len 1000 --presets meta-large -m 0.85 -o ../3_coassembly -t 12
```

```sh
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


anvi-profile -i BGR_130305.bam.sorted.bam -c ../5_anvio_profiles/contigs.db --output-dir ../5_anvio_profiles/BGR_130305

anvi-profile -i BGR_130527.bam.sorted.bam -c ../5_anvio_profiles/contigs.db --output-dir ../5_anvio_profiles/BGR_130527

anvi-profile -i BGR_130708.bam.sorted.bam -c ../5_anvio_profiles/contigs.db --output-dir ../5_anvio_profiles/BGR_130708

```



# visualisation

```sh
srun --reservation=biol217 --pty --mem=10G --nodes=1 --tasks-per-node=1 --cpus-per-task=1 --nodelist=n100 --partition=base /bin/bash
```

```sh
module load gcc12-env/12.1.0
module load miniconda3/4.12.0
conda activate anvio-8

anvi-display-contigs-stats contigs.db
```

neues Terminal

```sh
ssh -L 8060:localhost:8080 sunam232@caucluster.rz.uni-kiel.de
ssh -L 8080:localhost:8080 n100
```

```sh
cd .
anvi-merge /work_beegfs/sunam232/Metagenomics/5_anvio_profiles/BGR_130305/PROFILE.db /work_beegfs/sunam232/Metagenomics/5_anvio_profiles/BGR_130527/PROFILE.db /work_beegfs/sunam232/Metagenomics/5_anvio_profiles/BGR_130708/PROFILE.db -o /work_beegfs/sunam232/Metagenomics/6_anvi-merge/ -c /work_beegfs/sunam232/Metagenomics/5_anvio_profiles/contigs.db --enforce-hierarchical-clustering
```

```sh
cd /work_beegfs/sunam232/Metagenomics
anvi-cluster-contigs -p ./6_anvi-merge/PROFILE.db -c ./5_anvio_profiles/contigs.db -C METABAT --driver metabat2 --just-do-it --log-file log-metabat2

anvi-summarize -p ./6_anvi-merge/PROFILE.db -c ./5_anvio_profiles/contigs.db -o SUMMARY_METABAT -C METABAT
```


```sh
cd /work_beegfs/sunam232/Metagenomics
anvi-cluster-contigs -p ./6_anvi-merge/PROFILE.db -c ./5_anvio_profiles/contigs.db -C MAXBIN2 --driver maxbin2 --just-do-it --log-file log-maxbin2

anvi-summarize -p ./6_anvi-merge/PROFILE.db -c ./5_anvio_profiles/contigs.db -o SUMMARY_MAXBIN -C MAXBIN2
```


```sh
anvi-estimate-genome-completeness -c ./5_anvio_profiles/contigs.db -p ./6_anvi-merge/PROFILE.db -C METABAT
```

```sh
anvi-estimate-genome-completeness -p ./6_anvi-merge/PROFILE.db -c ./5_anvio_profiles/contigs.db --list-collections
```

```sh
module load gcc12-env/12.1.0
module load miniconda3/4.12.0
conda activate anvio-8

anvi-interactive -p ./6_anvi-merge/PROFILE.db -c ./5_anvio_profiles/contigs.db -C METABAT
```


```sh
module load gcc12-env/12.1.0
module load miniconda3/4.12.0
conda activate anvio-8

anvi-interactive -p ./6_anvi-merge/PROFILE.db -c ./5_anvio_profiles/contigs.db -C MAXBIN2
``

METABAT gives best quality for archaea binning & general less redundancy/ contamination

MAXBIN 
archaeum completion 96.05
archaeum contamination/ redundancy 80.02
Bacteria With high quality 3

METABAT
archaeum completion 97.3 
archaeum contamination/ redundancy 5.2
Bacteria With high quality 13 


```sh
anvi-summarize -p ./6_anvi-merge/PROFILE.db -c ./5_anvio_profiles/contigs.db --list-collections
anvi-summarize -c ./5_anvio_profiles/contigs.db -p ./6_anvi-merge/PROFILE.db -C METABAT -o SUMMARY_METABAT2 --just-do-it
```




```sh
anvi-estimate-genome-completeness -c ./5_anvio_profiles/contigs.db -p ./6_anvi-merge/PROFILE.db -C METABAT > METABAT_table.txt
```

```sh
module load gcc12-env/12.1.0
module load miniconda3/4.12.0
conda activate anvio-8

anvi-interactive -p /PATH/TO/merged_profiles/PROFILE.db -c /PATH/TO/contigs.db -C YOUR_COLLECTION
```


```sh
anvi-summarize -p /PATH/TO/merged_profiles/PROFILE.db -c /PATH/TO/contigs.db --list-collections

anvi-summarize -c /PATH/TO/contigs.db -p /PATH/TO/merged_profiles/profile.db -C METABAT2 -o SUMMARY_METABAT2 --just-do-it
```


```sh
cd ./SUMMARY_METABAT2/bin_by_bin/

mkdir ../../ARCHAEA_BIN_REFINEMENT

cp ./METABAT__25/*.fa ../../ARCHAEA_BIN_REFINEMENT/
cp ./METABAT__41/*.fa ../../ARCHAEA_BIN_REFINEMENT/
cp ./METABAT__14/*.fa ../../ARCHAEA_BIN_REFINEMENT/
```


# day 5

## chimera detection

```sh
module load gcc12-env/12.1.0
module load miniconda3/4.12.0
conda activate gunc
```

```sh
cd /work_beegfs/sunam232/Metagenomics/ARCHAEA_BIN_REFINEMENT/

mkdir GUNC

for i in *.fa; do gunc run -i "$i" -r /work_beegfs/sunam232/Databases/gunc_db_progenomes2.1.dmnd --out_dir GUNC --threads 10 --detailed_output; done
gunc plot -d ./GUNC/diamond_output/METABAT__25-contigs.diamond.progenomes_2.1.out -g ./GUNC/genes_calls/gene_counts.json
gunc plot -d ./GUNC/diamond_output/METABAT__14-contigs.diamond.progenomes_2.1.out -g ./GUNC/genes_calls/gene_counts.json
gunc plot -d ./GUNC/diamond_output/METABAT__41-contigs.diamond.progenomes_2.1.out -g ./GUNC/genes_calls/gene_counts.json
```

## Manual bin refinement
```sh
cd /work_beegfs/sunam232/Metagenomics/ARCHAEA_BIN_REFINEMENT/
anvi-refine -c ../5_anvio_profiles/contigs.db -C METABAT -p ../6_anvi-merge/PROFILE.db --bin-id METABAT__25
anvi-refine -c ../5_anvio_profiles/contigs.db -C METABAT -p ../6_anvi-merge/PROFILE.db --bin-id METABAT__41
anvi-refine -c ../5_anvio_profiles/contigs.db -C METABAT -p ../6_anvi-merge/PROFILE.db --bin-id METABAT__14
```

```sh
module load gcc12-env/12.1.0
module load miniconda3/4.12.0
conda activate anvio-8

anvi-refine -c /PATH/TO/contigs.db -C METABAT -p /PATH/TO/merged_profiles/PROFILE.db --bin-id METABAT__25
```


```sh
anvi-inspect -p ../6_anvi-merge/PROFILE.db -c ../5_anvio_profiles/contigs.db
anvi-script-get-coverage-from-bam
```

## Taxonomy

You will now add taxonomic annotations to your MAG.

-> associates the single-copy core genes in your contigs-db with taxnomy information:
```sh
cd /work_beegfs/sunam232/Metagenomics/

anvi-run-scg-taxonomy -c ./5_anvio_profiles/contigs.db -T 20 -P 2
```

-> makes quick taxonomy estimates for genomes, metagenomes, or bins stored in your contigs-db using single-copy core genes:
```sh
anvi-estimate-scg-taxonomy -c ./5_anvio_profiles/contigs.db -p ./6_anvi-merge/PROFILE.db --metagenome-mode --compute-scg-coverages --update-profile-db-with-taxonomy > tax.txt
```

```sh
anvi-summarize -p ./6_anvi-merge/PROFILE.db -c ./5_anvio_profiles/contigs.db --metagenome-mode -o ./SUMMARY_METABAT2 -C METABAT2
```


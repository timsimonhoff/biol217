cd $WORK/pangenomics/02_anvio_pangenome/V_jascida_genomes/

 ls *fasta | awk 'BEGIN{FS="_"}{print $1}' > genomes.txt

 # remove all contigs <2500 nt
 for g in `cat genomes.txt`
 do
     echo
     echo "Working on $g ..."
     echo
     anvi-script-reformat-fasta ${g}_scaffolds.fasta \
                                --min-len 2500 \
                                --simplify-names \
                                -o ${g}_scaffolds_2.5K.fasta
 done

 # generate contigs.db
 for g in `cat genomes.txt`
 do
     echo
     echo "Working on $g ..."
     echo
     anvi-gen-contigs-database -f ${g}_scaffolds_2.5K.fasta \
                               -o V_jascida_${g}.db \
                               --num-threads 4 \
                               -n V_jascida_${g}
 done

 # annotate contigs.db
 for g in *.db
 do
     anvi-run-hmms -c $g --num-threads 4
     anvi-run-ncbi-cogs -c $g --num-threads 4
     anvi-scan-trnas -c $g --num-threads 4
     anvi-run-scg-taxonomy -c $g --num-threads 4
 done

 anvi-compute-genome-similarity -e external-genomes.txt -o ANI -p ./my_pangenome/my_pangenome-PAN.db -T 12


 anvi-gen-genomes-storage -e external-genomes.txt -o own-GENOMES.db


```sh
anvi-pan-genome -g own-GENOMES.db --project-name my_pangenome --num-threads 4

anvi-get-sequences-for-gene-clusters -p ./my_pangenome/my_pangenome-PAN.db -g own-GENOMES.db --min-num-genomes-gene-cluster-occurs 9 --max-num-genes-from-each-genome 1 --concatenate-gene-clusters --output-file ./my_pangenome/my_pangenome-SCGs.fa
trimal -in ./my_pangenome/my_pangenome-SCGs.fa -out ./my_pangenome/my_pangenome-SCGs-trimmed.fa -gt 0.5 
iqtree -s ./my_pangenome/my_pangenome-SCGs-trimmed.fa -m WAG -bb 1000 -nt 8

echo -e "item_name\tdata_type\tdata_value" > my_pangenome/my_pangenome-phylogenomic-layer-order.txt

# add the newick tree as an order
echo -e "SCGs_Bayesian_Tree\tnewick\t`cat my_pangenome/my_pangenome-SCGs-trimmed.fa.treefile`"  >> my_pangenome/my_pangenome-phylogenomic-layer-order.txt

# import the layers order file
anvi-import-misc-data -p ./my_pangenome/my_pangenome-PAN.db -t layer_orders ./my_pangenome/my_pangenome-phylogenomic-layer-order.txt


anvi-display-pan -g own-GENOMES.db -p my_pangenome/my_pangenome-PAN.db

```
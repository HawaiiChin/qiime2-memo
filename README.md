# qiime2-memo
qiime2 on phyllosphere 16S rRNA amplicon
#use bowtie2 to remove host DNA first

bowtie2 -x host_removal/maize_index -f -p 28 -U /gss1/home/zlcui/qinhuayu/metaanalysis/Megahit/222222/SRR18673642.contigs.fa --un host_removal/nohost_SRR18673642_contigs.fasta --very-fast --sensitive --al-conc aligned 

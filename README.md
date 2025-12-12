# qiime2-memo
qiime2 on phyllosphere 16S rRNA amplicon
#use bowtie2 to remove host DNA first

bowtie2 -x host_removal/maize_index -f -p 28 -U /gss1/home/zlcui/qinhuayu/metaanalysis/Megahit/222222/SRR18673642.contigs.fa --un host_removal/nohost_SRR18673642_contigs.fasta --very-fast --sensitive --al-conc aligned 

#qiime2 code
qiime tools import --type 'SampleData[PairedEndSequencesWithQuality]' --input-path manifest.tsv --output-path demux.qza --input-format PairedEndFastqManifestPhred33V2
qiime demux summarize --i-data WF/demux.qza --o-visualization WF/summary.qzv

##vsearch
qiime vsearch merge-pairs --i-demultiplexed-seqs WF/demux.qza --o-merged-sequences WF/demux-joined.qza --o-unmerged-sequences vsearchunmerged.qza
（p-minmergelen, p-maxmergelen, p-mixlen, p-maxlen)
qiime demux summarize --i-data WF/demux-joined.qza --o-visualization WF/demux-joined.qzv
qiime quality-filter q-score --i-demux WF/demux-joined.qza --o-filtered-sequences WF/demux-joined-filtered.qza --o-filter-stats demux-joined-filter-stats.qza
qiime vsearch dereplicate-sequences --i-sequences WF/demux-joined-filtered.qza --o-dereplicated-table WF/derep-table.qza --o-dereplicated-sequences WF/vsearch-reps.qza
time qiime vsearch uchime-denovo --i-table WF/derep-table.qza --i-sequences WF/vsearch-reps.qza --o-chimeras vsearchchimera.qza --o-nonchimeras nonchimeras.qza --o-stats uchimestats.qza
qiime feature-table filter-features --i-table WF/derep-table.qza --m-metadata-file WF/nonchimeras.qza --o-filtered-table WF/table-seqs-nonchimeras.qza
qiime feature-table filter-seqs --i-data WF/vsearch-derep-reps.qza --m-metadata-file WF/nonchimeras.qza --o-filtered-data WF/rep-seqs-nonchimeras.qza

qiime vsearch cluster-features-de-novo --i-table WF/table-seqs-nonchimeras.qza --i-sequences WF/rep-seqs-nonchimeras.qza --p-perc-identity 0.97 --o-clustered-table table-0.97.qza --o-clustered-sequences rep-seqs-0.97.qza

qiime feature-table filter-features --i-table WF/table-0.97.qza --p-min-frequency 2 --o-filtered-table WF/filtered-table-0.97.qza
qiime feature-table filter-seqs --i-data WF/rep-seqs-0.97.qza --i-table WF/filtered-table-0.97.qza --o-filtered-data filtered-seqs-0.97.qza


*qiime feature-classifier classify-consensus-vsearch --i-reference-reads /gss1/home/zlcui/qinhuayu/gtdb_seqs.qza --i-reference-taxonomy /gss1/home/zlcui/qinhuayu/gtdb_tax.qza --i-query WF/filtered-seqs-0.97.qza --o-classification WF/taxonomy.qza --o-search-results WF/taxonomy-blast.qza
*qiime feature-classifier classify-sklearn --i-classifier gtdb_classifier_v6.qza --i-reads req-seqs-otu.qza --o-classification- gtdb-taxonomy.qza
qiime metadata tabulate --m-input-file WDMDADA2Visual/gtdb-taxonomy.qza --o-visualization gtdb-taxonomy.qzv
qiime taxa barplot --i-table WDMdataVsearch/table-otu.qza --i-taxonomy gtdb-taxonomy.qza --m-metadata-file manifest.tsv --o-visualization gtdb-barplot.qzv

biom convert -i /mnt/e/data_feature-table.biom -o /mnt/e/otu0.97.txt --to-tsv

qiime diversity core-metrics-phylogenetic --i-phylogeny rooted-tree.qza --i-table 2nddechlo/filtered-table-0.97.qza --p-sampling-depth 1103 --m-metadata-file manifest.tsv --output-dir diversity-core-metrics-phylogenetic

qiime feature-table filter-samples --i-table 2nddechlo/table-0.97.qza --m-metadata-file manifest.tsv --o-filtered-table phyllosphere-table.qza
#construct gtdb classifier
qiime rescript dereplicate --p-version "220" --p-domain "Bacteria" --o-ssequences gtdb_bac_seqs.qza --o-staxonomy gtdb_bac_tax.qza
 qiime rescript get-gtdb-data --p-version "220" --p-domain "Bacteria" --o-ssequences gtdb_bac_seqs.qza --o-staxonomy gtdb_bac_tax.qza
 from qiime2.plugins.rescript.methods import get_gtdb_data

 qiime rescript get-gtdb-data --p-version "220.0" --p-domain "Bacteria" --o-gtdb-sequences gtdb_bac_seqs.qza --o-gtdb-taxonomy gtdb_bac_tax.qza
 qiime rescript get-gtdb-data --p-version "220.0" --p-domain "Archaea" --o-gtdb-sequences gtdb_arc_seqs.qza --o-gtdb-taxonomy gtdb_arc_tax.qza

qiime feature-table merge-seqs --i-data gtdb_bac_seqs.qza --i-data gtdb_arc_seqs.qza --o-merged-data gtdb_seqs.qza
 
qiime feature-table merge-taxa --i-data gtdb_bac_tax.qza --i-data gtdb_arc_tax.qza --o-merged-data gtdb_tax.qza
qiime feature-classifier fit-classifier-naive-bayes --i-reference-reads gtdb_seqs.qza --i-reference-taxonomy gtdb_tax.qza --o-classifier gtdb_classifier_v6.qza

qiime rescript dereplicate --i-sequences gtdb_bac_seqs.qza --i-taxa gtdb_bac_tax.qza --o-dereplicated-sequences gtdb_bac_seqs_derep.qza --o-dereplicated-taxa gtdb_bac_tax_derep.qza
qiime rescript dereplicate --i-sequences gtdb_arc_seqs.qza --i-taxa gtdb_arc_tax.qza --o-dereplicated-sequences gtdb_arc_seqs_derep.qza --o-dereplicated-taxa gtdb_arc_tax_derep.qza
 
qiime feature-table merge-taxa --i-data gtdb_bac_tax_derep.qza --i-data gtdb_arc_tax_derep.qza --o-merged-data gtdb_tax.qza
qiime feature-table merge-seqs --i-data gtdb_bac_seqs_derep.qza --i-data gtdb_arc_seqs_derep.qza --o-merged-data gtdb_seqs.qza
qiime feature-classifier fit-classifier-naive-bayes --i-reference-reads gtdb_seqs.qza --i-reference-taxonomy gtdb_tax.qza --o-classifier gtdb_classifier_v6.qza

# tealeaf ( previously LeafCutterITI)

### By Xingpei Zhang and David A. Knowles

### Citations:
**Alamancos, G. P., Pagès, A., Trincado, J. L., Bellora, N., & Eyras, E. (2015). Leveraging transcript quantification for fast computation of alternative splicing profiles. RNA , 21(9), 1521–1531. https://doi.org/10.1261/rna.051557.115**

**Garrido-Martín, D., Palumbo, E., Guigó, R., & Breschi, A. (2018). ggsashimi: Sashimi plot revised for browser-and annotation-independent splicing visualization. PLoS computational biology, 14(8), e1006360.**

**Li, Y. I., Knowles, D. A., Humphrey, J., Barbeira, A. N., Dickinson, S. P., Im, H. K., & Pritchard, J. K. (2018). Annotation-free quantification of RNA splicing using LeafCutter. Nature Genetics, 50(1), 151–158. https://doi.org/10.1038/s41588-017-0004-9**

**Patro, R., Duggal, G., Love, M. I., Irizarry, R. A., & Kingsford, C. (2017). Salmon provides fast and bias-aware quantification of transcript expression. Nature Methods, 14(4), 417–419. https://doi.org/10.1038/nmeth.4197**


### Requirements (versions used for development)

- python (v3.10.11)

#### Python Dependencies
- numpy 
- pandas
- pyranges
- scipy

### R Dependencies (for tealeaf-ggsashimi)
- R
- ggplot2 
- data.table 
- gridExtra 

#### Additional Requirement for isoform quantification

- salmon (v1.10.0)

#### Other dependencies for Leafcutter as listed in https://github.com/davidaknowles/leafcutter/tree/master, especially for Leafcutter_ds

## tealeaf
A modified version of Leafcutter that detects and analyzes alternative splicing events by quantifying excised introns by utilizing isoform abundance and transcriptome annotation. Can also be install as a command line tool with `pip install tealeaf`

![tealeaf_Workflow](figures/tealeaf_workflow.png)



There are three parts of LeafcutterITI: 
- tealeaf_map_gen
- tealeaf_clustering (for bulk & bulk-like single-cell or pseudobulk)
- tealeaf_sc (for single-cell)
- tealeaf-ggsashimi (utlize adapted ggsashimi for sashimi plotting)

### tealeaf_map_gen
```
usage: python tealeaf_map_gen.py [-a/--annot] [--annot_source] [-o/--outprefix] 
                     [--maxintronlen] [--minintronlen] [-v/--virtual_intron] [--single_cell]
or when install with pip
leafcutterITI-tealeaf [-a/--annot] [--annot_source] [-o/--outprefix] [--maxintronlen]
                      [--minintronlen] [-v/--virtual_intron] [--single_cell]


Mandatory parameters:

-a, --annot     The transcriptome annotation gtf file for LeafcutterITI_map_gen to run with 

Optional Parameters:

--annot_source          The annotation source for the annotation, currently support Gencode and Stringtie 
                        (default: gencode)

-o, --outprefix         The prefix for output files (default: Leafcutter_)

--maxintronlen          The maximum allowed intron length for introns (default: 5,000,000)

--minintronlen          The minimum allowed intron length for introns (default: 50)

--no_quality_control  A flag on whether not to remove pseudogene, and decay transcript

-v, --virtual_intron    A flag on whether to compute virtual intron that can be used to capture
                        AFE and ALE usage, a testing feature

--single_cell           Whether to build matrices for isoform to intron and exon, required if dealing with\
                        single cell data from alevin-fry (default: True)
```


### tealeaf_clustering

```
usage: python tealeaf_clustering.py [--map] [--count_files] [--connect_file] [-a/--annot]
                    [--cluster_def] [-o/--outprefix] [--use_TPM] [--samplecutoff]
                    [--introncutoff] [-m/--minclucounts] [-r/--mincluratio] [--normalization_scale]
                    [--read_len] [--not_paired_end] [--overhang] [--sizing_factor]
or when install with pip
tealeaf-cluster [--map] [--count_files] [--connect_file] [-a/--annot]
                    [--cluster_def] [-o/--outprefix] [-n/--normalization] [--samplecutoff]
                    [--introncutoff] [-m/--minclucounts] [-r/--mincluratio] [--normalization_scale]
                    [--read_len] [--not_paired_end] [--overhang] [--sizing_factor]


Mandatory parameters:
--map             The isoforms to introns map generated from leafcutterITI_map_gen  

--count_files     A txt file that contain the sample names 

--connect_file    The intron-exon connectivity file generated from leafcutterITI_map_gen 

-a, --annot       The transcriptome annotation gtf file 


Optional Parameters:
--cluster_def           The definition used for cluster refinement, three def available, 1: overlap, 2: overlap+share_intron_splice_site, 
                        3: overlap+share_intron_splice_site+shared_exon_splice_site (default: 3)

-o, --outprefix         The prefix for output files (default: Leafcutter_)

--use_TPM               A flag on whether to use TPM or normalized count

--preprocessed          A flag on Whether the files provided are already normalized, mainly for rerunning the pipeline and don't 
                        perform normalization again
--normalization_scale   The mode use for normaliztion, whether the count/TPM scale is based on junction count simulation, local (gene level) or global level,
                        can only input junction, local, or global (default: junction)

--samplecutoff          Minimum Normalized count/TPM for an intron in a sample to count as exist (default: 0)

--introncutoff          Minimum Normalized count/TPM for an intron to count as exist(default 5)

--m, --minclucounts     Minimum Normalized count/TPM to support a cluster (default: 30)

-r, --mincluratio       Minimum fraction of reads in a cluster that supports an intron (default: 0.01)

--read_len              The read length of sequencing data, use to simulate junction count, only work when normalization_scale = "junction"

--not_paired_end        Whether the reads are not paired end use to simulate junction count, only work when normalization_scale = "junction"
                        (default: False)
--overhang              The oeverhand that would like to use, could be set to zero, use to simulate junction count, 
                        only work when normalization_scale = junction (default: 2)

--sizing_factor         The sizing factor for junction simulation normalization to better calibrate the p-values (default: 1)

```


### tealeaf_sc

```
usage: python tealeaf_sc.py [--alevin_dir] [--salmon_ref] [--ref_dir] [--barcodes_cluster] [--pseudobulk_samples]
                          [-n/--num_cell] [-k/--num_bootstrapping] [--min_eq] [--group_method] [--ref_prefix]
                          [--thread] [--cluster_def] [-o/--outprefix] [-n/--normalization] [--samplecutoff] 
                          [--introncutoff] [-m/--minclucounts] [-r/--mincluratio] [--preprocessed]
                          [--read_len] [--not_paired_end] [--overhang] [--sizing_factor]


or when install with pip
tealeaf-sc [--alevin_dir] [--salmon_ref] [--ref_dir] [--barcodes_cluster] [--pseudobulk_samples]
                          [-n/--num_cell] [-k/--num_bootstrapping] [--min_eq] [--group_method] [--ref_prefix]
                          [--thread] [--cluster_def] [-o/--outprefix] [--normalization_scale] [--samplecutoff] 
                          [--introncutoff] [-m/--minclucounts] [-r/--mincluratio] [--preprocessed]
                          [--read_len] [--not_paired_end] [--overhang] [--sizing_factor]


Mandatory parameters:

--alevin_dir            The directory for alevin results, the file should contain the eq matrix and other files

--salmon_ref            The reference used for salmon index, The salmon reference,  maybe spliceu or splicei

--ref_dir               leafcutterITI reference directory, which should contain the matrices for isoform to intron and exon

--barcodes_cluster      The file that records which barcodes belong to which cluster/cell type in the format 'barcode,cluster' 
                        this file will be used to generate pseudobulk samples 

--pseudobulk_samples    A txt file with barcodes to pseudobulk sample are expected in format 'barcode pseudobulk_ample', if \
                        this option != None, then it will overwrite the input to --barcodes_cluster, and use the file in this option \
                        for computation. Only one of barcodes_cluster or pseudobulk_samples is required

Optional Parameters:
--ref_prefix            The prefix that is used to generate isoform to intron map using
                        leacutterITI_map_gen (default: '')

--n,--num_cell          The number of cell/barcode that you would like to include in a pseudobulk sample, cluster/cell type that has fewer
                        cell/barcodes than this number will not included in the computation (default: 100)

-k,--num_bootstrapping  The number of bootstrapping samples generated for each cluster/cell type if using bootstrapping to generate pseudobulk sample (default: 30)

--min_eq                Minimum count for each eq class for it to be included in the EM (default: 5)

--pseudobulk_method     The pseudobulk sample generate method, could be metacells or bootstrapping (default: metacells)

--cluster_def           The definition used for cluster refinement, three def available, 1: overlap, 2: overlap+share_intron_splice_site, 
                        3: overlap+share_intron_splice_site+shared_exon_splice_site (default: 3)

-o, --outprefix         The prefix for output files (default: leafcutter_)

--thread                The number of threads used for parallel computation, should not be too large to avoid crash (default: 8)

--use_TPM               A flag on whether to use TPM or normalized count

--preprocessed          A flag on whether pseudobulk generation and EM were done, if true, then the pipeline starts from counting intron (default: False)

-v,--with_virtual       A flag on whether the map that contain virtual intron to capture AFE and ALE

--samplecutoff          Minimum Normalized count/TPM for an isoform in a sample to count as exist (default: 0.1)

--introncutoff          Minimum Normalized count/TPM for an intron to count as exist(default: 80)

--m, --minclucounts     Minimum Normalized count/TPM to support a cluster (default: 100)

-r, --mincluratio       Minimum fraction of reads in a cluster that supports an intron (default 0.01)

--normalization_scale   The mode use for normaliztion, whether the count/TPM scale is based on junction count simulation, local (gene level) or global level,
                        can only input junction, local, or global (default: junction)

--read_len              The read length of sequencing data, use to simulate junction count, only work when normalization_scale = "junction" (default: 100)

--not_paired_end        Whether the reads are not paired end use to simulate junction count, only work when normalization_scale = "junction" (default: False)
--overhang              The oeverhand that would like to use, could be set to zero, use to simulate junction count, 
                        only work when normalization_scale = junction (default: 2)

--sizing_factor         The sizing factor for junction simulation normalization to better calibrate the p-values (default: 1)

```


### tealeaf_ggsashimi


```
usage: python tealeaf-ggsashimi.py


or when install with pip
tealeaf-ggsashimi


Mandatory parameters:




```






## Detailed Tutorial to run the tealeaf

In this tutorial, we walk through all the steps to run the tealeaf pipeline. For each step, we discuss the possible parameters that can be changed, how to do so and the considerations involved in each of the parameters. Finally, we show example inputs and outputs of each step (with column explanations) so the user knows what to expect and can make custom files as needed.


### Step 0: Transcriptome annotation download or generation and Salmon isoform quantification

Example human transcriptome annotation can be downloaded from https://www.gencodegenes.org/human/


### Step 1: Isoform to intron map generation

In this step, tealeaf_map_gen will be used to generate a map that contains information about which isoform is generated by splicing which introns. The map will also contain information about which exon is in which isoform. This step only needs to run once for each unique transcriptome annotation gtf file. 

Sample run:
```
python tealeaf_map_gen -a gencode.v45.annotation.gtf --annot_source gencode -o sample_run_ --maxintronlen 5000000 --minintronlen 50 -v False          
```


Depending on the setting, two or four files will be generated.
- {out_prefix}isoform_intron_map.tsv
- {out_prefix}intron_exon_connectivity.tsv
- {out_prefix}isoform_intron_map_with_virtual.tsv
- {out_prefix}intron_exon_connectivity_with_virtual.tsv

where with_virtual mean virtual intron was used to capture all annotated AFE and ALE (a testing feature).

if annotation_source='gencode', an additional file will be generated to give out information about the possible isoform type that can be generated by splicing out each intron
- {out_prefix}intron_source_map.tsv (also a testing feature)

  
A record file that contains the parameters will also be generated

When --single_cell == True, five additional files will be generated. Two for sparse matrices in npz format, rows are isoforms, and columns are introns or exons. Three txt files record the row and column names. These file is essential for tealeaf-sc.




### Step 2: Salmon isoform quantification

tealeaf utlized pseudoalignment method Salmon for bulk and preprocessed pseudobulk data. For usage of Salmon please refer https://salmon.readthedocs.io/en/latest/salmon.html

For single-cell data, leafcutterITI utilized alevin-fry pipeline form Salmon. The usage of alevin-fry please refer https://alevin-fry.readthedocs.io/en/latest/. 
Specific notices, please use -d, --dump-eqclasses flag when using alevin-fry quant to obtain the eqclass matrix. Also, t2t mapping should be used instead of normal t2g mapping. t2t mapping can be easily obtained by replacing the gene col in t2g file with the transcripts.

In the rest of the tutorial, we assume RNA-seq data aligned to the transcriptome using Salmon or Alevin-fry. 

### Step 2.1: Single-cell clustering after alevin-fry

For single-cell data, after pseudoaligment, we will need to process the data and obtain a barcodes to clusters/cell_types csv file that have row in format 'barcode,cluster/cell_type'. 
There are different single-cell analysis tool can achieve this goal. For examples, Seurat and Scanpy. Any analysis tool could work as long as the barcodes to clusters/cell_types csv file is provided. 
For our analysis, we used Scanpy and tutorial for cell clustering with Scanpy could be found at https://scanpy.readthedocs.io/en/stable/tutorials/basics/clustering.html. 
After the clustering and cell labeling, the barcodes to clusters/cell_types could be export like 
`adata.obs[['cell_barcodes', 'cluster_name']].to_csv('barcode_to_cluster.csv', index = False, header = None)` 



### Step 3.1: tealeaf clustering for bulk or pseudobulk data

For this step, we assume the data we are processing are bulk or pseudobulk data, we will need the file generated from step 1, the file path and the name for the isoform quantification files generated by Salmon, and the transcriptome annotation


Sample run:
```
python tealeaf_clustering --map transcript_intron_map.tsv --count_files quantification_files.txt --connect_file intron_exon_connectivity.tsv -a gencode.v45.annotation.gtf --cluster_def 3 \
                                --normalization True -o sample_run_ --minclucounts 30 --mincluratio 0.01
```


Two main output files will be obtained:
- {outprefix}refined_cluster
- {outprefix}ratio_count

sample {out_prefix}refined_cluster
```
sample1.sf sample2.sf sample3.sf sample4.sf sample5.sf sample6.sf
chr1:17055:17233:clu_1 21.1 13 18 20 17 12 
chr1:17055:17606:clu_1 4 11.4 12 7 2 0 5 
chr1:17368:17606:clu_1 127 132 128 55 93 90 68 
chr1:668593:668687:clu_2 3 11.3 1 3 4 4 8 
chr1:668593:672093:clu_2 11 16 23 2.5 3 20 9
```

These two files are equivalent to Leafcutter clustering numers.counts.gz and counts.gz. It is worth noticing that the normalized count or TPM is not necessarily an integer, but the normalized count will exhibit a count-like property.



### Step 3.2: LeafcutterITI clustering for single-cell data

For this step, we assume the data we are processing are single-cell data. Results from Step 2 and Step 2.1 were obtained. The files required for this step will be 1. the salmon/alevin directory that contains files `gene_eqclass.txt.gz`, `geqc_counts.mtx`, and other relevant files from alevin-fry 2. Directory for where the mapping were generated by leafcutterITI-map_gen 3. the reference that salmon use to generate the index 4. a barcode to cluster file or barcodes to pseudobulk sample file. 

Other parameters are optional. 

Sample run:
```
LeafcutterITI-scITI --alevin_dir salmon/out_permit_know/quant_spliceu_t2t --salmon_ref salmon_index/spliceu.fa --ref_dir leafcutterITI_map_dir
                    --barcodes_cluster barcodes_clusters.txt  --cluster_def 3 --normalization True --preprocessed False -o sample_run_
                    --minclucounts 30 --mincluratio 0.01 -n 100
```

Similar to leacutterITI-clustering, two main output files will be obtained:
- {outprefix}refined_cluster
- {outprefix}ratio_count

This step will also generate other relevant files like `barcodes_pseudobulk.txt` that record each step of the computation.




### Step 4:
The output from step 3 is equivalent to results from leafcutter clustering, and the results are compatible with downstream analysis for Leafcutter, such as Leafcutter_ds and Leafviz. 
Further information and downstream analysis please refer to https://davidaknowles.github.io/leafcutter/index.html
















# SARS2_host_derived_insertions
This repository contains scripts and data for the direct RNA-seq analysis and generating figures described in paper "Putative host-derived insertions in the genomes of circulating SARS-CoV-2 variants" by Yiyan Yang, Keith Dufault-Thompson, Rafaela Salgado Fontenele, Xiaofang Jiang.

## Content
1. a pipeline to detect chimeric reads from direct RNA-seq data
2. a custom script to verify insertion with raw sequencing data
3. codes to generate Figures

## Prerequisites
All codes were run and tested on Linux
- Python >=3.8
- Biopython=1.76
- Minimap2
- Paftools
- NanoFilt
- datamash
- sratoolkit

## Workflow for direct RNA-seq analysis
### Input:
#### fastq files

### Output:
#### 1. read_chimeric_pattern.tsv:  
This file stores the final chimeric reads detected in the direct RNA-seq data.
- column1: chimeric pattern, could be "sh" or "hs" (sh: SARS2-host chimeria; hs: host-SARS2 chimera). 
- column2: read name.  
- column3: junction sites on references (ref1:position1,ref2:position2).  
- column4: junction start and end sites on read (ref1,ref2).  
- column5: selected region on read to get the averaged quality score for the junction region.  
- column6: the length of selected region on read in column 5.  
- column7: averaged quality score for the junction region.  
- column8: the percentage of the averaged quality score greater than the 1000-time simulated averaged quality score along the read.  
- column9: similar to column3, but more detailed information about alignment to references (ref1:ref1_length:map_start-map_end(strand),ref2:ref2_length:map_start-map_end(strand)).  
- column10: the sample name for this read.  
#### 2. sars2_insert_start.list:
column1: junction site relative to SARS-CoV-2 reference genome.
column2: chimeric pattern, could be "sh" or "hs".

### Steps
- Step.1: align raw reads fastq files to references and convert to paf format.
- Step.2: extract host-virus chimeric reads from all samples and merged into one file.
- Step.3: get junction start and end positions for each chimeric read and summarize them into a tab-delimited file.
- Step.4: filter out chimeric reads whose 1) junction length >= 15bp; 2) junction occurs within the last 50 nucleotides of the first gene sequence or within the first 50 nucleotides of the second gene sequence.
- Step.5: split fastq file into multiple single-read fastq files for following quality score checking.
- Step.6: do permutation test to check if the quality score within 20 bp of either side of the junction region is higher than the 20th percentile quality score for each read.
- Step.7: filter out chimeric reads that don't meet the demand in step 6.
- Step.8: annotate junctions with chimeric pattern either as "hs" (short for "5'-host-SARS2-3' chimeric read") or "sh" (short for "5'-SARS2-host-3' chimeric read").
- Step.9: get junction sites of chimeric reads relative to SARS-CoV-2 reference.

## Usage
```
#### direct RNA-seq analysis ####
# git repo
git clone https://github.com/ncbi/SARS2_host_derived_insertions.git
cd SARS2_host_derived_insertions

# download fastq data
./prepare_fastq_data.sh Homo_sapiens

# for Chlorocebus_sabaeus, please run:
# Notice: VeroInf24h.all.fastq need to be downloaded manually from https://osf.io/8f6n9/
# and CRR197323.fastq, CRR197322.fastq, CRR197321.fastq, CRR197320.fastq, CRR127517.fastq, and CRR127516.fastq need to be downloaded manually from https://bigd.big.ac.cn/ under accession number GSA:CRA002508 PRJCA002477.
# ./prepare_fastq_data.sh Chlorocebus_sabaeus

# preparing reference data into specific folder
./prepare_ref_data.sh Homo_sapiens

# for Chlorocebus_sabaeus, please run:
# ./prepare_ref_data.sh Chlorocebus_sabaeus

# Run on Linux
# Default host is Homo sapiens
./Pipeline.sh Homo_sapiens

# for Chlorocebus_sabaeus, please run:
# ./Pipeline.sh Chlorocebus_sabaeus

#### verify insertion with raw sequencing data ####
cd verify_insertion/WA-PHL-005726_data
./process.sh &> log
# get reads number
w_insert_read_ct=$(awk 'END{print NR}' w_insert.reads)
echo "reads exclusively aligned to the consensus genome with the insertion: ${w_insert_read_ct}"
wo_insert_read_ct=$(awk 'END{print NR}' wo_insert.reads)
echo "reads exclusively aligned to the consensus genome without the insertion: ${wo_insert_read_ct}"

cd verify_insertion/HI-H215617
./process.sh &> log
# get reads number
w_insert_read_ct=$(awk 'END{print NR}' w_insert.reads)
echo "reads exclusively aligned to the consensus genome with the insertion: ${w_insert_read_ct}"
wo_insert_read_ct=$(awk 'END{print NR}' wo_insert.reads)
echo "reads exclusively aligned to the consensus genome without the insertion: ${wo_insert_read_ct}"

#### generate figures ####
cd Figure1
Rscript --vanilla plot_figure1.R

cd Figure2
Rscript --vanilla plot_figure2.R
```

## Citation
Putative host-derived insertions in the genome of circulating SARS-CoV-2 variants
Yiyan Yang, Keith Dufault-Thompson, Rafaela Salgado Fontenele, Xiaofang Jiang
bioRxiv 2022.01.04.474799; doi: https://doi.org/10.1101/2022.01.04.474799

## Acknowledgements
We would like to thank Sofya K. Garushyants and Eugene V. Koonin for their code on how to perform permutation tests and plot the results provided at [covid_insertions_paper](https://github.com/garushyants/covid_insertions_paper) which was adjusted and used in this project.

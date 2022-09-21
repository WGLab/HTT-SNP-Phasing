# Haplotyping SNPs for allele-specific deletion of the expanded huntingtin (mHTT) gene

Huntington's disease (HD) is an autosomal dominant neurodegenerative disease. Individuals with HD suffer from progressive motor, cognitive and psychiatric disturbances over the course of 10-20 years. At the molecular level, HD is caused by a CAG trinucleotide repeat expansion in exon-1 of the huntingtin (HTT) gene located at chr4. In the normal population, the CAG repeat is in the range of 6-35. When expanded to ≥35 repeats, HD is likely to develop. In individuals with 36-39 repeats, there is partial penetrance with full penetrance when there are ≥40 repeats. 

Allele-specific CRISPR/Cas9 editing of the expanded huntingtin (mHTT) can be achived by taking advantage of heterozygous SNPs that either eliminate or create PAM sequences. To identify all potential SNPs that may be used for allele-specific deletion of the mHTT, we sequenced the exon-1 surrounding region (chr4:3069608-3079972, GRCh38) of two HD cohorts with a total of 1056 individuals using long-read sequencing. 


Sometimes long-read sequencing is available and people only have SNP genotypes identified from SNP arrays or short-read sequencing. In this case, phased SNPs (haplotypes) released in this study can be used as a reference panel for statistical phasing. In this repository, we provide a tutorial for statistical phasing using the HD haplotypes released in this study. 


## 1. Download `SHAPEIT`

In this tutorial, we will use [SHAPEIT](https://mathgen.stats.ox.ac.uk/genetics_software/shapeit/shapeit.html) to perform statistical phasing. `SHAPEIT` can be downloaded from [here](https://mathgen.stats.ox.ac.uk/genetics_software/shapeit/shapeit.html#download).


## 2. Reference haplotypes of HD population

In the `htt_reference_haplotypes` folder, We provide haplotypes of the French cohort. There are three files in this folder. These files were formatted so that they can be used directly as input of `SHAPEIT`. You don't need to modify these files unless you want to prepare your own reference panel. 

The `HTT_French_cohort_reference.sample` file describes the sample information. This dataset includes 348 HD samples. 

The `HTT_French_cohort_reference.legend` file describes the SNPs. Afer removing low-frequency SNPs (allele count < 9 in this dataset), 51 SNPs were included in the reference panel. There are four columns: SNP ID, position, first allele, second allele. SNP positions are based on GRCh38. For all SNPs, we use `R` and `A` to denote the reference and alternative allele respectively. **Of note, we use `chr4-3074876-CAG` to denote the CAG repeat in exon-1 of the HTT gene. We treat it as a 'SNP' in the phasing analysis although it is a repeat expansion variant. Similar to SNPs, the allele with normal-length CAG is denoted as `R` and the allele with expaned CAG repeat is denoted as `A`.** 
Haplotyping the CAG repeat is crutial because we need to known whether the SNPs are in normal HTT or mutant HTT.

The `HTT_French_cohort_reference.hap` file describes the haplotypes. It is a space-delimited text file with 51 rows and 696 columns. Each row corresponds to a SNP and each column is a haplotype (in the order of `HTT_French_cohort_reference.legend` and `HTT_French_cohort_reference.sample`). `0` means reference allele and `1` means alternative allele. 

Details about the file format can be found [here](https://mathgen.stats.ox.ac.uk/genetics_software/shapeit/shapeit.html#haplegsample)
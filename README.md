# Haplotyping SNPs for allele-specific deletion of the expanded huntingtin (mHTT) gene

Huntington's disease (HD) is an autosomal dominant neurodegenerative disease. Individuals with HD suffer from progressive motor, cognitive and psychiatric disturbances over the course of 10-20 years. At the molecular level, HD is caused by a CAG trinucleotide repeat expansion in exon-1 of the huntingtin (HTT) gene located at chr4. In the normal population, the CAG repeat is in the range of 6-35. When expanded to >35 repeats, HD is likely to develop. In individuals with 36-39 repeats, there is partial penetrance with full penetrance when there are ≥40 repeats. 

Allele-specific CRISPR/Cas9 editing of the expanded huntingtin (mHTT) can be achived by taking advantage of heterozygous SNPs that either eliminate or create PAM sequences. To identify all potential SNPs that may be used for allele-specific deletion of the mHTT, we sequenced the exon-1 surrounding region (chr4:3069608-3079972, GRCh38) of two HD cohorts with a total of 1056 individuals using long-read sequencing. 


Sometimes long-read sequencing is available and people only have SNP genotypes identified from SNP arrays or short-read sequencing. In this case, phased SNPs (haplotypes) released in this study can be used as a reference panel for statistical phasing. In this repository, we provide a tutorial for statistical phasing using the HD haplotypes released in this study. 


## 1. Download `SHAPEIT`

In this tutorial, we will use [SHAPEIT](https://mathgen.stats.ox.ac.uk/genetics_software/shapeit/shapeit.html) to perform statistical phasing. Instructions for downloading `SHAPEIT` are [here](https://mathgen.stats.ox.ac.uk/genetics_software/shapeit/shapeit.html#download). Academic users can download it for free. 


## 2. Reference haplotypes of HD population

In the `htt_reference_haplotypes` folder, we provide haplotypes of the French cohort. There are three files in this folder. You don't need to modify these files unless you want to prepare your own reference panel. 

The `HTT_French_cohort_reference.sample` file describes the sample information. This dataset includes 348 HD samples.

The `HTT_French_cohort_reference.legend` file describes the SNPs. Afer removing low-frequency SNPs (allele count < 9 in this dataset), 51 SNPs were included in the reference panel. Each row is an SNP. There are four columns: SNP ID, position, first allele, second allele. In our reference panel, SNP positions are based on GRCh38. The first 10 rows of the file is shown below.

|        id        | position | a0 | a1 |
|:----------------:|:--------:|:--:|:--:|
| chr4-3069952-C-G |  3069952 |  R |  A |
| chr4-3070120-C-G |  3070120 |  R |  A |
| chr4-3070451-C-T |  3070451 |  R |  A |
| chr4-3070504-G-A |  3070504 |  R |  A |
| chr4-3070599-G-A |  3070599 |  R |  A |
| chr4-3070636-C-G |  3070636 |  R |  A |
| chr4-3070672-A-G |  3070672 |  R |  A |
| chr4-3070714-C-G |  3070714 |  R |  A |
| chr4-3071436-A-G |  3071436 |  R |  A |

The SNP ID is in the format of 'Chrom-Position-Ref-Alt'. We did not use RS numbers because one RS number may be linked to multiple SNPs. 

For simplicity, we use `R` and `A` to denote the reference and alternative allele for all SNPs. The actual reference and alternative allele can be infered from the SNP ID. 

**NOTE: We use `chr4-3074876-CAG` to denote the CAG repeat in exon-1 of the HTT gene. We treat it as a 'SNP' in the phasing analysis although it is a repeat expansion variant. Similar to SNPs, a normal-length CAG repeat is denoted as `R` and an expaned CAG repeat is denoted as `A`. The position is assumed to the start position the repeat (3074876)** 
Haplotyping the CAG repeat is crutial because we need to known whether the SNPs are in normal HTT or mutant/expanded HTT. 

The `HTT_French_cohort_reference.hap` file describes the haplotypes. It is a space-delimited text file with 51 rows and 696 columns. Each row corresponds to a SNP and every two column is a sample (in the order of `HTT_French_cohort_reference.legend` and `HTT_French_cohort_reference.sample`). `0` means reference allele and `1` means alternative allele. 

Details about the file format can be found [here](https://mathgen.stats.ox.ac.uk/genetics_software/shapeit/shapeit.html#haplegsample)

## 3. Example data

We provide an example dataset (unphased genotypes) in the `example_data` folder. The dataset is in PED and MAP format. A detailed description of the formats can be found [here](https://mathgen.stats.ox.ac.uk/genetics_software/shapeit/shapeit.html#formats). 

The `example1.map` file describes the SNPs in the dataset. This file can be SPACE or TAB delimited. Each line corresponds to a SNP. There are four columns: 
- Chromosome number [integer]
- SNP ID [string]
- SNP genetic position (cM) [float]
- SNP physical position (bp) [integer]

The first 10 rows of the `example1.map` file are shown below. 

```
4	chr4-3069952-C-G	0	3069952
4	chr4-3070120-C-G	0	3070120
4	chr4-3070451-C-T	0	3070451
4	chr4-3070504-G-A	0	3070504
4	chr4-3070599-G-A	0	3070599
4	chr4-3070636-C-G	0	3070636
4	chr4-3070672-A-G	0	3070672
4	chr4-3070714-C-G	0	3070714
4	chr4-3071436-A-G	0	3071436
4	chr4-3071526-C-T	0	3071526
```
There is no header line. If the genetic position in [centiMorgan](https://en.wikipedia.org/wiki/Centimorgan) is unknown, it can be set to 0. The fourth column is the physical position in the chromosome (in our case it is based on GRCh38). 

In row 26, the CAG repeat is denoted as 
```
4	chr4-3074876-CAG	0	3074876
```

**NOTE: To be consitent with the `HTT_French_cohort_reference.legend` file, the ID of the CAG repeat must be `chr4-3074876-CAG` and the position must be `3074876`.**

The `example1.ped` file describes the individuals and unphased genotypes of each individual. This file can be SPACE or TAB delimited. Each line corresponds to a single individual. The first 6 columns are:

- Family ID [string]
- Individual ID [string]
- Father ID [string]
- Mother ID [string]
- Sex [integer] (1=male; 2=female; other=unknown)
- Phenotype [float] (-9=missing; 0=missing; 1=unaffected; 2=affected)

For HD individuals, we set the Phenotype value to 2. 

Columns 7 and 8 code for the observed alleles at the first SNP, columns 9 and 10 code for the observed alleles at the second SNP, and so on. Missing data can be coded as `0 0`.This file should have N lines and 2L+6 columns, where N and L are the numbers of individuals and SNPs contained in the dataset respectively.

NOTE: 
- If the HD individual has one normal HTT and one expanded HTT, the genotypes of the CAG repeat (`chr4-3074876-CAG`) can be coded as `0 1` or `1 0`. The order doesn't matter because the input PED file is considered to be unphased. 

- There are 51 SNPs in the reference panel but it is OK that the PED/MAP files include more/less SNPs. `SHAPEIT` will do a pre-phasing check to remove SNPs that only exist in the input data (PED/MAP files) or the reference panel. 


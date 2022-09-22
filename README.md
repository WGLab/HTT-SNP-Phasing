# Haplotyping SNPs for allele-specific deletion of the expanded huntingtin (mHTT) gene

Huntington's disease (HD) is an autosomal dominant neurodegenerative disease. Individuals with HD suffer from progressive motor, cognitive and psychiatric disturbances over the course of 10-20 years. At the molecular level, HD is caused by a CAG trinucleotide repeat expansion in exon-1 of the huntingtin (HTT) gene located at chr4. In the normal population, the CAG repeat is in the range of 6-35. When expanded to >35 repeats, HD is likely to develop. In individuals with 36-39 repeats, there is partial penetrance with full penetrance when there are ≥40 repeats. 

Allele-specific CRISPR/Cas9 editing of the expanded huntingtin (mHTT) can be achived by taking advantage of heterozygous SNPs that either eliminate or create PAM sequences. To identify all potential SNPs that may be used for allele-specific deletion of the mHTT, we sequenced the exon-1 surrounding region (chr4:3069608-3079972, GRCh38) of two HD cohorts with a total of 1056 individuals using long-read sequencing. 


Sometimes long-read sequencing is available and people only have SNP genotypes identified from SNP arrays or short-read sequencing. In this case, phased SNPs (haplotypes) released in this study can be used as a reference panel for statistical phasing. In this repository, we provide a tutorial for statistical phasing using the HD haplotypes released in this study.  **Please note that results from statistical phasing are based on probabilities and we strongly recommend experimental validation for downstream clinical applications.**

## Quick Start

1. Download `SHAPEIT` following instructions [here](https://mathgen.stats.ox.ac.uk/genetics_software/shapeit/shapeit.html#download). Academic users can download it for free. 

2. Run the following commands. 
```
# download reference haplotypes and example data
git clone https://github.com/WGLab/HTT-SNP-Phasing.git 

cd path/to/HTT-SNP-Phasing
mkdir -p output

# check input data
path/to/shapeit -check \
  -P ./example_data/example1 \
  -M ./genetic_map/genetic_map_hg38_chr4.txt \
  --input-ref ./htt_reference_haplotypes/HTT_French_cohort_reference.hap ./htt_reference_haplotypes/HTT_French_cohort_reference.legend ./htt_reference_haplotypes/HTT_French_cohort_reference.sample \
  --output-log ./output/example1

# output most likely haplotypes
path/to/shapeit  \
  -P ./example_data/example1 \
  -M ./genetic_map/genetic_map_hg38_chr4.txt \
  --input-ref ./htt_reference_haplotypes/HTT_French_cohort_reference.hap ./htt_reference_haplotypes/HTT_French_cohort_reference.legend ./htt_reference_haplotypes/HTT_French_cohort_reference.sample \
  --exclude-snp ./output/example1.snp.strand.exclude \
  --output-max ./output/example1.max

# output haplotype graph
path/to/shapeit  \
  -P ./example_data/example1 \
  -M ./genetic_map/genetic_map_hg38_chr4.txt \
  --input-ref ./htt_reference_haplotypes/HTT_French_cohort_reference.hap ./htt_reference_haplotypes/HTT_French_cohort_reference.legend ./htt_reference_haplotypes/HTT_French_cohort_reference.sample \
  --exclude-snp ./output/example1.snp.strand.exclude \
  --output-graph ./output/example1.graph

# sampling haplotypes from the graph to capture uncertainty
for seed in $(seq 1 100); do
    path/to/shapeit -convert \
      --input-graph ./output/example1.graph \
      --output-sample ./output/example1.haplotype_sample.seed$seed \
      --seed $seed;
done
rm shapeit*.log
```

## Detailed Instructions
### 1. Download `SHAPEIT` and our data

In this tutorial, we will use [SHAPEIT](https://mathgen.stats.ox.ac.uk/genetics_software/shapeit/shapeit.html) to perform statistical phasing. Instructions for downloading `SHAPEIT` are [here](https://mathgen.stats.ox.ac.uk/genetics_software/shapeit/shapeit.html#download). Academic users can download it for free. 

To download our data:

```
git clone https://github.com/WGLab/HTT-SNP-Phasing.git
```

### 2. Reference haplotypes of HD population

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

### 3. Example data

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

### 4. Preprocessing of input data

`SHAPEIT` can only phase SNPs that are in both input data and the reference panel with the same REF/ALT alleles. So the first step is to check the data and remove SNPs that cannot be used for phasing. A data check can be run using the following commands: 

```
cd path/to/HTT-SNP-Phasing

mkdir -p output

path/to/shapeit -check \
  -P ./example_data/example1 \
  -M ./genetic_map/genetic_map_hg38_chr4.txt \
  --input-ref ./htt_reference_haplotypes/HTT_French_cohort_reference.hap ./htt_reference_haplotypes/HTT_French_cohort_reference.legend ./htt_reference_haplotypes/HTT_French_cohort_reference.sample \
  --output-log ./output/example1
```

In the `output/example1.log` file, you will find the following information: 

```
Reading SNPs in [./htt_reference_haplotypes/HTT_French_cohort_reference.legend]
  * 10 reference panel sites included
  * 41 reference panel sites excluded

ERROR: Reference and Main panels are not well aligned:
  * #Missing sites in reference panel = 0
  * #Misaligned sites between panels = 41
  * #Multiple alignments between panels = 0
```

41 reference panel sites are excluded. This is because these SNPs are homozygous in this sample and `SHAPEIT` failed to find the alternative allele in our data. This error can be ignored because we don't need to phase homozygous SNPs. These sites can be excluded during a formal phasing run. The `output/example1.snp.strand.exclude` file records the positions of these SNPs. 


### 5. Phasing

#### 5.1 Output the most likely haplotype

If you only want to know the most likly haplotype, you can run the following command:

```
path/to/shapeit  \
  -P ./example_data/example1 \
  -M ./genetic_map/genetic_map_hg38_chr4.txt \
  --input-ref ./htt_reference_haplotypes/HTT_French_cohort_reference.hap ./htt_reference_haplotypes/HTT_French_cohort_reference.legend ./htt_reference_haplotypes/HTT_French_cohort_reference.sample \
  --exclude-snp ./output/example1.snp.strand.exclude \
  --output-max ./output/example1.max
```

Two files will be generated in the `./output` folder:  `example1.max.haps`  and `example1.max.sample`. The predicted haplotypes are store in the `example1.max.haps` file:

```
4 chr4-3073068-A-G 3073068 R A 0 1
4 chr4-3073964-G-C 3073964 R A 0 1
4 chr4-3074876-CAG 3074876 A R 0 1
4 chr4-3074932-A-C 3074932 R A 0 1
4 chr4-3074935-A-C 3074935 R A 0 1
4 chr4-3074938-A-C 3074938 R A 0 1
4 chr4-3074945-A-G 3074945 R A 0 1
4 chr4-3077210-A-G 3077210 R A 0 1
4 chr4-3078446-G-A 3078446 R A 0 1
4 chr4-3078472-A-G 3078472 R A 0 1
```

Please pay attention to the order of the `R` and `A` alleles. Sometimes `A` is before `R`. The first allele in this output file may be the first one appear in the PED file. In our case, all SNPs are in the same haplotype and the expanded CAG repeat (`chr4-3074876-CAG`) is in the other haplotype as the order of `A` and `R` is reversed in the third row: `4 chr4-3074876-CAG 3074876 A R 0 1`.

The predicted haplotypes are correct because this sample is one individual in the French cohort and we know answer. 

**NOTE: The output haplotype is the haplotype with max probablity. There are still uncertainties and it is recommanded use the following method to capture the phase uncertainty.**


#### 5.2 Capturing phase uncertainty

`SHAPEIT` can produce a haplotype graph and phase uncertainty can be captured by random sampling haplotypes from the graph. So the very first step is to produce the haplotype graph:

```
path/to/shapeit  \
  -P ./example_data/example1 \
  -M ./genetic_map/genetic_map_hg38_chr4.txt \
  --input-ref ./htt_reference_haplotypes/HTT_French_cohort_reference.hap ./htt_reference_haplotypes/HTT_French_cohort_reference.legend ./htt_reference_haplotypes/HTT_French_cohort_reference.sample \
  --exclude-snp ./output/example1.snp.strand.exclude \
  --output-graph ./output/example1.graph
```

This will generate a `example1.graph` file in the `./outpout` folder. `example1.graph` is a binary file and is not human-readable. 

To sample 100 haplotype sets for example, you can run the following shell script:

```
for seed in $(seq 1 100); do
    path/to/shapeit -convert \
      --input-graph ./output/example1.graph \
      --output-sample ./output/example1.haplotype_sample.seed$seed \
      --seed $seed;
done
rm shapeit*.log
```

This will generate 100 haplotypes in the `./output` folder. Your downstream analysis can be based on the average of the 100 haplotypes.

NOTE: The seed of each sampling process should be different so that these samples are generated using different random number sequences. 



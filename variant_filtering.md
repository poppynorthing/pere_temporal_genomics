# Variant Filtering

## Generating VCF stats
Use Bcftools v1.19 ([Danecek et al. 2021](https://academic.oup.com/gigascience/article/10/2/giab008/6137722?login=true)) to generate overall summary stats and vcftools v0.1.16 ([Danecek et al. 2011](https://academic.oup.com/bioinformatics/article/27/15/2156/402296)) to generate more specific variant stats.

```
#!bin/bash

ml bcftools
ml vcftools

VCF=joint_genotyped.2.vcf.gz

# Generate summary stats on pre-filtered vcf file
bcftools stats $VCF > $VCF.vcfstats

# Generate variant stats

    # report mean depth per individual
    vcftools --gzvcf $VCF --depth --out vcf_stats/vcf.2

    # report mean depth per site
    vcftools --gzvcf $VCF --site-mean-depth --out vcf_stats/vcf.2

    # report per-individual missingness
    vcftools --gzvcf $VCF --missing-indv --out vcf_stats/vcf.2

    # report per-site missingness
    vcftools --gzvcf $VCF --missing-site --out vcf_stats/vcf.2

    # report per-individual heterozyosity and Fis
    vcftools --gzvcf $VCF --het --out vcf_stats/vcf.2
```

```
# VCF Statistics
# Modified from JR Rick

library(tidyverse)

# Load files
idepth <- read_table('Data/vcfstats/vcf.2.idepth') # Mean depth per individual
imiss <- read_table('Data/vcfstats/vcf.2.imiss') # Mean missingness per individual
ihet <- read_table('Data/vcfstats/vcf.2.het') # Observed homo/heterozygosity per individual
ldepth <- read_table('Data/vcfstats/vcf.2.ldepth.mean') # Mean depth per site
lmiss <- read_table('Data/vcfstats/vcf.2.lmiss') # Mean missingness per site

# Generate relevant summary statistics
summary(idepth)
summary(imiss)
summary(ihet)
summary(ldepth)
summary(lmiss)

# Visualize

# Distribution of observed heterozygosity per individual
ihet %>%
  mutate(HET_O = 1-(`O(HOM)`/N_SITES)) %>%
  ggplot(aes(x=HET_O)) +
  geom_histogram() +
  theme_classic() +
  labs(x="Observed Heterozygosity")

# Distribution of inbreeding (F) per individual
ihet %>%
  ggplot(aes(x=F)) +
  geom_histogram() +
  theme_classic() +
  labs(x = "Inbreeding (F)")

# Distribution of depth per site w/ cutoff included

ldepth %>%
  ggplot() +
  geom_histogram(aes(x=MEAN_DEPTH)) +
  xlim(0,75) +
  geom_vline(xintercept = 20, color = "red", linetype = "dashed") +
  theme_classic()
```

## Filtering variants
Based on the above, wee subsequently only kept sites that had a depth greater than 20, a fraction of missing data greater than 0.3, and a minor allele frequency greater than 0.05 using Bcftools v1.19 (Danecek et al. 2021).

```

```

## Linkage pruning

We calculated linkage decay and pruned linked variants using (plink)[https://pmc.ncbi.nlm.nih.gov/articles/PMC1950838/] v1.9 (Purcell et al. 2007). 
```
#!bin/bash

ml plink/1.9

VCF=joint_genotyped.recalc.maf.m30.filtd20.snps.2.vcf.gz

# Calculate LD with plink
plink --vcf $VCF --double-id --allow-extra-chr \
--set-missing-var-ids @:# \
--thin 0.1 -r2 gz --ld-window 100 --ld-window-kb 1000 \
--ld-window-r2 0 \
--make-bed --out joint_genotyped
```

## References
Petr Danecek, Adam Auton, Goncalo Abecasis, Cornelis A. Albers, Eric Banks, Mark A. DePristo, Robert E. Handsaker, Gerton Lunter, Gabor T. Marth, Stephen T. Sherry, Gilean McVean, Richard Durbin, 1000 Genomes Project Analysis Group, The variant call format and VCFtools, Bioinformatics, Volume 27, Issue 15, August 2011, Pages 2156–2158, https://doi.org/10.1093/bioinformatics/btr330

Petr Danecek, James K Bonfield, Jennifer Liddle, John Marshall, Valeriu Ohan, Martin O Pollard, Andrew Whitwham, Thomas Keane, Shane A McCarthy, Robert M Davies, Heng Li, Twelve years of SAMtools and BCFtools, GigaScience, Volume 10, Issue 2, February 2021, giab008, https://doi.org/10.1093/gigascience/giab008

Purcell S, Neale B, Todd-Brown K, Thomas L, Ferreira MA, Bender D, Maller J, Sklar P, de Bakker PI, Daly MJ, Sham PC. PLINK: a tool set for whole-genome association and population-based linkage analyses. Am J Hum Genet. 2007 Sep;81(3):559-75. doi: 10.1086/519795. Epub 2007 Jul 25. PMID: 17701901; PMCID: PMC1950838.

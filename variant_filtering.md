# Variant Filtering

## Generating VCF stats
Use bcftools v1.19 ([Danecek et al. 2021](https://academic.oup.com/gigascience/article/10/2/giab008/6137722?login=true)) to generate overall summary stats and vcftools v0.1.16 ([Danecek et al. 2011](https://academic.oup.com/bioinformatics/article/27/15/2156/402296)) to generate more specific variant stats.

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

## Filtering variants


```
```

## References
Petr Danecek, Adam Auton, Goncalo Abecasis, Cornelis A. Albers, Eric Banks, Mark A. DePristo, Robert E. Handsaker, Gerton Lunter, Gabor T. Marth, Stephen T. Sherry, Gilean McVean, Richard Durbin, 1000 Genomes Project Analysis Group, The variant call format and VCFtools, Bioinformatics, Volume 27, Issue 15, August 2011, Pages 2156–2158, https://doi.org/10.1093/bioinformatics/btr330

Petr Danecek, James K Bonfield, Jennifer Liddle, John Marshall, Valeriu Ohan, Martin O Pollard, Andrew Whitwham, Thomas Keane, Shane A McCarthy, Robert M Davies, Heng Li, Twelve years of SAMtools and BCFtools, GigaScience, Volume 10, Issue 2, February 2021, giab008, https://doi.org/10.1093/gigascience/giab008

## Structure

```
!bin/bash

ml plink

plink --vcf final_joint_genotyped.recalc.maf.m30.d20.snps.2.vcf.gz --allow-extra-chr 0 --double-id --keep-allele-order --max-alleles 2 --make-bed --out final_joint_genotyped.2

```

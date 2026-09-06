# Variant calling

## Mark duplicate reads

PCR and optical duplicates were removed using [picard](https://github.com/broadinstitute/picard) vv2.23.4 (Picard Toolkit 2019). Make sure that picard has enough memory; I had to set mem-per-cpu to 16
to get this to run.

```
#!/bin/bash

file=$(cat ./file_lists/sample_ids.txt | sed -n ${SLURM_ARRAY_TASK_ID}p)

ml picard

BAMDIR=sequences/aligned

# Mark duplicates (pcr + optical) w/ picard

picard MarkDuplicates I=$BAMDIR/"$file"_merged.rg.bam O=$BAMDIR/"$file".markedDups.bam M="$file".dupMetrics.txt
```

## Call variants

First, make a reference dictionary for the [<i>Pectocarya recurvata</i> reference chromosomes](https://bsapubs.onlinelibrary.wiley.com/doi/10.1002/aps3.70008) (Northing et al. 2025).

```
#!/bin/bash

ml picard
REF=/xdisk/kdlugosch/pcnorthing/Genome/pere_ch.fa

# Create a sequence dictionary for the reference 
picard CreateSequenceDictionary R=$REF O=pere_ch.dict
```

Then, run [gatk](https://gatk.broadinstitute.org/hc/en-us) HaplotypeCaller vX (Van der Auwera and O'Connor 2020) to call variants.
```
#!/bin/bash

ml gatk
ml samtools

BAMDIR=sequences/aligned
VCFDIR=sequences/vcf
REF=/xdisk/kdlugosch/pcnorthing/Genome/pere_ch.fa

# Make an index of every bam file
samtools index $BAMDIR/"$file".markedDups.bam

# use GATK haplotype caller to call variants for ploidy = 2:
gatk HaplotypeCaller -R $REF -I $BAMDIR/"$file".markedDups.bam -ploidy 2 -O $VCFDIR/"$file".2.vcf.gz -ERC GVCF

# use GATK haplotype caller to call variants for ploidy = 4:
gatk HaplotypeCaller -R $REF -I $BAMDIR/"$file".markedDups.bam -ploidy 4 -O $VCFDIR/"$file".4.vcf.gz -ERC GVCF

```

## References

Genomics in the Cloud: Using Docker, GATK, and WDL in Terra. Van der Auwera, G. A & O'Connor, B. D O'Reilly Media, Inc., (1st Edition) edition, 2020.

Li, Heng, Bob Handsaker, Alec Wysoker, Tim Fennell, Jue Ruan, Nils Homer, Gabor Marth, Goncalo Abecasis, Richard Durbin, 1000 Genome Project Data Processing Subgroup, The Sequence Alignment/Map format and SAMtools, Bioinformatics, Volume 25, Issue 16, August 2009, Pages 2078–2079, https://doi.org/10.1093/bioinformatics/btp352

Northing, P. C., J. A. Pelosi, D. L. Venable, and K. M. Dlugosch. 2025. Chromosome-scale reference genome of Pectocarya recurvata, the species with the smallest reported genome size in Boraginaceae. Applications in Plant Sciences 13(3): e70008. https://doi.org/10.1002/aps3.70008

Picard Toolkit. 2019. Broad Institute, GitHub Repository. https://broadinstitute.github.io/picard/; Broad Institute

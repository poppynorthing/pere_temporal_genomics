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

First, make a dictionary for the <i>P. recurvata</i> reference chromosomes.

```
#!/bin/bash

ml picard
REF=/xdisk/kdlugosch/pcnorthing/Genome/pere_ch.fa

# Create a sequence dictionary for the reference 
picard CreateSequenceDictionary R=$REF O=pere_ch.dict
```

Then, run gatk vX (REF) to call variants.
```
#!/bin/bash

ml gatk



```

## References

Li, Heng, Bob Handsaker, Alec Wysoker, Tim Fennell, Jue Ruan, Nils Homer, Gabor Marth, Goncalo Abecasis, Richard Durbin, 1000 Genome Project Data Processing Subgroup, The Sequence Alignment/Map format and SAMtools, Bioinformatics, Volume 25, Issue 16, August 2009, Pages 2078–2079, https://doi.org/10.1093/bioinformatics/btp352

Picard Toolkit. 2019. Broad Institute, GitHub Repository. https://broadinstitute.github.io/picard/; Broad Institute

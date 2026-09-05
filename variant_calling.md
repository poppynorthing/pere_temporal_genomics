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


```

```

## References

Li, Heng, Bob Handsaker, Alec Wysoker, Tim Fennell, Jue Ruan, Nils Homer, Gabor Marth, Goncalo Abecasis, Richard Durbin, 1000 Genome Project Data Processing Subgroup, The Sequence Alignment/Map format and SAMtools, Bioinformatics, Volume 25, Issue 16, August 2009, Pages 2078–2079, https://doi.org/10.1093/bioinformatics/btp352

Picard Toolkit. 2019. Broad Institute, GitHub Repository. https://broadinstitute.github.io/picard/; Broad Institute

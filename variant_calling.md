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

Picard Toolkit. 2019. Broad Institute, GitHub Repository. https://broadinstitute.github.io/picard/; Broad Institute

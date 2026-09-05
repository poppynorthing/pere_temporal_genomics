# Variant calling

## Mark duplicate reads

PCR and optical duplicates were removed using picard vX (REF). Make sure that picard has enough memory; I had to set mem-per-cpu to 16
to get this to run.

```
#!/bin/bash

file=$(cat ./file_lists/sample_ids.txt | sed -n ${SLURM_ARRAY_TASK_ID}p)

ml samtools
ml picard
ml gatk

BAMDIR=sequences/aligned
REF=/xdisk/kdlugosch/pcnorthing/Genome/pere_ch.fa

# Mark duplicates (pcr + optical) w/ picard

picard MarkDuplicates I=$BAMDIR/"$file"_merged.rg.bam O=$BAMDIR/"$file".markedDups.bam M="$file".dupMetrics.txt
```

## Call variants


```

```

# Trimming raw sequences and aligning to reference

Raw PE sequence reads were trimmed using [fastp](https://github.com/opengene/fastp) v0.23.4 (Chen 2025). Specifically, we did xyz thing to trim the reads.
Code adapted from '01_SequenceProcessingAndAlignment.sh' script by Brandon Thomas Hendrickson ([github repo: HerbariumStructure_WhiteClover](https://github.com/Brandon-Thomas-Hendrickson/HerbariumStructure_WhiteClover)).

```
RAW=temporal_genomics/sequences/raw
TRIMDIR=/temporal_genomics/sequences/trimmed
FASTPDIR=/temporal_genomics/sequences/fastp_report
S1=_R1_001.fastq.gz
S2=_R2_001.fastq.gz

#Trim herbarium sequence reads

for i in $(cat temporal_genomics/file_lists/raw_sequence_prefixes.txt);
do fastp -i $RAW/$i$S1 -I $RAW/$i$S2 -o $TRIMDIR/$i$S1 -O $TRIMDIR/$i$S2 --detect_adapter_for_pe --cut_right --dedup -h $FASTPDIR/$i'.html' -g -w 16;
done
```

Description of alignment

```
code here
```


References:

Shifu Chen. fastp 1.0: An ultra-fast all-round tool for FASTQ data quality control and preprocessing. iMeta 4.5 (2025): e70078 https://doi.org/10.1002/imt2.70078

# Trimming raw sequences and aligning to reference

Raw PE sequence reads were trimmed using [fastp](https://github.com/opengene/fastp) v0.23.4 (Chen 2025). Specifically, we did xyz thing to trim the reads.
Code adapted from '01_SequenceProcessingAndAlignment.sh' script by Brandon Thomas Hendrickson ([github repo: HerbariumStructure_WhiteClover](https://github.com/Brandon-Thomas-Hendrickson/HerbariumStructure_WhiteClover)).

```
!/bin/bash

#SBATCH --job-name=fastp
#SBATCH --output=fastp%i%j.out
#SBATCH --partition=standard
#SBATCH --account=kdlugosch
#SBATCH --ntasks=1
#SBATCH --nodes=1
#SBATCH --mem-per-cpu=4gb
#SBATCH --cpus-per-task=2
#SBATCH --time=1-12:00:00
#SBATCH --array=1-190%25

file=$(cat ./file_lists/raw_sequence_prefixes.txt | sed -n ${SLURM_ARRAY_TASK_ID}p)

ml fastp

RAW=sequences/raw
TRIMDIR=sequences/trimmed
FASTPDIR=sequences/fastp_report

#Trim herbarium sequence reads

fastp -i $RAW/"$file"_R1_001.fastq.gz -I $RAW/"$file"_R2_001.fastq.gz -o $TRIMDIR/"$file"_R1_001.fastq.gz -O $TRIMDIR/"$file"_R2_001.fastq.gz --detect_adapter_for_pe --cut_right --dedup -h $FASTPDIR/"$file".html -g -w 16
```

Description of alignment

```
code here
```


References:

Shifu Chen. fastp 1.0: An ultra-fast all-round tool for FASTQ data quality control and preprocessing. iMeta 4.5 (2025): e70078 https://doi.org/10.1002/imt2.70078

# Trimming raw sequences and aligning to reference

## Trimming
Raw PE sequence reads were trimmed using [fastp](https://github.com/opengene/fastp) v0.23.4 (Chen 2025). We removed adapter sequences (--detect_adapter_for_pe), removed low-quality reads (--cut_right), and removed PCR duplicate reads (--dedup).
Code partially adapted from '01_SequenceProcessingAndAlignment.sh' script by Brandon Thomas Hendrickson ([github repo: HerbariumStructure_WhiteClover](https://github.com/Brandon-Thomas-Hendrickson/HerbariumStructure_WhiteClover)).

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

fastp -i $RAW/"$file"_R1_001.fastq.gz -I $RAW/"$file"_R2_001.fastq.gz \
-o $TRIMDIR/"$file"_R1_001.fastq.gz -O $TRIMDIR/"$file"_R2_001.fastq.gz \
--detect_adapter_for_pe --cut_right --dedup -h $FASTPDIR/"$file".html -g -w 16
```
## Alignment
Trimmed reads were aligned to the chromosomes from the <i>Pectocarya recurvata</i> reference genome (Northing et al. 2025) using [BWA](https://github.com/lh3/bwa) mem v0.7.18 (Li 2013) with default settings.

```
#!/bin/bash

#SBATCH --job-name=bwa
#SBATCH --output=bwa%j.out
#SBATCH --partition=standard
#SBATCH --account=kdlugosch
#SBATCH --ntasks=1
#SBATCH --nodes=1
#SBATCH --mem-per-cpu=4gb
#SBATCH --cpus-per-task=2
#SBATCH --time=1-12:00:00
#SBATCH --array=1-190%25

file=$(cat ./file_lists/raw_sequence_prefixes.txt | sed -n ${SLURM_ARRAY_TASK_ID}p)

ml bwa
ml samtools

GENOME=/xdisk/kdlugosch/pcnorthing/Genome/pere_ch.fa
TRIMDIR=sequences/trimmed
ALIGNDIR=sequences/aligned

#Index Pectocarya recurvata reference chromosomes
bwa index $GENOME

#Align trimmed reads to PERE reference genome and convert sams to bams

bwa mem -t 16 $GENOME $TRIMDIR/"$file"_R1_001.fastq.gz $TRIMDIR/"$file"_R2_001.fastq.gz > $ALIGNDIR/"$file".sam
samtools view -bS $ALIGNDIR/"$file".sam
samtools sort $ALIGNDIR/"$file".bam -@ 48 -o $ALIGNDIR/"$file".bam
rm $ALIGNDIR/"$file".sam

#Get alignment stats
samtools flagstat $ALIGNDIR/"$file".bam
```


References:
Li H. (2013) Aligning sequence reads, clone sequences and assembly contigs with BWA-MEM. arXiv:1303.3997v2 [q-bio.GN]. (if you use the BWA-MEM algorithm or the fastmap command, or want to cite the whole BWA package)
Northing PC, Pelosi JA, Venable DL, Dlugosch KM. Chromosome-scale reference genome of Pectocarya recurvata, the species with the smallest reported genome size in Boraginaceae. Appl Plant Sci. 2025 May 21;13(3):e70008. doi: 10.1002/aps3.70008.
Shifu Chen. fastp 1.0: An ultra-fast all-round tool for FASTQ data quality control and preprocessing. iMeta 4.5 (2025): e70078 https://doi.org/10.1002/imt2.70078

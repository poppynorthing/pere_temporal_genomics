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
--detect_adapter_for_pe --cut_right --trim_front1 10 --trim_front2 10 --dedup -h $FASTPDIR/"$file".html -g -w 16
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
samtools view -bS $ALIGNDIR/"$file".sam > $ALIGNDIR/"$file".bam
samtools sort $ALIGNDIR/"$file".bam -@ 48 -o $ALIGNDIR/"$file".bam
rm $ALIGNDIR/"$file".sam

```
Merge alignment reads from different lanes of sequencing using [samtools](https://academic.oup.com/bioinformatics/article/25/16/2078/204688) v1.19.2 merge (Li et al. 2009).

```
#!/bin/bash

#SBATCH --job-name=merge_bams
#SBATCH --output=merge_bams%j.out
#SBATCH --partition=standard
#SBATCH --account=kdlugosch
#SBATCH --ntasks=1
#SBATCH --nodes=1
#SBATCH --mem-per-cpu=4gb
#SBATCH --cpus-per-task=2
#SBATCH --time=4-12:00:00
#SBATCH --array=1-95%25

sample=$(cat ./file_lists/sample_ids.txt | sed -n ${SLURM_ARRAY_TASK_ID}p)

ml samtools

cd ./sequences/aligned/

# merge bams

samtools merge -f "$sample"_merged.bam "$sample"_007.bam "$sample"_001.bam
```
## Mapping and error rate assessment

Generate mapping rates using [samtools](https://academic.oup.com/bioinformatics/article/25/16/2078/204688) v1.19.2 flagstat (Li et al. 2009). Assess deamination-caused damage in our reads using [mapDamage](https://academic.oup.com/bioinformatics/article/29/13/1682/184965) v2.2.3 (Jónsson et al. 2013).

```
#!/bin/bash

source ~/.bashrc
file=$(cat ./file_lists/sample_ids.txt | sed -n ${SLURM_ARRAY_TASK_ID}p)
ml samtools
conda activate MAPDAMAGE

# Get alignment stats

bam=$ALIGNDIR/"$file".bam
reads=`samtools flagstat $bam -@ 24 | grep total | cut -f 1 -d' '`
mapped=`samtools flagstat $bam -@ 24 | grep mapped | head -n 1 | cut -f 1 -d' '`
echo "${file},${reads},${mapped}" >> mapping_stats.csv

# Assess damage

mapDamage -i $ALIGNDIR/$file.bam -r $REF

```

References:
Li, Heng, Bob Handsaker, Alec Wysoker, Tim Fennell, Jue Ruan, Nils Homer, Gabor Marth, Goncalo Abecasis, Richard Durbin, 1000 Genome Project Data Processing Subgroup, The Sequence Alignment/Map format and SAMtools, Bioinformatics, Volume 25, Issue 16, August 2009, Pages 2078–2079, https://doi.org/10.1093/bioinformatics/btp352

Li H. (2013) Aligning sequence reads, clone sequences and assembly contigs with BWA-MEM. arXiv:1303.3997v2 [q-bio.GN].

Jónsson, Hákon, Aurélien Ginolhac, Mikkel Schubert, Philip L. F. Johnson, Ludovic Orlando, mapDamage2.0: fast approximate Bayesian estimates of ancient DNA damage parameters, Bioinformatics, Volume 29, Issue 13, July 2013, Pages 1682–1684, https://doi.org/10.1093/bioinformatics/btt193

Northing PC, Pelosi JA, Venable DL, Dlugosch KM. Chromosome-scale reference genome of Pectocarya recurvata, the species with the smallest reported genome size in Boraginaceae. Appl Plant Sci. 2025 May 21;13(3):e70008. doi: 10.1002/aps3.70008.

Shifu Chen. fastp 1.0: An ultra-fast all-round tool for FASTQ data quality control and preprocessing. iMeta 4.5 (2025): e70078 https://doi.org/10.1002/imt2.70078



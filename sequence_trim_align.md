# Trimming raw sequences and aligning to reference

## Trimming
Raw PE sequence reads were trimmed using [fastp](https://github.com/opengene/fastp) v0.23.4 (Chen 2025). We removed adapter sequences (--detect_adapter_for_pe), removed low-quality reads (--cut_right), and removed PCR duplicate reads (--dedup).

```
!/bin/bash

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
Trimmed reads were aligned to the chromosome contigs from the [<i>Pectocarya recurvata</i> reference genome](https://bsapubs.onlinelibrary.wiley.com/doi/10.1002/aps3.70008) (Northing et al. 2025) using [BWA](https://github.com/lh3/bwa) mem v0.7.18 (Li 2013) with default settings.

```
#!/bin/bash

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
Before merging, add read group information to each bam using picard v2.23.4 (REF). This must be repeated for each lane of sequencing (this shows code for lane 1; I also did this for lane 7). This step is important for marking PCR and optical duplicates before calling variants.
```
#!/bin/bash

sample=$(cat ./file_lists/sample_ids.txt | sed -n ${SLURM_ARRAY_TASK_ID}p)

cd ./sequences/aligned

ml picard
ml samtools

BAMDIR=sequences/aligned
REF=/xdisk/kdlugosch/pcnorthing/Genome/pere_ch.fa

read_name=$(samtools view "$sample"_L001.bam | head -1 | cut -f1)
flowcell=$(echo "$read_name" | cut -d: -f3)
lane=$(echo "$read_name" | cut -d: -f4)

echo $read_name
echo $flowcell
echo $lane

# Add read group information to pre-merged bams

picard AddOrReplaceReadGroups \
    I="$sample"_L001.bam \
    O="$sample"_L001.rg.bam \
    SORT_ORDER=coordinate \
    RGID="${flowcell}.${lane}" \
    RGLB=GE-8688 \
    RGPL=illumina \
    RGPU="${flowcell}.${lane}" \
    RGSM="$sample"
```

Then, merge alignment reads from different lanes of sequencing using [samtools](https://academic.oup.com/bioinformatics/article/25/16/2078/204688) v1.19.2 merge (Li et al. 2009).

```
#!/bin/bash

sample=$(cat ./file_lists/sample_ids.txt | sed -n ${SLURM_ARRAY_TASK_ID}p)

ml samtools

cd ./sequences/aligned/

# merge bams

samtools merge -f "$sample"_merged.bam "$sample"_007.rg.bam "$sample"_001.rg.bam
```
## Mapping and error rate assessment

Generate mapping rates using [samtools](https://academic.oup.com/bioinformatics/article/25/16/2078/204688) v1.19.2 flagstat (Li et al. 2009). Assess deamination-caused damage in our reads using [mapDamage](https://academic.oup.com/bioinformatics/article/29/13/1682/184965) v2.2.3 (Jónsson et al. 2013). Note: in order to get mapDamage to run properly, I had to manually install an R package that wouldn't properly compile; to do this, run <b>conda install bioconda::r-rcppgsl</b> to install the package in your virtual environment w/ mapdamage.

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

## References:

Li, Heng, Bob Handsaker, Alec Wysoker, Tim Fennell, Jue Ruan, Nils Homer, Gabor Marth, Goncalo Abecasis, Richard Durbin, 1000 Genome Project Data Processing Subgroup, The Sequence Alignment/Map format and SAMtools, Bioinformatics, Volume 25, Issue 16, August 2009, Pages 2078–2079, https://doi.org/10.1093/bioinformatics/btp352

Li H. (2013) Aligning sequence reads, clone sequences and assembly contigs with BWA-MEM. arXiv:1303.3997v2 [q-bio.GN].

Jónsson, Hákon, Aurélien Ginolhac, Mikkel Schubert, Philip L. F. Johnson, Ludovic Orlando, mapDamage2.0: fast approximate Bayesian estimates of ancient DNA damage parameters, Bioinformatics, Volume 29, Issue 13, July 2013, Pages 1682–1684, https://doi.org/10.1093/bioinformatics/btt193

Northing PC, Pelosi JA, Venable DL, Dlugosch KM. Chromosome-scale reference genome of Pectocarya recurvata, the species with the smallest reported genome size in Boraginaceae. Appl Plant Sci. 2025 May 21;13(3):e70008. doi: 10.1002/aps3.70008.

Shifu Chen. fastp 1.0: An ultra-fast all-round tool for FASTQ data quality control and preprocessing. iMeta 4.5 (2025): e70078 https://doi.org/10.1002/imt2.70078



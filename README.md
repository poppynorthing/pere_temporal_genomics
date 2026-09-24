# Investigating temporal genomics in <i>Pectocarya recurvata</i>.
Authors: Poppy C. Northing, D. Larry Venable, Katrina M. Dlugosch

### This repo contains several markdown files with my annotated bioinformatic pipeline:

sequence_trim_align.md -- code for trimming reads through checking read mapping statistics. \
variant_calling.md -- code for variant calling through joint genotyping. \
variant_filtering.md -- code for variant filtering and statistics, including calculating LD and pruning linked SNPs.
ld_decay_calc_pcn_modified.py -- python script for calculating average LD intervals in the genome. Modified from https://speciationgenomics.github.io/ld_decay/ to run in python3

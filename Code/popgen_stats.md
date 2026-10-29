## Population genetics statistics


### Individual heterozygosity
```
# Load data
diploid.vcf <- read.vcfR("Data/final_joint_genotyped.recalc.maf.m30.filtd20.snps.2.vcf.gz")
popmap <- read.csv("Data/popmap.csv")

# Convert to genlight
diploid.genlight <- vcfR2genlight(diploid.vcf) # convert to genlight

# Calculate individual heterozygosity using gl.report.heterozygosity
diploid.genlight@other$loc.metrics <- data.frame(row.names = 1:nLoc(diploid.genlight))
diploid.genlight <- gl.filter.monomorphs(diploid.genlight)
het.diploid <- gl.report.heterozygosity(diploid.genlight, method = 'ind') %>% rename(Ind = ind.name)

het.diploid <- het.diploid %>% left_join(popmap, by = "Ind") # add metadata to heterozygosity data

het.diploid.sum <- het.diploid %>% group_by(temporal_bin) %>% summarize(n = n(),
                                                                        mean_Ho = mean(Ho)) # look at mean heterozygosity per bin

ggplot(het.diploid, aes(factor(temporal_bin), Ho)) +
  geom_boxplot(outlier.shape = NA) + geom_jitter(width = 0.15)

```

### Runs of Homozygosity
FROH = (total length of all ROH segments in the individual) / (total length of the genome assayed)

Use plink v1.9 to find ROHs:
```
plink --vcf vcf/final_vcfs/final_joint_genotyped.recalc.maf.m30.filtd20.snps.2.vcf.gz \
  --double-id --allow-extra-chr --set-missing-var-ids @:# \
  --homozyg --homozyg-snp 50 --homozyg-kb 500 --homozyg-density 50 \
  --homozyg-gap 1000 --homozyg-window-snp 50 --homozyg-window-het 1 \
  --homozyg-window-missing 5 --out $PWD/roh.2
```



```
```

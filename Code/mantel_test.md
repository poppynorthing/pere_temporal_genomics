## Testing for Genetic Isolation-by-Distance


Setup
```
# Load libraries
library(tidyverse)
library(vegan)
library(vcfR)
library(adegenet)
library(poppr)

# Load data
popmap <- read.csv("Data/popmap.csv") # sample metadata
diploid.vcf <- read.vcfR("Data/joint_genotyped.recalc.maf.m30.filtd20.snps.2.vcf.gz")

# Create genind object from vcf
diploid.genind <- vcfR2genind(diploid.vcf, ploidy = 2)

# Samples with complete coordinates (89/95 samples)
coords_ok <- popmap[complete.cases(popmap[, c("latitude", "longitudue")]), ]

# Only keep samples present in both the genind and the coordinates
keep <- intersect(indNames(diploid.genind), coords_ok$Ind)

# Subset the genind to only samples with coord info (in the same order)
gen_sub    <- diploid.genind[keep, ]
coords_sub <- coords_ok[match(keep, coords_ok$Ind), ]

stopifnot(identical(indNames(gen_sub), coords_sub$Ind))
```

Then, create genetic and geographic distance matrices.

```
# Calculate distance matrices
diploid.dist <- prevosti.dist(gen_sub) # genetic distance matrix
geo.dist     <- dist(coords_sub[, c("latitude", "longitudue")], method = "euclidean") # geographic distance matrix
```

Then, run mantel test

```
# Calculate isolation by distance

mantel.diploid <- mantel(diploid.dist, geo.dist, method = "pearson", permutations = 9999)
mantel.diploid

# Plot
iso.by.dist <- data.frame(gen = as.vector(diploid.dist), geo = as.vector(geo.dist))

mantel_label <- sprintf("r = %.3f\np = %s\nPermutations = %d",
                        mantel.diploid$statistic,
                        format.pval(mantel.diploid$signif, digits = 3, eps = 0.001),
                        mantel.diploid$permutations) # create label w/ test statistics for plot

ggplot(iso.by.dist, aes(geo, gen)) +
  geom_point(alpha = 0.3) +
  geom_smooth(method = "lm") +
  annotate("text",
           x = 0.8, y = 0.1,
           label = mantel_label,
           hjust = -0.1, vjust = 1.2,
           size = 4) +
  labs(x = "Geographic distance", y = "Genetic distance") +
  theme_bw()
```

### References

# Genetic Clustering PCA
Data wrangling and analyses done in R v4.4.1 using the following libraries: adegenet vX, poppr vX, tivyverse vX, vcfR vX, and readr vX. 

First, need to load libraries for running PCA and visualizing, load filtered genetic variants data (vcf), and convert files to appropriate format for R.

```
# Load Libraries
library(tidyverse)
library(vcfR)
library(adegenet)
library(readr)
library(poppr)
library(ggrepel)
library(patchwork)

# Load data
diploid.vcf <- read.vcfR("Data/joint_genotyped.recalc.maf.m30.filtd20.snps.2.vcf.gz")
#tetraploid.vcf <- read.vcfR()
popmap <- read.csv("Data/popmap.csv")

# Create genind objets
diploid.genind <- vcfR2genind(diploid.vcf, ploidy = 2)
#tetraploid.genind <- vcfR2genind(tetraploid.vcf, ploidy = 4)
pop(diploid.genind) <- popmap$status # add info about whether samples are historical or modern
X <- tab(diploid.genind, NA.method = "mean")
```

Then, run PCA*:

```
pca.diploid <- dudi.pca(X, scale = TRUE, scannf = FALSE, nf = 50)

pve.diploid <- (pca.diploid$eig / sum(pca.diploid$eig))*100
pve.diploid <- round(pve.diploid, digits = 2)

pca.diploid.df <- pca.diploid$li
pca.diploid.df$Ind <- rownames(pca.diploid.df)
pca.diploid.df.popmap <- inner_join(pca.diploid.df, popmap)
```

Finally, visualize PCA:

```
# Plot first and second PC
p1.pca.diploid <- ggplot() +
  geom_point(pca.diploid.df.popmap, mapping = aes(x = Axis1, y = Axis2, color = as.numeric(collection_year)), size = 3, alpha = 0.85) +
  scale_color_viridis_c(limits = range(pca.diploid.df.popmap$collection_year)) +
  #geom_text_repel(pca.diploid.df.popmap, mapping = aes(x = Axis1, y = Axis2, label = Ind), size = 3) +
  xlab(paste0("PC 1 (", pve.diploid[1], "% variation explained)")) +
  ylab(paste0("PC 2 (", pve.diploid[2], "% variation explained)")) +
  theme_bw() +
  theme(text = element_text(size = 16)) +
  labs(color = "Collection Year"); p1.pca.diploid

# Plot second and third PC
p2.pca.diploid <- ggplot() +
  geom_point(pca.diploid.df.popmap, mapping = aes(x = Axis2, y = Axis3, color = as.numeric(collection_year)), size = 3, alpha = 0.85) +
  scale_color_viridis_c(limits = range(pca.diploid.df.popmap$collection_year)) +
  geom_text_repel(pca.diploid.df.popmap, mapping = aes(x = Axis2, y = Axis3, label = Ind), size = 3) +
  xlab(paste0("PC 2 (", pve.diploid[2], "% variation explained)")) +
  ylab(paste0("PC 3 (", pve.diploid[3], "% variation explained)")) +
  theme_bw() +
  theme(text = element_text(size = 16)) +
  labs(color = "Collection Year"); p2.pca.diploid

# Plot third and fourth PC
p3.pca.diploid <- ggplot() +
  geom_point(pca.diploid.df.popmap, mapping = aes(x = Axis3, y = Axis4, color = as.numeric(collection_year)), size = 3, alpha = 0.85) +
  scale_color_viridis_c(limits = range(pca.diploid.df.popmap$collection_year)) +
  geom_text_repel(pca.diploid.df.popmap, mapping = aes(x = Axis3, y = Axis4, label = Ind), size = 3) +
  xlab(paste0("PC 3 (", pve.diploid[3], "% variation explained)")) +
  ylab(paste0("PC 4 (", pve.diploid[4], "% variation explained)")) +
  theme_bw() +
  theme(text = element_text(size = 16)) +
  labs(color = "Collection Year"); p3.pca.diploid

# Plot them all together with the same legend
p1.pca.diploid + p2.pca.diploid + p3.pca.diploid + plot_layout(guides = "collect")
```
\* Note: I also ran a pca with the same data in plink vv2.0.0 (ref) as a sanity check; the results were the same.
```
#!bin/bash

ml plink
plink --vcf joint_genotyped.recalc.maf.m30.filtd20.snps.2.vcf.gz --pca
```
### References

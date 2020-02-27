--- 
title: "Cassava GS SOPs"
author: "Marnin Wolfe"
date: "2020-02-27"
site: bookdown::bookdown_site
documentclass: book
bibliography: [book.bib, packages.bib]
biblio-style: apalike
link-citations: yes
description: "Documentation for genomic selection-related standard operating procedures used for NextGen Cassava Breeding."
---

# Overview {-}

## Workflows / Use Cases

The purpose of this document is as a manual, a teaching tool and a codebase to support genomic selection-related standard operating procedures (SOPs). This should serve to support the [NextGen Cassava Breeding Project](http://www.nextgencassava.org) and its community of practice partnerships. 

Initially, I will attempt to outline the core steps, providing standardized code where possible, and clearly pointing out the areas where breeding programs / analysts will need to provide input.

The actual GS analyses that I conducted in 2019 are documented on GitHub for [IITA](https://wolfemd.github.io/IITA_2019GS/), [NRCRI](https://wolfemd.github.io/NRCRI_2019GS/) and [NaCRRI](https://wolfemd.github.io/NaCRRI_2019GS/). Some of the datasets involved are too big for GitHub; the most important ones can be found in [this](https://drive.google.com/drive/folders/1sjFFS4sRAwxHALEcT8RTSuUdHJsIuGNK?usp=sharing) Google Drive folder.

**Use Case 1: ** Selection of parents for crossing.  
Analysis: Using GEBV (standard GBLUP model)

1. Step \@ref(prepTP): Prepare a training dataset
  * Download data from DB, "Clean" and format DB data
2. Step \@ref(curateTrials): Curate trials
  * remove outliers, select best model for each trial
3. Step \@ref(getBLUPs): Getting BLUPs from multiple trials
4. Step \@ref(getKinship): Review and QC of SNP marker data
  * construction kinship matrix, PCA to check pop. structure
5. Step \@ref(crossValidation): Evaluate prediction accuracy with cross-validation
6. Step \@ref(getGEBVs): Genomic prediction, get GEBVs for all selection candidates
7. Step \@ref(getGainEst): Genomic selection
  * Estimate genetic gain, Formulate selection index, upload GEBV to DB
  
**Use Case 2: ** Select clones for further phenotyping  
**Use Case 2a: ** Advancement of clones for variety development (e.g. harvest current PYTs and advance to AYTs)  
Analysis: Using GETGV (non-additive genomic prediction models)  
**Use Case 2b: ** Maintenance of diversity and prediction accuracy  
Analysis: Using STPGA (R package for selecting “optimal” subsets for phenotyping)

## Example data {#example_data}

Want to develop some standard functions using a small (50Mb or less dataset). Will start with some previously processed data and use it to select a subset of raw trials to use as examples. 


```r
library(tidyverse); library(magrittr)
cleanphenos <- readRDS(url("https://raw.github.com/wolfemd/IITA_2019GS/master/data/IITA_ExptDesignsDetected_72619.rds"))
cleanphenos2keep<-cleanphenos %>% 
      filter(grepl(".GS.|gain",studyName,ignore.case = T),
             studyYear %in% c("2016","2017"),
             !grepl("ungenotyped",studyName))
trialsForExample<-unique(cleanphenos2keep$studyName)
```

I've chosen 2 years of IITA data, just the GS-related trials.  

Should encompass RCBD and Augmented designs and the range of plot sizes. 

Next part will call on data located [here](https://drive.google.com/drive/folders/1sjFFS4sRAwxHALEcT8RTSuUdHJsIuGNK?usp=sharing), subset the raw data and save as example data.


```r
rawphenos<-read.csv("~/Google Drive/NextGenGS/NGC_BigData/DatabaseDownload_72419/2019-07-24T144915phenotype_download.csv",
         na.strings = c("#VALUE!",NA,".",""," ","-","\""),
         stringsAsFactors = F)
examplerawphenos<-rawphenos %>% 
      filter(studyName %in% trialsForExample)
write.csv(examplerawphenos,file="data/example_phenotype_download.csv",row.names = F)

read.csv("~/Google Drive/NextGenGS/NGC_BigData/DatabaseDownload_72419/2019-07-24T144144metadata_download.csv",
         na.strings = c("#VALUE!",NA,".",""," ","-","\""),
         stringsAsFactors = F) %>% 
      filter(studyName %in% trialsForExample) %>% 
      write.csv(.,file="data/example_metadata_download.csv",row.names = F)
```

Let's also make an example SNP dataset. Will sample a subset of SNPs and remove unecessary samples to make an example dosage matrix.

Sample 2000 SNPs. Create an easy to use data.frame **geno2pheno** with 2 columns:  

* **germplasmName** matches the cassavabase phenotype data download.
* **DNAname:** matches the `rownames` of the example snps dosage matrix **snps**.


```r
snps<-readRDS("~/Google Drive/NextGenGS/NGC_BigData/DosageMatrix_RefPanelAndGSprogeny_ReadyForGP_73019.rds")
dim(snps) # [1] 21856 68814

cleanphenos2keep<-cleanphenos2keep %>% 
      select(studyName,TrialData) %>% 
      unnest(cols = c(TrialData))

geno2pheno<-cleanphenos2keep %>% 
      select(germplasmName,FullSampleName) %>% 
      distinct %>% 
      filter(!is.na(FullSampleName),
             FullSampleName %in% rownames(snps)) %>% 
      rename(DNAname=FullSampleName)

set.seed(42)
snps<-snps[rownames(snps) %in% geno2pheno$DNAname,sample(1:ncol(snps),size = 2000, replace = F)]

saveRDS(snps,file="data/example_snps.rds")
saveRDS(geno2pheno,file="data/example_geno2pheno.rds")
```

## Appendices

1. Access to and use of computing resources
2. Learning resources
  - https://youtu.be/Qne86lxjgtg
  - https://youtu.be/bzUmK0Y07ck
  - https://youtu.be/D48JHU4llkk
  - [R for Data Science](https://r4ds.had.co.nz/)
3. Tidy data
  - (1) each variable in a column, (2) each observation in a row, (3) each cell is one value
4. Misc
  - https://malco.io/2018/11/05/why-should-i-use-the-here-package-when-i-m-already-using-projects/
  - [Wilson et al. 2017. _PloS Computation Biology_.] Good enough practices in scientific computing. (https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1005510)




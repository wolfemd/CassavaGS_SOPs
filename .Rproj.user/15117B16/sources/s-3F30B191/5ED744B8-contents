# Get BLUPs {#getBLUPs}

## Explanation of two-stage prediction

For now best to see, **EiB GS workshop from Sept. 2018** [here](https://drive.google.com/open?id=1ND7vCAEoKuv5wtQe_nPZhFSx3JJOe3tr), especially the **GSprocess_SupportingMaterial.html**.

**Two-stage** genomic prediction refers to the following procedure:

**Stage 1:** Fit a linear mixed model to the data *without* genomic data. Individuals (e.g. clones / accessions) are modeled as independent and identically distributed (*i.i.d.*) random effects. The BLUPs for this random effect represent the measurable total genetic values of each individual. All the experimental design variation, e.g. replication and blocking effects have been controlled for in the creation of our new response variable, the BLUPs from the gneotype random effect.

**Stage 2:** Using a modified version of the BLUPs from step 1 as the response variable, fit a genomic prediction model, which now has reduced size because the number of observations is now the same as the number of individuals.

**NOTE:** In the animal breeding literature **single-step** often refers to predictions that combine pedigree and marker information simultaneously. That *is not* our meaning here.

## Set-up training data chunks

Using the curated trial data (BLUPs?) from the [previous step](#curateTrials) as input.

## Fit lmer models

## Extract and de-regress BLUPs



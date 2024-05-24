# ALBACOTO_GWAS.github.io

First enter to your GenomeDK, find the PopGen folder and start an interactive job:
```sh
srun --mem-per-cpu=1g --time=3:00:00 --account=populationgenomics --pty bash
```
also activate the conda environment which has PLINK downloaded. Note to perform all the code in the same folder in which you have the .fam, .bim, .bed files


**GWAS QC USING PLINK**


**Identification of individuals with elevated missing data rates or outlying heterozygosity rate**
```sh
plink --bfile gwas_data --missing --out GWA-QC 
```
This command will create the files .imiss, .lmiss, .hh, .log and .nosex. The phenotype of individuals in the data is without gender information. So that is why we use “--allow-no-sex” option (which gives the .nosex file).

The fourth column in the file GWA-data.imiss (N_MISS) denotes the number of missing SNPs and the sixth column (F_MISS) denotes the proportion of missing SNPs per individual.
```sh
plink --bfile gwas_data --het --out GWA-QC-het 
```
This command will create the file GWA-data.het, in which the third column denotes the observed number of homozygous genotypes [O(Hom)] and the fifth column denotes the number of non-missing genotypes [N(NM)] per individual.


We than open R to create a plot in which the observed heterozygosity rate per individual is plotted on the x axis and the proportion of missing SNPs per individuals is plotted on the y axis.
To be able to charge the file .imiss in R we need to have it in our own computer, we obtain it from GenomeDK like this:
```sh
scp @login.genome.au.dk:/path/to/file .
```

Inside R we should also filter out outliers for the variables, and then save the file so we later remove it using PLINK. We have made a file with the FID and IID of all individuals that have a genotype missing rate >=0.03 or a heterozygosity rate that is more than 3 s.d. from the mean (this is calculated in R). To remove this file using PLINK we should type:
```sh
plink --bfile gwas_data --remove wrong_het_missing_values.txt --make-bed --out GWA-QC-nohet
```

We will create new bed, bim, fam files if we name the outpit differently or we will overwrite the ones we already had if we don't change the output name.


**Identification of duplicated or related individuals**
We have to calculate the identity by descent (IBD). 

We will “prune” the data and create a list of SNPs that are non-correlated. This can be done by the following command:
```sh
plink --bfile GWA-QC-nohet --allow-no-sex --indep-pairwise 500kb 5 0.2 --out GWA-QC
```
It saves the list of independent SNPs as .prune.in.  

To calculate IBD between each pair of individuals:
```sh
plink --bfile GWA-QC-nohet --allow-no-sex --extract GWA-QC.prune.in --genome --min 0.185 --out GWA-QC-ibd
```
The --min 0.185 option means that it will only print the calculated IBD if it is above 0.185 (Mean between second-degree relatives:0.25 and third-degree relatives:0.125). This produces a file with the extions .genome 


With Rstudio we should remove a member from each of the pairs that are too closely related from the data set. We will remove the individual mentioned first. 

To remove these individuals that we have obtained in R we will use again the --remove parameter and create updated bed/bim/fam files with: 
```sh
plink --bfile  GWA-QC-nohet --allow-no-sex --remove wrong_ibd.txt --make-bed --out GWA-QC-unique 
```
(The updated files are going to be -unique).
























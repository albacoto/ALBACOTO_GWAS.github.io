# ALBACOTO_GWAS.github.io

First enter to your GenomeDK, find the PopGen folder and start an interactive job:
```sh
srun --mem-per-cpu=1g --time=3:00:00 --account=populationgenomics --pty bash
```
also activate the conda environment which has PLINK downloaded. Note to perform all the code in the same folder in which you have the .fam, .bim, .bed files


**GWAS QC USING PLINK**


Identification of individuals with elevated missing data rates or outlying heterozygosity rate
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



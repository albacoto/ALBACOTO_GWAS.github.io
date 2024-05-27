# ALBACOTO_GWAS.github.io

First enter to your GenomeDK, find the PopGen folder and start an interactive job:
```sh
srun --mem-per-cpu=1g --time=3:00:00 --account=populationgenomics --pty bash
```
also activate the conda environment which has PLINK downloaded. Note to perform all the code in the same folder in which you have the .fam, .bim, .bed files


**GWAS QC USING PLINK**


**Identification of individuals with elevated missing data rates or outlying heterozygosity rate**
```sh
plink --bfile gwas_data --allow-no-sex --missing --out GWAS-QC1 
```
This command will create the files .imiss, .lmiss, .hh, .log and .nosex. The phenotype of individuals in the data is without gender information. So that is why we use “--allow-no-sex” option (which gives the .nosex file).

The fourth column in the file GWA-data.imiss (N_MISS) denotes the number of missing SNPs and the sixth column (F_MISS) denotes the proportion of missing SNPs per individual.
```sh
plink --bfile gwas_data --allow-no-sex --het --out GWAS-QC1-het 
```
This command will create the file GWA-data.het, in which the third column denotes the observed number of homozygous genotypes [O(Hom)] and the fifth column denotes the number of non-missing genotypes [N(NM)] per individual.


We then open R to create a plot in which the observed heterozygosity rate per individual is plotted on the x axis and the proportion of missing SNPs per individuals is plotted on the y axis. To calculate the observed heterozygosity rate per individual we should do it using the formula: Het = (N(NM) − O(Hom))/N(NM). 

Note: To be able to charge the file .imiss and .het in R we need to have it in our own computer, we obtain it from the cluster like this:
```sh
scp @login.genome.au.dk:/path/to/file .
```

Once in R we join the 2 files that we just talked about so we are able to create a plot where the observed heterozygosity rate per individual is on the x axis and the proportion of missing SNPs per individuals is plotted on the y axis. We therefore interpret the plot and results we obtain.


Inside R we should also filter out outliers for the variables, and then save the file so we later remove it in the file (using PLINK) and not only in R. 
We filter the outliers that have a genotype missing rate >=0.03 and a heterozygosity rate that is more than 3 s.d. from the mean. 

Note that we have to upload the txt file that we will obtain from R in the cluster using the following command:
```sh
scp /path/to/file @login.genome.au.dk:/path/to/directory/
```

To remove this file using PLINK we should type:
```sh
plink --bfile gwas_data --allow-no-sex --remove wrong_het_missing.txt --make-bed --out GWAS-QC2
```
Now we have created new bed, bim, fam files if we name the output differently or we will overwrite the ones we already had if we don't change the output name. This new files don't contain the outliers we just calculated above.


**Identification of duplicated or related individuals**

We have to calculate the identity by descent (IBD). 

1) We will “prune” the data and create a list of SNPs that are non-correlated. This is performed with the command:
```sh
plink --bfile GWAS-QC2 --allow-no-sex --indep-pairwise 500kb 5 0.2 --out GWAS-QC2
```
We obtain .prune.in and prune.out files. In the .prune.it we have the list of independent SNPs. 

2) To calculate IBD between each pair of individuals:
```sh
plink --bfile GWAS-QC2 --allow-no-sex --extract GWAS-QC2.prune.in --genome --min 0.185 --out GWAS-QC2
```
The --min 0.185 option means that it will only print the calculated IBD if it is above 0.185 (Mean between second-degree relatives:0.25 and third-degree relatives:0.125). This produces a file with the extention .genome.


With Rstudio we should remove a member from each of the pairs that are too closely related from the data set. We will remove the individual mentioned first. 

To remove these individuals that we have obtained in R we will use again the --remove parameter and create updated or new bed/bim/fam files with the same command as before: 
```sh
plink --bfile  GWAS-QC2 --remove ibd_filtered.txt --make-bed --out GWA-QC3 
```



**SNP QC**

SNPs with an excessive missing data rate

We need to run the --missing command again to generate the .lmiss with the missing data rate for each SNP: 
```sh
plink --bfile GWAS-QC3 --allow-no-sex --missing --out GWAS-QC3
```

The --test-missing command tests for association between missingness and case/control status, using Fisher's exact test. It produces a file with ".missing" suffix.
Run the test-missing command: plink --bfile GWA-QC-unique --allow-no-sex --test-missing --out GWA-QC-unique-missing
Warning: Skipping --test-missing since at least one case and one control is required. At this point we need to add the phenotypes variable in eye_color.txt distinguishing the phenotypes related with the eye color. 


**PCA**

We will use the pruned set of SNPs to calculate the relationship matrix and calculate the first 20 principle components (PCs): 
```sh
plink --bfile GWAS-QC2 --extract GWAS-QC2.prune.in --pca 20 --out GWAS-QC2
```
This calculates the eigenvalues and the eigenvectors.

We should load the eigenvectors file to R and make a plot with the 2 firsts PCs. 

Note: name the columns properly (22 columns) remember we performed the plink so that we would obtain a PCA of 20 (the first 2 columns are going to be the identifiers). 

We can use the eigenvalues to compute the variance explained by each PC and interpret the results.

From the plot with the PCs we must look if we have any outlier and if so get rid of it:
```sh
plink --bfile GWAS-QC2 --remove pca_outlier.txt --make-bed --out GWAS-QC4
```

ASSOCIATION TESTING:

Now, we should take into account the phenotypes of our eye color file. The pheno column in our .fam file contains the values of -9 for all individuals. We must change that for a binary phenotype of the eye color that we like (we can divide it in blue and brown eye color with 1 & 2 values). 

Note: add name columns for the other columns that are not phenotype.

Afterwards we must join the PCA dataset that we created in the step before (with the 20 PCs) with the dataset that contains the phenotypes. And if there are NA values we should remove them and update it in the files from the cluster using plink (--make-bed).

Following, from these new fam, bed, bim files we just obtained should do again all the processing of the data until we obtain the .eigenvec and .eigenval files to perform a new PCA.
And also we will do again the QC with the SNPs to obtain the distribution of missing data rates. 


To add the phenotypes we obtained from R we would have to perform the following command with a txt previously obtained from R (where we created the binary phenotype) to join the phenotypes in the .fam, .bim, .bed files:
```sh
plink --bfile GWAS-QC5 --pheno eye_color_id.txt --make-bed --out GWAS-QC6
```
With this we will obtain the .fam file with the phenotypes 1 and 2 for brown and blue and not with -9. Now that we have the phenotypes we should proceed with the last steps of QC. 


We shoud now run a new command:
```sh
plink --bfile GWAS-QC6 --test-missing --out GWAS-QC6
```
This will test for association between missingness and case/control status (which are the 2 phenotypes that we have), using Fisher's exact test. It produces a file ".missing".

Then, we make a list in R where the p-value < 1e-5. And we save the list as a txt. In this file we will have the low-quality SNPs, so we write the following command to remove them:
```sh
plink --bfile GWAS-QC6 --exclude fail-diffmiss-qc.txt --geno 0.05 --hwe 0.00001 --maf 0.01 --make-bed --out GWAS-QC7
```
In addition to removing SNPs identified with differential call rates between cases and controls, this command removes SNPs with call rate less than 95% with --geno option and deviation from HWE (p<1e-5) with the --hwe option. It also removes all SNPs with minor allele frequency less than a specified threshold using the --maf option.



















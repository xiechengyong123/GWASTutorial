# Association test

## Overview

<img width="900" alt="image" src="https://github.com/Cloufield/GWASTutorial/assets/40289485/1e4ab229-6f5a-44a0-9850-81244944b969">

## Genetic models

To test the association between a phenotype and genotypes, we need to group the genotypes based on genetic models.

There are three basic genetic models:

- Additive model (ADD)
- Dominant model (DOM)
- Recessive model (REC)

!!! info "Three genetic models"
    For example, suppose we have a biallelic SNP whose reference allele is A and alternative allele is G.
    
    There are three possible genotypes for this SNP: AA, AG, and GG.
    
    This table shows how we group different genotypes under each genetic model for association tests using linear or logistic regressions.
    
    |Genetic models|AA|AG|GG|
    |-|-|-|-|
    |Additive model|0|1|2|
    |Dominant model|0|1|1|
    |Recessive model|0|0|1|

!!! info "Contingency table and non-parametric tests" 
    A simple way to test association is to use the 2x2 or 2x3 contingency table. For dominant and recessive models,  Chi-square tests are performed using the 2x2 table. For the additive model, Cochran-Armitage trend tests are performed for the 2x3 table. However, the non-parametric tests do not adjust for the bias caused by other covariates like sex, age and so forth.
    
    <img width="900" alt="image" src="https://github.com/Cloufield/GWASTutorial/assets/40289485/f2b0f941-9405-4ebe-8e89-aef608eb9bb1">

## Association testing basics

For quantitative traits, we can employ a simple linear regression model to test associations:

$$ 
y = G\beta_G + X\beta_X + e
$$

- $G$ is the genotype matrix.
- $\beta_G$ is the effect size for variants.
- $X$ and $\beta_X$ are covariates and their effects.
- $e$ is the error term.

!!! info "Interpretation of linear regression"
    <img width="900" alt="image" src="https://github.com/Cloufield/GWASTutorial/assets/40289485/d5092f82-c5b5-4f08-873e-c2942955e634">


For binary traits, we can utilize the logistic regression model to test associations:

$$ 
logit(p) = G\beta_G + X\beta_X + e
$$

!!! info "Linear regression and logistic regression"
    <img width="900" alt="image" src="https://github.com/Cloufield/GWASTutorial/assets/40289485/1b902ad6-db06-44f7-9215-43b13a1a4fea">

## File Preparation

To perform genome-wide association tests, usually we need the following files:

- **Genotype file** (or dosage file) : usually in PLINK format, VCF format, or BGEN format.
- **Phenotype file** : plain text file.
- **Covariate file** (optional): plain text file. Usually covariates include age, sex, and top Principal Components. 

!!! example "Phenotype and covariate files"

    Phenotype file for a simulated binary trait; B1 is the phenotype name; 1 means the control, 2 means the case.
    
    ```txt title="1kgeas_binary.txt"
    FID IID B1
    0 HG00403 2 
    0 HG00404 1 
    0 HG00406 2 
    0 HG00407 2 
    0 HG00409 2 
    0 HG00410 2
    0 HG00419 1 
    0 HG00421 2 
    0 HG00422 1
    
    Covariate file (only top PCs calculated in the previous PCA section)
    
    ```txt title="plink_results_projected.sscore"
    #FID	IID	ALLELE_CT	NAMED_ALLELE_DOSAGE_SUM	PC1_AVG	PC2_AVG	PC3_AVG	PC4_AVG	PC5_AVG	PC6_AVG	    PC7_AVG	PC8_AVG	PC9_AVG	PC10_AVG
    0	HG00403	224016	224016	0.000246109	0.0292717	-0.0127437	-0.0135105	0.0301317	0.    0196699	0.0232392	-0.0205941	-0.00416543	0.0121819
    0	HG00404	224016	224016	-0.000716664	0.032043	-0.00573731	-0.0189504	0.0277385	-0.    0154478	-0.0136551	-0.00147269	0.00510851	0.0268694
    0	HG00406	224016	224016	0.00681476	0.0346759	-0.00706221	-0.0054266	-0.025458	-0.    0166505	-0.00510707	-0.00105581	-0.0224028	0.0202523
    0	HG00407	224016	224016	0.00695106	0.0244759	-0.00890072	-0.000694057	0.00946838	-0.    00773378	-0.0139923	-0.0204871	-0.0003338	0.0387022
    0	HG00409	224016	224016	-0.0023481	0.0213108	0.03235	0.0158793	0.026655	0.    00324455-0.0107152	0.0317714	-0.00764455	0.0155973
    0	HG00410	224016	224016	0.000926147	0.0198139	0.0475798	0.00314974	0.0275373	0.    041886	-0.0133896	0.0184717	-0.0143644	0.0291036
    0	HG00419	224016	224016	0.00580767	0.0369208	-0.00907507	0.00903163	-0.00649345	-0.    000472359	-0.00327011	0.0160456	-0.005133	0.0141021
    0	HG00421	224016	224016	0.000987901	0.0336895	-0.00697814	0.00557199	-0.0210537	0.    00700968	0.00319921	-0.0215999	0.00127686	0.0350116
    0	HG00422	224016	224016	0.00440288	0.0335901	-0.0125043	0.0135621	-0.0228428	0.    00492741	0.00445856	-0.00911147	-0.00312742	-0.00784459
    ```

## Association tests using PLINK

Please check https://www.cog-genomics.org/plink/2.0/assoc for more details.

We will perform logistic regression with firth correction for a simulated binary trait under the additive model using the 1KG East Asian individuals.

!!! note "Firth correction"
    Adding a penalty term to the log-likelihood function when fitting the logistic model, which results in less bias. - Firth, David. "Bias reduction of maximum likelihood estimates." Biometrika 80.1 (1993): 27-38.

!!! note "Quantitative traits"
    For quantitative traits, linear regressions will be performed and in this case, we do not need to add `firth` (since Firth correction is not appliable). 

!!! example "Sample codes for association test using plink for binary traits"
    ```
    genotypeFile="../01_Dataset/1KG.EAS.auto.snp.norm.nodup.split.maf005.thinp020"  #!please set this to your own path
    phenotypeFile="../01_Dataset/1kgeas_binary.txt" #!please set this to your own path
    covariateFile="../05_PCA/plink_results_projected.sscore"
    
    covariateCols=6-10
    colName="B1"
    threadnum=2
    
    plink2 \
    	--bfile ${genotypeFile} \
    	--pheno ${phenotypeFile} \
    	--pheno-name ${colName} \
    	--maf 0.01 \
    	--covar ${covariateFile} \
    	--covar-col-nums ${covariateCols} \
    	--glm hide-covar firth  firth-residualize single-prec-cc \
    	--threads ${threadnum} \
    	--out 1kgeas
    ```
!!! note
    Using the latest version of PLINK2, you need to add `firth-residualize single-prec-cc` to generate the results. (The algorithm and precision have been changed since 2023 for firth regression)

You will see a similar log like:

!!! example "Log"
    ```txt title="1kgeas.log"
    PLINK v2.00a3LM 64-bit Intel (12 Dec 2020)     www.cog-genomics.org/plink/2.0/
    (C) 2005-2020 Shaun Purcell, Christopher Chang   GNU General Public License v3
    Logging to 1kgeas.log.
    Options in effect:
      --bfile ../01_Dataset/1KG.EAS.auto.snp.norm.nodup.split.maf005.thinp020
      --covar ../05_PCA/plink_results_projected.sscore
      --covar-col-nums 6-10
      --glm hide-covar firth
      --maf 0.01
      --out 1kgeas
      --pheno ../01_Dataset/1kgeas_binary.txt
      --pheno-name B1
      --threads 2
    
    Start time: Tue Dec 27 23:02:52 2022
    15957 MiB RAM detected; reserving 7978 MiB for main workspace.
    Using up to 2 compute threads.
    504 samples (0 females, 0 males, 504 ambiguous; 504 founders) loaded from
    ../01_Dataset/1KG.EAS.auto.snp.norm.nodup.split.maf005.thinp020.fam.
    1122299 variants loaded from
    ../01_Dataset/1KG.EAS.auto.snp.norm.nodup.split.maf005.thinp020.bim.
    1 binary phenotype loaded (201 cases, 302 controls).
    5 covariates loaded from ../05_PCA/plink_results_projected.sscore.
    Calculating allele frequencies... done.
    0 variants removed due to allele frequency threshold(s)
    (--maf/--max-maf/--mac/--max-mac).
    1122299 variants remaining after main filters.
    --glm Firth regression on phenotype 'B1': done.
    Results written to 1kgeas.B1.glm.firth .
    End time: Tue Dec 27 23:04:40 2022
    ```

Let's check the first lines of the output:

!!! example "Association test results"
    ```txt title="1kgeas.B1.glm.firth"
    #CHROM	POS	ID	REF	ALT	A1	TEST	OBS_CT	OR	LOG(OR)_SE	Z_STAT	P	ERRCODE
    1	13273	1:13273:G:C	G	C	C	ADD	503	0.746149	0.282904	-1.03509	0.300628	.
    1	14599	1:14599:T:A	T	A	A	ADD	503	1.67693	0.240899	2.14598	0.0318742.
    1	14604	1:14604:A:G	A	G	G	ADD	503	1.67693	0.240899	2.14598	0.0318742.
    1	14930	1:14930:A:G	A	G	G	ADD	503	1.64359	0.242872	2.04585	0.0407708.
    1	69897	1:69897:T:C	T	C	T	ADD	503	1.69142	0.200238	2.62471	0.00867216.
    1	86331	1:86331:A:G	A	G	G	ADD	503	1.41887	0.238055	1.46968	0.141649	.
    1	91581	1:91581:G:A	G	A	A	ADD	503	0.931304	0.123644	-0.5755980.564887	.
    1	122872	1:122872:T:G	T	G	G	ADD	503	1.04828	0.182036	0.259034	0.795609	.
    1	135163	1:135163:C:T	C	T	T	ADD	503	0.676666	0.242611	-1.60989	0.    107422	.
    ```

!!! info "Usually, other options are added to enhance the sumstats"

    * --keep xxx/kiso2021/for_plink2/unrelated.sample.id	# Because the standard linear     regression does not account for the relatedness, the kinship-pruned samples in last steps are suggested.
    * --mach-r2-filter 0.7 2.0	# It allows to use only the variants passed an (MaCH)Rsq filter.  NOTE: when pgen file is used, the upper boundary should be 2.
    * --glm **cols=+a1freq,+machr2** firth-fallback **omit-ref**	# The `cols=` requests the  following columns in the sumstats: here are allele1 frequency and (MaCH)Rsq, `firth-fallback`     will test the common variants without firth correction, which could improve the speed,     `omit-ref` will force the ALT==A1==effect allele, otherwise the minor allele would be tested     (see the above result, which ALT may not equal A1).
    * --covar-variance-standardize	# To normalize the covariates which may at a huge scale, like     AGE**AGE.
    * --covar-name AGE SEX PC1-PC20	# Instead of setting the index of columns, directly specify the     column name.



## Genomic control 

Genomic control (GC) is a basic method for controlling for confounding factors including population stratification.
  
We will calculate the genomic control factor (lambda GC) to evaluate the inflation. The genomic control factor is calculated by dividing the **median of observed Chi square statistics** by the **median of Chi square distribution with the degree of freedom being 1** (which is approximately 0.455).

$$ 
\lambda_{GC} = {median(\chi^{2}_{observed}) \over median(\chi^{2}_1)} 
$$

Then, we can used the genomic control factor to correct observed Chi suqare statistics.

$$ 
\chi^{2}_{corrected} = {\chi^{2}_{observed} \over \lambda_{GC}} 
$$

Genomic inflation is based on the idea that most of the variants are not associated, thus no deviation between the observed and expected Chi square distribution, except the spikes at the end. However, if the trait is highly polygenic, this assumption may be violated.

Reference: Devlin, B., & Roeder, K. (1999). Genomic control for association studies. Biometrics, 55(4), 997-1004.

## Significant loci

Please check [Visualization using gwaslab](https://cloufield.github.io/GWASTutorial/Visualization/)

Loci that reached suggestive significance threhold (P value < 5e-6) :
```
SNPID	CHR	POS	EA	NEA	SE	Z	P	OR	N	STATUS
1:217437563:C:T	1	217437563	C	T	0.151157	-5.22793	1.714210e-07	0.453736	503	9999999
2:55574452:G:C	2	55574452	C	G	0.160948	-5.98392	2.178320e-09	0.381707	503	9999999
3:176524872:C:T	3	176524872	T	C	0.248418	4.92774	8.318440e-07	3.401240	503	9999999
3:193128900:G:A	3	193128900	A	G	0.153788	4.70811	2.500290e-06	2.062770	503	9999999
6:29919659:T:C	6	29919659	T	C	0.155457	-5.89341	3.782970e-09	0.400048	503	9999999
9:36660672:A:G	9	36660672	G	A	0.160275	5.63422	1.758540e-08	2.467060	503	9999999
11:56249438:A:G	11	56249438	G	A	0.188891	-4.77836	1.767350e-06	0.405518	503	9999999
```

## Visualization

To visualize the sumstats, we will create the Manhattan plot, QQ plot and regional plot.

Please check for codes : [Visualization using gwaslab](https://cloufield.github.io/GWASTutorial/Visualization/)

![image](https://user-images.githubusercontent.com/40289485/209681591-dc691764-7346-4936-80b4-528bc425a61e.png)

### Manhattan plot

Manhattan plot is the most classic visualization of GWAS summary statistics. It is a form of scatter plot. Each dot represents the test result for a variant. variants are sorted by its genome coordinates and are aligned along the X axis. Y axis shows the -log10(P value) for tests of variants in GWAS. 

!!! note
    This kind of plot was named after Manhattan in New York City since it resembles the Manhattan skyline.   

!!! info "A real Manhattan plot"
    <img width="686" alt="image" src="https://user-images.githubusercontent.com/40289485/209780549-54a24fdd-485b-4875-8f40-d6812eb644fe.png">
    I took this photo in 2020 just before the COVID-19 pandemic. It was a cloudy and misty day. Those birds formed a significance threshold line. And the skyscrapers above that line resembled the significant signals in your GWAS.  I believe you could easily get how the GWAS Manhattan plot was named. 

Data we need from sumstats to create Manhattan plots:

- Chromosome 
- Basepair position
- P value or -log10(P)

!!! tips "Steps to create Manhattan plot"
    
    1. sort the variants by genome coordinates.
    2. map the genome coordinates of variants to the x axis.
    3. convert P value to -log10(P).
    4. create the scatter plot.

### Quantile-quantile plot

Quantile-quantile plot (as known as Q-Q plot), is commonly used to compare an observed distribution with its expected distribution. For a specific point (x,y) on Q-Q plot, its y coordinate corresponds to one of the quantiles of the observed distribution, while its x coordinate corresponds to the same quantile of the expected distribution.

Quantile-quantile plot is used to check if there is any significant inflation in P value distribution, which usually indicates population stratification or cryptic relatedness. 

Data we need from sumstats to create the Manhattan plot:

- P value or -log10(P)

!!! tips "Steps to create Q-Q plot"
    
    Suppose we have `n` variants in our sumstats,
    
    1. convert the `n` P value to -log10(P).
    2. sort the -log10(P) values in asending order.
    3. get `n` numbers from `(0,1)` with equal intervals.
    4. convert the `n` numbers to -log10(P) and sort in ascending order.
    4. create scatter plot using the sorted -log10(P) of sumstats as Y and sorted -log10(P) we generated as X.

!!! note 
    The expected distribution of P value is a Uniform distribution from 0 to 1.

    $$P_{expected} \sim U(0,1)$$

### Regional plot

Manhattan plot is very useful to check the overview of our sumstats. But if we want to check a specific genomic locus, we need a plot with finer resolution. This kind of plot is called regional plot. It is basically the Manhattan plot of only a small region on the genome, with points colored by its LD r2 with the lead variant in this region.

Such a plot is especially helpful to understand the signal and loci, e.g., LD structure, independent signals and genes.

The regional plot for the loci of 2:55574452:G:C. 

Please check [Visualization using gwaslab](https://cloufield.github.io/GWASTutorial/Visualization/)

![image](https://user-images.githubusercontent.com/40289485/209681608-3973c546-ad52-4d77-a3a1-a1c60a7a0a97.png)


### GWAS-SSF

To standardize the format of GWAS summary statistics for sharing, GWAS-SSF format was proposed in 2022. This format is now used as the standard format for GWAS Catalog.

GWAS-SSF consists of :

1. a tab-separated data file with well-defined fields (shown in the following figure)
2. an accompanying metadata file describing the study (such as sample ancestry, genotyping method, md5sum, and so forth)

!!! example "Schematic representation of GWAS-SSF data file"
    <img width="800" alt="image" src="https://github.com/Cloufield/GWASTutorial/assets/40289485/9c6526ec-0742-426a-83e5-8134e37078c2">

!!! quote "GWAS-SSF"
    Hayhurst, J., Buniello, A., Harris, L., Mosaku, A., Chang, C., Gignoux, C. R., ... & Barroso, I. (2022). A community driven GWAS summary statistics standard. bioRxiv, 2022-07.

For details, please check:

- [https://github.com/EBISPOT/gwas-summary-statistics-standard](https://github.com/EBISPOT/gwas-summary-statistics-standard)
- [https://www.ebi.ac.uk/gwas/docs/summary-statistics-format](https://www.ebi.ac.uk/gwas/docs/summary-statistics-format)

# Principle component analysis (PCA)

PCA aims to find the **orthogonal directions of maximum variance** and project the data onto a new subspace with equal or fewer dimensions than the original one. Simply speaking, **GRM (genetic relationship matrix; covariance matrix)** is first estimated and then PCA is applied to this matrix to generate **eigenvectors** and **eigenvalues**. Finally, the $k$ eigenvectors with the largest eigenvalues are used to transform the genotypes to a new feature subspace.

!!! info "Genetic relationship matrix (GRM)"
    <img width="600" alt="image" src="https://github.com/Cloufield/GWASTutorial/assets/40289485/767d940c-0ade-47b9-b53e-8e55cc3e0591">
    
    Citation: Yang, J., Lee, S. H., Goddard, M. E., & Visscher, P. M. (2011). GCTA: a tool for genome-wide complex trait analysis. The American Journal of Human Genetics, 88(1), 76-82.
    
!!! example "A simple PCA"
    
    Source data:
    ```
    cov = np.array([[6, -3], [-3, 3.5]])
    pts = np.random.multivariate_normal([0, 0], cov, size=800)
    ```

    The red arrow shows the first principal component axis (PC1) and the blue arrow shows the second principal component axis (PC2). The two axes are orthogonal.
    
    <img width="600" alt="image" src="https://github.com/Cloufield/GWASTutorial/assets/40289485/124b8c3d-0f83-4936-ab08-342efd29660a">

!!! quote "Interpretation of PCs" 
    **The first principal component** of a set of p variables, presumed to be jointly normally distributed, is the derived variable formed as a linear combination of the original variables that **explains the most variance**. The second principal component explains the most variance in what is left once the effect of the first component is removed, and we may proceed through p iterations until all the variance is explained.

PCA is by far the most commonly used dimension reduction approach used in population genetics which could identify the difference in ancestry among the sample individuals. The population outliers could be excluded from the main cluster. For GWAS we also need to include top PCs to adjust for the population stratification.

Please read the following paper on how we apply PCA to genetic data:
Price, A., Patterson, N., Plenge, R. et al. Principal components analysis corrects for stratification in genome-wide association studies. Nat Genet 38, 904–909 (2006). https://doi.org/10.1038/ng1847 https://www.nature.com/articles/ng1847

So before association analysis, we will learn how to run PCA analysis first.

- [Preparation](#preparation)
- [PCA steps](#pca-steps)
- [Sample codes](#sample-codes)
- [Plotting the PCs](#plotting-the-pcs)
- [PCA-UMAP](#pca-umap)
- [References](#references)


## Preparation

### Exclude SNPs in high-LD or HLA regions
For PCA, we first exclude SNPs in high-LD or HLA regions from the genotype data. 

!!! quote "The reason why we want to exclude such high-LD or HLA regions"
    - Price, A. L., Weale, M. E., Patterson, N., Myers, S. R., Need, A. C., Shianna, K. V., Ge, D., Rotter, J. I., Torres, E., Taylor, K. D., Goldstein, D. B., & Reich, D. (2008). Long-range LD can confound genome scans in admixed populations. American journal of human genetics, 83(1), 132–139. https://doi.org/10.1016/j.ajhg.2008.06.005 


### Download BED-like files for high-LD or HLA regions

You can simply copy the list of high-LD or HLA regions in Genome build version(.bed format) to a text file `high-ld.txt`. 

!!! quote "High LD regions were obtained from" 
    [https://genome.sph.umich.edu/wiki/Regions_of_high_linkage_disequilibrium_(LD)](https://genome.sph.umich.edu/wiki/Regions_of_high_linkage_disequilibrium_(LD))


!!! info "High LD regions of hg19"
    ```txt title="high-ld-hg19.txt"
    1	48000000	52000000	highld
    2	86000000	100500000	highld
    2	134500000	138000000	highld
    2	183000000	190000000	highld
    3	47500000	50000000	highld
    3	83500000	87000000	highld
    3	89000000	97500000	highld
    5	44500000	50500000	highld
    5	98000000	100500000	highld
    5	129000000	132000000	highld
    5	135500000	138500000	highld
    6	25000000	35000000	highld
    6	57000000	64000000	highld
    6	140000000	142500000	highld
    7	55000000	66000000	highld
    8	7000000 13000000	highld
    8	43000000	50000000	highld
    8	112000000	115000000	highld
    10	37000000	43000000	highld
    11	46000000	57000000	highld
    11	87500000	90500000	highld
    12	33000000	40000000	highld
    12	109500000	112000000	highld
    20	32000000	34500000	highld
    ```

### Create a list of SNPs in high-LD or HLA regions

Next, use `high-ld.txt` to extract all SNPs which are located in the regions described in the file using the code as follows:
    

```
plink --file ${plinkFile} --make-set high-ld.txt --write-set --out hild
```

!!! example "Create a list of SNPs in the regions specified in `high-ld.txt` "
    
    ```
    plinkFile="../01_Dataset/1KG.EAS.auto.snp.norm.nodup.split.maf005.thinp020" #! Please set this to your own path
    
    plink \
    	--bfile ${plinkFile} \
    	--make-set high-ld-hg19.txt \
    	--write-set \
    	--out hild
    ```
    
    And all SNPs in the regions will be extracted to hild.set.
    
    ```
    $head hild.set
    highld
    1:48000156:C:G
    1:48002096:C:G
    1:48003081:T:C
    1:48004776:C:T
    1:48006500:A:G
    1:48006546:C:T
    1:48008102:T:G
    1:48009994:C:T
    1:48009997:C:A
    ```

For downstream analysis, we can exclude these SNPs using `--exclude hild.set`.

---------
## PCA steps

!!! info "Steps to perform a typical genomic PCA analysis"

    - 1. LD-Pruning (https://www.cog-genomics.org/plink/2.0/ld#indep)
    - 2. Removing relatives from calculating PCs (usually 2-degree) (https://www.cog-genomics.org/plink/2.0/distance#king_cutoff)
    - 3. Running PCA using un-related samples and independent SNPs (https://www.cog-genomics.org/plink/2.0/strat#pca)
    - 4. Projecting to all samples (https://www.cog-genomics.org/plink/2.0/score#pca_project)

!!! note "MAF filter for LD-pruning and PCA"
    For LD-pruning and PCA, we usually only use variants with MAF > 0.01 or MAF>0.05. Since the sample dataset only contains variants with MAF > 0.05. We will skip the MAF filtering here. But please do keep this in mind when you work with your own datasets. (You can simply add `--maf 0.01` or `--maf 0.05` when performing LD-pruning or PCA.)

---------
## Sample codes

!!! example "Sample codes for performing PCA"
    ```
    plinkFile="" #please set this to your own path
    outPrefix="plink_results"
    threadnum=2
    hildset = hild.set 
    
    # LD-pruning, excluding high-LD and HLA regions
    plink2 \
            --bfile ${plinkFile} \
    	    --threads ${threadnum} \
    	    --exclude ${hildset} \ 
    	    --indep-pairwise 500 50 0.2 \
            --out ${outPrefix}
    
    # Remove related samples using king-cuttoff
    plink2 \
            --bfile ${plinkFile} \
    	    --extract ${outPrefix}.prune.in \
            --king-cutoff 0.0884 \
    	    --threads ${threadnum} \
            --out ${outPrefix}
    
    # PCA after pruning and removing related samples
    plink2 \
            --bfile ${plinkFile} \
            --keep ${outPrefix}.king.cutoff.in.id \
    	    --extract ${outPrefix}.prune.in \
    	    --freq counts \
    	    --threads ${threadnum} \
            --pca approx allele-wts 10 \
            --out ${outPrefix}
    
    # Projection (related and unrelated samples)
    plink2 \
            --bfile ${plinkFile} \
    	    --threads ${threadnum} \
            --read-freq ${outPrefix}.acount \
    	    --score ${outPrefix}.eigenvec.allele 2 5 header-read no-mean-imputation variance-standardize \
            --score-col-nums 6-15 \
            --out ${outPrefix}_projected
    ```

After step 3, the `allele-wts 10` modifier requests an additional one-line-per-allele `.eigenvec.allele` file with the first `10 PCs` expressed as allele weights instead of sample weights.

We will get the `plink_results.eigenvec.allele` file, which will be used to project onto all samples along with an allele count `plink_results.acount` file.

In the projection, `score ${outPrefix}.eigenvec.allele 2 5` sets the `ID` (2nd column) and `A1` (5th column), `score-col-nums 6-15` sets the first 10 PCs to be projected.

Please check https://www.cog-genomics.org/plink/2.0/score#pca_project for more details on the projection.

!!! example "Allele weight and count files"
    ```txt title="plink_results.eigenvec.allele"
    #CHROM	ID	REF	ALT	A1	PC1	PC2	PC3	PC4	PC5	PC6	PC7	PC8	PC9	PC10
    1	1:13273:G:C	G	C	G	1.12369	-0.320826	-0.0206569	-0.218665	0.869801	0.378433	-0.0723841	-0.227555	0.0361673	-0.368192
    1	1:13273:G:C	G	C	C	-1.12369	0.320826	0.0206569	0.218665	-0.869801	-0.378433	0.0723841	0.227555	-0.0361673	0.368192
    1	1:14599:T:A	T	A	T	0.99902	-1.15824	-1.80519	-0.36774	0.179881	0.25242	0.068899	0.206564	-0.342483	0.103762
    1	1:14599:T:A	T	A	A	-0.99902	1.15824	1.80519	0.36774	-0.179881	-0.25242	-0.068899	-0.206564	0.342483	-0.103762
    1	1:14930:A:G	A	G	A	-0.0704343	-0.35091	-0.41535	-0.304856	0.081039	-0.49408	-0.0667606	-0.0698847	0.245836	0.330869
    1	1:14930:A:G	A	G	G	0.0704343	0.35091	0.41535	0.304856	-0.081039	0.49408	0.0667606	0.0698847	-0.245836	-0.330869
    1	1:69897:T:C	T	C	T	-0.514024	0.563153	-0.997768	-0.298234	-0.840608	-0.247155	0.545471	-0.675274	-0.787836	-0.509647
    1	1:69897:T:C	T	C	C	0.514024	-0.563153	0.997768	0.298234	0.840608	0.247155	-0.545471	0.675274	0.787836	0.509647
    1	1:86331:A:G	A	G	A	-0.169641	-0.0125126	-0.531174	-0.0219291	0.614439	0.140143	0.133833	-0.570109	0.392805	-0.065334
    ```
    
    ```txt title="plink_results.acount"
    #CHROM	ID	REF	ALT	ALT_CTS	OBS_CT
    1	1:13273:G:C	G	C	63	1004
    1	1:14599:T:A	T	A	90	1004
    1	1:14930:A:G	A	G	417	1004
    1	1:69897:T:C	T	C	879	1004
    1	1:86331:A:G	A	G	87	1004
    1	1:91581:G:A	G	A	499	1004
    1	1:122872:T:G	T	G	259	1004
    1	1:135163:C:T	C	T	91	1004
    1	1:233473:C:G	C	G	156	1004
    ```

Eventually, we will get the PCA results for all samples.

!!! example "PCA results for all samples"
    ```txt title="plink_results_projected.sscore"
    #FID	IID	ALLELE_CT	NAMED_ALLELE_DOSAGE_SUM	PC1_AVG	PC2_AVG	PC3_AVG	PC4_AVG	PC5_AVG	PC6_AVG	PC7_AVG	PC8_AVG	PC9_AVG	PC10_AVG
    0	HG00403	219504	219504	0.000643981	-0.0297502	-0.0151499	-0.0122381	0.0229149	0.0235408	-0.033705	-0.0075127	-0.0125402	0.00271677
    0	HG00404	219504	219504	-0.000492225	-0.031018	-0.00764244	-0.0204998	0.0284068	-0.00872449	0.0123353	-0.00492058	-0.00557003	0.0248966
    0	HG00406	219504	219504	0.00620984	-0.034375	-0.00898555	-0.00335076	-0.0217559	-0.0182433	0.00333925	-0.00760613	-0.0340018	0.00641082
    0	HG00407	219504	219504	0.00678586	-0.0239308	-0.00704419	-0.00466139	0.00985433	0.000889767	0.00679557	-0.0200495	-0.0131869	0.0350328
    0	HG00409	219504	219504	-0.00236345	-0.0231604	0.0320665	0.0145563	0.0236768	0.00704788	0.012859	0.0319605	-0.0130627	0.0110219
    0	HG00410	219504	219504	0.000670927	-0.0210665	0.0467767	0.00293079	0.0184061	0.045967	0.00384994	0.0212317	-0.0296434	0.0237174
    0	HG00419	219504	219504	0.00526139	-0.0369818	-0.00974662	0.00855412	-0.0053907	-0.00102057	0.0063254	0.0140126	-0.00600854	0.00732882
    0	HG00421	219504	219504	0.00038356	-0.0319534	-0.00648054	0.00311739	-0.022044	0.0064945	-0.0105273	-0.0276718	-0.00973368	0.0208449
    0	HG00422	219504	219504	0.00437335	-0.0323416	-0.0111979	0.0106245	-0.0267334	0.00142919	-0.00487295	-0.0124099	-0.00467014	-0.0188086
    ```

## Plotting the PCs 
You can now create scatterplots of the PCs using R or Python.

For plotting using Python:
[plot_PCA.ipynb](https://github.com/Cloufield/GWASTutorial/blob/main/05_PCA/plot_PCA.ipynb)

!!! example "Scatter plot of PC1 and PC2 using 1KG EAS individuals"
    <img width="500" alt="image" src="https://user-images.githubusercontent.com/40289485/209298567-d4871fd0-aaa4-4d90-a7db-bc6aa34ab011.png">

    Note : We only used 20% of all available variants. This figure only very roughly shows the population structure in East Asia.
 
Requirements:
- python>3
- numpy,pandas,seaborn,matplotlib

## PCA-UMAP
(optional) 
We can also apply another non-linear dimension reduction algorithm called UMAP to the PCs to further identfy the local structures. (PCA-UMAP)

For more details, please check:
- https://umap-learn.readthedocs.io/en/latest/index.html

An example of PCA and PCA-UMAP for population genetics:
- Sakaue, S., Hirata, J., Kanai, M., Suzuki, K., Akiyama, M., Lai Too, C., ... & Okada, Y. (2020). Dimensionality reduction reveals fine-scale structure in the Japanese population with consequences for polygenic risk prediction. Nature communications, 11(1), 1-11.

# References
- (PCA) Price, A., Patterson, N., Plenge, R. et al. Principal components analysis corrects for stratification in genome-wide association studies. Nat Genet 38, 904–909 (2006). https://doi.org/10.1038/ng1847 https://www.nature.com/articles/ng1847
- (why removing high-LD regions) Price, A. L., Weale, M. E., Patterson, N., Myers, S. R., Need, A. C., Shianna, K. V., Ge, D., Rotter, J. I., Torres, E., Taylor, K. D., Goldstein, D. B., & Reich, D. (2008). Long-range LD can confound genome scans in admixed populations. American journal of human genetics, 83(1), 132–139. https://doi.org/10.1016/j.ajhg.2008.06.005 
- (UMAP) McInnes, L., Healy, J., & Melville, J. (2018). Umap: Uniform manifold approximation and projection for dimension reduction. arXiv preprint arXiv:1802.03426.
- (UMAP in population genetics) Diaz-Papkovich, A., Anderson-Trocmé, L. & Gravel, S. A review of UMAP in population genetics. J Hum Genet 66, 85–91 (2021). https://doi.org/10.1038/s10038-020-00851-4 https://www.nature.com/articles/s10038-020-00851-4
- (king-cutoff) Manichaikul, A., Mychaleckyj, J. C., Rich, S. S., Daly, K., Sale, M., & Chen, W. M. (2010). Robust relationship inference in genome-wide association studies. Bioinformatics, 26(22), 2867-2873.

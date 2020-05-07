Diploid
=================================================
*EXPLANATION - made by Jan.28th 2020*

Because of the *TINY* mistake I made before about tetraploid taxa and diploid taxa, I have to make the former one as 'Pilot run' and this one as 'diploid' dataset. However, I should be aware that this contains an assumption, which is, we assumed that the hybrids between tetraploid and diploid are diploid, which might not be the cases. For this, we can test it later with **k-mer**. I'm not sure now, but it seems it could indicate the ploidy -- this information I got from Alex's NGG courses. We may discuss with this problem next meeting. 

Also, I want to set up some *RULES* here for the log:
1. the order should follow the date, date with the major analysis;
2. Each day should have a conclusion, where to stop, why (time or bugs), what to do next;
3. Record every increasing details that may change the direction of analysis, for example,  talking with Alex or other researchers or reading more reference -- make the flow as clear as you can and believe this is not wasting time. You will get a lot more benefits than those 'wasting time'.

=================================================
20200115-16_TASSEL-GBS
=================================================
**Step1: GBSSeqToTagDBPlugin**
::

  ./run_pipeline.pl -Xms100G -Xmx200G -GBSSeqToTagDBPlugin -db /localdisk/home/s1950737/Diploid/GBSV3.db -e ApeKI -i /disk2/Twyford_GBS_illumina -k /localdisk/home/s1950737/Diploid/DipKey.txt 

*Time consumed: Start time: 17:41 15/01/20 - 17:57 15/01/20; 16 minutes; screen -r diploid*

**Step2: TagExportToFastqPlugin**
::

  ./run_pipeline.pl -TagExportToFastqPlugin -db /localdisk/home/s1950737/Diploid/GBSV3.db -o /localdisk/home/s1950737/Diploid/tagsForAlign.fa.gz -c 1 -endPlugin

*Time consumed: 1 minute*

**Step3: Alingment**

''Step3-1: create an index from the reference genome'' is just the same as before, so can be directly used.

*Step3-2: Aligment*
::

  ./bwa aln -t 4 ../referenceGenome/anglica_HB050619_scaffoldsKeptAnno.fasta ../Diploid/tagsForAlign.fa.gz > tagsForAlign.sai

*Time consumed: 29 seconds*

::

  ./bwa samse ../referenceGenome/anglica_HB050619_scaffoldsKeptAnno.fasta tagsForAlign.sai ../Diploid/tagsForAlign.fa.gz > tagsForAlign.sam

*Time consumed: 9.5seconds*

**Step4: SAMToGBSdbPlugin**
::

  ./run_pipeline.pl -SAMToGBSdbPlugin -i ../Diploid/tagsForAlign.sam -db ../Diploid/GBSV3.db

*Time consumed: 1 minute*

**Step5: DiscoverySNPCallerPluginV2**  
::

  ./run_pipeline.pl -DiscoverySNPCallerPluginV2 -db ../Diploid/GBSV3.db

*Time consumed: 19:26 15/01/20 - 6:18 16/01/20, 11h*

**Step6: ProductionPipeline ProductionSNPCallerPluginV2**
::

  ./run_pipeline.pl -ProductionSNPCallerPluginV2 -db ../Diploid/GBSV3.db -e ApeKI -i /disk2/Twyford_GBS_illumina -k ../Diploid/DipKey.txt -o ../Diploid/productionHapMap_diploid.vcf

*Time consumed: 09:15 16/01/2020 - 14:47 23/11/2019; 10 minutes.*

=================================================
20200116_SNP-filter
=================================================

Program: TASSEL v5.0 GUI
-------------------------------------------

Filter:

1. MAF=0.01 with missing data (MD)<=75%;
2. MAF=0.01 with missing data (MD)<=80%;
3. MAF=0.05 with missing data (MD)<=75%;
4. MAF=0.05 with missing data (MD)<=80%;

Q:In TASSEL-GBS, there is a MAF, and in this step, we have another MAF, what's the difference?

Comparison:

1. Different filters with different SNPs and taxa remained; ==>> finally chose the filter with 'MAF=0.01 with missing data <= 75%'; *Consideration:* more taxa with less missing data;
2. Make the summary about the taxa information, including the species details, how many populations and individuals, store the information both in the <MAF0.01MD75> filter file in the 'taxa -- summary' sheet and <ComparisonFilters> file in the 'chosensummary' sheet.

=================================================
20200116_PLINK
=================================================

**Summary statistics**
-------------------------------------------

**Hardy-Weinberg equilibrium (HWE)**

The first try:
::

  .\plink.exe --vcf .\MAF0.01MD75.vcf --hardy --out HWE

*ERRORS AND DEBUGS -- the last one is working*
::
 
  ERROR: Invalid chromosome code 'SCAFFOLD100007|SIZE873_NO-HIT' on line 12 of.vcf file. (Use --allow-extra-chr to force it to be accepted.)
  DEBUG:  .\plink.exe --vcf MAF0.01MD75 --hardy --out HWE
  ERROR: Failed to open MAF0.01MD75. (--vcf expects a complete filename; did you forget '.vcf' at the end?)
  DEBUG: .\plink.exe --vcf MAF0.01MD75.vcf --hardy --out HWE
  ERROR: Invalid chromosome code 'SCAFFOLD100007|SIZE873_NO-HIT' on line 12 of .vcf file. (Use --allow-extra-chr to force it to be accepted.)
  DEBUG: .\plink.exe --vcf MAF0.01MD75.vcf --allow-extra-chr --hardy --out HWE

*Time consumed: instantly*

**TIPS**

PLINK will have the log file itself to tell the commands and results, including error information and debugs, which is really good. 

**Other Summary statistics**

::

  .\plink.exe --vcf MAF0.01MD75.vcf --allow-extra-chr --het --out SummaryStatistics

*Time consumed: instantly*

**Principal coordinate analysis (PCoA)**
-------------------------------------------
**Prepare in PLINK v1.9**
::

  FIRST: .\plink.exe --file .\MAF0.01MD75.vcf --cluster --mds-plot 4
  ERROR: Failed to open .\MAF0.01MD75.vcf.map.
  DEBUG: .\plink.exe --bfile .\MAF0.01MD75.vcf --genome --out MAF0.01MD75
  ERROR: Failed to open .\MAF0.01MD75.vcf.bed.
  DEBUG: .\plink.exe --file .\MAF0.01MD75.vcf --cluster
  ERROR: Failed to open .\MAF0.01MD75.vcf.map.
  DEBUG: .\plink.exe --bfile MAF0.01MD75-PLINK --genome --out MAF0.01MD75
  ERROR: Failed to open MAF0.01MD75-PLINK.bed.
  DEBUG: .\plink.exe --bfile .\MAF0.01MD75.vcf --update-ids recoded.txt --make-bed --out MAF0.01MD75-2
  ERROR: Failed to open MAF0.01MD75-PLINK.bed.
  DEBUG: .\plink.exe --file .\MAF0.01MD75.vcf --make-bed
  ERROR: Failed to open .\MAF0.01MD75.vcf.map.
  DEBUG: .\plink.exe --file .\MAF0.01MD75-PLINK.plk --make-bed
  ERROR: Invalid chromosome code 'SCAFFOLD100007|SIZE873_NO-HIT' on line 1 of .map file. (Use --allow-extra-chr to force it to be accepted.)
  DEBUG(WORKED): .\plink.exe --file .\MAF0.01MD75-PLINK.plk --allow-extra-chr --make-bed
  NEXT: .\plink.exe --bfile MAF0.01MD75-PLINK --genome --out MAF0.01MD75
  ERROR: Failed to open MAF0.01MD75-PLINK.bed.
  DEBUG(WORKED): change the file name from 'plink.bed', 'plink.bim', 'plink.fam' to 'MAF0.01MD75-PLINK.bed', 'MAF0.01MD75-PLINK.bim', 'MAF0.01MD75-PLINK.fam', then run the code again.
  ERROR: Invalid chromosome code 'SCAFFOLD100007|SIZE873_NO-HIT' on line 1 of .bim file. (Use --allow-extra-chr to force it to be accepted.)
  DEBUG(WORKED): .\plink.exe --bfile MAF0.01MD75-PLINK --allow-extra-chr --genome --out MAF0.01MD75
  NEXT(WORKED): .\plink.exe --file .\MAF0.01MD75-PLINK --allow-extra-chr --cluster 
  NEXT: .\plink.exe --file .\MAF0.01MD75-PLINK --allow-extra-chr --cluster --distance-matrix
  ERROR: --cluster and --neighbour cannot be used with non-IBS distance matrix calculations.
  DEBUG(WORKED): .\plink.exe --file .\MAF0.01MD75-PLINK --allow-extra-chr --cluster --matrix
  NEXT: .\plink.exe --file .\MAF0.01MD75-PLINK --allow-extra-chr --cluster --distance-matrix

**Summary at this moment**
---------------------------------------

1. In population structure, we need the PLINK format -- it's easy to get, either save as plink format using TASSEL-GUI or transform the format in PLINK itself. In PLINK format, we will get *PED* and *MAP* file, more details `Input files <http://zzz.bwh.harvard.edu/plink/data.shtml#ped>`_

2. Besides, we also need the binary files for the input file -- *bed* file. So we first need to get the binary file `Different out files <http://zzz.bwh.harvard.edu/plink/reference.shtml#output>`_

3. Because all these analyses require genome-wide coverage of autosomalSNPs and, this is based on human chromosomes, so we need to use *--allow-extra-chr* every time.

4. Summary for the input files and commands and out files:

MAF0.01MD75-PLINK.ped & MAF0.01MD75-PLINK.map 

==>>
::

  .\plink.exe --file .\MAF0.01MD75-PLINK.plk --allow-extra-chr --make-bed

==>>

plink.bed, plink.bim & plink.fam (change the name as MAF0.01MD75-PLINK.bed, MAF0.01MD75-PLINK.bim & MAF0.01MD75-PLINK.fam just to keep it easy for the following steps)

==>>
::

  .\plink.exe --bfile MAF0.01MD75-PLINK --allow-extra-chr --genome --out MAF0.01MD75

==>>

MAF0.01MD75.genome

==>> 
::

  .\plink.exe --file .\MAF0.01MD75-PLINK --allow-extra-chr --cluster --matrix

==>>

plink.cluster1, plink.cluster2, plink.cluster3, plink.mibs, plink.mibs.id

============================================================
20200116_Bayesian -- fastStructure
============================================================

Prepare in PLINK v1.9 -- LD-based SNPs
-----------------------------------------
::

  .\plink.exe --file MAF0.01MD75-PLINK --allow-extra-chr --indep-pairwise 50 5 0.5
  .\plink.exe --file MAF0.01MD75-PLINK --allow-extra-chr --extract plink.prune.in --make-bed --out pruneddata
  
  Total genotyping rate is 0.5294.
  97814 variants and 74 people pass filters and QC.
  Note: No phenotypes present.
  Warning: Skipping --indep-pairwise since there are less than two founders. (--make-founders may come in handy here.)

Q: there's no output file, what does this mean and how does it happen? -- need to be solved, so I just tried the SNPs without LD test in the following.

Reference: 'fastStructure <https://www.genetics.org/content/genetics/197/2/573.full.pdf>'_
Github download: 'fastStructure <https://rajanil.github.io/fastStructure/>'_

Github-server
--------------------------------------------------

**Getting the source code**
(in my directory -- s1950737)
::

  mkdir ~/proj
  cd ~/proj
  git clone https://github.com/rajanil/fastStructure
  cd ~/proj/fastStructure
  git fetch
  git merge origin/master #update the latest version

**Building Python extensions**
::

  locate libgsl.so  ##/usr/lib64/
  locate libgslcblas.so ##/usr/lib64/
  locate gsl/gsl_sf_psi.h ##/usr/include
  cd ../../../ #to home directory
  vim bashrc
    export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/lib64
    export CFLAGS="-I/usr/include"
    export LDFLAGS="-L/usr/lib64"
  source bashrc
  cd proj/fastStructure/vars
  python setup.py build_ext --inplace
  cd ../
  python setup.py build_ext --inplace

  python setup.py build_ext --inplace

**Input files**

*MAF0.01MD75-PLINK.bed*; *MAF0.01MD75-PLINK.bim*; *MAF0.01MD75-PLINK.fam* -- without LD test

**Excuting the code**
::

  python structure.py -K 3 --input=MAF0.01MD75-PLINK --output=MAF0.01MD75_output

*Time consumed: less than 1 minute*

Summary for 0116
--------------------------------------------------
1. Not quite sure for those input and output files in PLINK;
2. Not quite sure for those filtering, with sites and taxa, etc.;
3. Not quite sure for what's happening in PCoA in PLINK;
4. Need learn more about the next steps for fastSTRUCTURE -- DISTRUCT maybe, already downloaded the reference 'DISTRUCT <https://onlinelibrary.wiley.com/doi/full/10.1046/j.1471-8286.2003.00566.x>'_

============================================================
**2020123_Adjustment-meeting with Alex**
============================================================
Summary for the meeting with Alex:

1. SNPs filtering needs some adjustment, three dimensions: taxa, site, MAF; it's better to use PLINK v1.9 to to the filtering;
2. Recommendation for the PCA and the other analysis: R package **adegenet** 'adegenet <http://adegenet.r-forge.r-project.org/>'_

============================================================
20200127-28_SNP filtering
============================================================

Program: PLINK v1.9
------------------------------------------- 

TIPs: three parameters -- per site(>=75%); per taxa (75%); MAF(0.01)
::

  ./plink --file SNPCalling.plk --geno 0.25 --mind 0.25 --maf 0.01 --allow-extra-chr --make-bed --out MAF0.01TAXA75MD75


::

  ./plink --file SNPCalling.plk --geno 0.25 --mind 0.25 --maf 0.01 --allow-extra-chr --recode vcf-iid --out MAF0.01TAXA75MD75_cvf

  ./plink --file SNPCalling.plk --geno 0.75 --mind 0.75 --maf 0.01 --allow-extra-chr --recode vcf-iid --out MAF0.01TAXA75MD75_cvf2

  ./plink --file SNPCalling.plk --geno 0.75 --mind 0.75 --maf 0.05 --allow-extra-chr --recode vcf-iid --out MAF0.05TAXA75MD75_cvf

**Results and comments for Jan.27th**
------------------------------------------- 

1. by *make-bed* I got the binary file but I don't know how to check it in TASSEL;

2. Then I used *vcf-iid* to make vcf files, thus I can check it in TASSEL-GUI;

3. But I find with the other two commands I got the nearly same results with *maf0.01* and *maf0.05*;

4. Thus, I need more reading carefully with PLINK document.

**Rerun Round1 -- after reading document**
-------------------------------------------
*Step1. Missing rate per person: 25%*
::

  ./plink --file SNPCalling.plk --mind 0.25 --allow-extra-chr --recode --out clean1  ## exclude with more than 25% missing genotypes.

*Step2. Allele frequency: 0.01*
::

  ./plink --file clean1 --maf 0.01 --allow-extra-chr --recode --out clean2 ##include SNPs with MAF >=0.01. 

*Step3. Missing rate per SNP: 0.25*
::

  ./plink --file clean2 --geno 0.25 --allow-extra-chr --recode vcf-iid --out clean3.vcf ##Removing individuals with high missing genotype rates

**Results & comments**

There are only 7 taxa left with the threshold 0.25 of missing genotypes. -- it should be the opposite -- that is, more than 25% missing data will be removed. So, I need to set as 75% rather than 25%

**Rerun Round2 -- worked well**
-------------------------------------------
*Step1. Missing rate per person: 75% (Taxa)*
::

  ./plink --file SNPCalling.plk --mind 0.75 --allow-extra-chr --recode --out clean1.1  ## exclude with more than 75% missing genotypes.

*Step2. Allele frequency: 0.01 (MAF)*
::

  ./plink --file clean1.1 --maf 0.01 --allow-extra-chr --recode --out clean2.1 ##include SNPs with MAF >=0.01.
  
*Step3. Missing rate per SNP: 0.25 (Site)*
::

  ./plink --file clean2.1 --geno 0.25 --allow-extra-chr --recode vcf-iid --out clean3 ##Removing individuals with high missing genotype rates

**Results & comments**

The results seem really nice. Then I need to clean up all those process-files, i.e. those trying such as clean1, 2, 3 etc. Just keep those working files.

Then, the next step is trying different filtering conditions and decided which one to choose for the next steps. Before this, try another one, could this all be done in one command?
::

  ./plink --file SNPCalling.plk --allow-extra-chr --mind 0.75 --maf 0.01 --geno 0.25 --recode vcf-iid --out cleanT

Conclusion: yes, exactly the same. 

**Rerun Round more -- different filtering conditions**
----------------------------------------------------------
**TIPS: BEFORE RUN DIFFERENT FILTERING CONDITIONS, CLEAN UP ALL THE TRYINGS TO MAKE IT CLEAN AND CLEAR. THUS, IT STARTS FROM 1**

*Taxa:75%; MAF:0.01; Site:25%*
::

  ./plink --file SNPCalling.plk --allow-extra-chr --mind 0.75 --maf 0.01 --geno 0.25 --recode vcf-iid --out clean1

*Taxa:80%; MAF:0.01; Site:25%*
::

  ./plink --file SNPCalling.plk --allow-extra-chr --mind 0.80 --maf 0.01 --geno 0.25 --recode vcf-iid --out clean2

*Taxa:80%; MAF:0.01; Site:20%*
::

  ./plink --file SNPCalling.plk --allow-extra-chr --mind 0.80 --maf 0.01 --geno 0.20 --recode vcf-iid --out clean3

*Taxa:75%; MAF:0.05; Site:25%*
::

  ./plink --file SNPCalling.plk --allow-extra-chr --mind 0.75 --maf 0.05 --geno 0.25 --recode vcf-iid --out clean4

*Taxa:80%; MAF:0.05; Site:20%*
::

  ./plink --file SNPCalling.plk --allow-extra-chr --mind 0.80 --maf 0.05 --geno 0.20 --recode vcf-iid --out clean5

*Taxa:75%; MAF:0.01; Site:20%*
::

  ./plink --file SNPCalling.plk --allow-extra-chr --mind 0.75 --maf 0.01 --geno 0.20 --recode vcf-iid --out clean6

**Conclusion for SNPs filtering**
-------------------------------------------

1. Three dimensions: taxa, site(SNP), MAF, MAF matters less -- MAF=0.01 is exact the same as MAF=0.05, maybe it's just because the difference between 0.01 and 0.05 is too little;

2. Final chosen for the next steps are this filtering condition (clean6): *taxa-75%, MAF-0.01, site-20%*;

3. Next step is: fastStructure & PCA(R package -- adegenet.

============================================================
20200128-31_fastStructure
============================================================

Ref: 

`Case-study <https://doi.org/10.1007/s11295-019-1406-x>`_

`fastStructure <https://www.genetics.org/content/197/2/573>`_

`DISTRUCT <https://onlinelibrary.wiley.com/doi/10.1046/j.1471-8286.2003.00566.x>`_

**Plan: as referenced by case-study, try the following steps**

  *Step1:* PLINK v1.9, LD-based SNP pruning;

  *Step2:* run fastStructure, set K in different values;

  *Step3:* test best K, run build-in script (Raj. et al., 2014);

  *Step4:* visualized, DISTRUCT plots (Rosenberg 2004);

  *Step5:* subpopulations, R package 'adegenet' (Jombart and Ahimed 2011).

Step1: PLINK v1.9, LD-based SNP pruning (20200128)
---------------------------------------------------------------
Before LD-based SNP pruning, deal with the missing data, the same as before: [taxa-75%, maf=0.01, site=20%]

::

  ./plink --file SNPCalling.plk --allow-extra-chr --mind 0.75 --maf 0.01 --geno 0.20 --recode --out cleanFS

LD-based SNP pruning:
::

  ./plink --file cleanFS --allow-extra-chr --indep 50 5 2 --make-founders --out cleanFS1

Make input file for fastStructure; (i.e.: .bed, .bim, .fam)
::

  ./plink --file cleanFS --allow-extra-chr --extract cleanFS1.prune.in --make-bed --out pruneCleanFS

Step2: run fastStructure (20200128)
---------------------------------------------------------------
**Q before running: how many K do I need? Two aspects to consider: species/hybrids and geography; it's easy for species/hybrids but you need more checking in geography**

**Temporary summary for Jan. 28th**
1. In general, everything goes well, find the problem, solve the problem. The filtering is quite clear now, including how to make the data management in PLINK v1.9;

2. The problem for now is that I should deal with my Python version things first. There are two versions -- python2 and python3 in my Mac system, and somehow there will be some problems in Python3 and also python2. Only solve those problems that I can move to the next steps.

3. The next steps should be as following: 

  3-1. fastStructure: the K should be decided according to two parts (taxa and geography). For the taxa, the maximum should be 13, since we got 13 species/hybrids. The minimum should be xxx, not sure yet, should be working out tomorrow. For the geography, I need to check them again and then decide how many I should test in K value. 

  3-2. PCA/PCoA/DAPC: check out in R package 'adegenet' -- it also used in fastStructure. I didn't get the time to check it today. So it should be the next steps.

**Jan.29 analysis in server -- fastStructure & choose K**
::

  ./python structure.py -K 15 --input=pruneCleanFS --output=pruneCleanFS.output

Then, run the other 14 times, K from 1-14; then *Choosing model complexity:*
::

  ./python chooseK.py --input=pruneCleanFS.output

*Results:*
::

  Model complexity that maximizes marginal likelihood = 4
  Model components used to explain structure in data = 6

**Jan.30 analysis in Mac -- distruct, visulasation**

::

  ./python3 distruct.py -K 6 --input=pruneCleanFS.output --output=pruneCleanFS0.svg
  ## This is only for the population, we don't know the exact combination in each population.

::

  ./python3 distruct.py -K 6 --input=pruneCleanFS.output --output=pruneCleanFS1.svg --popfile=Popfile1.txt
  ## This includes both the taxa name and species/hybrids names; in this distruct.py, xmin=0.01; width=1; label fontsize=3
  ./python3 distruct.py -K 6 --input=pruneCleanFS.output --output=pruneCleanFS1.svg --popfile=Popfile2.txt
  ## This only includes the species/hybrids names, and it will clustering in different species/hybrids; in this distruct.py, xmin=0.1; width=0.95; label fontsize=7

**Summaries - problems & solutions:**
-------------------------------------------------------

1. Use server to run fastStructure, run K from 1-15, got the best K is 6;

2. When use the DISTRUCT, the version of python in server is not required( *ImportError: No module named matplotlib.pyplot* -- and now *Matplotlib requires Python >=3.6* ), so moved to MAC. 

3. However, there are always some problems with the distruct.py because this script based on python2 before. So I corrected the syntax error then it works. -- DON'T BE PANIC and ANXIOUS, it's not a big deal, just search it and solve it. That's it. 

4. For the visualization: Still I haven't found how to change the artboards, but instead, I can change the place of the figure and the label fontsize. **It should be improved**

5. Next step: to find out the location, to see if they're clustering in locations and also PCA.

**geography-20200130-31**

1. Use the latitude & longitude information and the grid map in British, relocate their places as grid number, then link them with taxa number/name to make the structure map; 

2. The similar parameters with taxa

::

  ./python3 distruct.py -K 6 --input=0_DIPLOID/pruneCleanFS.output --output=geography1.svg --popfile=geography1.txt
  ## This includes both the taxa number and geographical info; in this distruct.py, xmin=0.01; width=1; label fontsize=3
  ./python3 distruct.py -K 6 --input=0_DIPLOID/pruneCleanFS.output --output=geography2.svg --popfile=geography2.txt
  ## This includes both the taxa name_geographical info; in this distruct.py, xmin=0.01; width=1; label fontsize=3
  
============================================================
20200131_meeting with Alex
============================================================
1. We can make graph every time we did a filter, according to the graph we will find the best filter condition. -- this time is fine but it's quite useful when the datasets become much more complicated;

2. LD filter, make sure that the LD won't have any influence to the downstream analysis. Especially for the STRUCTURE, because it assumes that all markers are dependently. There are several methods to check out it. 

 2-1. **PLINK:** *--indep* flag, which is based on *variance inflation factor* ; *--indep-pairwise* , which is based on pairwise genotypic correlation. In my previous trying, I used *--indep 50 5 2* , which means, 50 for window size in SNPs, 5 SNPs to shift the window at each step, and VIF threshold is 2. -- VIF of 1 would imply that the SNP is completely independent of all other SNPs. 

 2-2. VCF tools

 2-3. ipyrad

so my next step is to try *--indep 50 5 1* and also *--indep-pairwise 50 5 0.5* ; and the other two.

============================================================
20200204_New LD pruning & FS
============================================================
PLINK
---------------------------
**use VIF=1**
::

  ./plink --file ./2_PrepareForFS_20200128/cleanFS --allow-extra-chr --indep 50 5 1 --make-founders --out unlinkVIF1
  ./plink --file ./2_PrepareForFS_20200128/cleanFS --allow-extra-chr --extract unlinkVIF1.prune.in --make-bed --out unlinkVIF1 
  ./plink --file ./2_PrepareForFS_20200128/cleanFS --allow-extra-chr --extract unlinkVIF1.prune.in --recode vcf-iid --out unlinkvif1

**USE pairwise**
::

  ./plink --file ./2_PrepareForFS_20200128/cleanFS --allow-extra-chr --indep-pairwise 50 5 0.5 --make-founders --out pairwise0.5
  ./plink --file ./2_PrepareForFS_20200128/cleanFS --allow-extra-chr --extract pairwise0.5.prune.in --make-bed --out pairwise05 
  ./plink --file ./2_PrepareForFS_20200128/cleanFS --allow-extra-chr --extract pairwise0.5.prune.in --recode vcf-iid --out pairwise05

**summaries**
1. For VIF, the SNPs of VIF=2 is 12712 (original cleanFS is 20818); the SNPs of VIF=1 is 4851;

2. For pairwise, the SNPs of 0.5 is 13326. I need more information of the difference between VIF and pairwise, but here we should use those *unlink-SNPs* to try the structure. 

FS-FastStructure
---------------------------
**analysis in server -- fastStructure & choose K; before this, move all the files of VIF=2 to folder VIF2**

::

  python structure.py -K 15 --input=unlinkVIF1 --output=unlinkVIF1.output

Then, run the other 14 times, K from 1-14; then *Choosing model complexity:*
::

  python chooseK.py --input=unlinkVIF1.output

*Results:*
::

  Model complexity that maximizes marginal likelihood = 2
  Model components used to explain structure in data = 6

**analysis in Mac -- distruct, visulasation**

::

  python3 distruct.py -K 6 --input=unlinkVIF1.output --output=unlinkP6.svg --popfile=popfile.txt
  ## This includes both the taxa name and species/hybrids names; in this distruct.py, xmin=0.01; width=1; label fontsize=3
  python3 distruct.py -K 6 --input=unlinkVIF1.output --output=unlinkG6.svg --popfile=./0_DIPLOID/geography1.txt
  ## This only includes the species/hybrids names, and it will clustering in different species/hybrids; in this distruct.py, xmin=0.01; width=0.98; label fontsize=3

To put the same parameters for graphs, use different K and same pop file.
This includes the taxa and geographical information; in this distruct.py, xmin=0.035; width=0.98; label fontsize=3
::

  python3 distruct.py -K 5 --input=unlinkVIF1.output --output=unlinkG5.svg --popfile=./0_DIPLOID/geography1.txt
  
This includes both the taxa name and species/hybrids names; in this distruct.py, xmin=0.01; width=1; label fontsize=3
::

  python3 distruct.py -K 5 --input=unlinkVIF1.output --output=unlinkP5.svg --popfile=popfile.txt

============================================================
20200204_R-PCA & DAPC - 'adegenet' package in R
============================================================
**TIPS** 

This is a long time since last time I did something in R and python. There are a lot of packages updated during this period, even with the R and python versions. Thus, remember every time when you met some problems you should check the system or version first. Make sure that this is not the version mistakes. For this time, I updated my R version and all the packages needed. Just keep this in mind, and then continue. 

glPca
------------------------
Description:

  These functions implement Principal Component Analysis (PCA) for massive SNP datasets stored as *genlight* object. 

Usage:
::  

  glPca(x, center = TRUE, scale = FALSE, nf = NULL, loadings = TRUE, alleleAsUnit = FALSE, useC = TRUE, parallel = FALSE, n.cores = NULL, returnDotProd = FALSE, matDotProd = NULL)

First, I need to change the format as *genind*

**vcf to genind**
::

  library(adegenet)
  library(ade4)
  library(vcfR)
  vcf <- read.vcfR('clean6.vcf')
  my_genind <- vcfR2genind(vcf)

**problem: vcfs do not contain population information**

TRY different types to solve the problem:

 1. PLINK;
 2. IPYRAD;
 3. adegenet;
 4. the other possible softwares.


============================================================
20200210_PCoA-PLINK
============================================================
REFERENCE: https://github.com/GELOG/adam-ibs/wiki/Plink-IBS-MDS-Tutorial
::

  ## Step1-1. Make the input file (SNP filtering)
  ./plink --file SNPCalling.plk --allow-extra-chr --mind 0.75 --maf 0.01 --geno 0.20 --recode --out PCOA1
  *input:  SNPCalling.plk.map; SNPCalling.plk.ped*
  *output: PCOA1.map, PCOA1.ped; PCOA1.nosex; PCOA1.irem*

  ## Step1-2. creating the binary dataset (--make-bed)
  ./plink --file PCOA1 --allow-extra-chr --make-bed --out PCOA2
  *input: PCOA1.map; PCOA1.ped*
  *output: PCOA2.bed, PCOA2.bim, PCOA2.fam; PCOA2.log; PCOA2.nosex*

  ## Step1-3. calculating the pairwise IBS metrics (--genome)
  ./plink --bfile PCOA2 --allow-extra-chr --genome --out ibs1
  *input: PCOA2.bed, PCOA2.bim, PCOA2.fam*
  *output: ibs1.genome; ibs1.log, ibs1.nosex*

  ## Step2-1. performing an IBS clustering analysis (--cluster)
  ./plink --bfile PCOA2 --allow-extra-chr --read-genome ibs1.genome --out ibs2 --cluster
  *input: PCOA2.bed, PCOA2.bim, PCOA2.fam; ibs1.genome*
  *output: ibs2.cluster[1-3]; ibs2.log, ibs2.nosex*

  ## Step2-2. performing an IBS clustering analysis with two constraints (--cluster --ppc --cc)
  ./plink --bfile PCOA2 --allow-extra-chr --read-genome ibs1.genome --out ibs3 --cluster --ppc 0.01 --cc
  *input: PCOA2.bed, PCOA2.bim, PCOA2.fam; ibs1.genome*
  *output: ibs3.cluster[1-3]; ibs3.log, ibs3.nosex*
 
  ## Step2-3 performing an IBS similarity matrix (--cluster --matrix)
  ./plink --bfile PCOA2 --allow-extra-chr --read-genome ibs1.genome --out ibs4 --cluster --matrix
  **ERROR:  --read-genome is pointless with --ibs-matrix unless --ppc is also present.**
  **Debug: ./plink --bfile PCOA2 --allow-extra-chr --read-genome ibs1.genome --out ibs4 --cluster --matrix --ppc 0.01**
  *input: PCOA2.bed, PCOA2.bim, PCOA2.fam; ibs1.genome*
  *output: ibs4.cluster[1-3]; ibs4.mibs, ibs4.mibs.id; ibs4.log, ibs4.nosex*

  ## Step2-4. performing an IBS distance matrix (--cluster --distance-matrix)
  ./plink --bfile PCOA2 --allow-extra-chr --read-genome ibs1.genome --out ibs5 --cluster --distance-matrix --ppc 0.01
  **ERROR: --cluster and --neighbour cannot be used with non-IBS distance matrix calculations**
  *input: PCOA2.bed, PCOA2.bim, PCOA2.fam; ibs1.genome*
  *output should be: ibs5.cluster[1-3], ibs5.hh, ibs5.mdist*

  ## Step3. performing a MDS analysis (--mds-plot)
  ./plink --bfile PCOA2 --allow-extra-chr --read-genome ibs1.genome --out mds1 --cluster --mds-plot 4
  *input: PCOA2.[bed, bim, fam]; ibs1.genome*
  *output: mds1.cluster[1-3]; mds1.mds*

  ## Step4. Visualizing the MDS Plot (using R software)
  # copy-paste this code in R shell
  png("mds1-strat.png");
  p <- read.table("mds1.mds", header=T);
  plot( p$C1 , p$C2 , pch=20 , cex=2 , col=p$SOL+1);
  dev.off();
  *input: mds1.mds*
  *output: mds1-start.png*

**Summaries & questions for PCoA in PLINK**
----------------------------------------------
1. In my datasets, the PLINK-ibs clustering do not have *ibs.hh* output file, either of *.cluster0* . **Why? What's wrong here?**

2. I'm not quite sure about the results of PCA in R i.e. *mds1-strat.png* , in the reference example, *{Chinese cluster at the left, Japanese cluster at the right}* , where do they get this information? 

3. In my reference paper *Brazilian peach* they used *cmdscale* which requires *dist* file as input. Since I failed in *--distance-matrix* so either of this I couldn't get the result. 

============================================================
20200210_DAPC-adegenet
============================================================
**'dapc'** -- see the test.R script.
::
  
  library(adegenet)
  library(ade4)
  library(vcfR)
  ## Step1. get input file [vcfR2genind]
  vcf <- read.vcfR('clean6.vcf')
  dip <- vcfR2genind(vcf)

  ./plink --file SNPCalling.plk --allow-extra-chr --mind 0.75 --maf 0.01 --geno 0 --recode vcf-iid --out Nomiss



https://grunwaldlab.github.io/Population_Genetics_in_R/gbs_analysis.html

============================================================
20200217_Summaries of DIPLOID-GBS analsis
============================================================
Questions summaries
----------------------------

1. The input and output files;

2. The meaning of parameters, how to choose and what's the difference;

3. How to search for the tutorials, what's the worthy questions to be get deeper and be spending more time?

4. The understanding of the resutls, the graphs, the plots and how to make the adjustment based on the results?

Structure summaries
----------------------------

1. SNP Calling: TASSEL-GBS;

2. SNP filtering: PLINK v1.9;

3. Check out the data: TASSEL-GUI;

4. Population genetics analyses: fastStructure; PCA/PCoA/DAPC (R-adegenet);

5. Tutorials:
  **Paper:** Genome-wide SNP discovery through GBS of Brazilian peach breeding germplasm `Liane et al., 2020 <https://link.springer.com/article/10.1007/s11295-019-1406-x>`_ ;
  
  **Tutorial:** `GBS analysis <https://grunwaldlab.github.io/Population_Genetics_in_R/gbs_analysis.html#exercises>`_

============================================================
20200220_F-statistics
============================================================
Data-management
------------------------------

Fst should be at least two individuals per population, so I deleted those only have one individuals. And use *--extract* flag in PLINK v1.9 to make the sub-individuals:

*Input:*
  
  1. LessMissing dataset(taxa/MAF/site: 75/0.01/5) - plink format: LessMissing.ped/map

  2. NoMissing dataset(taxa/MAF/site: 75/0.01/0) - plink format: NoMissing.ped/map
  
  3.extract.txt: two columns -- FID(Family ID); IID(Individual ID); Here, my FID are all -9(missing), can check use *head xxx.ped*

*Code:*
::

  ./plink --allow-extra-chr --file LessMissing --keep extract.txt --make-bed --out LMFST
  ./plink --allow-extra-chr --file NoMissing --keep extract.txt --make-bed --out NMFST

*Output:*

  1. LMFST.bed/bim/fam

  2. NMFST.bed/bim/fam

FST analysis
--------------------------------

*Input:*

  1. LMFST/NMFST.bed/bim/fam

  2. cluster file: with *--within* flag: three columns-- FID(Family ID); IID(Individual ID); Cluster. Cluster used four different types, *x1, x2, x3* from LessMissing dataset; *x5* from NoMissing datasets. Not a big difference

*code:*

::

  ./plink --allow-extra-chr --bfile LMFST --fst --out x1-k4 --within x1-k4.txt
  ./plink --allow-extra-chr --bfile LMFST --fst --out x2-k2 --within x2-k2.txt
  ./plink --allow-extra-chr --bfile LMFST --fst --out x3-k4 --within x3-k4.txt
  ./plink --allow-extra-chr --bfile NMFST --fst --out x5-k3 --within x5-k3.txt

*Output:*

  1. xxx.log/fst/nosex

Summaries
--------------------------------
1. Not a big difference among different sets -- mean Fst around 22%; weighted Fst around 20%;

2. Should check more about what's the difference between those two; also, the other types of *Fst* -- among populations not only the average of all the SNPs?? --Not quite sure about this part.

3. Also tried *vcfTools* but didn't work, something wrong with my C++ system. It might just something wrong with system, check out more tomorrow. 
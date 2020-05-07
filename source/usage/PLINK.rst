PLINK Recipe
=================================================

Webpages: 
'PLINK <http://zzz.bwh.harvard.edu/plink/>'_
'PLINK v1.9home <https://www.cog-genomics.org/plink/1.9/>'_
'PLINK v1.9GoogleGroup <https://groups.google.com/d/forum/plink2-users>'_

=================================================
Summary
=================================================
PLINK is a free, open-source whole genome association analysis toolset, designed to perform a range of basic, large-scale analyses in a computationally efficient manner.

The focus of PLINK is purely on analysis of genotype/phenotype data, so there is no support for steps prior to this (e.g. study design and planning, generating genotype or CNV calls from raw data). Through integration with gPLINK and Haploview, there is some support for the subsequent visualization, annotation and storage of results.

PLINK (one syllable) is being developed by Shaun Purcell whilst at the Center for Human Genetic Research (CHGR), Massachusetts General Hospital (MGH), and the Broad Institute of Harvard & MIT, with the support of others.  

**Good points** it contains the log including all the flags we used in the calculation and the time, the environment, etc. including all the information we needed. 
 
=================================================
How it works
=================================================
In the download zip files, it contains several files:

1. the main PLINK 1.9 binary;
2. the GPLv3 license;
3. the prettify utility for generating clean space-delimited text tables;
4. the small files toy.ped and toy.map (example input files)

**PILOT TRY** 

in MAC:
::

  ./plink --file toy --freq --out toy_analysis

or in Windows:
::
  
  .\plink.exe --file toy --freq --out toy_analysis

Explanation:

1. this software contains some flags, like *--file*, *--freq* and *--out* -- just like the plugins in TASSEL; -- there are a lot of flags and in the documents are pretty clear.

2. the first flag **--file** is the input file, the second flag means calculations to perform; the last one is the output file prefix;

3. input file format: 

including *.ped* file: including family ID, individual ID, paternal ID, sex and phenotype -- 6 columns info. 

and, *.map* file: including 4 columns: chromosome, rs or snp identifier, genetic distance, base-pair position. 

-- this can be changed according to different needs. **I shall keep this in mind and so far I can use the default.**

4. There are a lot of flags and input and output files format should be references in the webpage; **TIPS for logs:** we can use *--out* to name the log file, thus it will be easier to trace back;

5. Rerun is useful for those little change -- *e.g.: ./plink --rerun run1.log --maf 0.1 --out run2* ; 

6. we can also use the script file: *./plink --script myscript1.txt* ; make all the commands we need in the script.

What I need here
-----------------

1. summary statistics: deviations from HWE (Hardy-Weinberg equilibrium) and MAF for each SNP;

2. observed heterozygosity (Ho), expected heterozygosity (He),  and inbreeding coefficient (F) for each genotype;

3. principal coordinate analysis (PCoA);

4. LD-based SNP pruning -- also prepare for the fastSTRUCTURE.

=================================================
Q & A 
=================================================
1. What's the meaning by testing HWE?

=================================================
Introduction and meaning for me
=================================================

Data management
-------------------------------------------------


=================================================
Population stratification
=================================================





















PilotRun
=================================================

=================================================
20191113_Step1: GBSSeqToTagDBPlugin
=================================================

Wiki website: `GBSSeqToTagDBPlugin <https://bitbucket.org/tasseladmin/tassel-5-source/wiki/Tassel5GBSv2Pipeline/GBSSeqToTagDBPlugin>`_

**Required parameters:**
::
  
  -db  ## <Output Database file>: Output Database File
  -e  ## <Enzyme> : Enzyme used to create the GBS library if it differs from the one listed in the key file.
  -i  ## <Input Directory>: Input directory containing FASTQ files in text or gzipped text. NOTE: Directory will be searched recursively and should be written WITHOUT a slash after its name.
  -k  ## <Key File> : Key file listing barcodes distinguishing the samples.
 

My code -- first try:
::
  ./run_pipeline.pl -GBSSeqToTagDBPlugin -db /localdisk/home/s1950737/Diploid/GBSV2.db -e ApeKI -i /disk2/Twyford_GBS_illumina -k /localdisk/home/s1950737/Diploid/DipKey.txt 

Error info:
:: 
  
  ERROR net.maizegenetics.plugindef.ThreadedPluginListener - Out of Memory: GBSSeqToTagDBPlugin could not complete task:
  Current Max Heap Size: 1531 Mb
  Use -Xmx option in start_tassel.pl or start_tassel.bat to increase heap size. Included with tassel standalone zip.

My code -- second try:
::
  ./run_pipeline.pl -Xms100G -Xmx200G -GBSSeqToTagDBPlugin -db /localdisk/home/s1950737/Diploid/GBSV2.db -e ApeKI -i /disk2/Twyford_GBS_illumina -k /localdisk/home/s1950737/Diploid/DipKey.txt 

*Time consumed: Start time: 17:00 15/11/19; screen -r diploid2*

=================================================
20191118_Step2: TagExportToFastqPlugin
=================================================
**Parameters required:**
::

  -c ## <Min Count>: Minimum count of reads for a tag to be output (Default: 1) This is a total of X count across all taxa, not in each individual taxon.
  -db ## <Input DB> : Input database file with tags and taxa distribution (REQUIRED)
  -o ## <Output File> : Output fastq file to use as input for BWA or Bowtie2 (REQUIRED)

My code (screen -r diploid2): 

::

  ./run_pipeline.pl -fork1 -TagExportToFastqPlugin -db /localdisk/home/s1950737/Diploid/GBSV2.db -o /localdisk/home/s1950737/Diploid/tagsForAlign.fa.gz -c 1 -endPlugin -runfork1

*Time consumed: 10 minutes*


=================================================
20191118_Step3: Alingment
=================================================

**Bowtie or BWA?** -- BWA

  ' *Bowtie 2* is an ultrafast and memory-efficient tool for aligning sequencing reads to long reference sequences. It is particularly good at aligning reads of about 50 up to 100s or 1,000s of characters, and particularly good at aligning to relatively long (e.g. mammalian) genomes. Bowtie 2 indexes the genome with an FM Index to keep its memory footprint small: for the human genome, its memory footprint is typically around 3.2 GB. Bowtie 2 supports gapped, local, and paired-end alignment modes.'

  ' *BWA* is a software package for mapping low-divergent sequences against a large reference genome, such as the human genome. It consists of three algorithms: BWA-backtrack, BWA-SW and BWA-MEM. The first algorithm is designed for Illumina sequence reads up to 100bp, while the rest two for longer sequences ranged from 70bp to 1Mbp. BWA-MEM and BWA-SW share similar features such as long-read support and split alignment, but BWA-MEM, which is the latest, is generally recommended for high-quality queries as it is faster and more accurate. BWA-MEM also has better performance than BWA-backtrack for 70-100bp Illumina reads.'

 
**BWA** (download and move to server by SSH copy)

*First step:running with BWA is to create an index from the reference genome* -- (screen -r diploid2)
::

  bwa index -a bwtsw disk2/Twyford_GBS_files/Diploid/anglica_HB050619_scaffoldsKeptAnno.fa

**Errors and debug:**
::

  ERROR -- bwa: command not found
  DEBUG -- git clone https://github.com/lh3/bwa.git
           cd bwa; make
           ./bwa index -a bwtsw disk2/Twyford_GBS_files/Diploid/anglica_HB050619_scaffoldsKeptAnno.fa

  Error -- [bwa_idx_build] fail to open file 'disk2/Twyford_GBS_files/Diploid/anglica_HB050619_scaffoldsKeptAnno.fa' : No such file or directory
  DEBUG -- ./bwa index -a bwtsw /disk2/Twyford_GBS_files/Diploid/anglica_HB050619_scaffoldsKeptAnno.fasta

  Error -- Pack FASTA... [bns_fasta2bntseq] fail to open file '/disk2/Twyford_GBS_files/Diploid/anglica_HB050619_scaffoldsKeptAnno.fasta.pac' : Permission denied
  DEBUG -- sudo ./bwa index -a bwtsw /disk2/Twyford_GBS_files/Diploid/anglica_HB050619_scaffoldsKeptAnno.fasta  

  Error -- s1950737 is not in the sudoers file.  This incident will be reported.
  DEBUG -- cp /disk2/Twyford_GBS_files/Diploid/anglica_HB050619_scaffoldsKeptAnno.fasta .
           ./bwa index -a bwtsw ../anglica_HB050619_scaffoldsKeptAnno.fasta

*Time consumed: 6 minutes*

*Second step:Alignment (2-1: The file created from the "bwa aln ..." call is a ".sai" file containing the suffix array coordinates of all short reads loaded to BWA.)* -- (screen -r diploid2)
::

  ./bwa aln -t 4 ../anglica_HB050619_scaffoldsKeptAnno.fasta ../Diploid/tagsForAlign.fa.gz > tagsForAlign.sai

Errors and debug:
::

  **ERROR** -- [bwa_aln] 17bp reads: max_diff = 2
  [bwa_aln] 38bp reads: max_diff = 3
  [bwa_aln] 64bp reads: max_diff = 4
  [bwa_aln] 93bp reads: max_diff = 5
  [bwa_aln] 124bp reads: max_diff = 6
  [bwa_aln] 157bp reads: max_diff = 7
  [bwa_aln] 190bp reads: max_diff = 8
  [bwa_aln] 225bp reads: max_diff = 9
  [bwa_aln] fail to locate the index
  **DEBUG** -- ./bwa aln -t 4 ../anglica_HB050619_scaffoldsKeptAnno.fasta ../Diploid/tagsForAlign.fa.gz > tagsForAlign.sai

*Time consumed: 17 seconds*

**Summary for this moment**

1. the output files in [bwa] will store in the same folder with reference genomes, and I put it into [/localdisk/home/s1950737] -- all the other outputs (including output of step one -- 
[anglica_HB050619
_scaffoldsKeptAnno.fasta.amb]; 
[anglica_HB050619
_scaffoldsKeptAnno.fasta.ann]; 
[anglica_HB050619
_scaffoldsKeptAnno.fasta.bwt]; 
[anglica_HB050619
_scaffoldsKeptAnno.fasta.pac] 
and output of step two -- 
[anglica_HB050619
_scaffoldsKeptAnno.fasta.sa]) 
are in this folder. 
So in this step, I moved them into the the new folder [referenceGenome] to make it clear and tidy.

*Second step: Alignment (2-2): There is one more conversion to obtain the .sam file.* -- (screen -r diploid2)
::

  ./bwa samse ../referenceGenome/anglica_HB050619_scaffoldsKeptAnno.fasta tagsForAlign.sai ../Diploid/tagsForAlign.fa.gz > tagsForAlign.sam

*Time consumed: 5.38 seconds*

=================================================
20191118_Step5: DiscoverySNPCallerPluginV2
=================================================
**Required parameters:**
::

  -db ## <Input GBS Database> : Input Database file if using SQLite (REQUIRED)
  
My code:
::

  ./run_pipeline.pl -fork1 -DiscoverySNPCallerPluginV2 -db ../Diploid/GBSV2.db -endPlugin -runfork1

*Time consumed: 3 seconds*

=================================================
20191118_ProductionSNPCallerPluginV2
=================================================
**Required parameters:**
::

  -db ## <Input GBS Database> : Input Database file if using SQLite (REQUIRED)
  -e ## <Enzyme> : Enzyme used to create the GBS library (REGQUIRED)
  -i ## <Input Directory> : Input directory containing fastq AND/OR qseq files (REQUIRED)
  -k ## <Key File> : Key file listing barcodes distinguishing the sample (REQUIRED)
  -o ## <Output Genotypes File> : Output (target) genotypes file to which is added new genotypes. VCF format is the default. if the file specified has suffix ".h5" output will be to an HDF5 file. (REQUIRED)

My code:
::

  ./run_pipeline.pl -fork1 -ProductionSNPCallerPluginV2 -db ../Diploid/GBSV2.db -e ApeKI -i /disk2/Twyford_GBS_illumina -k ../Diploid/DipKey.txt -o ../Diploid/productionHapMap_diploid.vcf -endPlugin -runfork1 

Errors and debug:
::

  ERROR -- net.maizegenetics.analysis.gbs.v2.ProductionSNPCallerPluginV2 - No snp positons found with quality score of 0.0. Please run UpdateSNPPositionQualityPlugin to add quality scores for your positions, then select snp positions within a quality range you have specified.
  DEBUG -- FORGOT ONE STEP!!!

=================================================
20191118_Step6: SNPQualityProfilerPlugin
=================================================
**Required parameters:**
::

  -db ## <Output Database file> : Name of output file (e.g. GBSv2.db) (REQUIRED)
  
what's this? I do not really understand...

=================================================
20191120_Step4: SAMToGBSdbPlugin
=================================================
**Summary for this moment**
1. I confused some procedure before, skipping the step 4(i.e. SAMToGBSdbPlugin), so I need to go back this step. Before, I need to clean up some output files, they located in different folders confused me a lot. First, let's link the steps with outfiles and input files, here we go:
2. Step1 (i.e. GBSSeqToTagDBPlugin): FASTAQ files(floder); Key File ==>> GBSV2.db ( *Diploid* )
   Step2 (i.e. TagExportToFastqPlugin): GBSV2.db ( *Diploid* ) ==>> tagsForAlign.fa.gz ( *Diploid* )
   Step3 (i.e. BWA alingment): 
     Step3-1: reference genome ( *referenceGenome* ) ==>> several index files -- amb, bwt, pac, ann, sa ( *referenceGenome* )
     Step3-2: reference genome ( *referenceGenome* ); tagsForAlign.fa.gz ( *Diploid* ) ==>> tagsForAlign.sai ( *bwa* )
     Step3-3: reference genome ( *referenceGenome* ); tagsForAlign.fa.gz ( *Diploid* ); tagsForAlign.sai ( *bwa* ) ==>> tagsForAlign.sam ( *bwa* ) 

**Here, I should also typed the path for tagsForAlign.sai and tagsForAlign.sam to Diploid. Then continue step4:** 
   
   Step4 (i.e. SAMToGBSdbPlugin): tagsForAlign.sam ( *Diploid* ) ==>> GBSV2.db ( *Diploid* ) -- rewrite
   Step5 (DiscoverySNPCallerPluginV2): 


**Required parameters:**
::

  -i ## <SAM Input file> (REQUIRED)
  -db ## <GBS DB file> (REQUIRED)

My code (screen -r diploid2)
::

  ./run_pipeline.pl -SAMToGBSdbPlugin -i ../Diploid/tagsForAlign.sam -db ../Diploid/GBSV2.db

*Time concumed: 1 minute*

=================================================
20191120_Step5: DiscoverySNPCallerPluginV2
=================================================
**Required parameters:**
::

  -db ## <Input GBS Database> : Input Database file if using SQLite (REQUIRED)
  
My code:
::

  ./run_pipeline.pl -DiscoverySNPCallerPluginV2 -db ../Diploid/GBSV2.db

*Time consumed: 14:27 20/11/19 - it seems endless, adjustment in the following* -- it is because connection reset, so do each step in screen not in the local terminal!!!!!

screen -r diploid2: recalculate from 11:07 21/11/19 - 18:36 21/11/19, 7.5h

=================================================
20191125_Step6: SNPQualityProfilerPlugin
=================================================
**Required parameters:**
::

  -db ## <Input GBS Database> : Input Database file if using SQLite (REQUIRED)
  
My code:
::

  ./run_pipeline.pl -SNPQualityProfilerPlugin -db ../Diploid/GBSV2.db

*Time consumed: instantly*

=================================================
20191125_Step7: UpdateSNPPositionQualityPlugin
=================================================
**Required parameters:**
::

  -db ## <Input GBS Database> : Input Database file if using SQLite (REQUIRED)
  -qsFile ## <Quality Score File> : A tab delimited txt file containing headers CHROM( String), POS (Integer) and QUALITYSCORE(Float) for filtering. (REQUIRED)
  
My code:
::

  ./run_pipeline.pl -UpdateSNPPositionQualityPlugin -db ../Diploid/GBSV2.db -qsFile (???what's this?)

*Time consumed: *

=================================================
20191123_ProductionPipeline ProductionSNPCallerPluginV2
=================================================
**Required parameters:**
::

  -db ## <Input GBS Database> : Input Database file if using SQLite (REQUIRED)
  -e ## <Enzyme> : Enzyme used to create the GBS library (REGQUIRED)
  -i ## <Input Directory> : Input directory containing fastq AND/OR qseq files (REQUIRED)
  -k ## <Key File> : Key file listing barcodes distinguishing the sample (REQUIRED)
  -o ## <Output Genotypes File> : Output (target) genotypes file to which is added new genotypes. VCF format is the default. if the file specified has suffix ".h5" output will be to an HDF5 file. (REQUIRED)

My code:
::

  ./run_pipeline.pl -ProductionSNPCallerPluginV2 -db ../Diploid/GBSV2.db -e ApeKI -i /disk2/Twyford_GBS_illumina -k ../Diploid/DipKey.txt -o ../Diploid/productionHapMap_diploid.vcf

*Time consumed: 14:37 23/11/2019 - 14:47 23/11/2019; 10 minutes.*

=================================================
Q & A
=================================================

1. output:
  Why there is a GBSV1.db in the first step (the outoput of GBSSeqToTagDBPlugin)?


Q2. In the following example:

::

   ./run_pipeline.pl -fork1 -TagExportToFastqPlugin -db /Users/lcj34/git/tassel-5-test/tempdir/GBS/Chr9_10-20000000/GBSv2.db -o /Users/lcj34/git/tassel-5-test/tempDir/GBS/Chr9_10-20000000/tagsForAlign.fa.gz -c 1 -endPlugin -runfork1

What's the fork1 and endPlugin for?

A2: In java, it supposed to support different forks running at the same time. There won't be anything wrong if without fork1. The same for the endPlugin.

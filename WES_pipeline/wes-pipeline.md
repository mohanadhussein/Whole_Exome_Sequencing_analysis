---
# yaml-language-server: $schema=schemas\page.schema.json
Object type:
    - Page
Backlinks:
    - Exome Analysis
Creation date: "2023-12-28T06:06:34Z"
Created by:
    - Mohanad Hussein
Links:
    - files\fastqc_manual.pdf
id: bafyreie6hogsbvvhqjabuickdotlpemiqs5jxtzftq4ftnl7sob7p362re
---
# WES Pipeline   
Exome analysis   
# IGV   
- Download IGV through ubuntu terminal using:   
   
```
wget https://data.broadinstitute.org/igv/projects/downloads/2.16/IGV_Linux_2.16.2_WithJava.zip 
```
   
# Raw data QC   
FastQ format considered the standard for storing NGS data as they store phred-quality scores of reads.   
Common tools for QC: FastQC FastQ screen, FastX-Toolkit and NGS QC Toolkit
check this super tool: [https://multiqc.info/](https://multiqc.info/)    
> The parameters for QC   

1. Sequence quality score distribution   
2. Read length distribution   
3. GC content distribution   
4. Sequence duplication level   
5. PCR amplification issue   
6. Biasing of k-mers   
7. Over-represented sequences   
   
> FastQC manual   

[FastQC_Manual](files\fastqc_manual.pdf)    
- Documentation and explanation of modules found on [https://www.bioinformatics.babraham.ac.uk/projects/fastqc/](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/)    
- [https://people.duke.edu/~ccc14/duke-hts-2018/bioinformatics/quality\_scores.html#Phred+33](https://people.duke.edu/~ccc14/duke-hts-2018/bioinformatics/quality_scores.html#Phred+33)    
   
```
fastqc TEX162_299_278_S2_L004_R1_001.fastq
fastqc TEX162_299_278_S2_L004_R2_001.fastq
#the output files are named automatically
#a better practice would be calling the GUI and using it for analysis, but this is not feasible by convention in WSL, the graphical display env needs config first
```
# Trimming   
> Available tools   

- skerwe: looks for adaptors only, no quality trimming for paired ends   
- Btrim: a bit old but fine   
- trimmomatic: illumina only!   
- fastp: faster than trimmomatic and more space efficient cus written in C++   
   
> https://www.youtube.com/watch?v=nOJ5RGkKwnQ  good resource   

> Logic behind trimming   

the part that binds the DNA to the flow cell is the adapter, it is followed by sample-specific index that differentiate replicates. Trimming aims to remove low quality sequeces and adapters used in illumina sequencing platform (or any other platform)   
> IlluminaClip:   

- **2:** This is the number of mismatches allowed in the seed region (first 16 bases) of the adapter sequence before discarding a potential match. A lower value increases stringency.   
- **30:** This is the minimum adapter match length required for trimming. Shorter matches are ignored.   
- **10:** This is the maximum allowed quality score within the adapter region. Bases above this score are considered part of the adapter and trimmed.   
- **2:** This is the minimum required base quality after trimming. Bases below this score are also trimmed.   
   
> Leading:   

3: This parameter defines the leading quality filtering. Bases with a quality score below 3 at the beginning of the reads are trimmed.   
> Trailing:    

3:This parameter defines the trailing quality filtering. Bases with a quality score below 3 at the end of the reads are trimmed.   
> Minlen:   

36: This specifies the minimum allowed read length after trimming. Reads shorter than 36 bases are discarded.   
- [https://bookdown.org/ggiaever/wrangling-genomics/trimming-and-filtering.html](https://bookdown.org/ggiaever/wrangling-genomics/trimming-and-filtering.html)    
   
```
trimmomatic PE -threads 4 TEX162_299_278_S2_L004_R1_001.fastq.gz \ TEX162_299_278_S2_L004_R2_001.fastq.gz \ #second input
TEX162_299_278_S2_L004_R1_trimmed.fastq.gz \ #first output 
TEX162_299_278_S2_L004_R1_trimmed_orphan.fastq.gz \ #first output
TEX162_299_278_S2_L004_R2_trimmed.fastq.gz \ #second output
TEX162_299_278_S2_L004_R2_trimmed_orphan.fastq.gz \ #second output
ILLUMINACLIP:Nextera.fasta:2:30:10:2:True LEADING:3 TRAILING:3 MINLEN:36

```
# Alignment & post-process   
- first decompress the trimmed fasta files if they end with gzip   
   
```
gzip -dk file_name.gz 
```
- download the reference genome from [https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/000/001/405/GCA\_000001405.15\_GRCh38/seqs\_for*alignmentpipelines.ucscids/*](https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/000/001/405/GCA_000001405.15_GRCh38/seqs_for_alignment_pipelines.ucsc_ids/)* 
Download the following files: fna.bwa\_index.tar.gz, fna.fai,fna.gz
*see ref builds:* *[https://gatk.broadinstitute.org/hc/en-us/articles/360035890951-Human-genome-reference-builds-GRCh38-or-hg38-b37-hg19](https://gatk.broadinstitute.org/hc/en-us/articles/360035890951-Human-genome-reference-builds-GRCh38-or-hg38-b37-hg19)    
   
> useful reading: https://lh3.github.io/2017/11/13/which-human-reference-genome-to-use    
> https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/000/001/405/GCA_000001405.15_GRCh38/seqs_for_alignment_pipelines.ucsc_ids/README_analysis_sets.txt    
>    

```
#for files that are not downloaded when pressing over them use the following
wget file_link_copied_from_browser
```
- run the alignment using bwa   
   
```
#note: the indexed files must be compatible with alignment tool used, otherwrise you have to perform ref_genome indexing yourself.
#note: reference genome files dowonloaded are named in a specific way that allows calling interfile communcication, do not change names, call the ref genome as illustrated here
bwa mem -t 8 reference_genome.fna R1_file.fq.gz R2_file.fq.gz > output.sam
```
- conversion of alignment result (i.e SAM) into BAM   
   
```
samtools view -b TEX162_MH.sam > TEX162_MH.bam

```
- sorting by name for fixmate (filling read attributes)   
   
```
samtools sort -n -O bam -o output_file.bam input_file.sam

#see: http://samtools.sourceforge.net/samtools.shtml , http://samtools.sourceforge.net/SAM1.pdf to understand

samtools fixmate -@ 10 -m nsorted.sam fixmate.sam

```
- resorting by position for markdup   
   
```
samtools sort -o $dir/positionsorted.sam $dir/fixmate.sam

#see https://www.biostars.org/p/107402/ to understand dupicates
samtools markdup -r fixmate.sam $dir/markdup.sam

```
- to remove duplicate, fixmating (i.e )   
- indexing Bam files   
   
```
#to view bam in IGV you need to use the sorted bam file and its indexed version
samtools index file_name.sorted.bam
```
- quality control for mapping   
   
```
samtools flagstat file_name.sorted.bam > file_name.txt
#for more details see https://www.htslib.org/doc/samtools-flagstat.html 
```
## Trick & note
   
```
#you can combine the steps of converting sam>bam and sorting by using the pipe operator
bwa mem -t 8 reference_genome.fna R1_file.fq.gz R2_file.fq.gz |samtools sort -o output_file.sorted.bam

#after aligning and converting samt>bam, sam files can be omitted for ever and only bam can be used for spece-iffeciency
```
# Variant calling-GATK
   
> To understand the reference genome known variants: see https://www.ncbi.nlm.nih.gov/variation/docs/glossary/    
> and https://www.ncbi.nlm.nih.gov/variation/docs/human_variation_vcf/    

- gatk requires a reference genome in a particular format described at [https://gatk.broadinstitute.org/hc/en-us/articles/360035531652-FASTA-Reference-genome-format](https://gatk.broadinstitute.org/hc/en-us/articles/360035531652-FASTA-Reference-genome-format)    
- I am using the common\_all\_20180418.vcf known variant list from here: [https://ftp.ncbi.nlm.nih.gov/snp/organisms/human\_9606/VCF/GATK/](https://ftp.ncbi.nlm.nih.gov/snp/organisms/human_9606/VCF/GATK/)    
   
```
#create the dict and fai files, be carefull that the following indexing will overwrite the index file you already have for your genome because both commands name output file automatically.
gatk CreateSequenceDictionary -R ref.fna

#no need to return to samtools_env, gatk has it integrated
samtools faidx ref.fna
```
- Indel Realignment   
   
```
#usually indel realignment is done but web sources say that they are not necessary anymore if you use a variant caller that performs haplotype realignment like "haplotypecaller"
see:
https://github.com/broadinstitute/gatk-docs/blob/master/gatk3-tutorials/(howto)_Perform_local_realignment_around_indels.md
```
- Base Recalibration   
   
```
#GATK assumes read groupo in the alignment bam file, if no read groups exists, add at least 1 using picard (which is actually integrated to gatk), see:
#http://broadinstitute.github.io/picard/command-line-overview.html#AddOrReplaceReadGroups, https://www.biostars.org/p/43897/#43918 

gatk AddOrReplaceReadGroups I=TEX162_MH.fixcsortdup.bam O=TEX162_MH.fixcsortdupRG.bam RGID=1 RGLB=lib1 RGPL=illumina RGPU=unit1 RGSM=1

# Using GATK jar /home/mh/gatk-4.4.0.0/gatk-package-4.4.0.0-local.jar

#Recalibrate base quality bu using a benchmark dataset (i.e known var list)
gatk BaseRecalibrator -I TEX162_MH.fixcsortdupRG.bam -R /mnt/e/Exome_data_MH/Reference\ genome/GRCh38/GCA_000001405.15_GRCh38_no_alt_plus_hs38d1_analysis_set.fna --known-sites common_all_20180418.vcf.gz -O TEX162_MH_recal.table

#Applying recabalibration according to table prodiced in prev step, see: https://gatk.broadinstitute.org/hc/en-us/articles/360037055712-ApplyBQSR 
gatk ApplyBQSR -R /mnt/e/Exome_data_MH/Reference\ genome/GRCh38/GCA_000001405.15_GRCh38_no_alt_plus_hs38d1_analysis_set.fna -I TEX162_MH.fixcsortdupRG.bam -bqsr TEX162_MH_recal.table -O TEX162_MH.fixcsortdupRG.recal.bam

gatk HaplotypeCaller  -R /mnt/e/Exome_data_MH/Reference\ genome/GRCh38/GCA_000001405.15_GRCh38_no_alt_plus_hs38d1_analysis_set.fna -D common_all_20180418.vcf.gz --native-pair-hmm-threads 10  -I TEX162_MH.fixcsortdupRG.recal.bam -O TEX162_MH.recalSNPs.vcf




```
- Variant Recalibration (valid for multiple WES only)
   
   
```
#documentation recommends annotating vcf with VariantAnnotator, but haplotypecaller produces the descent version of annotation required

#for recalibration, variant datasets can be found in: https://console.cloud.google.com/storage/browser/genomics-public-data/resources/broad/hg38/v0;tab=objects?prefix=&forceOnObjectsSortingFiltering=false 

#downloading these datasets from the browser seems to be problematic, so download them using wget instead.

#check the header of any dataset using bcftools head <file_name.vcf/gz>, transfer VCFs to gz

bcftools view resources_broad_hg38_v0_Homo_sapiens_assembly38.dbsnp138.vcf -Oz -o resources_broad_hg38_v0_Homo_sapiens_assembly38.dbsnp138.vcf.gz

#indexing dbSNP, GATK generally uses CSI but here only TBI is working
bcftools index -t resources_broad_hg38_v0_Homo_sapiens_assembly38.dbsnp138.vcf.gz

gatk VariantRecalibrator -R /mnt/e/Exome_data_MH/Reference\ genome/GRCh38/GCA_000001405.15_GRCh38_no_alt_plus_hs38d1_analysis_set.fna -V TEX162_MH.recalSNPs.vcf --resource:1000G,known=false,training=true,truth=true,prior=10.0 /mnt/e/Exome_data_MH/Reference\ genome/1000G_phase1.snps.high_confidence.hg38.vcf.gz --resource:dbsnp,known=true,training=false,truth=false,prior=2.0 /mnt/e/Exome_data_MH/Reference\ genome/resources_broad_hg38_v0_Homo_sapiens_assembly38.dbsnp138.vcf.gz -an MQ -mode SNP -tranche 100.0 -tranche 99.9 -tranche 99.0 -tranche 90.0 -O TEX162_MH.VarRecal.recal --tranches-file TEX162_MH.VarRecal.tranches --rscript-file TEX162_MH.VarRecal.plot.R

gatk ApplyVQSR -R /mnt/e/Exome_data_MH/Reference\ genome/GRCh38/GCA_000001405.15_GRCh38_no_alt_plus_hs38d1_analysis_set.fna -V TEX162_MH.recalSNPs.vcf -O TEX162_MH.ApplyVqsr.vcf.gz --truth-sensitivity-filter-level 99.0 --tranches-file TEX162_MH.VarRecal.tranches --recal-file TEX162_MH.VarRecal.recal -mode SNP

```
- Single WES VR command    
   
```
gatk VariantRecalibrator -R /mnt/e/Exome_data_MH/Reference\ genome/Variant_Recalibration/Homo_sapiens_assembly38.fasta -tranche 100.0 -tranche 99.95 -tranche 99.9 -tranche 99.5 -tranche 99.0 -tranche 97.0 -tranche 96.0 -tranche 95.0 -tranche 94.0 -tranche 93.5 -tranche 93.0 -tranche 92.0 -tranche 91.0 -tranche 90.0 -V TEX162_MH.recalSNPs.vcf --resource:hapmap,known=false,training=true,truth=true,prior=15.0 /mnt/e/Exome_data_MH/Reference\ genome/Variant_Recalibration/hapmap_3.3.hg38.vcf.gz --resource:1000G,known=false,training=true,truth=false,prior=10.0 /mnt/e/Exome_data_MH/Reference\ genome/Variant_Recalibration/1000G_omni2.5.hg38.vcf.gz --resource:1000G,known=false,training=true,truth=false,prior=10.0 /mnt/e/Exome_data_MH/Reference\ genome/Variant_Recalibration/1000G_phase1.snps.high_confidence.hg38.vcf.gz -an QD -an MQ -an MQRankSum -an ReadPosRankSum -an FS -an SOR -mode SNP -O TEX162_MH.VarRecal.recal --tranches-file TEX162_MH.VarRecal.tranches

gatk ApplyVQSR -R /mnt/e/Exome_data_MH/Reference\ genome/Variant_Recalibration/Homo_sapiens_assembly38.fasta -V TEX162_MH.recalSNPs.vcf -O TEX162_MH.ApplyrecalSNPs.vcf --truth-sensitivity-filter-level 97.5 --tranches-file TEX162_MH.VarRecal.tranches --recal-file TEX162_MH.VarRecal.recal -mode SNP

```

---
# yaml-language-server: $schema=schemas\page.schema.json
Object type:
    - Page
Backlinks:
    - Exome Analysis
Creation date: "2023-12-12T14:47:34Z"
Created by:
    - Mohanad Hussein
Links:
    - https-annovar-openbioinformatics-org-en-latest.md
Emoji: ⚙️
id: bafyreigt4seq7uc4n77d7zfpapnoqyisscieqgu5ntlv4lv5tpxa6bpfr4
---
# Installations & configurations   
Codes and commands used in ubuntu to analyze the exome data for TEX162 and TEX163    
- Since samtools work in linux, you must use the ubuntu cmd through WSL and mount to your local data file.    
- A good practice to work in conda environments inside ubuntu. I did that during the Bioinformatics course from future learn. The same steps can be used over and over again with optimizing for the operating system and enviorment/package versions. See documentation on Linux.    
   
```
#mounting to the working directory
cd /mnt
#activating previously created conda env
conda env list
conda activate samtools_env
#checking samtools version
samtools version
#coverage
```
- The tool required for FastQ>BAM is bwa-mem2 (see documentation:[https://github.com/bwa-mem2/bwa-mem2](https://github.com/bwa-mem2/bwa-mem2) ) but you can download if from conda: [https://anaconda.org/bioconda/bwa-mem2](https://anaconda.org/bioconda/bwa-mem2)  which is way more convinient.     
   
# ANNOVAR   
- WANNOVAR (the web version of ANNOVAR) cannot handle WGS files due to size limitation, so using the CLI tool is mandatory   
- ANNOVAR is written in PERL and needs installing PERL if not already installed, most linux distributions come with PERL installed with the required modules   
- to install ANNOVAR, go to the [website ](https-annovar-openbioinformatics-org-en-latest.md)and register to receive the source link   
   
### Installation and Setup   
```
# check if perl and its modules is installed in the system (if not, use conda)
perl -v
perl -MLWP::Simple -e 1 

# install the annovar source
wget http://www.openbioinformatics.org/annovar/download/0wgxR2rIVP/annovar.latest.tar.gz

# decompress it
tar -zxvf annovar.latest.tar.gz

```
### Installing Databases   
- Unlike WANNOVAR, ANNOVAR is not online and thus requires installing the databases -from which you will pull the variants- locally. This requires a considerable amount of space (e.g gnomad211\_genome is 30 GB)   
   
```
# these commands install the additional databases
# update the list as needed

perl annotate_variation.pl -buildver hg19 -downdb -webfrom annovar refGeneWithVer humandb/ # refGeneWithVer specifies the version of refgen NM nomenclature (e.g NM_123.5)
perl annotate_variation.pl -buildver hg19 -downdb -webfrom annovar gnomad211_genome humandb/ # gnomAD v2.1.1 genomes
perl annotate_variation.pl -buildver hg19 -downdb -webfrom annovar clinvar_20250721 humandb/
perl annotate_variation.pl -buildver hg19 -downdb -webfrom annovar 1000g2015aug humandb/
perl annotate_variation.pl -buildver hg19 -downdb -webfrom annovar avsnp150 humandb/
perl annotate_variation.pl -buildver hg19 -downdb -webfrom annovar dbnsfp47a humandb/ # functional annotation database
```
### Running ANNOVAR   
- if the input is vcf, which is usually the case, but unfortunately not the default in ANNOVAR, specifying `-vcfinput` automatically triggers running the `convert2annovar.pl` script that creates an `.avinput` file that is used internally as input instead of VCF.   
- It is not possible -at least atm- to generate a csv directly from VCF, either convert it first using `convert2annovar.pl ` or use the generated .avinput from running the following script then rerun the script with changing input to .avinput and replacing the `-vcfinput` flag with `-csvout` (note: both can't be used together)   
   
```
# running annovar to annotate variants in a VCF file 

input_vcf="/input.vcf"
output_prefix="/output

perl table_annovar.pl "$input_vcf" \
	humandb/ \
	-buildver hg19 \
	-out "$output_prefix" \
	-protocol refGeneWithVer,gnomad211_genome,1000g2015aug_all,clinvar_20131105,clinvar_20250721,avsnp150,dbnsfp47a \
	-operation g,f,f,f,f,f,f \
	-remove -vcfinput -nastring . -polish 
```
   

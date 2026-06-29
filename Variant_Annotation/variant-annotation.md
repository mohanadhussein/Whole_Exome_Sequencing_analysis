---
# yaml-language-server: $schema=schemas\page.schema.json
Object type:
    - Page
Backlinks:
    - Exome Analysis
Creation date: "2025-10-10T12:52:48Z"
Created by:
    - Mohanad Hussein
Emoji: "\U0001F4D5"
id: bafyreihkmaitefrxpqc6ihfxtq6av5lsa4fymd4g3kn36s6frzdncsu4ze
---
# Variant Annotation   
# ANNOVAR   
> Doc: https://annovar.openbioinformatics.org/en/latest/    

AnnovarFAQ: [https://annovar.openbioinformatics.org/en/latest/misc/faq/](https://annovar.openbioinformatics.org/en/latest/misc/faq/)    
- since the web version doesn't accept large exome VCF files or WGS VCF files, running the CLI ANNOVAR is necessary   
- since ANNOVAR is a locally installed tool, we need to install the databases locally as well. This requires a considerably large memory for both storage and running   
- ANNOVAR documentation includes a lot of details, some of which might be confusing so follow this for simplicity   
   
### Installing Databases   
- Unlike WANNOVAR, ANNOVAR is not online and thus requires installing the databases -from which you will pull the variants- locally. This requires a considerable amount of space (e.g gnomad211\_genome is 30 GB)   
- Selection of databases depends on your interest   
   
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
- if the input is vcf, which is usually the case, but unfortunately not the default in ANNOVAR, specifying `-vcfinput` automatically triggers running the `convert2annovar.pl` script that creates an `.avinput` file that is used internally as input instead of VCF (even though it is literally same as vcf but no HEADER).   
- It is not possible -at least atm- to generate a CSV directly from VCF, either convert it first using `convert2annovar.pl ` or use the generated `.avinput `from running the following script then rerun the script with changing input to `.avinput` and replacing the `-vcfinput` flag with `-csvout` (note: both can't be used together)   
   
```
# running annovar to annotate variants in a single VCF file 

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
### Optimized ANNOVAR running   
- With large files VCF files and large databases, it is not ideal to run everything in one run, but rather splitting the task into small pieces   
- We can split the VCF into chromosome\_wise.vcf, then running annotation either directly on the VCF(which will lead to having nested loops), or running conversion (VCF→`.avinput`) first then annotating `.avinput` directly. Here, i follow the second approach to keep the code clear   
- First, we split the vcf by chromosomes (note: requires bcftools):   
   
```

# splitting a VCF by chromomosome to handle large files 
vcf_file="file_name_.vcf.gz"
for i in {1..22} X Y MT; do
    bcftools view -r $i -Oz -o file_name_${i}.vcf.gz $vcf_file
done

```
- second, we convert each VCF\_chr to ANNOVAR specific input format `.avinput` :   
   
```
working_dir="vcf_dir"
output_dir="output_dir"
for i in {1..22} X Y MT; do
    vcf_chr="${working_dir}file_name_${i}.vcf.gz"
    annovar_chr="${output_dir}file_name_${i}.avinput"
    convert2annovar.pl -includeinfo -withfreq -format vcf4 "$vcf_chr" > "$annovar_chr"
done
```
- then we run annotation for each ANNOVAR file (note: this may last for a long time):   
   
```
for i in {1..22} X Y MT; do
    input_annovar="${output_dir}file_name_${i}.avinput"
    
    # running annotation with avinput file per chromosome
    perl table_annovar.pl "$input_annovar" humandb/ -buildver hg19 -out "$input_annovar" -remove -protocol refGeneWithVer,gnomad211_genome,1000g2015aug_all,clinvar_20250721,avsnp150,dbnsfp47a -operation g,f,f,f,f,f -nastring . -csvout -polish
done

```
   
   

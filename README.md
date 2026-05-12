# Streptomyces-sialometabolism

##Dowload genomes
```
datasets download genome taxon "Streptomyces" --reference --include protein

#unzip file
unzip ncbi_dataset.zip

#list filenames
cd ncbi_dataset/data
ls -d  GCF* > GCF_Streptomyces_ID
mv GCF_Streptomyces_ID ../../ & cd ../../

#count file
wc -l GCF_Streptomyces_ID
```
## rename files based on directories and fasta header
```
bash rename_Strepto_files.sh
ls ncbi_dataset/data/GCF*/ #check if everything is ok

#rename fasta header

for file in GCF*/*faa; do mv $file ../; done #move files
for f in *.faa; do sed -i "s/^>/>${f}_/" "$f"; done
```

## Get report of genomes
```
datasets summary genome accession --inputfile GCF_Streptomyces_ID --as-json-lines | dataformat tsv genome > Streptomyces_GCF_summary

#get unique rows and remove duplicates
awk -F'\t' '!seen[$1]++' Streptomyces_GCF_summary > Streptomyces_GCF_summary_unique
```

# Creation of HMM models

fasta files were retrieved from this [link](https://zenodo.org/records/20030591)


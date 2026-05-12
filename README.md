# Streptomyces-sialometabolism

#Dowload genomes
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

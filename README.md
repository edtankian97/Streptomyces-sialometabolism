# Streptomyces-sialometabolism

## Tools to download
```
conda install bioconda::mafft==7.525 #mafft v. 7.525
conda install bioconda::hmmer==3.4 #hmmer v. 3.4
conda install bioconda::seqkit==2.13.0 #seqkit
```
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
Download **final_fastas.tar.gz** file and move to **Protein_database** folder
```
#Descompress file
tar -xf final_fastas.tar.gz
cd final_fastas/
```
Use the fasta files inside **FASTAS_without_extra** folder
```
cd FASTAS_without_extra
ls FASTAS_without_extra
#copy files of Sia biosynthesis pathways
cp neuC_curado.fasta neuB_curado.fasta ../../

#copy files of Sia catabolism pathways
cp nanH_curado.fasta nanT_curado.fasta nanA_curado.fasta nanK_curado.fasta nanE_curado.fasta nagA_curado.fasta nagB_curado.fasta ../../
```
Step of CD-HIT was alrealdy done, so we can skip it
Step of alignment with mafft
```
bash mafft_align.sh
```
Alignment outputs are located in
```
Protein_database/MAFFT_alignment
```
## HMM model creation
```
bash hmm_models_create.sh
```
HMM models are located in:
```
Protein_database/HMM_models
```

## HMM annotation
```
bash hmm_annotation_script.sh
```

HMM annotation results are located in:
```
# output files
Protein_database/HMM_RESULT/output_hmm_Strepto

#coverage results
Protein_database/HMM_RESULT/cover_hmm_Strepto
```

#Filtration of coverage, bit-score and e-value

### Coverage
```
#Join coverage result
cd ../HMM_RESULT/cover_hmm_Strepto
find ./ -type f -name '*coverage' -exec cat {} + > Strepto_HMM_coverage.tsv
wc -l Strepto_HMM_coverage.tsv

#filter by coverage
awk '$1 != "inf" && $1 >= 40' Strepto_HMM_coverage.tsv > Strepto_HMM_coverage_filtered.tsv

#Count lines
# before filter
wc -l Strepto_HMM_coverage.tsv

#after filter
wc -l Strepto_HMM_coverage_filtered.tsv
```

Retrieve filenames to be moved
```
awk '{print $2}' Strepto_HMM_coverage_filtered.tsv > Strepto_HMM_coverage_filtered_ID.tsv
cp Strepto_HMM_coverage_filtered_ID.tsv ../output_hmm_Strepto/
```
Move files
```
cd ../output_hmm_Strepto/
 while read id; do
 cp "${id}" ../filtered_hmm_out_Strepto
done < Strepto_HMM_coverage_filtered_ID.tsv
```

###evalue

Join all files inside **filtered_hmm_out_Strepto** folder
```
cd ../filtered_hmm_out_Strepto
ls filtered_hmm_out_Strepto
cat *_output.tsv  > all_coverage_filtered_hmm_Strepto_output.tsv

#see the file
head all_coverage_filtered_hmm_Strepto_output.tsv

#Format output file
#remove hastag
sed '/#/d' all_coverage_filtered_hmm_Strepto_output.tsv > all_coverage_filtered_hmm_Strepto_output_wo_hastag.tsv

#tabular format
sed  's/ \{1,\}/\t/g' all_coverage_filtered_hmm_Strepto_output_wo_hastag.tsv > all_coverage_filtered_hmm_Strepto_output_wo_hastag_and_tabular_formatted.tsv

#remove unnecessary columns
cut -d $'\t' -f1-25 all_coverage_filtered_hmm_Strepto_output_wo_hastag_and_tabular_formatted.tsv > all_coverage_filtered_hmm_Strepto_output_wo_hastag_and_tabular_formatted_cutted.tsv
```

#Check file inside R environment. Follow script **01.HMM_Strepto_evalue_filter.ipynb** held inside **scripts/jupyter_scripts**

Final output files are located in 
```
Streptomyces_PhD/
```

## InterProScan analysis

First remove header
```
#remove header
for file in target_IDs*; do
sed  '1d' $file > "${file%.tsv}_wo_header.tsv";
done

#rename files faa
sed -i 's/ .*//' *.faa

#move files into proteins folder
cp *_wo_header.tsv ./ncbi_dataset
cd ./ncbi_dataset
```
Retrieve sequences
```
bash retrieve_seqs_interpro.sh
```
Verify that the number of entries matches across files.
```
wc -l *
grep -c ">" *retrieved_now
```

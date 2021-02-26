### Detailed Data processing ###
----------
For this assignement, I summarize the workflow as:

Original data<br>
:....Separate files maize and teosinte -> transpose files<br>
:<br>
:........create headers files<br>
:........remove headers from files<br>
:........sort genotypes and snps' files<br>
:........concatenate headears<br>
:<br>
:............sort by chromosome and position<br>
:<br>
:........................create a file for unknown positions<br>
:........................create a file for multiple positions snps<br>
:<br>
:............filter out unknown and multiple position snps<br>
:<br>
:................replace the missing data symbol<br>
:....................sort by chromosome and position (ascending)<br>
:........................create 10 files, one for each chromosome<br>
:<br>
:................replace the missing data symbol<br>
:....................sort by chromosome and position (decreasing)<br>
:........................create 10 files, one for each chromosome<br>   

This same worflow is applied to both species, yielding 44 files total.<br>
Many intermediate files, can be used for both species.<br>
All intermediate files are indicated with the prefix `tem` to facilitate their removal after data processing.

----------
File `fang_et_al_genotypes.txt`: <br>
Select of genotypes based on the species.<br> 

- `grep -E` for allow searching between multiple options for each species.<br>

- `cut --complement -f 2-3` to remove unneeded columns. <br>

- Two files created, one for each species.<br>

<br>

	grep -E "(Group|ZMPBA|ZMPIL|ZMPJA)" fang_et_al_genotypes.txt | cut --complement -f 2-3 > tem_teosinte_genotypes.txt
	grep -E "(Group|ZMMIL|ZMMLR|ZMMMR)" fang_et_al_genotypes.txt | cut --complement -f 2-3 > tem_maize_genotypes.txt

Transpose each genotype file using `transpose.awk`
	
	awk -f transpose.awk tem_maize_genotypes.txt > tem_transposed_maize_genotypes.txt
	awk -f transpose.awk tem_teosinte_genotypes.txt > tem_transposed_teosinte_genotypes.txt

From the file `snp_position.txt`:<br>
Use `cut -f` to take columnms 1, 3 and 4, which contains SNP_ID, Chromosome and Position, and stored them in a new file. <br>

	cut -f 1,3,4 snp_position.txt > tem_positions
Create a file excluding header line using `grep -v ^SNP_ID`.<br>
The output is is sorted by the first column `sort -k1,1` and stored as a new file.<br>

	grep -v ^SNP_ID tem_positions | sort -k1,1 > tem_sorted_positions

Back in the transposed genotypes files:<br>

- Use `-grep -v ^Sample file` filters reverse everything started with the `Sample`, for removing the header.<br>

- The output is piped to `-sort -k1,1` by the first column (Sample_ID).<br>

- Sorted output stored as new files.<br>

<br>

	grep -v ^Sample tem_transposed_maize_genotypes.txt | sort -k1,1 > tem_sorted_maize_genotypes
	grep -v ^Sample tem_transposed_teosinte_genotypes.txt | sort -k1,1 > tem_sorted_teosinte_genotypes

Create header files using `head -n 1` for the SNPs' file and for each genotype file.<br>
	
	head -n 1 tem_transposed_maize_genotypes.txt > tem_maize_header
	head -n 1 tem_transposed_teosinte_genotypes.txt > tem_teosinte_header
	head -n 1 tem_positions > tem_position_header
	
Add headers to all the sorted files using `cat`. In each case `` indicates file1 is on top of file2.

	cat tem_maize_header ``tem_sorted_maize_genotypes > tem_sorted_maize_complete
	cat tem_teosinte_header ``tem_sorted_teosinte_genotypes > tem_sorted_teosinte_complete
	cat tem_position_header ``tem_sorted_positions > tem_sorted_positions_complete


Join SNPs file `-1` to each genotypes file `-1` using the first column `1` as a common column (SNP_ID), and saved as a new file.<br>
```--header``` will indicate to ignore the first row.<br>
```-t $'\t'``` indicates to ```join``` that the files use tab as a delimiter.<br>

	join -1 1 -2 1 -t $'\t' --header tem_sorted_positions_complete tem_sorted_maize_complete > tem_merged_maize
	join -1 1 -2 1 -t $'\t' --header tem_sorted_positions_complete tem_sorted_teosinte_complete > tem_merged_teosinte

**First set of output files: Unknown and multiple position SNPs**<br>
Create a directory for storing the output files using `mkdir`

	mkdir output
The first four files will be multiple and unknown positions for maize and teosinte.<br>
For filtering the joint dataset `grep -E` allows to keep all the data rows containing either matching pattern `(Chromosome|unknown)`, allowing to keep the headers. Files stored in the output directory.

	grep -E '(Chromosome|unknown)' tem_merged_maize > ./output/maize_unknown_pos_snps.txt
	grep -E '(Chromosome|multiple)' tem_merged_maize > ./output/maize_multiple_pos_snps.txt
	grep -E '(Chromosome|unknown)' tem_merged_teosinte > ./output/teosinte_unknown_pos_snps
	grep -E '(Chromosome|multiple)' tem_merged_teosinte > ./output/teosinte_multiple_pos_snps

Retain SNPs from the joint files which are not unknown/multiple using `grep -v -E`.<br>
Filtered outputs are stored as new files.<br>

	grep -v -E "(unknown|multiple)" tem_merged_maize > tem_merged_maize_chromosomes
	grep -v -E "(unknown|multiple)" tem_merged_teosinte > tem_merged_teosinte_chromosomes

Create headers from the filtered files (only chromosomes), for upcoming steps.<br>
	
	head -n 1 tem_merged_maize_chromosomes > tem_merged_maize_headers
	head -n 1 tem_merged_teosinte_chromosomes > tem_merged_teosinte_headers

**Second set of output files: different sortings and charaters for missing data**
This section includes a pipe and a `for` loop, for each combination of sorting and species.<br> 

The pipe goes from a filtered file with "valid" SNPs only, to the output of a file for each chromosome. Pipe's outputs are files in the working directory. 

The `for` loop just add a header row to each file, and save them in the `./output` with their final name.

*Pipe 1: Replace, sort, split and store*

- In the joined and filtered file,`sed 's/?\/?/?/g'` subtitutes `'s/`the matching pattern `?/?` by a single `?`, globally `/g'`. The `\` in between of that matching pattern, is an escape character for `sed` given that `/` are part of the syntax.<br>
 
- The previous output is send for sorting by `sort`, by using columns 2 (Chromosome) and 3 (Position), considering both as numerical columnms. Although positions were the most important to take into consideration, I sorted the chromosome column for verifications.<br>
 
- The last `awk` command will create a file for each different value in column 2 (11, 10 chromosomes and 1 for for the header), where all the lines with the same value will be writen together. The output will be 11 separate files for each joined file. The output goes to the working directory.<br>

<br>
  	
	sed 's/?\/?/?/g' tem_merged_maize_chromosomes | sort -k2,2n -k3,3n | awk -F '\t' '{print $0 > "tem_m_inc_c_"$2}'
	
	sed 's/?\/?/?/g' tem_merged_teosinte_chromosomes | sort -k2,2n -k3,3n | awk -F '\t' '{print $0 > "tem_t_inc_c_"$2}' 

*For loop: add headers and save*
The loop `for`will iterate in a sequence from 1 to 10.
The `do` action is merging the header lines saved before, to each of the newly created chromosome files. The output of this loop, goes to the output folder.
 
	
	for i in $(seq 1 10); do cat tem_merged_teosinte_headers ``tem_t_inc_c_$i > "./output/teosinte_inc_c_"$i".txt"; done
	for i in $(seq 1 10); do cat tem_merged_maize_headers ``tem_m_inc_c_$i > "./output/maize_inc_c_"$i".txt"; done

This section of code is similar to the previous one. The two main differences in the workflow are:

- `sed 's/?\/?/-/g'` will globally replace the pattern `?/?` by a `-`<br>

- `sort -k2,2n -k3,3nr` sort the output of sed, by column 2 and column 3 as numeric, but sorting column 3 (Positions) in reverse order.

<br>

	sed 's/?\/?/-/g' tem_merged_maize_chromosomes | sort -k2,2n -k3,3nr | awk -F '\t' '{print $0 > "tem_m_dec_c_"$2}'
	sed 's/?\/?/-/g' tem_merged_teosinte_chromosomes | sort -k2,2n -k3,3nr | awk -F '\t' '{print $0 > "tem_t_dec_c_"$2}'


	for i in $(seq 1 10); do cat tem_merged_maize_headers ``tem_m_dec_c_$i > "./output/maize_dec_c_"$i".txt"; done
	for i in $(seq 1 10); do cat tem_merged_teosinte_headers ``tem_t_dec_c_$i > "./output/teosinte_dec_c_"$i".txt"; done
	

take the file merged\_maize\_chromosomes, globally replace the ?/? by a -, sort by chromosome and position in reverse order, and send each different group of values in the second column to a temporary

	
	

**Removing all the temporary files**
All temporary files were named including the prefix `tem_` in order to facilitate their remotion after finishing the data processing.

	rm tem*
#UNIX ASSIGNMENT#


### Data inspection ###
----------

**Attributes for ``fang_et_al_genotypes.txt``** <br>

1. File size <br>
I used the `ls` command for getting information about file size. Flag `-lh` allows an output in readable units.

		$ ls -lh fang_et_al_genotypes.txt
		-rw-r--r--. 1 jpanelo domain users 11M Feb 21 10:47 fang_et_al_genotypes.txt

2. Number of lines in the file <br>

Using `wc -l` gives the number of lines in the file 

		$ wc -l fang_et_al_genotypes.txt
		2783 fang_et_al_genotypes.txt

The count of 2783 does not necessarily represents the number of data rows. `grep -c` combined to the matching pattern exclude all the potential newlines in the file:<br>
	
		$ grep -c "[^\\n\\t]" fang_et_al_genotypes.txt
		2783

After inspectioning usind `head -n 5`, the first row seems to be the header. If multiple data files comes with the same format, a safe way to count for the number of actual data rows would be:

		 $ head -n -1 fang_et_al_genotypes.txt | wc -l
		2782

3. Number of columns <br>
`awk` with `-F` standing for the field separator used for computing the number of fields.

		$ head -n -1 fang_et_al_genotypes.txt |  awk -F '\t' '{print NF; exit}'
		986

4. Duplicated ID's <br>
The combination between `sort`, `uniq -d`, and `wc` commands computes the number of duplicated IDs. 

		$  sort fang_et_al_genotypes.txt |  uniq -d | wc -l
		0

5. Working groups<br>
In addition, the third columnm indicates the group which these IDs' belongs to. By combining `cut` for selecting the third column, `sort` for sorting the file, `uniq` for condense unique IDs and `wc -l` for counting the number of lines, we notice the IDs blongs to 16 different groups.  

		$ grep -v ^Sample fang_et_al_genotypes.txt | cut -f3 | sort | uniq | wc -l
		16
Printing the list of the 16 groups, ZMMLR and ZMPBA are the groups with the largest number of IDs, whereas the remainder 14 groups includes a low number of IDs compared to these two. 

		 $ grep -v ^Sample fang_et_al_genotypes.txt | cut -f3 | sort | uniq -c | sort -rn
		   1256 ZMMLR
		    900 ZMPBA
		    290 ZMMIL
		     75 ZMXCH
		     69 ZMXCP
		     41 ZMPIL
		     34 ZMPJA
		     27 ZMMMR
		     22 TRIPS
		     17 ZLUXR
		     15 ZDIPL
		     10 ZMHUE
		      9 ZPERR
		      7 ZMXNO
		      6 ZMXIL
		      4 ZMXNT

By inspecting this file, I learnt:
- This dataset has 2782 genotypes<br>
- There are 983 SNPs genotypes in this population, given that the first three columns refers to Sample IDs and some other infomations
- The reaserchers worked with are several "groups" of maize and teosinte, which might give some hints about the diversity the researchers explored. However, most of the genotypes belongs to a very few groups.


**Attributes for `snp_position.txt`** <br>

1. File size <br>

		ls -lh snp_position.txt
		-rw-r--r--. 1 jpanelo domain users 81K Feb 21 10:47 snp_position.txt

2. Number of lines in the file <br>
	
		$ wc -l snp_position.txt
		984 snp_position.txt
The count of 984 stands for the total number of lines

		$ grep -c "[^\\n\\t]" snp_position.txt
		984
The count of 984 stands for the total number of lines excludind newlines

		$ head -n -1 snp_position.txt| wc -l
		983
Similar to the first file, we can get the number of actual data rows by counting lines excluding the header.

3. Number of columns <br>
		
		$ head -n -1 snp_position.txt |  awk -F '\t' '{print NF; exit}'
		15

4. Duplicated SNPs ID <br>
Counting the number of unique 	

		$ sort snp_position.txt | uniq -d | wc -l
		0

5. Chromosomes <br>
`cut -f 3` selects the third column <br>
`grep -v '[a-z]'` filters excludes all non-numerical elements in the column, given that chromosomes are stored as numbers <br>
`sort -n` re-order the dataset considering the sorted colummn as numerical. <br>
`uniq -c` compress the dataset into a list with unique classes for each chromosome.
 
	   	$ cut -f 3 snp_position.txt | grep -v '[a-z]' | sort -n | uniq -c
	    155 1
	    127 2
	    107 3
	     91 4
	    122 5
	     76 6
	     97 7
	     62 8
	     60 9
	     53 10
In addition, 33 SNPs ID are cathegorized under 'multiple' and 'unknown' positions. 
 	
		 $ cut -f 3 snp_position.txt | grep -v '[0-9]' | sort | uniq -c
	      1 Chromosome
	      6 multiple
	     27 unknown

By inspecting this file, I learnt:
- The number os SNPs was indeed, 983<br>
- Some chromosomes shows larger number of snps than others. However, Chromosome 1 and Chromosome 10 showed the larger and smaller amount of SNPs respectively 
- 33 SNPs were not mapped to a specific chromosome.

### Data processing ###
----------

###Maize data###
	grep -E "(Group|ZMMIL|ZMMLR|ZMMMR)" fang_et_al_genotypes.txt | cut --complement -f 2-3 > tem_maize_genotypes.txt
	awk -f transpose.awk tem_maize_genotypes.txt > tem_transposed_maize_genotypes.txt
	cut -f 1,3,4 snp_position.txt > tem_positions
	grep -v ^SNP_ID tem_positions | sort -k1,1 > tem_sorted_positions
	grep -v ^Sample tem_transposed_maize_genotypes.txt | sort -k1,1 > tem_sorted_maize_genotypes
	head -n 1 tem_transposed_maize_genotypes.txt > tem_maize_header
	head -n 1 tem_positions > tem_position_header
	cat tem_maize_header ``tem_sorted_maize_genotypes > tem_sorted_maize_complete
	cat tem_position_header ``tem_sorted_positions > tem_sorted_positions_complete
	join -1 1 -2 1 -t $'\t' --header tem_sorted_positions_complete tem_sorted_maize_complete > tem_merged_maize
	grep -E '(Chromosome|unknown)' tem_merged_maize > ./output/maize_unknown_pos_snps.txt
	grep -E '(Chromosome|multiple)' tem_merged_maize > ./output/maize_multiple_pos_snps.txt
	grep -v -E "(unknown|multiple)" tem_merged_maize > tem_merged_maize_chromosomes
	head -n 1 tem_merged_maize_chromosomes > tem_merged_maize_headers
	sed 's/?\/?/?/g' tem_merged_maize_chromosomes | sort -k2,2n -k3,3n | awk -F '\t' '{print $0 > "tem_m_inc_c_"$2}'
	for i in $(seq 1 10); do cat tem_merged_maize_headers ``tem_m_inc_c_$i > "./output/maize_inc_c_"$i".txt"; done
	sed 's/?\/?/-/g' tem_merged_maize_chromosomes | sort -k2,2n -k3,3nr | awk -F '\t' '{print $0 > "tem_m_dec_c_"$2}'
	for i in $(seq 1 10); do cat tem_merged_maize_headers ``tem_m_dec_c_$i > "./output/maize_dec_c_"$i".txt"; done
	rm tem*

This code: <br>
- Filter out the groups belonging to maize from the file `fang_et_al_genotypes.txt`, and reads `snp_position.txt`<br>
- Transpose the data set so SNPs are in rows, genotypes in columns.<br>
- Sort and join the snp file and the genotypes file, so then each individual SNP in the genotypes file include chromosome and SNP position on the chromosome.<br>
- Creates two separate files for the SNPs with unknown positions and those matching in multiple positions.<br>
- Replaces the missing data symbol from `?/?` to `?`, sort Chromosomes and positions in ascending order, and creates a file in the `./output` directory with genotypic data for each individual Chromosome (10).<br>
- Replaces the missing data symbol from `?/?` to `-`, sort Chromosomes in ascending order, Positions in descending order, and creates a file in the `./output` directory with genotypic data for each individual Chromosome (10).<br>
- Clean the working directory out of temporary files.

###Teosinte data ###

	grep -E "(Group|ZMPBA|ZMPIL|ZMPJA)" fang_et_al_genotypes.txt | cut --complement -f 2-3 > tem_teosinte_genotypes.txt
	awk -f transpose.awk tem_teosinte_genotypes.txt > tem_transposed_teosinte_genotypes.txt
	cut -f 1,3,4 snp_position.txt > tem_positions
	grep -v ^SNP_ID tem_positions | sort -k1,1 > tem_sorted_positions
	grep -v ^Sample tem_transposed_teosinte_genotypes.txt | sort -k1,1 > tem_sorted_teosinte_genotypes
	head -n 1 tem_transposed_teosinte_genotypes.txt > tem_teosinte_header
	head -n 1 tem_positions > tem_position_header
	cat tem_teosinte_header ``tem_sorted_teosinte_genotypes > tem_sorted_teosinte_complete
	cat tem_position_header ``tem_sorted_positions > tem_sorted_positions_complete
	join -1 1 -2 1 -t $'\t' --header tem_sorted_positions_complete tem_sorted_teosinte_complete > tem_merged_teosinte
	grep -E '(Chromosome|unknown)' tem_merged_teosinte > ./output/teosinte_unknown_pos_snps
	grep -E '(Chromosome|multiple)' tem_merged_teosinte > ./output/teosinte_multiple_pos_snps
	grep -v -E "(unknown|multiple)" tem_merged_teosinte > tem_merged_teosinte_chromosomes
	head -n 1 tem_merged_teosinte_chromosomes > tem_merged_teosinte_headers
	sed 's/?\/?/?/g' tem_merged_teosinte_chromosomes | sort -k2,2n -k3,3n | awk -F '\t' '{print $0 > "tem_t_inc_c_"$2}'
	for i in $(seq 1 10); do cat tem_merged_maize_headers ``tem_m_inc_c_$i > "./output/maize_inc_c_"$i".txt"; done
	sed 's/?\/?/-/g' tem_merged_teosinte_chromosomes | sort -k2,2n -k3,3nr | awk -F '\t' '{print $0 > "tem_t_dec_c_"$2}'
	for i in $(seq 1 10); do cat tem_merged_teosinte_headers ``tem_t_dec_c_$i > "./output/teosinte_dec_c_"$i".txt"; done
	rm tem*

This code: <br>
- Filter out the groups belonging to Teosinte from the file `fang_et_al_genotypes.txt`, and reads `snp_position.txt`<br>
- Transpose the data set so SNPs are in rows, genotypes in columns.<br>
- Sort and join the snp file and the genotypes file, so then each individual SNP in the genotypes file include chromosome and SNP position on the chromosome.<br>
- Creates two separate files for the SNPs with unknown positions and those matching in multiple positions.<br>
- Replaces the missing data symbol from `?/?` to `?`, sort Chromosomes and positions in ascending order, and creates a file in the `./output` directory with genotypic data for each individual Chromosome.
- Replaces the missing data symbol from `?/?` to `-`, sort Chromosomes in ascending order, Positions in descending order, and creates a file in the `./output` directory with genotypic data for each individual Chromosome.
- Clean the working directory out of temporary files.
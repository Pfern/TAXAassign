TAXAassign_v0.3
===============

 TAXAassign is useful for annotating sequences at different taxonomic levels using  NCBI's taxonomy

* The taxonomic assignment is resolved using NCBI’s Taxonomy and running NCBI’s Blast against locally-installed NCBI’s nt database to minimize execution time.
* Version 0.3 has many orders of magnitude improvement in speed over previous version. 
* To minimize the execution time, we use GNU Parallel, a shell tool for executing jobs in parallel on multicore computers. We split the sequence file into fixed size chunks and then run blastn in parallel on these chunks on separate cores. For a 16SrRNA dataset comprising 1000 most abundant OTU sequences, matching at most 100 reference sequences against a local NCBI’s NT database took 18.9 minutes on 45 cores. A speedup of 30 times or more is achieved this way.
* To find Genbank ID to Taxa ID mapping, one can use gi_taxid_nucl.dmp.gz from ftp://ftp.ncbi.nih.gov/pub/taxonomy/  which is updated every Monday around 2am EST on NCBI’s website. However, searching Taxa IDs through this beastly 4.4GB+ unzipped file is time consuming and slows down the pipeline as noticed in the previous version. To improve the performance, instead of using this file, we use the latest version of NCBI blast 2.28+ which also gives Taxa ID for each hit and so we skip this step altogether in this version.
* To improve the time for finding parent Taxa IDs for a given Taxa ID, instead of using the flat file taxdump.tar.gz from the above ftp site, we use BioSQL and host NCBI’s taxonomy data in a local MySQL server. Furthermore, we use GNU Parallel to run multiple SQL queries in parallel on multiple records of blast output file thus reducing the execution time significantly
* Both NCBI’s taxonomy database on MySQL server/sqlite3 and local nt database can be updated frequently by submitting a cron job on the server scheduled to run when the server is less busy i.e. at night time and thus the information does not get outdated.
* All the time consuming steps in the TAXAassign pipeline are not repeated on reruns. Thus, if pipeline has finished processing the data and you want to run it again with a different taxonomic level, it will skip blasting the fasta file. To see what you have already done, output gets logged in TAXAassign.log file in the current folder. This is useful for debugging, should a problem arise.
* If you have the OTUs abundance table (in csv or tsv format) along with OTUs sequences, the output file generated from TAXAassign can then be used withhttp://userweb.eng.gla.ac.uk/umer.ijaz/bioinformatics/convIDs.pl to annotate the table.
* For better understanding of BioSQL, refer to my tutorial http://userweb.eng.gla.ac.uk/umer.ijaz/bioinformatics/BIOSQL_tutorial.pdf 

Downloading sqlite3 database
============================

```
cd <TAXAassign_directory>
mkdir database
cd database
wget http://userweb.eng.gla.ac.uk/umer.ijaz/bioinformatics/db.sqlite.gz
gunzip sqlite.db.gz
```

Test run on test.fasta
======================
Step 1: To test TAXAassign, we will use a small dataset test.fasta comprising 10 unknown sequences. To do so, create a folder and copy test.fasta provided in the installation directory
```
[uzi@quince-srv2 ~/]$ mkdir check_TAXAassign; cd check_TAXAassign 
[uzi@quince-srv2 ~/check_TAXAassign]$ cp ~/<TAXAassign_directory>/data/test.fasta .
[uzi@quince-srv2 ~/check_TAXAassign]$ ls
test.fasta
```
Step 2: You may execute TAXAassign.sh without any arguments to look at the syntax. If you are running TAXAassign on a multicore server, you may turn the parallel processing option on by -p switch and then set the number of cores that you want to use using -c switch.
You can specify the total number of reference matches per query sequence using -r switch, percentage identity or similarity using -m switch, and query sequence coverage using -c switch.
TAXAassign takes a consensus of multiple hits, so if you specify -t 70 then it means the sequence will be classified only if more than 70% of the hits agree on the same taxonomic level assignment. 
```
[uzi@quince-srv2 ~/check_TAXAassign]$ bash ~/TAXAassign_v0.3/TAXAassign.sh 
Script to annotate sequences at different taxonomic levels using  NCBI's taxonomy

Usage:
    bash TAXAassign.sh -f <fasta_file.fasta> [options]
Options:
    -p Turn parallel processing on
    -c Number of cores to use (Default: 10)
    -r Number of reference matches (Default: 10)
    -m Minimum percentage ident in blastn (Default: 97)
    -q Minimum query coverage in blastn (Default: 97)
    -t Consensus threshold (Default: 90)
```

Step 3: You can then run the file test.fasta as follows:

```    
[uzi@quince-srv2 ~/check_TAXAassign]$ bash ~/TAXAassign_v0.3/TAXAassign.sh -p -c 10 -t 70 -f test.fasta
[2013-06-26 15:37:51] TAXAassign v0.3. Copyright (c) 2013 Computational Microbial Genomics Group, University of Glasgow, UK
[2013-06-26 15:37:51] Using  /home/opt/ncbi-blast-2.2.28+/bin/blastn
[2013-06-26 15:37:51] Using  /home/uzi/TAXAassign_v0.3/scripts/blast_concat_taxon.py
[2013-06-26 15:37:51] Using  /home/uzi/TAXAassign_v0.3/scripts/blast_gen_assignments.pl
[2013-06-26 15:37:51] Using  parallel
[2013-06-26 15:37:51] Blast against NCBI's nt database with minimum percent ident of 97%, maximum of 10 reference sequences, and evalue of 0.0001 in blastn.
[2013-06-26 15:38:32] blastn using GNU parallel took 41 seconds for test.fasta.
[2013-06-26 15:38:32] test_B.out generated successfully!
[2013-06-26 15:38:32] Filter blastn hits with minimum query coverage of 97%.
[2013-06-26 15:38:32] test_BF.out generated successfully!
[2013-06-26 15:38:32] Annotate blastn hits with NCBI's taxonomy data.
[2013-06-26 15:38:34] test_BFT.out generated successfully!
[2013-06-26 15:38:34] Generate taxonomic assignment tables from blastn hits with consensus threshold of 70%.
[2013-06-26 15:38:34] test_ASSIGNMENTS.csv, test_PHYLUM.csv, test_CLASS.csv, test_ORDER.csv, test_FAMILY.csv, test_GENUS.csv, and test_SPECIES.csv generated successfully!
[2013-06-26 15:38:34] Sequences assigned at phylum level: 10/10 (100.00000%)
[2013-06-26 15:38:34] Sequences assigned at class level: 10/10 (100.00000%)
[2013-06-26 15:38:34] Sequences assigned at order level: 10/10 (100.00000%)
[2013-06-26 15:38:34] Sequences assigned at family level: 10/10 (100.00000%)
[2013-06-26 15:38:34] Sequences assigned at genus level: 10/10 (100.00000%)
[2013-06-26 15:38:34] Sequences assigned at species level: 2/10 (20.00000%)
[uzi@quince-srv2 ~/check_TAXAassign]$ cat test_ASSIGNMENTS.csv
seq6,Proteobacteria,Gammaproteobacteria,Pseudomonadales,Pseudomonadaceae,Pseudomonas,__Unclassified__
seq3,Proteobacteria,Gammaproteobacteria,Pseudomonadales,Pseudomonadaceae,Pseudomonas,__Unclassified__
seq7,Proteobacteria,Gammaproteobacteria,Pseudomonadales,Pseudomonadaceae,Pseudomonas,__Unclassified__
seq9,Proteobacteria,Betaproteobacteria,Burkholderiales,Burkholderiaceae,Ralstonia,Ralstonia solanacearum
seq2,Actinobacteria,Actinobacteria,Actinomycetales,Nocardiaceae,Rhodococcus,__Unclassified__
seq10,Actinobacteria,Actinobacteria,Actinomycetales,Microbacteriaceae,Microbacterium,Microbacterium laevaniformans
seq8,Proteobacteria,Alphaproteobacteria,Caulobacterales,Caulobacteraceae,Brevundimonas,__Unclassified__
seq1,Proteobacteria,Alphaproteobacteria,Caulobacterales,Caulobacteraceae,Brevundimonas,__Unclassified__
seq4,Proteobacteria,Deltaproteobacteria,Desulfovibrionales,Desulfovibrionaceae,Desulfovibrio,__Unclassified__
seq5,Bacteroidetes/Chlorobi group,Sphingobacteriia,Sphingobacteriales,Sphingobacteriaceae,Pedobacter,__Unclassified__
[uzi@quince-srv2 ~/check_TAXAassign]$ cat test_PHYLUM.csv
Actinobacteria,2
Bacteroidetes/Chlorobi group,1
Proteobacteria,7
[uzi@quince-srv2 ~/check_TAXAassign]$ cat test_CLASS.csv
Actinobacteria,2
Alphaproteobacteria,2
Betaproteobacteria,1
Deltaproteobacteria,1
Gammaproteobacteria,3
Sphingobacteriia,1
[uzi@quince-srv2 ~/check_TAXAassign]$ cat test_ORDER.csv
Actinomycetales,2
Burkholderiales,1
Caulobacterales,2
Desulfovibrionales,1
Pseudomonadales,3
Sphingobacteriales,1
[uzi@quince-srv2 ~/check_TAXAassign]$ cat test_FAMILY.csv
Burkholderiaceae,1
Caulobacteraceae,2
Desulfovibrionaceae,1
Microbacteriaceae,1
Nocardiaceae,1
Pseudomonadaceae,3
Sphingobacteriaceae,1
[uzi@quince-srv2 ~/check_TAXAassign]$ cat test_GENUS.csv
Brevundimonas,2
Desulfovibrio,1
Microbacterium,1
Pedobacter,1
Pseudomonas,3
Ralstonia,1
Rhodococcus,1
[uzi@quince-srv2 ~/check_TAXAassign]$ cat test_SPECIES.csv
Microbacterium laevaniformans,1
Ralstonia solanacearum,1
__Unclassified__,8
[uzi@quince-srv2 ~/check_TAXAassign]$ 

```

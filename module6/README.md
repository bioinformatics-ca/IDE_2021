---
layout: tutorial_page
permalink: /epidemiology_2021_EPD_IMS
title: Infectious Disease Epidemiology Analysis
header1: Emerging Pathogen Detection and Identification using Metagenomic Samples
header2: Infectious Disease Genomic Epidemiology
image: /site_images/CBW_Metagenome_icon.jpg
home: https://bioinformaticsdotca.github.io/epidemiology_2021
description: Emerging Pathogen Detection and Identification using Metagenomic Samples
author: Aaron Petkau and Gary Van Domselaar
modified: October 2nd, 2021
---

# Table of contents
1. [Introduction](#intro)
2. [Software](#software)    
3. [Setup](#setup)
4. [Exercise](#exercise)
5. [Final words](#final)

<a name="intro"></a>
# 1. Introduction

This tutorial aims to introduce a variety of software and concepts related to detecting emerging pathogens from a complex host sample. The provided data and methods are derived from real-world data, but have been modified to either illustrate a specific learning objective or to reduce the complexity of the problem. Contamination and a lack of large and accurate databases render detection of microbial pathogens difficult. As a disclaimer, all results produced from the tools described in this tutorial and others must also be verified with supplementary bioinformatics or wet-laboratory techniques.

<a name="software"></a>
# 2. List of software for tutorial

* [fastp][]
* [multiqc][]
* [KAT][]
* [Kraken2][]
* [Krona][]
* [Megahit][]
* [Quast][]
* [NCBI blast][]

<a name="setup"></a>
# 3. Exercise setup

## 3.1. Copy data files

To begin, we will copy over the exercises to `~/workspace`. This let's use view the resulting output files in a web browser.

**Commands**
```bash
cp -r ~/CourseData/IDE_data/module6/module6_workspace/ ~/workspace/
cd ~/workspace/module6_workspace/analysis
```

When you are finished with these steps you should be inside the directory `/home/ubuntu/workspace/module6_workspace/analysis`. You can verify this by running the command `pwd`.

**Output after running `pwd`**
```
/home/ubuntu/workspace/module6_workspace/analysis
```

## 3.2. Activate environment

Next we will activate the [conda](https://docs.conda.io/en/latest/) environment, which will have all the tools needed by this tutorial pre-installed. To do this please run the following:

**Commands**
```bash
conda activate cbw-emerging-pathogen
```

You should see the command-prompt (where you type commands) switch to include `(cbw-emerging-pathogen)` at the beginning, showing you are inside this environment. You should also be able to run one of the commands like `kraken2` and see output:

**Output after running `kraken2 --version`**
```
Kraken version 2.1.2
Copyright 2013-2021, Derrick Wood (dwood@cs.jhu.edu)
```

<a name="exercise"></a>
# 4. Exercise

## 4.1. Patient Background:

A 41-year-old man was admitted to a hospital 6 days after the onset of disease. He reported fever, chest tightness, unproductive cough, pain and weakness. Preliminary investigations excluded the presence of influenza virus, *Chlamydia pneumoniae*, *Mycoplasma pneumoniae*, and other common respiratory pathogens. After 3 days of treatment the patient was admitted to the intensive care unit, and 6 days following admission the patient was transferred to another hospital.

To further investigate the cause of illness, a sample of bronchoalveolar lavage fluid (BALF) was collected from the patient and metatranscriptomic sequencing was performed (that is, the RNA from the sample was sequenced). In this lab, you will examine the metatranscriptomic data using a number of bioinformatics methods and tools to attempt to identify the cause of the illness.

*Note: The patient information and data was derived from a real study (shown at the end of the lab).* 

## 4.2. Overview

We will proceed through the following steps to attempt to diagnose the situation.

* Trim and clean sequence reads using `fastp`
* Filter host (human) reads with `kat`
* Run Kraken2 with a bacterial and viral database to look at the taxonomic makeup of the reads.
* Assemble metatranscriptome with `megahit`
* Examine assembly using `quast` and `blast`

---

## Step 1: Clean and examine quality of the reads

Reads that come directly off of a sequencer may be of variable quality which might impact the downstream analysis. We will use the software [fastp][] to both clean and trim reads (removing poor-quality reads) as well as examine the quality of the reads. To do this please run the following:

**Commands**
```bash
# Time: 30 seconds
fastp --detect_adapter_for_pe --in1 ../data/emerging-pathogen-reads_1.fastq.gz --in2 ../data/emerging-pathogen-reads_2.fastq.gz --out1 cleaned_1.fastq --out2 cleaned_2.fastq
``` 

You should see the following as output:

**Output**
```
Detecting adapter sequence for read1...
No adapter detected for read1

Detecting adapter sequence for read2...
No adapter detected for read2
[...]
Insert size peak (evaluated by paired-end reads): 150

JSON report: fastp.json
HTML report: fastp.html

fastp --detect_adapter_for_pe --in1 ../data/emerging-pathogen-reads_1.fastq.gz --in2 ../data/emerging-pathogen-reads_2.fastq.gz --out1 cleaned_1.fastq --out2 cleaned_2.fastq
fastp v0.22.0, time used: 34 seconds
```

### Examine output

You should now be able to nagivate to <http://YOUR-MACHINE/module6_workspace/analysis> and see some of the output files. In particular, you should be able to find **fastp.html**, which contains a report of the quality of the reads and how many were removed. Please take a look at this report now:

![fastp-report][]

This should show an overview of the quality of the reads before and after filtering with `fastp`. Using this report, please anser the following questions.

### Step 1: Questions

1. Compare the **total reads** and **total bases** before and after filtering in the `fastp` report. How do they differ?
2. Compare the **Q30 bases** (number of bases with a quality score > 30) before and after filtering? How do they compare (compare the percentage values)?

---

## Step 2: Host read filtering

The next step is to remove any host reads (in this case Human reads) from our dataset as we are not focused on examining host reads. There are several different tools that can be used to filter out host reads such as Kraken2, BLAST, KAT and others. In this demonstration, we have selected to run KAT followed by Kraken2, but you could likely accomplish something similar directly in Kraken2.

Command documentation is available [here](http://kat.readthedocs.io/en/latest/using.html#sequence-filtering)

KAT works by breaking down each read into small fragements of length *k*, k-mers, and comparing them to a k-mer database of the human reference genome. Subsequently, the complete read is either assigned into a matched or unmatched file if 10% of the k-mers in the read have been found in the human database.

![kat-overview][]

Let's run KAT now.

**Commands**
```bash
# Time: 3 minutes
kat filter seq -t 4 -i -o filtered --seq cleaned_1.fastq --seq2 cleaned_2.fastq ~/CourseData/IDE_data/module6/db/kat_db/human_kmers.jf
```

The arguments for this command are:

* `--seq --seq2` arguments to provide corresponding forward and reverse fastq reads (the cleaned reads from `fastp`)
* `-i` whether to output sequences not found in the kmer hash, rather than those with a database hit (host sequences). 
* `-o filtered` Provide prefix for all files generated by the command. In our case, we will have two output files **filtered.in.R1.fastq** and **filetered.in.R2.fastq**.
* `~/CourseData/IDE_data/module6/db/kat_db/human_kmers.jf` the human k-mer database

As the command is running you should see the following output on your screen:

**Output**
```
Running KAT in filter sequence mode
-----------------------------------

Loading hashes into memory... done.  Time taken: 40.7s

Filtering sequences ...
Processed 100000 pairs
Processed 200000 pairs
[...]
Finished filtering.  Time taken: 122.3s

Found 1127908 / 1306231 to keep

KAT filter seq completed.
Total runtime: 163.0s
```

If the command was successful, your current directory should contain two new files:

* `filtered.in.R1.fastq`
* `filtered.in.R2.fastq`

These are the set of reads minus any reads that matched the human genome.

---

## Step 3: Classify reads against the Kraken database

Now that we have most, if not all, host reads filtered out, it’s time to classify the remaining reads.

Database selection is one of the most crucial parts of running Kraken. One of the many factors that must be considered is the computational resources available. Our current AWS image for the course has only 16G of memory. A major disadvantage of Kraken is that it loads the entire database into memory. With the [standard viral, bacterial, and archael database](https://benlangmead.github.io/aws-indexes/k2) on the order of 50 GB we would be unable to run the full database on the course machine. To help mitigate this, Kraken allows reduced databases to be constructed, which will still give reasonable results. We have constructed our own smaller Kraken database using only bacterial, human, and viral data. We will be using this database.

Lets run the following command in our current directory to classify our reads against the kraken database.

**Commands**
```bash
# Time: 1 minute
kraken2 --db ~/CourseData/IDE_data/module6/db/kraken2_db --threads 4 --paired --output kraken_out.txt --report kraken_report.txt --unclassified-out kraken2_unclassified#.fastq filtered.in.R1.fastq filtered.in.R2.fastq
```

This should produce output similar to below:

**Output**
```
Loading database information... done.
1127908 sequences (315.54 Mbp) processed in 7.344s (9214.6 Kseq/m, 2577.83 Mbp/m).
  880599 sequences classified (78.07%)
  247309 sequences unclassified (21.93%)
```

Let's examine the text-based report of Kraken2:

**Commands**
```bash
less kraken_report.txt
```

This should produce output similar to the following:

```
 21.93  247309  247309  U       0       unclassified
 78.07  880599  30      R       1       root
 78.01  879899  124     R1      131567    cellular organisms
 76.37  861411  19285   D       2           Bacteria
 56.53  637572  2       D1      1783270       FCB group
 56.53  637558  1571    D2      68336           Bacteroidetes/Chlorobi group
 56.39  635982  1901    P       976               Bacteroidetes
 55.10  621496  35      C       200643              Bacteroidia
 55.09  621417  19584   O       171549                Bacteroidales
 53.18  599872  2464    F       171552                  Prevotellaceae
 52.96  597396  397538  G       838                       Prevotella
  4.74  53473   53473   S       28137                       Prevotella veroralis
[...]
```

This will show the top taxonomic ranks (right-most column) as well as the percent and number of reads that fall into these categories (left-most columns). For example: the very first row `21.93  247309  247309  U       0       unclassified` shows us that **247309 (21.93%)** of the reads processed by Kraken2 are unclassified (remember we only used a database containing bacterial, viral, and human representatives). More details about how to read this report can be found at <https://github.com/DerrickWood/kraken2/wiki/Manual#sample-report-output-format>.

---

## Step 4: Generate an interactive html-based report using Krona

Instead of reading a text-based report like above, we can visualize this information using [Krona][].

To generate a Krona figure, we first must make a file, `krona_input.txt`, containing a list of read IDs and the taxonomic IDs these reads were assigned. We can then run `ktImportTaxonomy` to create the figure. To do this, please run the following commands.

**Commands**
```bash
# Time: 1 second
cut -f2,3 kraken_out.txt > krona_input.txt

# Time: 30 seconds
ktImportTaxonomy krona_input.txt -o krona_report.html
```

Let’s look at what Krona generated. Return to your web browser and refresh the page from Step 1 to see the new files added in the `module6_workspace/analysis` directory.

Click on **final_web_report.html**. *Note: if this is not working, what you should see is shown in the image [krona-all.png][]*.

### Step 4: Questions

1. What does the distribution of taxa found in the reads look like? Is there any pathogen here that could be consistent with a cause for the patients symptoms?
2. This data was derived from RNA (instead of DNA) and some viruses are RNA-based. Take a look into the **Viruses** category in Krona (by expanding this category). Is there anything here that could be consistent with the patient's symptoms? *Note: if you cannot expand the **Viruses** category what you should see is shown in this image [krona-viruses.png][].*.
3. Given The results of Krona, can you form a hypothesis as to the cause of the patient's symptoms?

---

## Step 5: Metatranscriptomic assembly

In order to investigate the data further we will assemble the metatranscriptome using the software [Megahit][]. To do this please run the following:

**Commands**
```bash
# Time: 6 minutes
megahit -t 4 -1 filtered.in.R1.fastq -2 filtered.in.R2.fastq -o megahit_out
```

If everything is working you should expect to see the following as output:

**Output**
```
2021-09-30 11:53:35 - MEGAHIT v1.2.9
2021-09-30 11:53:35 - Using megahit_core with POPCNT and BMI2 support
2021-09-30 11:53:35 - Convert reads to binary library
2021-09-30 11:53:36 - b'INFO  sequence/io/sequence_lib.cpp  :   75 - Lib 0 (/media/cbwdata/workspace/module6_workspace/analysis/filtered.in.R1.fastq,/media/cbwdata/workspace/module6_workspace/analysis/filtered.in.R2.fastq): pe, 2255816 reads, 151 max length'
2021-09-30 11:53:36 - b'INFO  utils/utils.h                 :  152 - Real: 1.9096\tuser: 1.8361\tsys: 0.3320\tmaxrss: 166624'
2021-09-30 11:53:36 - k-max reset to: 141
2021-09-30 11:53:36 - Start assembly. Number of CPU threads 4

[...]

2021-09-30 11:58:01 - Assemble contigs from SdBG for k = 141
2021-09-30 11:58:02 - Merging to output final contigs
2021-09-30 11:58:02 - 3112 contigs, total 1536607 bp, min 203 bp, max 29867 bp, avg 493 bp, N50 463 bp
2021-09-30 11:58:02 - ALL DONE. Time elapsed: 267.449160 seconds
```

Once everything is completed, you will have a directory `megahit_out/` with the output. Let's take a look at this now:

**Commands**
```bash
ls megahit_out/
```

**Output**
```
checkpoints.txt  done  final.contigs.fa  intermediate_contigs  log  options.json
```

It's specifically the **final.contigs.fa** file that contains our metatranscriptome assembly. This will contain the largest *contiguous* sequences Megahit was able to construct from the sequence reads. We can look at the contents with the command `head`:

**Commands**
```bash
head megahit_out/final.contigs.fa
```

**Output**
```
>k141_0 flag=1 multi=3.0000 len=312
ATACTGATCTTAGAAAGCTTAGATTTCATCTTTTCAATTGGTGTATCGAATTTAGATACAAATTTAGCTAAGGATTTAGACATTTCAGCTTTATCTACAGTAGAGTATACTTTAATATCTTGAAGTACACCAGTTACTTTAGACTTAATCAAAATTTTACCCAAATCATTAACTAGATCTTTAGAATCAGAATTCTTTTCTACCATTTTAGCGATGATATCTGTTGCATCTTGATCTTCAAATGAAGATCTATATGACATGATAGTTTGACCTTCTTGTAGTTGAGATCCAACTTCTAAACATTCGATGTCT
>k141_1570 flag=1 multi=2.0000 len=328
GAGCATCGCGCAGAAGTATCTGTACTCCCTTTACTCCACGCAAGTCTTTCTCATACTCACGCTCGACACCCATCTTACCGATATAATCTCCCGGCTGATAGTACTCGTCTTCCTCAATATCACCCTGACTCACCTCTGCAACATCCCCAAGGACATGTGCAGCGATAGCTCGTTGATACTGACGAACACTACGTTTCTGAATATAAAAGCCTGGAAAACGATAGAGTTTCTCTTGGAAGGCGCTAAAGTCTTTATCACTCAATTGGCTCAAGAATAGTTGCTGCGTAAAGCGAGAGTAACCCGGATTCTTACTCCTATCCTTGATCCC
[...]
```

It can be a bit difficult to get an overall idea of what is in this file, so in the next step we will use the software [Quast][] to summarize the assembly information.

---

## Step 6: Evaluate assembly with Quast

[Quast][] can be used to provide summary statistics on the output of assembly software. We will run Quast on our data by running the following command:

**Commands**
```bash
# Time: 2 seconds
quast -t 4 megahit_out/final.contigs.fa
```

You should expect to see the following as output:

**Output**
```
/home/ubuntu/.conda/envs/cbw-emerging-pathogen/bin/quast -t 4 megahit_out/final.contigs.fa

Version: 5.0.2

System information:
  OS: Linux-5.11.0-1017-aws-x86_64-with-debian-bullseye-sid (linux_64)
  Python version: 3.7.10
  CPUs number: 4

Started: 2021-09-30 14:54:58

[...]

Finished: 2021-09-30 14:55:00
Elapsed time: 0:00:01.768326
NOTICEs: 1; WARNINGs: 0; non-fatal ERRORs: 0

Thank you for using QUAST!
```

Quast writes it's output to a directory `quast_results/`, which includes HTML and PDF reports. We can view this using a web browser by navigating to <http://[YOUR-MACHINE]/module6_workspace/analysis/quast_results/latest/icarus.html>. From here, click on **Contig size viewer**. You should see the following:

![quast-contigs.png][]

This shows the length of each contig in the `megahit_out/final.contigs.fa` file, sorted by size.

### Step 6: Questions

1. What is the length of the largest contig in the genome? How does it compare to the length of the 2nd and 3rd largest contigs?
2. Given that this is RNASeq data (i.e., sequences derived from RNA), what is the most common type of RNA you should expect to find? What is the approximate lengths of these RNA fragments? Is the largest contig an outlier (i.e., is it much longer than you would expect)?
3. Is there another type of source for this RNA fragment that could explain it's length? Possibly a [Virus][https://en.wikipedia.org/wiki/Coronavirus#Genome]?

---

## Step 7: Use BLAST to look for existing organisms

In order to get a better handle on what the identity of this large contig could be, let's use [BLAST][] to compare to a database of existing viruses. Please run the following:

**Commands**
```
# Time: seconds
seqkit sort --by-length --reverse megahit_out/final.contigs.fa | seqkit head -n 50 > contigs-50.fa
blastn -db ~/CourseData/IDE_data/module6/db/blast_db/ref_viruses_rep_genomes_modified -query contigs-50.fa -html -out blast_results.html
```

Here, we first use [seqkit][] to sort all contigs by length (`seqkit sort --by-length ...`) and we then extract only the top **50** longest contigs (`seqkit head -n 50`) and write these to a file **contigs-50.fa** (`> contigs-50.fa`).

Next, we run [BLAST][] on these top 50 longest contigs using a pre-computed database of viral genomes (`blastn -db ~/CourseData/IDE_data/module6/db/blast_db/ref_viruses_rep_genomes_modified -query contigs-50.fa ...`). The (`-html -out blast_results.html`) tells BLAST to write it's results as an HTML file.

To view these results, please browse to <http://YOUR-MACHINE/module6_workspace/analysis/blast_results.html> to view the ouptut `blast_results.html` file. This should look something like below:

![blast-report.png][]

### Step 7: Questions

1. What is the closest match for the longest contig you find in your data? Recall that if a pathogen is an emerging/novel pathogen then you may not get a perfect match to any existing organisms.
2. Using the BLAST report alongside all other information we've gathered, what can you say about what pathogen may be causing the patient's symptoms?

---

<a name="final"></a>
# 5. Final words

Congratulations, you've finished this lab. As a final check on your results, you can use [NCBI's online tool](https://blast.ncbi.nlm.nih.gov/Blast.cgi) to perform a BLAST on a larger database to see if you get any better matches.

The source of the data and patient background information can be found at <https://doi.org/10.1038/s41586-020-2008-3> (note: clicking this link will reveal what the illness is). The only modification made to the original metatranscriptomic reads was to reduce them to 10% of the orginal file size.


[fastp]: https://github.com/OpenGene/fastp
[multiqc]: https://multiqc.info/
[KAT]: https://kat.readthedocs.io/en/latest/
[Kraken2]: https://ccb.jhu.edu/software/kraken2/
[Krona]: https://github.com/marbl/Krona/wiki
[Megahit]: https://github.com/voutcn/megahit
[NCBI blast]: https://blast.ncbi.nlm.nih.gov/Blast.cgi
[seqkit]: https://bioinf.shenwei.me/seqkit/
[fastp-report]: images/fastp.png
[kat-overview]: images/kat.png
[krona-all.png]: images/krona-all.png
[krona-viruses.png]: images/krona-viruses.png
[quast-contigs.png]: images/quast-contigs.png
[Quast]: http://quast.sourceforge.net/quast
[blast-report.png]: images/blast-report.png

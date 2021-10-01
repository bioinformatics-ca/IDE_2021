---
layout: tutorial_page
permalink: /epidemiology_2021_EPD_IMS
title: Infectious Disease Epidemiology Analysis
header1: Phylogenetics and Phylodynamics of SARS-CoV-2
header2: Infectious Disease Epidemiology Analysis 2021
image: /site_images/CBW_Metagenome_icon.jpg
home: https://bioinformaticsdotca.github.io/epidemiology_2021
description: Phylogenetics and Phylodynamics of SARS-CoV-2
author: Aaron Petkau, Finlay Maguire, and Rob Beiko
modified: September 30, 2021
---

# Table of contents
1. [Introduction](#intro)
2. [Software](#software)    
2. [Exercise Setup](#setup)
3. [Exercise 1](#ex1)
4. [Exercise 2](#ex2)
5. [Paper](#paper)

<a name="intro"></a>
# 1. Introduction

In this tutorial we will be performing phylogenetic and phylodynamic analysis of a set of sequenced SARS-CoV-2 genomes and will be using this information to gain insight into the early introduction of SARS-CoV-2 into Scotland beginning around March 2020. The source of this data is available in the below study:

> da Silva Filipe, A., Shepherd, J.G., Williams, T. *et al.* Genomic epidemiology reveals multiple introductions of SARS-CoV-2 from mainland Europe into Scotland. *Nat Microbiol* **6**, 112–122 (2021). https://doi.org/10.1038/s41564-020-00838-z

We will be referring to this study to compare the phylogenetic trees and other results we generate to the already-published results.

The sequenced genomes from this study are available as part of the [COG-UK Consortium](https://www.cogconsortium.uk/tools-analysis/public-data-analysis-2/) and have already been downloaded to your machines.

<a name="software"></a>
# 2. List of software for tutorial

* [Augur][]
* [mafft][]
* [iqtree][]
* [treetime][]
* [Auspice][]

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

<a name="ex1"></a>
# 4. Exercise

## 4.1. Patient Background:

*Write background here.*

## 4.2. Overview

We will proceed through the following steps to attempt to diagnose the situation.

* Trim and clean reads using `fastp`
* Filter host reads with `kat`
* Run Kraken2 with a bacterial and viral database to look at the taxonomic makeup of the reads.
* Assemble metatranscriptome with `megahit`
* Examine assembly using `quast` and `blast`

---

### Step 1: Extract a subset of all SARS-CoV-2 genomes to work with

---

[Augur]: https://docs.nextstrain.org/projects/augur/en/stable/index.html
[mafft]: https://mafft.cbrc.jp/alignment/software/
[iqtree]: http://www.iqtree.org/
[treetime]: https://treetime.readthedocs.io/en/latest/
[Auspice]: https://auspice.us/

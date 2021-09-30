---
layout: tutorial_page
permalink: /IDE_2021_molecular_typing_lab
title: Molecular Typing Lab 
header1: Workshop Pages for Students
header2: Genome-scale Multilocus Sequence Typing Lab
image: /site_images/CBW_wshop-epidem_map-icon.png
home: https://bioinformaticsdotca.github.io/IDE_2021
description: Molecular Typing CBW Tutorial 
author: Dillon Barker
modified: 2021-10-04 
---

# Molecular Typing CBW Tutorial
## Dillon Barker - 2021-10-04

### Learning Objectives

Students will learn to: 

- Interpret genome-scale MLST results
- Compare allelic typing methods
- Visualize phylogenetic trees using R-lang and `ggtree`
- Clean noisy data
- Investigate an outbreak detected via routine surveillance

### Preamble

These exercises will take you through the process of analyzing surveillance data
for a foodborne bacterial pathogen. First you'll analyze the population using a
lower-resolution method, and then improve upon that with high-resolution core
genome multilocus sequence typing. The cgMLST call data will be noisy, and
you'll clean it to improve resolution and reliability. By projecting source and
temporal data on a phylogenetic tree, you'll identify an outbreak amongst the
sporadic cases in the surveillance data. Finally, you will further resolve the
outbreak using SNP typing.


Finally, if you finish an exercise early, I have included some bonus exercises.
These won't be covered in the tutorial and are not required to be done, but are
an elaboration upon material taught in the main exercise if you have spare time.

-----

We'll be looking at data derived from a surveillance program that isolated
_Campylobacter jejuni_ from wild raccoons. This raccoon population is being used
as a model system for _Campylobacter_ transmission in humans. Samples have been
taken from a population of wild raccoons, isolated, and sequenced on an Illumina
MiSeq. Your lab's bioinformatician has already taken care of assembling whole
genome sequences as well as  _in silico_ molecular typing including 7-gene
Multilocus Sequence Typing (MLST), and a 594-locus core genome multilocus
sequence typing scheme. You have been provided with these data, and you are to
analyze it for outbreak detection.

NB - This dataset really has been derived from a real-life raccoon surveillance 
program, however it has been modified to make it more amenable for teaching. 
Please feel free to ask me what modifications I've applied.  


![](https://upload.wikimedia.org/wikipedia/commons/thumb/e/ed/Raccoon_%28Procyon_lotor%29_2.jpg/711px-Raccoon_%28Procyon_lotor%29_2.jpg)

### Exercise 0 - Setting Up

For this tutorial, we'll be using R interactively through RStudio, an integrated
development environment for the R Language.

You can connect to your CBW instance of RStudio Server via your favourite web
browser at:

```
<your.public.ip>:8080
```

You can get your machine's public IP address by running:

```bash
curl http://checkip.amazonaws.com
```

We will use an R Script file as a scratch pad for writing and running R code.

Create one by clicking on **File** → **New File** → **R Script**

You can type code into the R Script, and you can execute code by selecting it 
and **Ctrl+Enter**. Test that everything is working by loading the libraries
we'll be using for the rest of the tutorial and printing a message to yourself
in the console.

```r
library(ape)
library(ggtree)
library(treeio)
library(dplyr)
library(readr)
library(tidyr)
library(tibble)
library(aplot)
library(stringr)
library(ggplot2)

print("Hello, world!")
```

### Exercise 1


#### Load and Display MLST

```r
```
#### Load raw cgMLST Data

#### Data Cleaning



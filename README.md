# Metagenomics Summer School

Course materials for the Genomics Aotearoa Metagenomics Summer School, to be hosted at the University of Auckland in December.

A draft timetable for the day is provided below, but please keep in mind that this is subject to change as we evaluate our course material.

---

## Workshop exercises

### Day 1

1. [Bash scripting](https://github.com/GenomicsAotearoa/metagenomics_summer_school/blob/master/materials/day1/ex1_bash_scripting.md)
1. [Quality filtering raw reads](https://github.com/GenomicsAotearoa/metagenomics_summer_school/blob/master/materials/day1/ex2_quality_filtering.md)
1. [Assembly (part 1)](https://github.com/GenomicsAotearoa/metagenomics_summer_school/blob/master/materials/day1/ex3_assembly.md)
1. [Assembly (part 2)](https://github.com/GenomicsAotearoa/metagenomics_summer_school/blob/master/materials/day1/ex4_assembly.md)

### Day 2

1. [Evaluating the overnight assembly](https://github.com/GenomicsAotearoa/metagenomics_summer_school/blob/master/materials/day2/ex5_evaluating_assemblies.md)
1. [Binning (part 1, read mapping)](https://github.com/GenomicsAotearoa/metagenomics_summer_school/blob/master/materials/day2/ex6_initial_binning.md)
1. [Binning (part 2, initial binning)](https://github.com/GenomicsAotearoa/metagenomics_summer_school/blob/master/materials/day2/ex7_initial_binning.md)
1. [Binning (part 3, dereplication)](https://github.com/GenomicsAotearoa/metagenomics_summer_school/blob/master/materials/day2/ex8_bin_dereplication.md)

### Day 3

1. [Bin refinement](https://github.com/GenomicsAotearoa/metagenomics_summer_school/blob/master/materials/day3/ex9_refining_bins.md)
1. [Gene prediction](https://github.com/GenomicsAotearoa/metagenomics_summer_school/blob/master/materials/day3/ex10_gene_prediction.md)
1. [Gene annotation (part 1)](https://github.com/GenomicsAotearoa/metagenomics_summer_school/blob/master/materials/day3/ex11_gene_annotation_part1.md)
1. [Gene annotation (part 2)](https://github.com/GenomicsAotearoa/metagenomics_summer_school/blob/master/materials/day3/ex12_gene_annotation_part2.md)

### Day 4

1. [Presentation of data](https://github.com/GenomicsAotearoa/metagenomics_summer_school/blob/master/materials/day4/ex13_presentation.md)

---

## Timetable

### Day 1 - 10<sup>th</sup> December 2019

|Time|Event|Session leader|
|:---|:---|:---|
|9:00 am – 9:45 am|**Introduction**<br>Welcome<br>*TASK: Logging into NeSI*|David Waite|
|9:45 am – 10:30 am|**Bash scripting**|Dinindu Senanayake<br>Ngoni Faya|
|10:30 am – 10:50 am|**Morning tea break**||
|10:50 am – 11:30 am|**Bash scripting** (continued)|Dinindu Senanayake<br>Ngoni Faya|
|11:30 am – 12:00 pm|**The metagenomics decision tree**<br>*TASK: Dividing into working groups*<br>*TASK: Select a goal for your project*|Kim Handley|
|12:00 pm – 12:45 pm|**Break for lunch**||
|12:45 pm – 1:45 pm|**Quality filtering raw reads**<br>*TASK: Visualisation with *FastQC**<br>*TASK: Read trimming and adapter removal*<br>**Diagnosing poor libraries**<br>**Common issues and best practice**|Florian Pichlmuller|
|1:45 pm – 3:00 pm|**Assembly (part 1)**<br>Choice of assemblers<br>*TASK: Considerations for parameters, and when to stop!*<br>*TASK: Submitting jobs to NeSI via slurm*|David Waite|
|3:00 pm – 3:20 pm|**Afternoon tea break**||
|3:20 pm – 3:45 pm|**Assembly (part 2)**<br>*TASK: Submitting variant assemblies to NeSI*|David Waite|
|4:00 pm – 5:00 pm|**End of day wrap up**<br>Attendees can work with their own data, if available|Kim Handley<br>David Waite|

### Day 2 - 11<sup>th</sup> December 2019

|Time|Event|Session leader|
|:---|:---|:---|
|9:00 am – 9:30 am|**Introduction**<br>Overview of yesterday, questions|Kim Handley|
|9:30 am – 10:30 am|**Evaluating the overnight assembly**<br>*TASK: Run evaluation tool/script*|David Waite|
|10:30 am – 10:50 am|**Morning tea break**||
|10:50 am – 11:20 am|**Overview of binning history**|Kim Handley|
|11:20 am – 12:00 pm|**Binning (part 1)**<br>*TASK: Short contig removal*<br>*TASK: Read mapping*|David Waite|
|12:00 pm – 12:45 pm|**Break for lunch**||
|12:45 pm – 1:15 pm|**Overview of binning history (continued)**<br>Key parameters and strategies for binning|Kim Handley|
|1:15 pm – 1:45 pm|**Binning (part 2)**<br>*TASK: Multi-binning strategy*|Kim Handley|
|1:45 pm - 3:00 pm|**Binning (part 3)**<br>*TASK: Bin dereplication via DAS_Tool*<br>*TASK: Evaluating bins using CheckM*|Kim Handley|
|3:00 pm – 3:20 pm|**Afternoon tea break**||
|3:20 pm – 4:00 pm|**Binning (part 4)**<br>Discuss additional dereplication strategies, such as **dRep**<br>How to work with viral and eukaryotic bins<br>Dealing with organisms which possess minimal genomes|Kim Handley<br>David Waite|
|4:00 pm – 5:00 pm|**End of day wrap up**<br>Attendees can work with their own data, if available|Kim Handley<br>David Waite|

### Day 3 - 12<sup>th</sup> December 2019

|Time|Event|Session leader|
|:---|:---|:---|
|9:00 am – 9:30 am|**Introduction**<br>Overview of yesterday, questions<br>Overview of today|David Waite|
|9:30 am – 11:30 am|**Bin refinement**<br>*TASK: Working with *VizBin***<br>Other strategies (*ESOMana*)|David Waite|
|10:30 am – 10:50 am|**Morning tea break**||
|10:50 am – 11:30 am|**Gene prediction**<br>Introduce **prodigal**, discuss single vs meta mode<br>Discuss what **prodigal** can't find, discuss other tools (**RNAmer**, **Aragorn**, etc)|Christina Straub|
|11:30 am – 12:00 pm|**Gene annotation (part 1)**<br>BLAST-like gene annotation using **usearch** or **diamond**<br>Introduce the different databases, highlight our reasons for *KEGG*<br>Evaluating the quality of gene assignment<br>Differences in taxonomies (*GTDB*, *NCBI* etc)|David Waite|
|12:00 pm – 12:45 pm|**Break for lunch**||
|12:45 pm – 3:00 pm|**Gene annotation (part 2)**)<br>*TASK: Overview of the BLAST XML output*<br>*TASK: Looking at gene networks in **MEGAN***<br>*TASK: Tie findings to your initial hypothesis*|David Waite|
|3:00 pm – 3:20 pm|**Afternoon tea break**||
|3:20 pm – 4:00 pm|**Gene annotation**<br>Using online resources (*KEGG*, *BioCyc*, *MetaCyc*, *HydDB*, *PSORT*)<br>*TASK: Tie findings to your initial hypothesis*|Kim Handley|
|4:00 pm – 5:00 pm|**End of day wrap up**<br>Attendees can work with their own data, if available|Kim Handley<br>David Waite|

### Day 4 - 13<sup>th</sup> December 2019

|Time|Event|Session leader|
|:---|:---|:---|
|9:00 am – 9:30 am|**Introduction**<br>Overview of yesterday, questions<br>Overview of today|Kim Handley|
|9:30 am – 10:30 am|**Gene annotation** (refresher)<br>*TASK: Tie findings to your initial hypothesis*|Kim Handley|
|10:30 am – 10:50 am|**Morning tea break**||
|10:50 am – 12:00 pm|**Present and discuss findings**<br>*TASK: Each group to give a casual discussion of their data*<br>*TASK: What they were looking for, what did they find?*|Kim Handley|
|12:00 pm – 12:45 pm|**Break for lunch**||
|12:45 pm – 3:00 pm|**Presentation of data**<br>How do visualise findings - Metabolism maps, heatmaps, gene trees*<br>*TASK: Gene synteny alignments and heatmaps*|David Waite|
|3:00 pm – 3:20 pm|**Afternoon tea break**||
|3:20 pm – 4:00 pm|**End of day wrap up**<br>Final discussion|Kim Handley<br>David Waite|

----

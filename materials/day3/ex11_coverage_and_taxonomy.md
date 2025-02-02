# Per-sample coverage and assigning taxonomy

### Objectives

* Calculate per-sample coverage stats for the filtered prokaryote bins
* Calculate per-sample coverage stats for the viral contigs output by `VIBRANT`
* Assign taxonomy to the refined bins
* Introduction to `vContact2` for predicting taxonomy of viral contigs
* *Appendix*: viral taxonomy prediction via `vContact2`

---

### Calculate per-sample coverage stats of the filtered prokaryote bins

One of the first questions we often ask when studying the ecology of a system is: What are the pattens of abundance and distribution of taxa across the different samples? With bins of metagenome-assembled genome (MAG) data, we can investigate this by mapping the quality-filtered unassembled reads back to the refined bins to then generate coverage profiles. Genomes in higher abundance in a sample will contribute more genomic sequence to the metagenome, and so the average depth of sequencing coverage for each of the different genomes provides a proxy for abundance in each sample. 

As per the preparation step at the start of the binning process, we can do this using read mapping tools such as `Bowtie`, `Bowtie2`, and `BBMap`. Here we will follow the same steps as before using `Bowtie2`, `samtools`, and `MetaBAT`'s `jgi_summarize_bam_contig_depths`, but this time inputting our refined filtered bins. 

These exercises will take place in the `8.coverage_and_taxonomy/` folder. Our final filtered refined bins from the previous bin refinement exercise have been copied to the `8.coverage_and_taxonomy/filtered_bins/` folder.

First, concatenate the bin data into a single file to then use to generate an index for the read mapper.

```bash
cd /nesi/nobackup/nesi02659/MGSS_U/<YOUR FOLDER>/8.coverage_and_taxonomy/

cat filtered_bins/*.fna > filtered_bins.fna
```

Now build the index for `Bowtie2` using the concatenated bin data. We will also make a new directory `bin_coverage/` to store the index and read mapping output into.

```bash
mkdir -p bin_coverage/

# Load Bowtie2
module load Bowtie2/2.3.5-GCC-7.4.0

# Build Bowtie2 index
bowtie2-build filtered_bins.fna bin_coverage/bw_bins
```

Map the quality-filtered reads (from `../3.assembly/`) to the index using `Bowtie2`, and sort and convert to `.bam` format via `samtools`.

Example slurm script:

```bash 
#!/bin/bash -e
#SBATCH -A nesi02659
#SBATCH --res SummerSchool
#SBATCH -J mapping_filtered_bins
#SBATCH --time 00:05:00
#SBATCH --mem 1GB
#SBATCH --ntasks 1
#SBATCH --cpus-per-task 10
#SBATCH -e mapping_filtered_bins.err
#SBATCH -o mapping_filtered_bins.out

module load Bowtie2/2.3.5-GCC-7.4.0 SAMtools/1.8-gimkl-2018b

cd /nesi/nobackup/nesi02659/MGSS_U/<YOUR FOLDER>/8.coverage_and_taxonomy/

# Step 1
for i in sample1 sample2 sample3 sample4;
do

  # Step 2
  bowtie2 --minins 200 --maxins 800 --threads 10 --sensitive \
          -x bin_coverage/bw_bins \
          -1 ../3.assembly/${i}_R1.fastq.gz -2 ../3.assembly/${i}_R2.fastq.gz \
          -S bin_coverage/${i}.sam

  # Step 3
  samtools sort -@ 10 -o bin_coverage/${i}.bam bin_coverage/${i}.sam

done
```

Finally, generate the per-sample coverage table for each contig in each bin via `MetaBAT`'s `jgi_summarize_bam_contig_depths`.

```bash 
# Load MetaBAT
module load MetaBAT/2.13-GCC-7.4.0

# calculate coverage table
jgi_summarize_bam_contig_depths --outputDepth bins_cov_table.txt bin_coverage/sample*.bam
```

The coverage table will be generated as `bins_cov_table.txt`. As before, the key columns of interest are the `contigName`, and each `sample[1-n].bam` column.

*NOTE: It is usually necessary to also normalise coverage values based on sample depth. For example, by normalising all coverages for each sample based on either minimum or average library size (the number of sequencing reads per sample). In this case, the mock metagenome data we have been working with were already of equal depth, and so we will omit any normalisation step here. But this would not be the case with your own sequencing data sets.*

*NOTE: Here we are generating a per-sample table of coverage values for **each contig** within each bin. To get per-sample coverage of **each bin** as a whole, we will need to generate average coverage values based on all contigs contained within each bin. We will do this in `R` during our data visualisation exercises on day 4 of the workshop, leveraging the fact that we added bin IDs to the sequence headers.*

---

### Calculate per-sample coverage stats of viral contigs

Here we can follow the same steps as outlined above for the bin data, but with a concatenated *fastA* file of viral contigs. 

To quickly recap: 

* In previous exercises, we first used `VIBRANT` to identify viral contigs from the assembled reads, generating a new fasta file of viral contigs: `spades_assembly.m1000.phages_combined.fna` 
* We then processed this file using `CheckV` to generate quality information for each contig, and to further trim any retained (prokaryote) sequence on the ends of prophage contigs. 

The resultant *fasta* files generated by `CheckV` (`proviruses.fna` and `viruses.fna`) have been copied to to the `8.coverage_and_taxonomy/checkv` folder for use in this exercise.

*NOTE: due to the rapid mutation rates of viruses, with full data sets it will likely be preferable to first further reduce viral contigs down based on a percentage-identity threshold using a tool such as `BBMap`'s `dedupe.sh`. This would be a necessary step in cases where you had opted for generating multiple individual assemblies or mini-co-assemblies (and would be comparable to the use of a tool like `dRep` for prokaryote data), but may still be useful even in the case of single co-assemblies incorporating all samples.*

We will first need to concatenate these files together.

```bash
cat checkv/proviruses.fna checkv/viruses.fna > checkv_combined.fna
```

Now build the index for `Bowtie2` using the concatenated viral contig data. We will also make a new directory `viruses_coverage/` to store the index and read mapping output into.

```bash
mkdir -p viruses_coverage/

# Load Bowtie2
module load Bowtie2/2.3.5-GCC-7.4.0

# Build Bowtie2 index
bowtie2-build checkv_combined.fna viruses_coverage/bw_viruses
```

Map the quality-filtered reads (from `../3.assembly/`) to the index using `Bowtie2`, and sort and convert to `.bam` format via `samtools`.

Example slurm script:

```bash 
#!/bin/bash -e
#SBATCH -A nesi02659
#SBATCH --res SummerSchool
#SBATCH -J mapping_filtered_viruses
#SBATCH --time 00:05:00
#SBATCH --mem 1GB
#SBATCH --ntasks 1
#SBATCH --cpus-per-task 10
#SBATCH -e mapping_filtered_viruses.err
#SBATCH -o mapping_filtered_viruses.out

module load Bowtie2/2.3.5-GCC-7.4.0 SAMtools/1.8-gimkl-2018b

cd /nesi/nobackup/nesi02659/MGSS_U/<YOUR FOLDER>/8.coverage_and_taxonomy/

# Step 1
for i in sample1 sample2 sample3 sample4;
do

  # Step 2
  bowtie2 --minins 200 --maxins 800 --threads 10 --sensitive \
          -x viruses_coverage/bw_viruses \
          -1 ../3.assembly/${i}_R1.fastq.gz -2 ../3.assembly/${i}_R2.fastq.gz \
          -S viruses_coverage/${i}.sam

  # Step 3
  samtools sort -@ 10 -o viruses_coverage/${i}.bam viruses_coverage/${i}.sam

done
```

Finally, generate the per-sample coverage table for each viral contig via `MetaBAT`'s `jgi_summarize_bam_contig_depths`.

```bash 
# Load MetaBAT
module load MetaBAT/2.13-GCC-7.4.0

# calculate coverage table
jgi_summarize_bam_contig_depths --outputDepth viruses_cov_table.txt viruses_coverage/sample*.bam
```

The coverage table will be generated as `viruses_cov_table.txt`. As before, the key columns of interest are the `contigName`, and each `sample[1-n].bam` column.

*NOTE: As mentioned above, it is usually necessary to also normalise coverage values based on sample depth. For example, by normalising all coverages for each sample based on either minimum or average library size (the number of sequencing reads per sample). In this case, the mock metagenome data we have been working with were already of equal depth, and so we will omit any normalisation step here. But this would not be the case with your own sequencing data sets.*

*NOTE: Unlike the prokaryote data, we have not used a binning process on the viral contigs (since many of the binning tools use hallmark characteristics of prokaryotes in the binning process). Here, `viruses_cov_table.txt` is the final coverage table. This can be combined with `CheckV` quality and completeness metrics to, for example, examine the coverage profiles of only those viral contigs considered to be "High-quality" or "Complete".* 

---

### Assign taxonomy to the refined bins

It is always valuable to know the taxonomy of our binned MAGs, so that we can link them to the wider scientific literature. In order to do this, there are a few different options available to us:

1. Extract 16S rRNA gene sequences from the MAGs and classify them
1. Annotate each gene in the MAG and take the consensus taxonomy
1. Use a profiling tool like `Kraken`, which matches pieces of DNA to a reference database using *k*-mer searches
1. Identify a core set of genes in the MAG, and use these to compute a species phylogeny

For this exercise, we will use the last option in the list, making use of the `GTDB-TK` software (available on [github](https://github.com/Ecogenomics/GTDBTk)) to automatically identify a set of highly conserved, single copy marker genes which are diagnostic of the bacterial (120 markers) and archaeal (122 markers) lineages. Briefly, `GTDB-TK` will perform the following steps on a set of bins.

1. Attempt to identify a set of 120 bacterial marker genes, and 122 archaeal marker genes in each MAG.
1. Based on the recovered numbers, identify which domain is a more likely assignment for each MAG
1. Create a concatenated alignment of the domain-specific marker genes, spanning approximately 41,000 amino acid positions
1. Filter the alignment down to approximately 5,000 informative sites
1. Insert each MAG into a reference tree create from type material and published MAGs
1. Scale the branch lengths of the resulting tree, as described in [Parks et al.](https://www.ncbi.nlm.nih.gov/pubmed/30148503), to identify an appropriate rank to each branch event in the tree
1. Calculate ANI and AAI statistics between each MAG and its nearest neighbours in the tree
1. Report the resulting taxonomic assignment, and gene alignment

This can all be achieved in a single command, although it must be performed through a slurm script due to the high memory requirements of the process.

```bash
#!/bin/bash
#SBATCH -A nesi02659
#SBATCH -J gtdbtk_test
#SBATCH --res SummerSchool
#SBATCH --time 00:30:00
#SBATCH --mem 140GB
#SBATCH --cpus-per-task 10
#SBATCH -e gtdbtk_test.err
#SBATCH -o gtdbtk_test.out

module load GTDB-Tk/0.2.2-gimkl-2018b-Python-2.7.16

cd /nesi/nobackup/nesi02659/MGSS_U/<YOUR FOLDER>/8.coverage_and_taxonomy

gtdbtk classify_wf -x fna --cpus 10 --genome_dir filtered_bins/ --out_dir gtdbtk_out/
```

As usual, lets look at the parameters here

|Parameter|Function|
|:---|:---|
|**classify_wf**|Specifies the sub-workflow from `GTDB-TK` that we wish to use|
|**-x ...**|Specify the file extension for MAGs within our input directory.<br>Default is *.fna*, but it's always good practice to specify it anyway|
|**--cpus ...**|Number of CPUs to use when finding marker genes, and performing tree insertion operations|
|**--genome_dir ...**|Input directory containing MAGs as individual *fastA* files|
|**--out_dir ...**|Output directory to write the final set of files|

Before submitting your job, think carefully about which set of MAGs you want to classify. You could either use the raw `DAS_Tool` outputs in the `../6.bin_refinement/dastool_out/_DASTool_bins/` folder, the renamed set of bins in the `../6.bin_refinement/example_data_unchopped/` folder, the set of curated bins in the `filtered_bins/` folder, or your own set of refined bins. Whichever set you choose, make sure you select the correct input folder and extension setting as it may differ from the example here.

When the task completes, you will have a number of output files provided. The main ones to look for are `gtdbtk.bac120.summary.tsv` and `gtdbtk.arch122.summary.tsv` which report the taoxnomies for your MAGs, split at the domain level. These file are only written if MAGs that fall into the domain were found in your data set, so for this exercise we do not expect to see the `gtdbtk.arch122.summary.tsv` file.

If you are interested in performing more detailed phylogenetic analysis of the data, the filtered multiple sequence alignment (MSA) for the data are provided in the `gtdbtk.bac120.msa.fasta` and `gtdbtk.arch122.msa.fasta` files.

Have a look at your resulting taxonomy. The classification of your MAGs will be informative when addressing your research goal for this workshop.

---

### Introduction to *vContact2* for predicting taxonomy of viral contigs

Even more so than prokaryote taxonomy, establishing a coherent system for viral taxonomy is complex and continues to evolve. Just in the last year, the International Committee on Taxonomy of Viruses ([ICTV](https://talk.ictvonline.org/)) overhauled the classification code into [15 hierarchical ranks](https://www.nature.com/articles/s41564-020-0709-x). Furthermore, the knowledge gap in databases of known and taxonomically assigned viruses remains substantial, and so identifying the putative taxonomy of viral contigs from environmental metagenomics data remains challenging.

There are a number of approaches that can be used to attempt to predict the taxonomy of the set of putative viral contigs output by programs such as `VIBRANT`, `VirSorter`, and `VirFinder`. [vContact2](https://www.nature.com/articles/s41587-019-0100-8) is one such method that uses 'guilt-by-contig-association' to predict the potential taxonomy of viral genomic sequence data based on relatedness to known viruses within a reference database (such as viral RefSeq). The principle is that, to the extent that the 'unknown' viral contigs cluster closely with known viral genomes, we can then expect that they are closely related enough to be able to predict a shared taxonomic rank. 

*NOTE: Anecdotally, however, in my own experience with this processes I have unfortunately been unable to predict the taxonomy of the vast majority of the viral contigs ouput by `VIBRANT`, `VirSorter`, or `VirFinder` from an environmental metagenomic data set (due to not clustering closely enough with known viruses in the reference database).*

Running `vContact2` can require a considerable amount of computational resources, and so we won't be running this in the workshop today. The required process is outlined in the [Appendix](#appendix-viral-taxonomy-prediction-via-vcontact2) below for reference, should you wish to experiment with this on your own data in the future. 

For today, we have provided the final two output files from this process when applied to our mock metagenome data. These can be viewed in the folder `8.coverage_and_taxonomy/vConTACT2_Results/` via `head` or `less`.

```bash
less vConTACT2_Results/genome_by_genome_overview.csv
```

```bash
less vConTACT2_Results/tax_predict_table.txt
```

A few notes to consider: 

* You will see that the `genome_by_genome_overview.csv` file contains entries for the full reference database used as well as the input viral contigs (contigs starting with `NODE`). 
* You can use a command such as `grep "NODE" vConTACT2_Results/genome_by_genome_overview.csv | less` to view only the lines for the input contigs of interest. 
  * Note also that these lines however will *not* contain taxonomy information. 
  * See the notes in the [Appendix](#appendix-viral-taxonomy-prediction-via-vcontact2) below for further information about why this might be.
* As per the notes in the [Appendix](#appendix-viral-taxonomy-prediction-via-vcontact2) below, the `tax_predict_table.txt` file contains *predictions* of potential taxonomy (and or taxonom*ies*) of the input viral contigs for order, family, and genus, based on whether they clustered with any viruses in the reference database.
  * Bear in mind that these may be lists of *multiple* potential taxonomies, in the cases where viral contigs clustered with multiple reference viruses representing more than one taxonomy at the given rank.
  * *NOTE: The taxonomies are deliberately enclosed in square brackets (`[ ]`) to highlight the fact that these are **predictions**, rather than definitive taxonomy **assignments**.

---

### Appendix: viral taxonomy prediction via *vContact2*

*NOTE: these steps are based on having `vContact2` set up as a `conda` environment. This documentation will be updated should `vContact2` become available as a NeSI module.*

Further information for installing and running of vContact2 can also be found [here](https://bitbucket.org/MAVERICLab/vcontact2/src/master/).

**1. Predict genes via `prodigal`**

```bash
cd /nesi/nobackup/nesi02659/MGSS_U/<YOUR FOLDER>/8.coverage_and_taxonomy
```

Example slurm script:

```bash
#!/bin/bash -e
#SBATCH -A nesi02659
#SBATCH -J prodigal
#SBATCH --time 00:05:00
#SBATCH --mem=1GB
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=2
#SBATCH -e prodigal.err
#SBATCH -o prodigal.out

# Load dependencies
module load Prodigal/2.6.3-GCC-9.2.0

# Set up working directories
cd /nesi/nobackup/nesi02659/MGSS_U/<YOUR FOLDER>/8.coverage_and_taxonomy
mkdir -p viral_taxonomy

# Run main analyses 
srun prodigal -p meta -q \
-i checkv_combined.fna \
-a viral_taxonomy/checkv_combined.faa 
```

**2. Generate required mapping file for `vContact2`**

Use `vContact2`'s `vcontact2_gene2genome` script to generate the required mapping file from the output of `prodigal`.

*NOTE: update `/path/to/conda/envs/vContact2/bin` in the below script to the appropraite path.*

```bash
# activate vcontact2 conda environment
module purge
module load Miniconda3
source activate vContact2

# Load dependencies
export PATH="/path/to/conda/envs/vContact2/bin:$PATH"
module load DIAMOND/0.9.32-GCC-9.2.0
module load MCL/14.137-gimkl-2020a

# run vcontact2_gene2genome
vcontact2_gene2genome -p viral_taxonomy/checkv_combined.faa -o viral_taxonomy/viral_genomes_g2g.csv -s 'Prodigal-FAA'

# deactivate conda environment
conda deactivate
```

**3. Run `vContact2`**

Example slurm script:

*NOTE: update `/path/to/conda/envs/vContact2/bin` and `/path/to/conda/envs/vContact2/bin/cluster_one-1.0.jar` in the below script to the appropraite paths.*

```bash
#!/bin/bash -e
#SBATCH -A nesi02659
#SBATCH -J vcontact2
#SBATCH --time 02:00:00
#SBATCH --mem=20GB
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=20
#SBATCH -e vcontact2.err
#SBATCH -o vcontact2.out

# Set up working directories
cd /nesi/nobackup/nesi02659/MGSS_U/<YOUR FOLDER>/8.coverage_and_taxonomy/viral_taxonomy/

# activate vcontact2 conda environment
module purge
module load Miniconda3
source activate vContact2

# Load dependencies
export PATH="/path/to/conda/envs/vContact2/bin:$PATH"
module load DIAMOND/0.9.32-GCC-9.2.0
module load MCL/14.137-gimkl-2020a

# Run vcontact2
srun vcontact2 \
-t 20 \
--raw-proteins checkv_combined.faa \
--rel-mode Diamond \
--proteins-fp viral_genomes_g2g.csv \
--db 'ProkaryoticViralRefSeq201-Merged' \
--c1-bin /path/to/conda/envs/vContact2/bin/cluster_one-1.0.jar \
--output-dir vConTACT2_Results

# deactivate conda environment
conda deactivate
```

**4. Predict taxonomy of viral contigs based on ouput of `vContact2`**

`vContact2` doesn't actually *assign* taxonomy to your input viral contigs. It instead provides an output outlining which reference viral genomes your viral contigs clustered with (if they clustered with any at all). Based on how closely they clustered with any reference genome(s), you can then use this to *predict* the likely taxonomy of the contig. 

From the `vContact2` online docs:

> One important note is that the taxonomic information is not included for user sequences. This means that each user will need to find their genome(s) of interest and check to see if reference genomes are located in the same VC. If the user genome is within the same VC subcluster as a reference genome, then there's a very high probability that the user genome is part of the same genus. If the user genome is in the same VC but not the same subcluster as a reference, then it's highly likely the two genomes are related at roughly genus-subfamily level. If there are no reference genomes in the same VC or VC subcluster, then it's likely that they are not related at the genus level at all.

The summary output of `vContact2` is the file `vConTACT2_Results/genome_by_genome_overview.csv`. As the comment above notes, one approach would be to search this file for particular contigs of interest, and see if any reference genomes fall into the same viral cluster (VC), using this reference to predict the taxonomy of the contig of interest.

The following `python` script is effectively an automated version of this for all input contigs (*Note: this script has not been widely tested, and so should be used with some degree of caution*). This script groups together contigs (and reference genomes) that fall into each VC, and then for each, outputs a list of all taxonomies (at the ranks of 'Order', 'Family', and 'Genus', separately) that were found in that cluster. The predictions (i.e. the list of all taxonomies found in the same VC) for each rank and each contig is output to the table `tax_predict_table.txt`. 

*NOTE: The taxonomies are deliberately enclosed in square brackets (`[ ]`) to highlight the fact that these are **predictions**, rather than definitive taxonomy **assignments**.*

For future reference, a copy of this script is available for download [here](https://github.com/GenomicsAotearoa/metagenomics_summer_school/blob/master/materials/scripts/vcontact2_tax_predict.py)

```bash
module load Python/3.8.2-gimkl-2020a

cd /nesi/nobackup/nesi02659/MGSS_U/<YOUR FOLDER>/8.coverage_and_taxonomy/

./vcontact2_tax_predict.py \
-i viral_taxonomy/vConTACT2_Results/genome_by_genome_overview.csv \
-o viral_taxonomy/
```

---

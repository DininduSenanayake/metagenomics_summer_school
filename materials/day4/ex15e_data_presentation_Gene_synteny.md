# Presentation of data: Gene synteny

### Objectives

* Build a sulfur assimilation gene alignment figure to investigate gene synteny using `R`

---

### Build a sulfur assimilation gene alignment figure to investigate gene synteny using `R`

When investigating the evolution of genomes, we sometimes want to consider not only the presence/absence of genes in a genome, but also how they are arranged in an operon. For this exercise, we are going to visualise several sulfur assimilation genes from bin_2, bin_3 and bin_4, comparing their arrangement among the organisms.

For this exercise, navigate to the folder `11.data_presentation/annotations/`. You have been provided with a copy of the `prodigal` gene predictions for each of the bins (`.faa` files), an annotation output table using multiple databases (`.aa` files), a small table of the annotation of some key genes of interest (`cys.txt` files), and blastn output (`blast*.txt`) comparing the genes of interest from these organisms. The annotation files were created by manually searching the annotations obtained in the previous exercises.

*NOTE: Refer to [gene_synteny_grab_GOI.md](https://github.com/GenomicsAotearoa/metagenomics_summer_school/blob/master/materials/resources/gene_synteny_grab_GOI.md) for more information on how the `cys.txt` files were produced.*

#### Part 1 - Parsing files in bash

We will be performing this exercise in two stages. Firstly, in `bash`, we will use `cut` and `tail` to pull out the genes of interest listed in the `*cys.txt` files from the `prodigal` files. The gene names will then be used to create a table of gene coordinates from the `prodigal` output using `grep`, `cut`, and `sed`.

For these `bash` steps, we will need to return to our logged in NeSI terminal. You can use the same terminal you have been using for the rest of the workshop if you wish. But fortunately, the NeSI `Jupyter hub` includes a terminal window as well. 

Launch a standard **terminal** within the NeSI [Jupyter hub](https://jupyter.nesi.org.nz/hub/login):

* click on the `+` button in the top left of the screen (directly below the `File` drop-down menu) to bring up the `Jupyter` launcher window
* launch the terminal using the bottom at the bottom left of the pane (under "Other"). 

From here, we can run the `bash` commands below.

First, navigate to the `11.data_presentation/annotations/` folder, and then perform the `cut` and `tail` steps outlined above.

```bash
cd /nesi/nobackup/nesi02659/MGSS_U/<YOUR FOLDER>/11.data_presentation/annotations/

cut -f3 bin_2_cys.txt | tail -n+2 > bin_2_cys.genes
cut -f3 bin_3_cys.txt | tail -n+2  > bin_3_cys.genes
cut -f3 bin_4_cys.txt | tail -n+2  > bin_4_cys.genes
```

We now have three new files, ending with the `.genes` suffix which are simply a list of the genes that we wish to extract from the `prodigal` files. We can then use each of these files as input for `grep` to pull out the *fastA* entries that correspond to these genes.

```bash
grep -f bin_2_cys.genes bin_2.genes.faa | cut -f1,2,3,4 -d "#" | sed 's/>//g' | sed 's/#/\t/g' > bin_2_cys.coords
grep -f bin_3_cys.genes bin_3.genes.faa | cut -f1,2,3,4 -d "#" | sed 's/>//g' | sed 's/#/\t/g' > bin_3_cys.coords
grep -f bin_4_cys.genes bin_4.genes.faa | cut -f1,2,3,4 -d "#" | sed 's/>//g' | sed 's/#/\t/g' > bin_4_cys.coords
```

As previously, we will quickly go through the steps of this command:

```bash
grep -f bin_2_cys.genes bin_2_cys.faa | cut -f1,2,3,4 -d "#" | sed 's/>//g' | sed 's/#/\t/g' > bin_2_cys.coords
|                                     |                      |              |                |
|                                     |                      |              |                Write the output
|                                     |                      |              |
|                                     |                      |              Replace each '#' character with tab
|                                     |                      |
|                                     |                      For each line, replace the '>' with empty text
|                                     |
|                                     Split the results into columns delimited by the # character,
|                                     then take columns 1 - 4.
| Select the lines of the bin_2_cys.faa file that contain entries found in the bin_2_cys.genes file.
```

Check the content of the `.coords` files now. You should see something like the following:

```bash
cat bin_2_cys.coords
# caef037ad318582c885d129aee6008b1_3761    3966783         3967781         1
# caef037ad318582c885d129aee6008b1_3762    3967943         3968761         1
# caef037ad318582c885d129aee6008b1_3763    3968772         3969641         1
# caef037ad318582c885d129aee6008b1_3764    3969645         3970634         1
```

If you recall from the previous exercise on gene prediction, we have taken the first four entries from each line of the `prodigal` output, which consists of:

1. The gene name, written as [CONTIG]\_[GENE]
1. The start position of the gene
1. The stop position of the gene
1. The orienation of the gene

We will now use these tables, together with the annotation tables to create the gene synteny view in `R`.

#### Part 2 - Producing the figure in *R*

First, move back to the `Jupyter Notebook` pane where we have `R` running as the kernel (or relaunch a new `R 4.0.1` Notebook). 

There are two `R` libaries we need to load for this exercise.

##### Set working directory and load *R* libraries

```R
setwd('/nesi/nobackup/nesi02659/MGSS_U/<YOUR FOLDER>/11.data_presentation/annotations/')

library(dplyr)
library(genoPlotR)
```
    
    Attaching package: ‘dplyr’
    
    The following objects are masked from ‘package:stats’:
    
        filter, lag
    
    The following objects are masked from ‘package:base’:
    
        intersect, setdiff, setequal, union
    
    Loading required package: ade4
    Loading required package: grid


##### Part 2.1 - Load coordinate files

We can now begin importing the data. First, we will import the `.coords` files, and set column names to the files.


```R
bin_2_coords = read.table('bin_2_cys.coords', header=F, sep='\t', stringsAsFactors=F)
colnames(bin_2_coords) = c('name', 'start', 'end', 'strand')
bin_3_coords = read.table('bin_3_cys.coords', header=F, sep='\t', stringsAsFactors=F)
colnames(bin_3_coords) = c('name', 'start', 'end', 'strand')
bin_4_coords = read.table('bin_4_cys.coords', header=F, sep='\t', stringsAsFactors=F)
colnames(bin_4_coords) = c('name', 'start', 'end', 'strand')
```

Take a look at the content of each of these data.frames, by entering their names into the terminal. You should notice that the coordinates occur at quite different positions between the genomes. If we were looking at complete genomes, this would indicate their position relative to the *origin of replication*, but as these are unclosed genomes obtained from MAGs, they reflect the coordinates upon their particular *contig*.

We now parse these data.frames into the *dna_seg* data class, which is defined by the `genoPlotR` library.


```R
bin_2_ds = dna_seg(bin_2_coords)
bin_3_ds = dna_seg(bin_3_coords)
bin_4_ds = dna_seg(bin_4_coords)
bin_4_ds
```


<table>
<caption>A dna_seg: 7 × 11</caption>
<thead>
	<tr><th scope=col>name</th><th scope=col>start</th><th scope=col>end</th><th scope=col>strand</th><th scope=col>col</th><th scope=col>fill</th><th scope=col>lty</th><th scope=col>lwd</th><th scope=col>pch</th><th scope=col>cex</th><th scope=col>gene_type</th></tr>
	<tr><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;chr&gt;</th></tr>
</thead>
<tbody>
	<tr><td>4f94130388f7df8dfba7383a12b9de4d_588 </td><td>625636</td><td>626724</td><td>-1</td><td>blue</td><td>blue</td><td>1</td><td>1</td><td>8</td><td>1</td><td>arrows</td></tr>
	<tr><td>4f94130388f7df8dfba7383a12b9de4d_589 </td><td>626721</td><td>627599</td><td>-1</td><td>blue</td><td>blue</td><td>1</td><td>1</td><td>8</td><td>1</td><td>arrows</td></tr>
	<tr><td>4f94130388f7df8dfba7383a12b9de4d_590 </td><td>627609</td><td>628442</td><td>-1</td><td>blue</td><td>blue</td><td>1</td><td>1</td><td>8</td><td>1</td><td>arrows</td></tr>
	<tr><td>4f94130388f7df8dfba7383a12b9de4d_591 </td><td>628508</td><td>629296</td><td>-1</td><td>blue</td><td>blue</td><td>1</td><td>1</td><td>8</td><td>1</td><td>arrows</td></tr>
	<tr><td>4f94130388f7df8dfba7383a12b9de4d_592 </td><td>629387</td><td>630298</td><td>-1</td><td>blue</td><td>blue</td><td>1</td><td>1</td><td>8</td><td>1</td><td>arrows</td></tr>
	<tr><td>4f94130388f7df8dfba7383a12b9de4d_593 </td><td>630410</td><td>630715</td><td>-1</td><td>blue</td><td>blue</td><td>1</td><td>1</td><td>8</td><td>1</td><td>arrows</td></tr>
	<tr><td>4f94130388f7df8dfba7383a12b9de4d_594 </td><td>630774</td><td>631781</td><td>-1</td><td>blue</td><td>blue</td><td>1</td><td>1</td><td>8</td><td>1</td><td>arrows</td></tr>
</tbody>
</table>


By looking at the table, we can see that the genes in bin_4 are in reversed order (strand = -1), so we might want to change the color of the gene to make sure they are different than bin_2 and bin_3.


```R
bin_4_ds$col = "#1a535c"
bin_4_ds$fill = "#1a535c"
```

##### Part 2.3 - Load annotation tables

Then, we can load the annotation tables we have into `R` and take a look at them.


```R
bin_2_ann = read.table('bin_2_cys.txt', header=T, sep='\t', stringsAsFactors=F) %>%
cbind(., bin_2_coords) %>%
select(Annotation, start1=start, end1=end, strand1=strand)
bin_3_ann = read.table('bin_3_cys.txt', header=T, sep='\t', stringsAsFactors=F) %>%
cbind(., bin_3_coords) %>%
select(Annotation, start1=start, end1=end, strand1=strand)
bin_4_ann = read.table('bin_4_cys.txt', header=T, sep='\t', stringsAsFactors=F) %>%
cbind(., bin_4_coords) %>%
select(Annotation, start1=start, end1=end, strand1=strand)
bin_2_ann
bin_4_ann
```


<table>
<caption>A data.frame: 4 × 4</caption>
<thead>
	<tr><th scope=col>Annotation</th><th scope=col>start1</th><th scope=col>end1</th><th scope=col>strand1</th></tr>
	<tr><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><td>sbp; sulfate-binding protei                                       </td><td>3966783</td><td>3967781</td><td>1</td></tr>
	<tr><td>cysT; sulfate transporter CysT                                    </td><td>3967943</td><td>3968761</td><td>1</td></tr>
	<tr><td>cysW; sulfate transporter CysW                                    </td><td>3968772</td><td>3969641</td><td>1</td></tr>
	<tr><td>cysA; sulfate.thiosulfate ABC transporter ATP-binding protein CysA</td><td>3969645</td><td>3970634</td><td>1</td></tr>
</tbody>
</table>




<table>
<caption>A data.frame: 7 × 4</caption>
<thead>
	<tr><th scope=col>Annotation</th><th scope=col>start1</th><th scope=col>end1</th><th scope=col>strand1</th></tr>
	<tr><th scope=col>&lt;chr&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th><th scope=col>&lt;dbl&gt;</th></tr>
</thead>
<tbody>
	<tr><td>cysA; sulfate transport ATP-binding ABC transporter protein</td><td>625636</td><td>626724</td><td>-1</td></tr>
	<tr><td>cysW; sulfate transport ABC transporter protein            </td><td>626721</td><td>627599</td><td>-1</td></tr>
	<tr><td>cysU; sulfate transport ABC transporter protein            </td><td>627609</td><td>628442</td><td>-1</td></tr>
	<tr><td>Domain of unknown function 2                               </td><td>628508</td><td>629296</td><td>-1</td></tr>
	<tr><td>Domain of unknown function 2                               </td><td>629387</td><td>630298</td><td>-1</td></tr>
	<tr><td>possible predicted diverged CheY-domain                    </td><td>630410</td><td>630715</td><td>-1</td></tr>
	<tr><td> sbp1; Prokaryotic sulfate-/thiosulfate-binding protein    </td><td>630774</td><td>631781</td><td>-1</td></tr>
</tbody>
</table>



We need to create one more table with descriptive information. This is an *annotation* object, which contains the name of each gene in the figure along with the coordinates to write the name. x1 = starts of the gene, x2 = end of the gene


```R
annot1 <- annotation(x1 = c(bin_2_ds$start[1], bin_2_ds$start[2],bin_2_ds$start[3],bin_2_ds$start[4]),
                    x2 = c(bin_2_ds$end[1], bin_2_ds$end[2], bin_2_ds$end[3], bin_2_ds$end[4]),
                    text = c("sbp", "cysT","cysW","cysA"),
                    rot = 0, col = "black")
annot2 <- annotation(x1 = c(bin_4_ds$start[1], bin_4_ds$start[2],bin_4_ds$start[3],bin_4_ds$start[4],bin_4_ds$start[7]),
                    x2 = c(bin_4_ds$end[1], bin_4_ds$end[2], bin_4_ds$end[3], bin_4_ds$end[6], bin_4_ds$end[7]),
                    text = c("cysA","cysW","cysU","unknown domain" ,"sbp"),
                    rot = 0, col = "black")
```

##### Part 2.3 - Creating a comparison table

Then, we can parse the blast output as comparison file among the genomes. genoPlotR can read tabular files, either user-generated tab files (read_comparison_from_tab), or from BLAST output (read_comparison_from_blast). To produce files that are readable by genoPlotR, the -m 8 or 9 option should be used in blastall, or -outfmt 6 or 7 with the BLAST+ suite.


```R
blast1 = read_comparison_from_blast("blast_bin2_bin3.txt")
blast2 = read_comparison_from_blast("blast_bin3_bin4.txt")
```

What it does here is to set the line color according to the direction of the gene match and the color gradient is based on the percent identity of the matches. Lighter color indicates weaker match and darker color indicates strong match.

Now we can generate the plot. Running this command in `RStudio` or our `Jupyter Notebook` running the `R` kernel interactively loads the figure. 

*NOTE: the commented-out lines below (the two lines starting with `#`) will not run. Un-comment these if you wish to save the figure to file rather than opening it in the `Jupyter` or `RStudio` viewer. (N.b. alternatives using a similar command are also available other formats, including `tiff` and `png`).

```R
#png("genoplot.png", width = 8, height = 4, res=300)
plot_gene_map(dna_segs = list(bin_2_ds,bin_3_ds,bin_4_ds), 
              gene_type = "arrows", dna_seg_labels = c("bin_2", "bin_3","bin_4"), 
              comparisons = list(blast1,blast2), dna_seg_scale = TRUE, 
              override_color_schemes=TRUE,annotations=list(annot1,NULL,annot2),
              annotation_height=1.7, dna_seg_label_cex = 1,main = "Sulfur assimilation")
#dev.off()
```


![png](https://github.com/GenomicsAotearoa/metagenomics_summer_school/blob/MGSS2020_DEV/materials/figures/ex15_gene_synteny_fig1.png)


Careful analysis would be needed to determine whether this is a genuine rearrangement relative to the rest of the genome, as these are draft genomes and contig orientation can either be forward or reverse. In this case, you can see that genes in bin_4 are in reversed order relative to the other bin contigs, hence, we can manually rotate the contig.

##### Rotate the contig and update the annotation

Rotate the orientation of the contig from bin_4:

```R
blast2 = mutate(blast2, direction = 1)

annot3 <- annotation(x1 = c(bin_4_ds$start[1], bin_4_ds$start[2],bin_4_ds$start[3],bin_4_ds$start[4],bin_4_ds$start[7]),
                    x2 = c(bin_4_ds$end[1], bin_4_ds$end[2], bin_4_ds$end[3], bin_4_ds$end[6], bin_4_ds$end[7]),
                    text = c("cysA","cysW","cysU","unknown domain" ,"sbp"),
                    rot = 0, col = "black")
#edit the color
bin_4_ds$col = "blue"
bin_4_ds$fill = "blue"
```

Regenerate the figure:

```R
#AFTER ROTATING
#png("genoplot_rotated.png", width = 8, height = 4, res=300)
plot_gene_map(dna_segs = list(bin_2_ds,bin_3_ds,bin_4_ds), 
              gene_type = "arrows", dna_seg_labels = c("bin_2", "bin_3","bin_4"), 
              comparisons = list(blast1,blast2), dna_seg_scale = TRUE, 
              override_color_schemes=TRUE,annotations=list(annot1,NULL,annot3),
              annotation_height=1.7, dna_seg_label_cex = 1,xlims=list(NULL, NULL, 
              c(Inf, -Inf)),main = "Sulfur assimilation")
#dev.off()
```


![png](https://github.com/GenomicsAotearoa/metagenomics_summer_school/blob/MGSS2020_DEV/materials/figures/ex15_gene_synteny_fig2.png)


All done! We can see here that compared to bin_2 and bin_3 the following differences are apparent in the gene operon for bin_4:

1. Three genes with unknown domain/function present in bin_4 in between *cysU* and *SBP*
2. *cysW* and *cysA* are highly conserved among the genomes and have higher similarity compared to *cysU* and *SBP*.

---

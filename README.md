# bulkRNAseq

This is the conceptual framework for my bulk RNA-seq data analysis pipeline. All bulk RNA-seq data is derived from FAC-sorted cells from the mouse (mm10) main olfactory epithelium after olfactory fear conditioning. This is a two-factor study with two groups in the primary factor (condition) and six groups in the secondary factor (timepoints). Other information is provided in the sample table (sex, age, etc). 

- Species: Mus musculus (mm10) 
- Tissue: main olfactory epithelium (sorted)
- Cell Populations: GFP+ ("green") and GFP- ("negative") 
- Conditions: paired (experimental) and unpaired (control) 
- Timepoints: 1-day, 3-day, 7-day, 10-day, 14-day (19-day pending) 

While I have bulk RNA-seq data from the GFP- cell population, I am focusing on the GFP+ cell population in all of my analysis. In my experience, the GFP- cell population contains too much of a diversity of cell types to get any meaningful signal above the noise. Sequencing the GFP- cell population will require single-cell RNA-seq, in which we can better isolate signal coming from specific cell types. Single-cell RNA-seq experiments are pending. :) 

Keep in mind that I collected data from all timepoints independent of one another, so for that reason: 1) batch correction for timepoint is not possible (that would have required me to prepare all timepoints in the same batch, which would have been a technical nightmare), and 2) up until now, I have analyzed all timepoints independent of one another. Thus far, I have performed all of my existing analysis using the DESeq2 R package. DESeq2 normalizes for size factor and dispersion to fit a negative binomial generalized linear model and identify differentially expressed genes (DEGs) between the paired and unpaired conditions at each timepoint. 

(Love, M.I., Huber, W. & Anders, S. Moderated estimation of fold change and dispersion for RNA-seq data with DESeq2. Genome Biol 15, 550 (2014). https://doi.org/10.1186/s13059-014-0550-8) 

Now I want to combine all timepoints into one massive dataset and do a complete meta-analysis. I'm hoping to extract some information from it, despite not being able to do a proper batch correction. I plan to use a few different but similar tools to look at this big dataset: 1) weighted gene co-expression network analysis (WGCNA), 2) pathway analysis of gene expression (iPAGE), and 3) finding informative regulatory elements (FIRE). iPAGE and FIRE will be done together. 

# WGCNA: Weighted Gene Co-expression Network Analysis 

(derived from Langfelder, P., Horvath, S. WGCNA: an R package for weighted correlation network analysis. BMC Bioinformatics 9, 559 (2008). https://doi.org/10.1186/1471-2105-9-559 and refino.bio tutorial) 

Tutorial: https://alexslemonade.github.io/refinebio-examples/04-advanced-topics/network-analysis_rnaseq_01_wgcna.html 

WGCNA is used for: 

- finding clusters (modules) of highly correlated genes
- visualizing clusters 
- relating modules to each other 

WGCNA requires at least 15 samples to produce meaningful results. Thankfully I have 72 samples (as of 1/14/2024). I plan to run all 72 samples in the first WGCNA analysis, and then I'll follow that with running only the GFP+ samples. 

The input for WGCNA is bulk RNA-seq data that has been normalized and transformed with DESeq2 (with the counts() and vst() functions, respectively). 

Download/load packages: tidyverse (which contains ggplot2), DESeq2, magrittr, impute, WGCNA 

Create two files: 1) sample table and 2) raw counts matrix. The sample table contains all metadata about each sample (library name, sort ("GFP" or "negative"), treatment ("unpaired" or "paired"), timepoint, sex, age in weeks) and the path to each sample's associated bam file. Then, take the information from the sample table to create a raw counts matrix with i genes (rows) x j samples (columns), which shows the unnormalized count of each genes in each sample. Remove rows of genes with no mapped reads in any sample. (Note: If we need to toggle with WGCNA, we can raise the cutoff here. Instead of removing just the genes with no mapped reads in any sample, we can also remove the genes with less than 50 total counts across all samples, or another selected cutoff.) 

Take the raw counts matrix, and normalize all counts. There are several ways to normalize counts in an RNA-seq dataset. FPKM/RPKM normalizes for read depth and transcript length, but it has largely been replaced in the literature with TPM. TPM is good for comparing gene expression within a sample (i.e. gene A is transcribed higher than gene B within sample X), but it is not good for comparing gene expression between samples (i.e. gene A is transcribed higher in sample X than in sample Y). To compare gene expression between samples, use the DESeq2 Normalized Counts method with the counts() function, which normalizes each gene's expression to that gene's expression across all samples. (Disclaimer: When using Normalized Counts, keep the transcript length in mind. Longer transcripts will appear to have higher expression than shorter transcripts, proportional to the transcript length. But that shouldn't change the DEGs because DESeq() is designed to be a cross-sample comparison.) 

After normalizing counts, transform the counts using the vst() function. 

HERE IS WHERE I'M STUCK. I now need to use functions from the WGCNA package, but I'm unable to properly install WGCNA on the Aphrodite server. I have tried exporting the dds_norm dataset (normalized and transformed) and opening it on my own computer, but my version of R/RStudio doesn't seem to support WGCNA package installation either. 

But once I get unstuck, the next steps are: 

1) Format normalized data for WGCNA:

Extract normalized counts to a matrix and transpose it for WGCNA input. 

2) Determine parameters for WGCNA:

To identify modules, WGCNA needs us to input the appropriate power parameter. WGCNA has the pickSoftThreshold() function to help us decide this parameter. Based on the parameter we choose, WGCNA will decide which genes are close to each other and delineate discrete modules of correlated genes. (Note: Read the section 4.6 of refine.bio tutorial for specific tips on picking the right parameter.) 

3) Run WGCNA:

Run the largest block the computer can handle, and give it time. 

4) Explore and plot results:

This part will take time too. Work on visualizing the data in every possible way. :) 

See how other papers are visualizing WGCNA data: 

Li, J., Zhou, D., Qiu, W. et al. Application of Weighted Gene Co-expression Network Analysis for Data from Paired Design. Sci Rep 8, 622 (2018). https://doi.org/10.1038/s41598-017-18705-z

Niu X, Zhang J, Zhang L, Hou Y, Pu S, Chu A, Bai M and Zhang Z (2019) Weighted Gene Co-Expression Network Analysis Identifies Critical Genes in the Development of Heart Failure After Acute Myocardial Infarction. Front. Genet. 10:1214. doi: 10.3389/fgene.2019.01214 

Farhadian, M., Rafat, S.A., Panahi, B. et al. Weighted gene co-expression network analysis identifies modules and functionally enriched pathways in the lactation process. Sci Rep 11, 2367 (2021). https://doi.org/10.1038/s41598-021-81888-z 

Bioinformagician also made a WGCNA tutorial on Youtube: https://www.youtube.com/watch?v=gYE59uEMXT4&t=71s 

# iPAGE/FIRE 

(derived from conversations with Saeed Tavazoie and his tools at https://iget.c2b2.columbia.edu/) 

iPAGE reference: Goodarzi 2009: https://doi.org/10.1016/j.molcel.2009.11.016

FIRE reference: Elemento 2007: https://doi.org/10.1016/j.molcel.2007.09.027 

This analysis method doesn't really have a working tutorial, but we'll figure it out nonetheless. Saeed proposed this method to me as an agnostic way to analyze my bulk RNA-seq data. It seems fairly similar to WGCNA, but it's different in a few key ways: 

The goal of iPAGE is to identify pathways that may be perturbed in the experimental condition (paired) and to pinpoint the regulators of those pathways. iPAGE is based on the concept of mutual information (MI) (Cover and Thomas 2006), which detects dependencies between observations. iPAGE quantifies MI between pathways and expression profiles. 

The goal of FIRE is to identify cis-regulatory elements that best explain gene expression in the data sets. 

Then, compare the deregulated pathways uncovered by iPAGE and the regulatory elements uncovered by FIRE to see if there's crossover and/or discovery of putative binding sites. 

To install and use iPAGE: https://tavazoielab.c2b2.columbia.edu/iPAGE/tutorial.html 

1) Create a tab-delimited text file with transcript-IDs of all genes in the first column (must be REFSEQ IDs) and gene expression levels from all samples in each column thereafter. Normalize (and transform?) this data set. See if I can take dds_norm from the WGCNA analysis, replace the gene names in the first column with transcript-IDs, and use that as input for iPAGE. 

2) Cluster this matrix! Try various k-values for clustering.

3) Use the cluster ID numbers as iPAGE input, and see what comes up. 

To install and use FIRE: https://tavazoielab.c2b2.columbia.edu/FIRE/tutorial.html 

Same steps as iPAGE. 

Compare iPAGE x FIRE results! 

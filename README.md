# bulkRNAseq

This is the conceptual framework for my bulk RNA-seq data analysis pipeline. All bulk RNA-seq data is derived from FAC-sorted cells from the mouse (mm10) main olfactory epithelium after olfactory fear conditioning. This is a two-factor study with two groups in the primary factor (condition) and six groups in the secondary factor (timepoints). Other information is provided in the sample table (sex, age, etc). 

Species: Mus musculus (mm10) 
Tissue: main olfactory epithelium (sorted)
Cell Populations: GFP+ ("green") and GFP- ("negative") 
Conditions: paired (experimental) and unpaired (control) 
Timepoints: 1-day, 3-day, 7-day, 10-day, 14-day (19-day pending) 

While I have bulk RNA-seq data from the GFP- cell population, I am focusing on the <u>GFP+ cell population</u> in all of my analysis. In my experience, the GFP- cell population contains too much of a diversity of cell types to get any meaningful signal above the noise. Sequencing the GFP- cell population will require single-cell RNA-seq, in which we can better isolate signal coming from specific cell types. 

Keep in mind that I collected data from all timepoints independent of one another, so for that reason: 1) batch correction is not possible (that would have required me to prepare samples from all timepoints in the same batch, which would have been a technical nightmare), and 2) up until now, I have analyzed all timepoints independent of one another. 

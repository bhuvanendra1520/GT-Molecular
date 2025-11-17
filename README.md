# ctDNA variant analysis for metastatic colorectal cancer


This repository contains solution and my approach for GT Molecular Bioinformatics Scientist Fall 2025 take home exercise. The main goal of this exercise is to analyze circulating tumor DNA from one colorectal cancer patient, compare pre and post treatment samples, and perform a focused variant analysis on a resistance related gene of interest.

This analysis uses patient CTC347 from the NCBI database.

> Circulating tumor DNA sequencing in colorectal cancer patients treated with first line chemotherapy with anti EGFR  
> BioProject PRJNA714799

---

## Patient and samples

Patient selected  
* CTC347

Samples used  
* Pre treatment ctDNA  SRR13973720  
* Post treatment ctDNA SRR13973721

## Methods overview

I followed a standard ctDNA variant calling workflow

1. Data retrieval from SRA using `prefetch` and `fasterq dump` for the two ctDNA samples  
2. Quality control and trimming with `fastp`  
3. Alignment to the hg38 reference genome with `bwa mem`  
4. Reference indexing with `bwa`, `samtools`, and `picard`  
5. Group assignment and duplicate removal with `picard`  
6. Variant calling with `gatk Mutect2` in tumor only mode for each sample  
7. Extraction of non header VCF records into plain text for downstream processing  
8. Collapsing variants to genomic keys based on `CHROM POS REF ALT` to avoid treating the same variant as different when INFO or FORMAT fields change  
9. Set based comparison of pre versus post variant keys to define  
   * shared variants  
   * variants lost after treatment  
   * variants gained after treatment  


## Gene focused analysis KRAS

* KRAS mutations are central to resistance against anti EGFR therapy in colorectal cancer.
* Emergence of KRAS positive subclones in ctDNA after treatment is a known mechanism of acquired resistance.


In this notebook I have defined:

1. Define the KRAS locus on hg38 chromosome 12  
2. Parse the Mutect2 VCF files and extract only variants that fall inside the KRAS genomic region  
3. Compute allele fractions and depth for these variants based on sample level format fields  
4. Join the KRAS variants with the global key based classification to tag each as shared, lost, or gained  
5. Build a pre vs post table of KRAS allele fractions and visualize them as paired bars

In patient CTC347 all KRAS locus variants are detected only in the post treatment sample. No KRAS variants are called in the pre treatment sample at the thresholds used. Several post treatment KRAS variants show high fractions above 0.6, indicating that KRAS positive subclones became major contributors to total ctDNA after therapy.

This pattern is consistent with therapy driven selection of KRAS mutant resistant clones under anti EGFR treatment.

---

## Running in Google Colab

To make the analysis easy, the notebook is designed to run in Google Colab starting from compact result files.

Typical workflow consists of:

1. Place the repository folder in Google Drive under a convenient path  
2. Open `notebooks/GTM.ipynb` in Colab  
3. Run the cell that mounts Google Drive  
4. Run all cells from top to bottom

The notebook will

* load the two VCF files from the folder  
* regenerate `pre.keys` and `post.keys` if needed  
* recompute `shared.keys`, `lost_after_treatment.keys`, and `gained_after_treatment.keys`  
* extract and visualize KRAS variants


---

## Interpretation summary

Key findings for patient CTC347

* The global variant profile changes dramatically between pre and post treatment ctDNA, with many variants lost and many more gained.
* When variants are collapsed by genomic position and allele, a relatively small core set remains shared across both time points.
* Within the KRAS locus, no variants are detected before therapy and multiple variants appear only after therapy, often with high fractions.

Taken together, these observations support a narrative of strong clonal evolution and selection under anti EGFR therapy, with KRAS driven resistant subclones emerging and expanding over the course of treatment.

---

## Ideas for marketing and presentation visuals

This notebook concludes with a short vision section that outlines how these results could be communicated to non technical stakeholders. Suggested visuals include

* A simple pre vs post bar chart of KRAS variant allele fractions  
* A timeline style slide showing treatment, sampling time points, and emergence of KRAS mutations  
* A schematic of ctDNA shedding and resistant clone expansion under targeted therapy  
* A single patient summary figure that could be reused across marketing or educational material to illustrate the value of ctDNA monitoring

These ideas are described in text and can be used as a starting point for design ready graphics.


## Acknowledgements

I would like to acknowledge GT Molecular for creating this take home exercise and for the opportunity to demonstrate my ability to design, run, and interpret a complete ctDNA bioinformatics workflow. The problem is well aligned with real diagnostic pipelines and provided a meaningful way to showcase both technical and biological reasoning.

I also acknowledge the authors of the study:

*Circulating tumor DNA sequencing in colorectal cancer patients treated with first line chemotherapy with anti EGFR*  
BioProject: PRJNA714799  whose publicly accessible datasets enabled this analysis.

To build foundational intuition around the workflow steps used in this project, I reviewed several freely available educational resources, including:

- **"How to download sequencing data from SRA NCBI using SRA Toolkit"**  
  https://www.youtube.com/watch?v=zE851fWCYQg

- **"WGS Variant Calling: Variant calling with GATK - Part 1 | Detailed NGS Analysis Workflow"**  
  https://www.youtube.com/watch?v=iHkiQvxyr5c

- **"Genomic Variants | Dr Disha Sharma"**  
  https://www.youtube.com/watch?v=JEdjyXgKtKM

These videos helped me undertsand the intution behind each preprocessing and variant calling step, including FASTQ retrieval, alignment, duplicate marking, and GATK based variant detection.

All tools used throughout the analysis—`fastp`, `bwa`, `samtools`, `picard`, `gatk Mutect2`, and standard Python packages—are open source, and I acknowledge the developers and maintainers who make reproducible genomics work possible.

Lastly, I acknowledge the broader scientific community for its commitment to open data and transparent methods, which allows meaningful exploration of real-world patient samples and the biological stories they reveal.


# 16S Soil Microbiome Analysis — Project Summary

## Project goal

Develop a reproducible and reusable Nextflow workflow for the analysis of
16S rRNA amplicon sequencing data from vineyard soil.

The biological question is:

> Does soil management (tillage vs cover crop/no-tillage) affect soil microbial community composition in a non-irrigated Xynisteri vineyard at harvest?

The workflow is designed so that the same computational pipeline can be
applied to additional 16S datasets by changing the input data and metadata.

---

## Study design

Current dataset:

* 8 soil samples
* 4 tillage samples
* 4 cover crop samples
* Cultivar: Xynisteri
* Irrigation: no irrigation
* Sampling stage: harvest
* Paired-end Illumina MiSeq 16S amplicon sequencing

---

## Workflow

```text
Raw FASTQ
   |
   v
FastQC
   |
   v
Cutadapt
   |
   v
FastQC after trimming
   |
   v
DADA2 filtering
   |
   v
Error-rate learning
   |
   v
DADA2 denoising
   |
   v
Paired-end merging
   |
   v
ASV abundance table
   |
   v
Taxonomic assignment
   |
   v
Community analysis
   |
   v
Statistical analysis
```

---

## Data organization

Raw sequencing data are stored locally in:

```text
data/raw/
```

Raw FASTQ files are intentionally excluded from GitHub.

Sample metadata are stored in:

```text
assets/samplesheet.csv
```

Example:

```text
sample,err,treatment,irrigation,cultivar,stage
ERR4702541,ERR4702541,tillage,no_irrigation,Xynisteri,harvest
ERR4702533,ERR4702533,cover_crop,no_irrigation,Xynisteri,harvest
```

The metadata are kept separate from the workflow logic so that the pipeline
can be reused with other datasets.

---

## Implemented workflow

### 1. Input and metadata

The workflow reads the sample metadata and associates each sample with its
paired-end FASTQ files.

Sample metadata are propagated through the Nextflow channels together with
the sequencing data.

### 2. Quality control

FastQC is run on the raw sequencing reads.

Outputs:

* FastQC HTML reports
* FastQC ZIP reports

### 3. Adapter trimming

Cutadapt removes sequencing adapters from paired-end reads.

### 4. Post-trimming quality control

FastQC is run again on the trimmed reads.

### 5. DADA2 filtering

Reads are filtered using DADA2 based on:

* truncation length
* expected errors
* ambiguous bases
* quality scores
* PhiX removal

Filtering statistics are saved for each sample.

### 6. Error-rate learning

DADA2 learns forward and reverse sequencing error models using the filtered
reads from all samples.

### 7. Denoising

DADA2 performs sample-level denoising and infers amplicon sequence variants
(ASVs).

### 8. Paired-end merging

Forward and reverse denoised reads are merged using DADA2.

### 9. ASV table

Merged ASVs are combined across samples into a single abundance matrix.

Current output:

```text
results/dada2/table/asv_table.rds
results/dada2/table/asv_table.tsv
```

Current dataset:

* 8 samples
* 8,953 ASVs
* 625,399 total reads

---

## Planned analysis

### 10. Chimera removal

Remove chimeric sequences using DADA2 and generate a final non-chimeric ASV
table.

### 11. Taxonomic assignment

Assign taxonomy to ASVs using an appropriate reference database.

Potential reference databases:

* SILVA
* GTDB
* other validated 16S databases

Output:

```text
ASV → Kingdom → Phylum → Class → Order → Family → Genus → Species
```

### 12. Community dataset

Combine:

```text
ASV abundance table
        +
taxonomy
        +
sample metadata
```

to create the main dataset for ecological analysis.

### 13. Alpha diversity

Calculate within-sample diversity metrics such as:

* Observed ASVs
* Shannon diversity
* Simpson diversity

Compare diversity between:

```text
Tillage
vs
Cover crop
```

### 14. Beta diversity

Calculate between-sample community dissimilarity using metrics such as:

* Bray-Curtis
* Jaccard
* UniFrac, if phylogenetic information is available

Visualize community structure using:

* PCoA
* NMDS

### 15. Statistical analysis

Test whether microbial community composition differs between soil management
conditions.

The analysis should account for relevant experimental variables and
potential confounders.

### 16. Visualization

Generate figures including:

* Alpha diversity plots
* PCoA plots
* Taxonomic composition plots
* Relative abundance plots
* Differential abundance plots, where appropriate

Figures will be stored in:

```text
results/figures/
```

### 17. Final report

Summarize:

* dataset and metadata
* sequencing quality
* filtering
* DADA2 processing
* ASV composition
* taxonomy
* diversity
* statistical analysis
* biological interpretation

---

## Reproducibility

The complete workflow can be executed with:

```bash
nextflow run main.nf
```

After modifying the workflow, previously completed processes can be reused
with:

```bash
nextflow run main.nf -resume
```

Nextflow caching allows completed processes to be reused when their inputs
and process definitions have not changed.

---

## Reusability

For a new 16S dataset, the main inputs should be:

```text
data/raw/
assets/samplesheet.csv
```

along with parameters appropriate for the sequencing experiment.

The workflow is designed to separate:

```text
INPUT DATA
     +
METADATA
     +
WORKFLOW LOGIC
     +
RESULTS
```

This allows the same workflow to scale from a small pilot dataset to larger
experiments without rewriting the complete analysis.

---

## Current project status

### Completed

* [x] Public 16S dataset selected
* [x] Sample metadata defined
* [x] Raw FASTQ organization
* [x] Nextflow DSL2 workflow
* [x] Metadata/FASTQ association
* [x] FastQC on raw reads
* [x] Cutadapt adapter trimming
* [x] FastQC after trimming
* [x] DADA2 quality filtering
* [x] DADA2 error-rate learning
* [x] DADA2 denoising
* [x] Paired-end read merging
* [x] ASV abundance table

### Next steps

* [ ] Chimera removal
* [ ] Taxonomic assignment
* [ ] Combine ASV table with taxonomy and metadata
* [ ] Alpha diversity
* [ ] Beta diversity / PCoA
* [ ] Statistical testing
* [ ] Taxonomic visualization
* [ ] Final figures
* [ ] Final report

---

## Key Nextflow principle

The workflow separates:

```text
INPUT DATA
    +
METADATA
    +
WORKFLOW LOGIC
    +
RESULTS
```

This makes the analysis:

* reproducible
* scalable
* reusable
* traceable
* easier to maintain
* suitable for larger datasets and HPC environments


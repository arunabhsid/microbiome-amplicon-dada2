# microbiome-amplicon-dada2

This is a reproducible **16S rRNA amplicon** processing workflow using [DADA2](https://benjjneb.github.io/dada2/): raw paired-end FASTQ files to an **Amplicon Sequence Variant (ASV) table** with **taxonomy**, which can be used for downstream analysis in R (phyloseq/vegan). Scripts have been adapted from the official DADA2 tutorial. 

The pipeline runs end-to-end on a small public example dataset in a few minutes on a laptop, and is written to be adapted to your own 16S data by changing a single config block.

> The example data is public and downloaded with the commands below; for your own study you point the script at your own FASTQ files.

---

## Final output

| Output | Description |
|---|---|
| `ASV_counts.tsv` | Feature table: ASVs × samples (read counts) |
| `ASV_taxonomy.tsv` | Taxonomy (Kingdom → Species) for each ASV |
| `ASVs.fasta` | ASV ID → DNA sequence |
| `track_reads.tsv` | Reads surviving each step (QC sanity check) |
| `plots/` | Read-quality and error-model diagnostic plots |

---

## Repository contents

```
microbiome-amplicon-dada2/
├── README.md
├── .gitignore
└── dada2_16S_tutorial.R              
```

---

## Prerequisites

- **R** (≥ 4.x) and the **DADA2** package:
  ```r
  if (!requireNamespace("BiocManager", quietly = TRUE)) install.packages("BiocManager")
  BiocManager::install("dada2")
  ```
- A terminal with `wget` or `curl` and `unzip`
  (Windows: use WSL or Git Bash; in R set `multithread = FALSE`).

---

## Step 1: Download the example data

The example is the **mothur MiSeq SOP** dataset: mouse-gut samples, 16S **V4** region, 2×250 Illumina MiSeq (Schloss lab). Primers have already been removed.

```bash
# raw FASTQ files
wget https://mothur.s3.us-east-2.amazonaws.com/wiki/miseqsopdata.zip
unzip miseqsopdata.zip          # creates a folder: MiSeq_SOP/
# (alternatively, use: curl -O https://mothur.s3.us-east-2.amazonaws.com/wiki/miseqsopdata.zip)
```

You should now have `MiSeq_SOP/` containing files such as `F3D0_S188_L001_R1_001.fastq.gz`.

## Step 2: Download the reference database (SILVA 138.2, DADA2-formatted)

Place both files inside the `MiSeq_SOP/` folder:

```bash
cd MiSeq_SOP
wget https://zenodo.org/records/14169026/files/silva_nr99_v138.2_toGenus_trainset.fa.gz
wget https://zenodo.org/records/14169026/files/silva_v138.2_assignSpecies.fa.gz
cd ..
```

> Download links occasionally change. If one fails, get the current URLs from the DADA2 pages:
> [example data](https://benjjneb.github.io/dada2/tutorial.html) · [reference databases](https://benjjneb.github.io/dada2/training.html).
> The current SILVA DADA2 files classify **bacteria and archaea only** (not eukaryotes).

## Step 3: Configure and run

Edit the CONFIG block at the top of `dada2_16S_tutorial.R` so the paths point at your `MiSeq_SOP/` folder, then:

```bash
Rscript dada2_16S_tutorial.R
```

Results are written to the output folder set in the config (default `MiSeq_SOP/dada2_out/`).

---

## Running on your own data

The code mostly remains same, only the **parameters** will need to be changed. The essentials:

1. **Remove primers first** (e.g. with `cutadapt`)
2. **Set `truncLen` from your own quality plots**, and make sure forward + reverse reads still **overlap** after truncation (or merging fails).
3. **Match the region/primers** to what was sequenced (V4, V3–V4, V1–V2 …).
4. **Read `track_reads.tsv` every run** - it's the fastest way to catch a bad parameter.

---


## Citation

If you use this workflow, please cite:

- Callahan BJ, McMurdie PJ, Rosen MJ, et al. (2016) *DADA2: High-resolution sample inference from Illumina amplicon data.* **Nature Methods** 13:581–583.
- Quast C, Pruesse E, Yilmaz P, et al. (2013) *The SILVA ribosomal RNA gene database project.* **Nucleic Acids Res** 41:D590–D596.
- Example data: Kozich JJ, Westcott SL, Baxter NT, et al. (2013), mothur MiSeq SOP (Schloss lab).


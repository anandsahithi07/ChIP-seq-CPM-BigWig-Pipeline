# ChIP-seq CPM BigWig Pipeline

## Project summary

This project processes paired-end ChIP-seq FASTQ data and creates CPM-normalized
BigWig tracks for genome-browser visualization in Integrated Genome Browser
(IGB). It trims reads, aligns them to the *Schizosaccharomyces pombe* reference,
removes PCR duplicates, and converts the resulting alignments into coverage
tracks. The scripts also include optional MACS3 peak calling.

In short: **raw ChIP-seq reads → cleaned alignments → CPM-normalized BigWig
tracks → visualization in IGB**.

## Workflow

For each configured sample, the pipeline:

1. Finds matching paired-end FASTQ files (`R1`/`R2`, or `_1`/`_2` naming).
2. Trims adapters and low-quality bases with Trim Galore.
3. Aligns the trimmed reads with Bowtie2.
4. Converts and processes alignments with Samtools: fix mate information,
   coordinate-sort, remove PCR duplicates, and index the final BAM.
5. Runs deepTools `bamCoverage` to create a BigWig using CPM normalization and
   10-base bins.
6. In the comprehensive script, calls peaks with MACS3 and exports results to
   the Windows Downloads folder.
7. Removes large intermediate SAM/BAM and trimmed-read files after processing.

CPM means **counts per million mapped reads**. It scales coverage by sequencing
depth to make track values more comparable across samples. The BigWig is a
compact, indexed coverage format suitable for genome browsers.

## Inputs and configuration

- Paired-end compressed FASTQ files (`.fastq.gz`).
- A matching *S. pombe* Bowtie2 reference index. The scripts expect its prefix
  at `./reference/spombe_index`.
- Sample IDs and file locations configured in the script.

The scripts currently use WSL/Linux paths, including a raw-data directory under
`/mnt/c/Users/venka/Downloads/` and an export location under the Windows
Downloads folder. **Update these paths and the sample ID list for your own
computer and experiment before running.** Check that each sample ID matches its
FASTQ filenames and that the reference genome matches the assembly you plan to
use in IGB.

## Software requirements

Run the pipeline in a Linux or WSL environment with:

- Python 3
- Trim Galore (and its required Cutadapt/FastQC dependencies)
- Bowtie2
- Samtools
- deepTools (`bamCoverage`)
- MACS3 (only for the comprehensive script's peak-calling step)

The scripts in this folder are saved as `.txt` records and may contain terminal
session text as well as Python code. They are **not necessarily directly
executable as-is**. Before running, extract and review the intended Python
script, save it as a `.py` file, and confirm its paths, sample IDs, and commands.
Run it from the project directory so relative folders such as `reference/` and
`bigwig/` resolve as expected.

## Output files

The comprehensive pipeline writes results into these project folders:

- `bigwig/` — CPM-normalized BigWig files, named like
  `<sample>_sample_cpm.bw`.
- `bam_files/` — duplicate-removed BAM files and their `.bai` indexes.
- `macs3_peaks/` — MACS3 peak output (when peak calling is enabled).

The scripts also copy some or all final files to the configured Windows
Downloads folder. Temporary trimmed reads and intermediate alignments may be
deleted when a sample finishes.

## View BigWig tracks in IGB

1. Open IGB and select the genome assembly that exactly matches the reference
   used for Bowtie2 alignment.
2. Open or load the `.bw` file for a sample (for example,
   `2UM_sample_cpm.bw`) using IGB's local-file loading option.
3. Navigate to a chromosome or genomic region of interest. The coverage values
   are displayed as a signal track.
4. Load additional sample BigWig files to compare their coverage in the same
   region. Adjust track height or display settings as needed.

BigWig coordinates are meaningful only with the matching genome assembly.
Using a different assembly can make the signal appear at incorrect locations or
not display.

## Notes

- BigWig coverage is generated from the duplicate-removed BAM with
  `--normalizeUsing CPM --binSize 10`.
- Peak calling is a separate analysis from BigWig generation. Review the
  MACS3 treatment/control design and genome-size setting for the experiment
  before interpreting peaks.
- Check the run log for missing FASTQ pairs or failed commands before assuming
  that all configured samples completed successfully.

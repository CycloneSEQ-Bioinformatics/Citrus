![image](https://github.com/user-attachments/assets/47158133-d60d-4eca-8f8c-aa34eed11b94)
# Citrus
This tool is designed to efficiently demultiplex barcodes from sequencing data. It streamlines the preprocessing of genomic data by demultiplexing barcodes, enhancing the accuracy of downstream analyses.

## No Installation Required  
**Ready to Use After Extraction**  

This software is designed for ease of use without the need for a complicated installation process. Simply extract the contents of the software package to your preferred location on your system, and it's ready to go.  

**How to Start:**  
1. Download the `.tar.gz` file that matches your CPU architecture (`x86_64` or `aarch64`).
2. Extract the contents to a desired location.  
3. Run the `citrus` to start the software.  

The release binaries are dynamically linked against glibc and require glibc 2.28 or later on the host (RHEL 8/9, Rocky 8/9, Debian 10+, Ubuntu 20.04+, ARM servers). No other external dependency is needed.  

## Example Usage  
**Mode 1 barcode demultiplexing**  
```bash
citrus -m 1 -f sample.fq.gz -b barcodes.txt -o output_dir -n sample -e 0.2 -s 150,100 -t threads
```
**Mode 2 barcode demultiplexing**  
```bash
citrus -m 2 -f sample.fq.gz -b barcodes.txt -o output_dir -n sample -e 0.2 -s 150,100 -t threads
```
**Multiple input files**  
```bash
citrus -m 1 -f sample1.fq.gz sample2.fq.gz -b barcodes.txt -o output_dir -n sample
```
**BAM input**  
Unmapped BAM files carrying MM/ML tags are accepted; all other tags are passed through unchanged.  
```bash
citrus -m 1 --bam sample.bam -b barcodes.txt -o output_dir -n sample
```
**Predefined barcode kit**  
```bash
citrus -m 2 -f sample.fq.gz -k CycloneSEQ_24_Barcode_Library_Prep_Set -o output_dir -n sample
```

## Command-line Arguments  
| Name | Description | Required | Default |
| :---: | --- | :---: | :---: |
| `-m, --mode` | Set a mode to demultiplex barcodes. 1: demultiplex the 5' end barcode and conditionally demultiplex the 3' end barcode if identified; 2: demultiplex barcodes at both the 5' and 3' ends | Yes | - |
| `-f, --fastq` | Input one or more fastq files, separated by spaces | Yes¹ | - |
| `--bam` | Input one or more unmapped BAM files carrying MM/ML tags, separated by spaces | Yes¹ | - |
| `-b, --barcode` | Input one or more barcode files. If there are both-ends barcodes in a read, please separate them with a tab in the input file. When both-ends barcodes complement each other in reverse, only the 5' end barcode is needed. For reads with two barcodes on one end, the barcode files should be comma-separated | Yes² | - |
| `-k, --kits` | Use a predefined barcode kit. Mutually exclusive with `-b` | Yes² | - |
| `-q, --testseq` | Input a test sequence file | No | - |
| `-o, --outdir` | Output directory | Yes | - |
| `-n, --sample` | Sample name | Yes | - |
| `-L, --min_len` | Sequences shorter than the minimum length will be directly classified as QC-failed | No | 0 |
| `-Q, --min_qual` | Read quality lower than the minimum quality will be directly classified as QC-failed | No | 7.0 |
| `-e, --err` | Threshold of Levenshtein Distance value (integer) or sequencing error rate (float value between 0 and 1). Use a comma to separate two numbers, when there are different thresholds for the 5' and 3' ends | No | 0.2,0.2 |
| `-s, --shift` | Threshold of shift length for identifying barcode sequences. Use a comma to separate two numbers, when there are different thresholds for the 5' and 3' ends | No | 150,100 |
| `-u, --trim_len` | Set trimmed length for barcode recognition when 3' end sequencing is incomplete | No | 0 |
| `-c, --seed_size` | Seed size | No | 6 |
| `-d, --step_size` | Step size | No | 1 |
| `-t, --thread` | Number of threads | No | 4 |
| `-r, --retain` | Retain barcode sequences on reads | No | - |
| `-g, --degenerate` | Allow degenerate bases in barcodes and primers | No | - |
| `-i, --intermediate` | Output intermediate results | No | - |
| `-F, --filter_chimeric` | Filter chimeric reads | No | - |
| `-a, --append` | Append to existing output files instead of overwriting them | No | - |
| `-h, --help` | Print help | No | - |
| `-V, --version` | Print version | No | - |

¹ Either `--fastq` or `--bam` is required; the two are mutually exclusive.  
² Either `--barcode` or `--kits` is required; the two are mutually exclusive.

**Note on defaults:** when a predefined kit is used with `-k`, the kit's tuned parameters replace the defaults listed above. Parameters you set explicitly on the command line always take precedence.

## Barcode File
The barcode file must contain two or three tab-separated columns: `ID`, `5' barcode sequence`, and optionally `3' barcode sequence`. The `ID` can be the number of a barcode, or names like sample name, species name, or tissue name, etc. Additionally, one `ID` can correspond to multiple pairs of barcode sequences.  
As shown in the example, if the `ID` is `BC1`, the barcode file would be structured as follows:  
<table><tr><td>BC1</td><td>ATCG</td><td>TCAG</td></tr></table>

![image](https://github.com/user-attachments/assets/89bc2b4f-f1e0-4725-8827-80d8e8a2623f)

**Single-end barcodes** — one barcode per line; the same sequence is used for both ends:

```
barcode_id	barcode_sequence
BC01	ATCGATCG
BC02	GCTAGCTA
```

**Dual-end barcodes** — two barcodes per line:

```
barcode_id	5p_barcode	3p_barcode
BC01	ATGC	GGTA
BC02	TTGCATCA	CCTAGGTA
```

When the 5' and 3' adapters are different, put both adapters for the same barcode ID on one line. Citrus automatically recognizes both sequencing orientations for that adapter pair. For example:

```
BC01	ATGC	GGTA
```

This configuration matches forward reads such as:

```
5' ATGC ... TACC 3'
```

where `TACC` is the reverse complement of `GGTA`. It also matches the opposite orientation:

```
5' GGTA ... GCAT 3'
```

where `GCAT` is the reverse complement of `ATGC`. Both orientations are written to the `BC01` output. Use `--mode 2` when both the 5' and 3' adapters must be detected; use `--mode 1` when reads may be classified by the 5' adapter only.

**Avoid mirrored pairs.** If the 5' and 3' adapters are drawn from one shared pool and every combination is listed, the pairs `(i, j)` and `(j, i)` are duplicates:

```
barcode_id	5p_barcode	3p_barcode
LB1_LB1	ATCGATCG	ATCGATCG
LB1_LB2	ATCGATCG	GCTAGCTA
LB2_LB2	GCTAGCTA	GCTAGCTA
LB2_LB1	GCTAGCTA	ATCGATCG
```

Citrus recognizes both sequencing orientations of every adapter pair, so `LB1_LB2` and `LB2_LB1` end up with exactly the same matching rule. A read carries one adapter at each end, and the adapter pair `ATCGATCG` / `GCTAGCTA` cannot be told apart from the mirrored pair `GCTAGCTA` / `ATCGATCG`, so the reads of both IDs are written to whichever ID sorts first by name while the mirrored ID stays empty. No warning is emitted.

> **Note.** List only one of the two orderings, e.g. every `i <= j` combination (N(N+1)/2 IDs for N barcodes: 300 for a 24-barcode pool, 4656 for a 96-barcode pool). Reads of either orientation are then reported under that single ID.
>
> Listing all combinations also repeats each 5' adapter across many IDs, so a read ends up with many equally good 5' candidates. In `--mode 1` this disables the fallback that classifies a read by its 5' adapter alone, and reads whose 3' adapter cannot be matched are discarded as LD-filtered. Use `--mode 2` for this layout.

## Predefined Barcode Kits
Use `-k` instead of `-b` to demultiplex with a built-in barcode kit:

| Kit | Alias | Barcodes |
| :---: | --- | :---: |
| `CycloneSEQ_24_Barcode_Library_Prep_Set` | `24_Ligation_Barcodes` | 24 |
| `CycloneSEQ_96_Barcode_Library_Prep_Set` | `96_Ligation_Barcodes` | 96 |
| `CycloneSEQ_Rapid_24_Barcode_Library_Prep_Set` | - | 24 |

## Output
```
output/
├── {sample}_{barcode_id}/
│   ├── {sample}_{barcode_id}_01.fastq.gz            # Demultiplexed reads
│   └── {sample}_{barcode_id}_01.fastq.stats.tsv     # Per-barcode read/base counts
├── Discarded/
│   ├── {sample}.length_filtered.fq.gz               # Length-filtered reads
│   ├── {sample}.qval_filtered.fq.gz                 # Quality-filtered reads
│   ├── {sample}.ld_filtered.fq.gz                   # Edit-distance-filtered reads
│   └── {sample}.chimeric.fq.gz                      # Chimeric reads
├── Intermediate_results/                            # (with -i)
│   ├── {sample}.splitted_record.tsv
│   ├── {sample}.ld_filtered_record.tsv
│   └── {sample}.chimeric_record.tsv
├── {sample}_demux.stats.txt                         # Detailed statistics
├── {sample}_barcodes.stats.tsv                      # Per-barcode statistics table
└── {sample}.barcode_cumulative.json                 # Cumulative JSON summary
```

For BAM input, the demultiplexed and discarded outputs use the `.bam` extension instead of `.fastq.gz`/`.fq.gz`, and the per-barcode statistics file is named `{sample}_{barcode_id}_01.bam.stats.tsv`.

### Demultiplexed and discarded reads
`{sample}_{barcode_id}_01.fastq.gz` holds the reads assigned to that barcode, in standard FASTQ format (four lines per read) and gzip-compressed. The barcode sequences are trimmed from the read ends by default; `--retain` keeps them in the output.

The four files under `Discarded/` collect the reads that did not reach a barcode, one file per category:

| File | Reads it contains |
|---|---|
| `{sample}.length_filtered.fq.gz` | shorter than `--min_len` |
| `{sample}.qval_filtered.fq.gz` | mean quality lower than `--min_qual` |
| `{sample}.ld_filtered.fq.gz` | no barcode matched within the `--err` threshold |
| `{sample}.chimeric.fq.gz` | reads detected as chimeric; this category is only detected when `-F` is used, and the file stays empty otherwise |

Without `-F`, chimeric reads are not detected and stay in the demultiplexed output of the barcode they matched.

### `{sample}_{barcode_id}_01.fastq.stats.tsv`
Located in each barcode sub-directory, a TSV file with a header line and one data row:

```
TotalReadNum	TotalBaseNum	PassBaseNum
{read_count}	{base_count}	{pass_base_count}
```

| Column | Meaning |
|--------|---------|
| `TotalReadNum` | reads assigned to this barcode (same as `Read_count` below) |
| `TotalBaseNum` | total bases of those reads, counted on the original read length |
| `PassBaseNum` | bases left after barcode trimming (same as `Passed_base_count` below) |

`TotalReadNum` and `TotalBaseNum` are not affected by `--retain`; `PassBaseNum` is, and equals `TotalBaseNum` when `--retain` is used. The values are cumulative, i.e. they include the runs appended earlier with `-a`. Discarded categories have no sub-directory and produce no such file.

### `{sample}_barcodes.stats.tsv`
A per-barcode TSV table: one header line, then one row per classified barcode. The discarded categories are not listed. Rows are sorted by barcode ID in natural order — digit runs are compared numerically, so `barcode2` precedes `barcode10`.

**Normal mode (default) columns:**

| Column | Meaning |
|--------|---------|
| `Barcode` | barcode ID |
| `Read_count` | number of reads assigned to this barcode |
| `Base_count` | total bases for this barcode, summed over original read lengths |
| `Passed_base_count` | total bases of the retained sequence after trimming |
| `Read_N50` | N50 computed from the post-trim read length distribution |
| `Read_mean_Q` | mean read Q of the barcode, taken from the Q value carried in each read ID and averaged in error-rate space, weighted by the reads' base counts (two decimals). Reads whose ID carries no parseable Q are excluded; the value is `0.00` when none of the barcode's reads carries a Q |

**Accuracy mode (`-q`) appends 8 columns after the above 6:**

| Column | Meaning |
|--------|---------|
| `True_read_count` | reads verified as correctly classified |
| `False_read_count` | reads verified as misclassified |
| `Short_read_count` | reads too short to validate |
| `Chimeric-I_read_count` | chimeric reads detected by seed density |
| `Chimeric-II_read_count` | chimeric reads detected by matching two test sequences |
| `Unknown_read_count` | unknown / undetermined reads |
| `Accuracy(%)` | `True_read_count` as a percentage of validated reads |
| `Index_hopping_rate(%)` | `False_read_count` as a percentage of validated reads |

**Effect of `--retain` on this file:** `Read_count` and `Base_count` are unchanged; `Passed_base_count` and `Read_N50` increase, because they are computed over the sequence after barcode trimming, which `--retain` leaves intact.

### `{sample}_demux.stats.txt`
A full statistics report in plain text, made of three parts:

1. **Per-barcode table**: the same columns as `{sample}_barcodes.stats.tsv` (including the accuracy columns in accuracy mode), sorted by barcode ID in the same natural order.
2. **Type / Read / Base summary table**: cross-barcode totals, one row per category, with read count, base count, and percentage of the total:

   | Row | Meaning |
   |-----|---------|
   | `Total` | all reads / bases |
   | `Demultiplexed_count` / `Demultiplexed_percentage` | successfully assigned to a barcode |
   | `Length-filtered_count` / `Length-filtered_percentage` | length filter |
   | `Qval-filtered_count` / `Qval-filtered_percentage` | quality filter |
   | `LD-filtered_count` / `LD-filtered_percentage` | edit-distance filter |
   | `Chimeric-filtered_count` / `Chimeric-filtered_percentage` | chimera filter |
   | `CV` | coefficient of variation of read / base counts across barcodes |

   Accuracy mode appends `Accuracy(%)` and `Index_hopping_rate(%)` rows.
3. **Barcode distribution matrix** (accuracy mode only): a confusion matrix with true barcodes as rows and predicted barcodes as columns, showing index hopping flows. Rows and columns follow the same natural barcode-ID order as the per-barcode table.

### `{sample}.barcode_cumulative.json`
A cumulative JSON summary, **generated only in normal mode** (not in accuracy mode). Top-level fields:

| Field | Meaning |
|-------|---------|
| `run_total_passed_reads` / `run_total_passed_bases` | total QC-passing reads / bases |
| `run_classified_reads` / `run_classified_bases` | classified read / base counts |
| `run_unclassified_reads` / `run_unclassified_bases` | unclassified (discarded) read / base counts |
| `run_classified_reads_rate` / `run_classified_bases_rate` | corresponding rates |
| `run_unclassified_reads_rate` / `run_unclassified_bases_rate` | corresponding rates |
| `run_barcode_cumulatvie_info` | array, one entry per barcode |

Each entry in `run_barcode_cumulatvie_info` describes one barcode:

| Field | Meaning |
|-------|---------|
| `barcode_id` | barcode ID |
| `sequence` | the barcode sequence registered for this ID |
| `reads` / `bases` | read and base counts of this barcode |
| `reads_rate` / `bases_rate` | share of the classified reads / bases held by this barcode |
| `split_rate` | reserved field, currently always `0.0` |
| `reads_q_val` | modal read Q of this barcode |
| `total_reads_base` | bases left after barcode trimming |
| `N50` | N50 of the post-trim read length distribution |
| `q_bins` / `q_counts` | read quality histogram: the bin edges (80.0 to 100.0 in steps of 0.1) and the number of reads in each bin |

This file supports accumulation across runs with `-a`.

### `Intermediate_results/` (with `-i`)
Three TSV files, each with a header line and one row per read:

| File | Columns |
|------|---------|
| `{sample}.splitted_record.tsv` | `Record_ID`, `Barcode_ID`, `LD_of_5_end`, `End_pos_of_5_end`, `Seed_match_pos`, `LD_of_3_end`, `End_pos_of_3_end` |
| `{sample}.ld_filtered_record.tsv` | `Record_ID`, `Barcode_ID`, `LD`, `Seed_match_pos` |
| `{sample}.chimeric_record.tsv` | `Record_ID`, `Seed_count_per_win` |

These files are meant for visualisation and troubleshooting of individual reads.

## Performance
As shown in the diagram, sequencing data size is 10.37G, average read length is 693.26bp, the number of barcodes is 24, and 10 threads were used.  
**Note:** Memory usage increases with the average read length, and runtime increases with the number of barcodes.  
<img width="1000" height="600" alt="memory_usage" src="https://github.com/user-attachments/assets/f0339535-9cbf-41c5-a429-d8890973f11a" />

## Authors
夏小双 Xiaoshuang Xia (xiaxiaoshuang@genomics.cn)

## License and Usage Restrictions
**Research Use Only**  

This software is provided strictly for individual research purposes. Commercial use is strictly prohibited. This means:  
**Allowed:** Personal academic research, personal learning, and non-commercial experimentation.  
**Not Allowed:** Any form of commercial application, distribution, or use that generates revenue directly or indirectly. This includes, but is not limited to, integration into commercial products, offering this software as a service, or using it for commercial gain.  

For commercial licensing or permissions, please contact us.

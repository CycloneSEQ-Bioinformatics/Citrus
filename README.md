![image](https://github.com/user-attachments/assets/47158133-d60d-4eca-8f8c-aa34eed11b94)
# Citrus
This tool is designed to efficiently demultiplex barcodes from sequencing data. It streamlines the preprocessing of genomic data by demultiplexing barcodes, enhancing the accuracy of downstream analyses.

## No Installation Required  
**Ready to Use After Extraction**  

This software is designed for ease of use without the need for a complicated installation process. Simply extract the contents of the software package to your preferred location on your system, and it's ready to go.  

**How to Start:**  
1. Download the `.tar.gz` file.
2. Extract the contents to a desired location.  
3. Run the `citrus` to start the software.  

## Example Usage  
**Mode 1 barcode demultiplexing**  
`citrus -m 1 -f sample.fq.gz -b barcodes.txt -o output_dir -n sample -e 0.2 -s 150,100 -t threads`  
**Mode 2 barcode demultiplexing**  
`citrus -m 2 -f sample.fq.gz -b barcodes.txt -o output_dir -n sample -e 0.2 -s 150,100 -t threads`

## Command-line Arguments  
| Name | Description | Required | Default |
| :---: | --- | :---: | :---: |
| `-m, --mode` | Set a mode to demultiplex barcodes. 1: demultiplex the 5' end barcode and conditionally demultiplex the 3' end barcode if identified; 2: demultiplex barcodes at both the 5' and 3' ends | Yes | - |
| `-f, --fastq` | Input a fastq file | Yes | - |
| `-b, --barcode` | Input one or more barcode files. If there are both-ends barcodes in a read, please separate them with a tab in the input file. When both-ends barcodes complement each other in reverse, only the 5' end barcode is needed. For reads with two barcodes on one end, the barcode files should be comma-separated | Yes | - |
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
| `-h, --help` | Print help | No | - |
| `-V, --version` | Print version | No | - |

## Barcode File
The barcode file must contain three tab-separated columns: `ID`, `5' barcode sequence`, and `3' barcode sequence`. The `ID` can be the number of a barcode, or names like sample name, species name, or tissue name, etc. Additionally, one `ID` can correspond to multiple pairs of barcode sequences.  
As shown in the example, if the `ID` is `BC1`, the barcode file would be structured as follows:  
<table><tr><td>BC1</td><td>ATCG</td><td>TCAG</td></tr></table>

![image](https://github.com/user-attachments/assets/89bc2b4f-f1e0-4725-8827-80d8e8a2623f)

## Versions
This tool offers two versions:  
**GNU version (recommended):** Faster and more memory-efficient. Requires a recent version(>=2.35) of glibc on your Linux system.  
**Musl version:** Highly portable and compatible with a wider range of Linux systems, including those with older glibc versions.  
As shown in the diagram, sequencing data size is 4.66G, the number of barcodes is 24, and 10 threads were used.  
![9d0d6bc7154580c998284ee3de8468d](https://github.com/user-attachments/assets/6004304c-2ea0-4d88-97b6-98df846a8dee)

## Authors
夏小双 Xiaoshuang Xia (xiaxiaoshuang@genomics.cn)

## License and Usage Restrictions
**Research Use Only**  

This software is provided strictly for individual research purposes. Commercial use is strictly prohibited. This means:  
**Allowed:** Personal academic research, personal learning, and non-commercial experimentation.  
**Not Allowed:** Any form of commercial application, distribution, or use that generates revenue directly or indirectly. This includes, but is not limited to, integration into commercial products, offering this software as a service, or using it for commercial gain.  

For commercial licensing or permissions, please contact us.

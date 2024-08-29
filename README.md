![image](https://github.com/user-attachments/assets/47158133-d60d-4eca-8f8c-aa34eed11b94)
# Citrus
This tool is designed to efficiently split barcodes from sequencing data. It streamlines the preprocessing of genomic data by separating barcodes, enhancing the accuracy of downstream analyses.

## No Installation Required  
**Ready to Use After Extraction**  

This software is designed for ease of use without the need for a complicated installation process. Simply extract the contents of the software package to your preferred location on your system, and it's ready to go.  

**How to Start:**  
1. Download the `.zip` file.
2. Extract the contents to a desired location.  
3. Run the `citrus` to start the software.  

## Example Usage
**Single-end barcode splitting**  
`citrus -m 1 -f sample.fq.gz -b barcodes.txt -o output_dir -n sample -e 6 -s 150 -t threads`  
**Paired-end barcode splitting**  
`citrus -m 2 -f sample.fq.gz -b barcodes.txt -o output_dir -n sample -e 6,6 -s 150,50 -t threads`

## Command-line Arguments
```
Usage: citrus [OPTIONS] --mode <mode> --fastq <fastq file> --barcode <barcode file> --outdir <output dir> --sample <sample>

Options:
  -m, --mode <mode>              Set a mode to split barcodes. 1: split the 5' end barcode and conditionally split the 3' end
                                 barcode if identified; 2: split barcodes at both the 5' and 3' ends [possible values: 1, 2]
  -f, --fastq <fastq file>       Input a fastq file
  -b, --barcode <barcode file>   Input a barcode file. If there are multiple barcodes in a read, please separate them with a tab
                                 in the input file. When two barcodes complement each other in reverse, only the 5' end barcode
                                 is needed
  -q, --testseq <test seq file>  Input a test sequence file
  -o, --outdir <output dir>      Output directory
  -n, --sample <sample>          Sample name
  -e, --err <err threshold>      Threshold of Levenshtein Distance value or sequencing error rate. Use a comma to separate two
                                 numbers, when there are different thresholds for the 5' and 3' ends [default: 6]
  -s, --shift <shift threshold>  Threshold of shift length for identifying barcode sequences. Use a comma to separate two
                                 numbers, when there are different thresholds for the 5' and 3' ends [default: 150,50]
  -u, --trim_len <trim len>      The length of the 3' end barcode sequence to be trimmed [default: 0]
  -c, --seed_size <seed size>    Seed size [default: 5]
  -d, --step_size <step size>    Step size [default: 1]
  -t, --thread <thread>          Number of threads [default: 4]
  -r, --retain                   Retain barcode sequences on reads
  -g, --degenerate               Allow degenerate bases in barcodes and primers
  -h, --help                     Print help
  -V, --version                  Print version
```

## Barcode File
The barcode file must contain three tab-separated columns: `ID`, `5' barcode sequence`, and `3' barcode sequence`.  
The `ID` can be the number of a barcode, or names like sample name, species name, or tissue name, etc.  
Additionally, one `ID` can correspond to multiple pairs of barcode sequences.
As shown in the example, if the `ID` is `BC1`, the barcode file would be structured as follows:  
<table><tr><td>BC1</td><td>ATCG</td><td>TCAG</td></tr></table>

![image](https://github.com/user-attachments/assets/89bc2b4f-f1e0-4725-8827-80d8e8a2623f)


## Versions
This tool offers two versions:  
**GNU version (recommended):** Faster and more memory-efficient. Requires a recent version(>=2.31) of glibc on your Linux system.  
**Musl version:** Highly portable and compatible with a wider range of Linux systems, including those with older glibc versions.  
As shown in the diagram, sequencing data size is 4.66G, with 10 threads.  
![9d0d6bc7154580c998284ee3de8468d](https://github.com/user-attachments/assets/6004304c-2ea0-4d88-97b6-98df846a8dee)

## Authors
夏小双 Xiaoshuang Xia (xiaxiaoshuang@genomics.cn)

## License and Usage Restrictions
**Research Use Only**  

This software is provided strictly for individual research purposes. Commercial use is strictly prohibited. This means:  
**Allowed:** Personal academic research, personal learning, and non-commercial experimentation.  
**Not Allowed:** Any form of commercial application, distribution, or use that generates revenue directly or indirectly. This includes, but is not limited to, integration into commercial products, offering this software as a service, or using it for commercial gain.  

For commercial licensing or permissions, please contact us.

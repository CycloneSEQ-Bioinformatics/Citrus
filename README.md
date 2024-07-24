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
  -m, --mode <mode>              Set a mode to split barcodes. 1: only front end has a barcode, 2: each end has a barcode [possible values:
                                 1, 2]
  -f, --fastq <fastq file>       Input a fastq file
  -b, --barcode <barcode file>   Input a barcode file. If there are multiple barcodes in a read, please separate them with a tab in the
                                 input file. When two barcodes complement each other in reverse, only the front-end barcode is needed
  -q, --testseq <test seq file>  Input a test sequence file
  -o, --outdir <output dir>      Output directory
  -n, --sample <sample>          Sample name
  -e, --err <err threshold>      Threshold of Levenshtein Distance value or sequencing error rate. Use a comma to separate two numbers, when
                                 there are different thresholds for mode 2 [default: 6]
  -c, --seed_size <seed size>    Seed size [default: 5]
  -d, --step_size <step size>    Step size [default: 1]
  -s, --shift <shift threshold>  Threshold of shift length. Use a comma to separate two numbers, when there are different thresholds for
                                 mode 2 [default: 150,50]
  -t, --thread <thread>          Number of threads [default: 4]
  -r, --retain                   Retain barcode sequences on reads
  -g, --degenerate               Allow degenerate bases in barcodes and primers
  -h, --help                     Print help
  -V, --version                  Print version
```

## Authors
夏小双 Xiaoshuang Xia (xiaxiaoshuang@genomics.cn)

## License and Usage Restrictions
**Research Use Only**  

This software is provided strictly for individual research purposes. Commercial use is strictly prohibited. This means:  
**Allowed:** Personal academic research, personal learning, and non-commercial experimentation.  
**Not Allowed:** Any form of commercial application, distribution, or use that generates revenue directly or indirectly. This includes, but is not limited to, integration into commercial products, offering this software as a service, or using it for commercial gain.  

For commercial licensing or permissions, please contact us.

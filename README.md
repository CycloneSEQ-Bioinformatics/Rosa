<p align="center">
  <img src="https://github.com/user-attachments/assets/0256b97b-d393-45d2-9105-301baad3fc87" alt="Rosa Logo" width="600">
</p>

# Rosa

Rosa is a quality control and error analysis tool designed specifically for long-read sequencing data. It provides comprehensive quality assessment to help researchers understand data quality, identify potential sequencing error patterns, and inform downstream analysis.

## Features

- **Sequence Quality Analysis**: Quality assessment of raw FASTQ/FASTA files
- **Alignment Quality Analysis**: Alignment-based quality assessment from BAM files
- **Error Pattern Analysis**: Identification and analysis of sequencing error types
- **Quality Report Generation**: Detailed HTML quality control reports

## Requirements

Rosa depends on the following external tools (must be installed and available in PATH):

| Tool | Purpose |
| :---: | --- |
| minimap2 | Sequence alignment |
| samtools | BAM file processing |
| seqtk | Sequence processing |
| mosdepth | Depth analysis |

## Installation

### Via Pre-compiled Wheel Package

```bash
conda env create -f environment.yml -p ./env
conda activate ./env
pip install -v rosa-*.whl

# Verify installation
rosa --help
```

## Usage

### Basic Syntax

```bash
rosa [options] [arguments]
```

### Analysis Modes

Rosa supports two analysis modes:

**Sequence Analysis Mode** (without reference):

```bash
# FASTQ file
rosa -i input.fastq -o output_dir -n sample_name

# FASTA file
rosa -i input.fasta -o output_dir -n sample_name

# Convert from BAM
rosa -b input.bam -o output_dir -n sample_name
```

**Alignment Analysis Mode** (with reference, performs both sequence and alignment analysis):

```bash
# From FASTQ
rosa -i input.fastq -r reference.fasta -o output_dir -n sample_name

# From BAM
rosa -b input.bam -r reference.fasta -o output_dir -n sample_name
```

> **Note**: If the input BAM file was generated using minimap2, ensure the `--eqx` option was used during alignment.

## Command-line Arguments

### Preset Mode

| Argument | Description | Default |
| :---: | --- | :---: |
| `-x, --preset` | Analysis mode: `default` (standard QC) or `rnaseq` (RNA-seq data) | default |

### Sequence Analysis Arguments

| Argument | Description | Default |
| :---: | --- | :---: |
| `-i, --sequence-path` | Input FASTQ/FASTA file(s). Supports multiple files (space-separated) or a .txt file containing a list of file paths | - |

### Alignment Analysis Arguments

| Argument | Description | Default |
| :---: | --- | :---: |
| `-b, --bam-path` | Input BAM file | - |
| `-r, --reference-path` | Reference genome FASTA file | - |

### External Tool Configuration

| Argument | Description | Default |
| :---: | --- | :---: |
| `--minimap2-path` | Path to Minimap2 executable | minimap2 |
| `--minimap2-args` | Minimap2 command-line arguments (must include `--eqx`) | auto |
| `--samtools-path` | Path to samtools executable | samtools |
| `--seqtk-path` | Path to seqtk executable | seqtk |
| `--mosdepth-path` | Path to mosdepth executable | mosdepth |

**Default minimap2-args**:
- `default` mode: `-a -k 16 -w 13 -A 2 -B 4 -O 4,41 -E 2,1 -s 180 -U70,1000000 --eqx --secondary=no`
- `rnaseq` mode: `-ax splice -uf -k14 --eqx --secondary=no`

### General Arguments

| Argument | Description | Default |
| :---: | --- | :---: |
| `-o, --output-dir` | Output directory | rosa_report |
| `-n, --sample-name` | Sample name | N/A |
| `-t, --threads` | Number of threads | 4 |
| `--sample-size` | Number of reads to analyze; set to -1 for all reads | 100000 |
| `--seed` | Random sampling seed | 42 |
| `--keep-intermediates` | Retain intermediate files | False |
| `--verbose` | Enable verbose logging | False |
| `--debug` | Enable debug output | False |

## Examples

### Basic Sequence Analysis

```bash
# Single file
rosa -i sample.fastq -o qc_results -n "Sample_001"

# Multiple files
rosa -i sample_1.fastq sample_2.fastq -o qc_results -n "Sample_001"

# Using file list
rosa -i files.txt -o qc_results -n "Sample_001"
```

### Alignment Analysis

```bash
# From FASTQ
rosa -i sample.fastq -r reference.fasta -o qc_results -n "Sample_001"

# From BAM
rosa -b aligned.bam -r reference.fasta -o qc_results -n "Sample_001"
```

### RNA-seq Analysis

```bash
rosa -i sample.fastq -r reference.fasta -o qc_results -n "Sample_RNA" -x rnaseq
```

### Advanced Configuration

```bash
# Full data analysis with multi-threading
rosa -i sample.fastq -o qc_results -n "Sample_001" -t 16 --sample-size -1

# Keep intermediate files with verbose logging
rosa -b aligned.bam -r reference.fasta -o qc_results -n "Sample_001" \
     --keep-intermediates --verbose

# Custom minimap2 arguments
rosa -i sample.fastq -r reference.fasta -o qc_results -n "Sample_001" \
     --minimap2-args "-a -k 16 -w 13 -A 2 -B 4 -O 4,41 -E 2,1 -s 180 -U70,1000000 --eqx --secondary=no"
```

## Output

```
rosa_report/
├── fastx_analyses/              # Sequence analysis results
│   ├── fastx_statistics.json    # Sequence statistics
│   ├── fastx_table.tsv          # Per-sequence attributes
│   ├── figures/                 # Visualization plots
│   └── results/                 # Data results
├── bam_analyses/                # Alignment analysis results
│   ├── coverage_analysis.json   # Coverage analysis data
│   ├── figures/                 # Visualization plots
│   └── results/                 # Data results
├── report.html                  # Comprehensive HTML report
├── metadata.tsv                 # Analysis metadata
├── command.txt                  # Execution command
└── Rosa.log                     # Run log
```

### Example Report

An example HTML report is available in the [examples](./examples) directory. The report includes detailed interpretations alongside each result visualization.

## Known Issues

The following issues are known in the current version and will be fixed in the next release:

1. **Homo/heteropolymer accuracy plot label**: The x-axis label should display "10" instead of "≥10".

2. **Substitution error statistics inconsistency**: The sum of substitution errors in the substitution error analysis exceeds the substitution count in the overall error analysis. This is caused by different denominators being used. In a future update, the denominator will be unified to: `Total length of (matches + mismatches + insertions + deletions)`.

3. **Strand orientation handling in substitution analysis**: Currently, substitution errors are calculated directly from alignment results without additional strand-specific processing. Future versions will reverse-complement negative strand alignments before calculating statistics.

4. **Homo/heteropolymer length statistics bias**: Due to regex greedy matching behavior, homopolymers longer than the upper bound are counted incorrectly:
   - For a 12bp homopolymer (AAAAAAAAAAAA): The regex `(?:A){3,10}` greedily matches the first 10 A's (length=10), leaving 2 A's which don't meet the minimum of 3, so they are ignored.
   - For a 13bp homopolymer (AAAAAAAAAAAAA): The regex matches the first 10 A's, then the remaining 3 A's trigger a second match, resulting in double counting.
   - This causes accuracy statistics at length 10 to be artificially lower than actual values.

## Notes

1. **Minimap2 Arguments**: The `--eqx` option is required for minimap2 alignment; otherwise, Rosa cannot correctly analyze error patterns
2. **Sampling**: Default sampling of 100,000 reads balances statistical accuracy and runtime efficiency
3. **Threading**: Use `-t` to increase threads for faster analysis, though multi-threading acceleration is limited
4. **Dependencies**: Ensure all required tools are installed and available in PATH

## Troubleshooting

### Check Dependencies

```bash
minimap2 --version
samtools --version
seqtk
mosdepth --version
```

### Slow Analysis

- Avoid `--sample-size -1` for full data analysis unless necessary
- Use BAM files directly if already available
- Increase thread count with `-t` to speed up alignment

### Debug Mode

Use `--debug` and `--verbose` for detailed runtime information:

```bash
rosa -i sample.fastq -o qc_results -n "Sample" --verbose --debug
```

## Version

Current version: Rosa 1.1.0

## Authors

Haibing Ma 马海兵 (mahaibing@genomics.cn)  
Jiayuan Zhang 张嘉远 (zhangjiayuan@genomics.cn)

## License

**Research Use Only**

This software is provided strictly for individual research purposes. Commercial use is strictly prohibited.

- **Allowed**: Personal academic research, learning, and non-commercial experimentation
- **Not Allowed**: Any form of commercial application, distribution, or use that generates revenue directly or indirectly. This includes, but is not limited to, integration into commercial products, offering this software as a service, or using it for commercial gain. 

For commercial licensing or permissions, please contact us.
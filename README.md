<p align="center">
    <img src="figures/logo.png" title="Restrander" alt="Restrander" width="300">
</p>

A vignette, tutorial and guide for the read orientation software, [Restrander](https://github.com/jakob-schuster/restrander).

## Table of Contents

- [Introduction](#introduction)
- [General usage](#general-usage)
    - [Installation & setup](#installation--setup)
    - [Basic restranding](#basic-restranding)
        - [Output statistics](#output-statistics)
    - [Configuration files](#configuration-files)
        - [Switching configurations](#switching-configurations)

## Introduction

In transcriptomic analyses, it is helpful to keep track of the strand of the RNA molecules. However, Oxford Nanopore long-read cDNA sequencing protocols generate reads that correspond to either the first or second-strand cDNA, therefore the strandedness of the initial transcript has to be inferred bioinformatically.

Library preparation for cDNA sequencing adds different oligonucleotide sequences on the ends of each read. The template-switching oligo (TSO, also known as Strand-Switching Primer SSP) is found on the 5’ end of forward reads, and on the 3’ end of reverse reads (as a reverse complement). The sequence of the reverse transcription primer (RTP, also known as oligo(dT) VN primer or VNP in some protocols), is conversely found at the 3’ end of forward reads (as a reverse complement) and the 5’ end of reverse reads. When reads are sequenced from the 5’ end, forward reads and reverse reads can be differentiated by the order in which the TSO and RTP appear. In addition to these primers, polyA tails are present near the end of forward reads, while complementary polyT tails are found near the start of reverse reads.

Restrander parses an input fastq, infers the orientation of each read, and prints to an output fastq. The strand of each read is recorded with the `strand` tag, either `+` or `-`. Each read from the reverse strand is replaced with its reverse-complement, ensuring all reads in the output have the same orientation as the original transcripts.

Only well-formed reads are included in the main output file; reads whose strand cannot be inferred are marked with `strand=?`, and optionally filtered out into an “unknown” fastq to be handled separately by the user. If Restrander is configured to detect artefacts, these artefactual reads will also be placed in the “unknown” fastq.

In a typical cDNA-seq analysis pipeline, Restrander would be applied after basecalling, and before mapping.

## General usage

In this vignette, we will go through some usage examples for Restrander. We'll use tiny slices of data to illustrate the features.

### Installation & setup

```bash
# download the vignette, so you have access to the data slices
git clone https://github.com/jakob-schuster/restrander-vignette.git
cd restrander-vignette

# for the convenience of this vignette, 
# install Restrander right here in the vignette repo
git clone https://github.com/jakob-schuster/restrander.git
cd restrander
make
```

### Basic restranding

First, we'll try Restrander on some PCR-cDNAseq data, sequenced using SQK-PCS109 on PromethION. A tiny sample of 20000 reads is included with this vignette. The sample can be found in `data/PCB109.fq.gz`.

Run Restrander by giving it the input file, the output destination and a configuration file.

```bash
# run this in the directory of this vignette
# if you're getting errors, make sure the path names are correct!
./restrander/restrander \
    data/PCB109.fq.gz \
    data/PCB109-restranded.fq.gz \
    restrander/config/PCB109.json
```

#### Output statistics

On your terminal, underneath the diagnostic information about Restrander, you'll see some output statistics. They should look similar to this:

```json
{
    "stats": {
        "artefactStats": {
            "no artefact": 20
        },
        "strandStats": {
            "+": 7,
            "-": 13
        },
        "totalReads": 20
    }
}
```

These statistics are useful, as they quantify the read orientations and artefact presence in the input data, as well as indicating how well Restrander performed. The stats are formatted as a json, and it's a good idea to pipe them out using `>` whenever you run Restrander:

```bash
./restrander/restrander \
    data/PCB109.fq.gz \
    data/PCB109-restranded.fq.gz \
    restrander/config/PCB109.json \
        > output-stats.json
```

### Configuration files

Since each library preparation protocol uses its own set of primers, you'll need to provide Restrander with the configuration file specific to your protocol. The Restrander download includes configurations for several protocols in the `config` directory. 

<table>
    <thead>
        <tr>
            <th>Config file</th>
            <th>Intended use</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>PCB109.json</td>
            <td><a href="https://store.nanoporetech.com/cdna-pcr-sequencing-kit.html">Oxford Nanopore Technologies cDNA-PCR sequencing kit SQK-PCS109</a></td>
        </tr>
        <tr>
            <td>PCB111.json</td>
            <td><a href="https://store.nanoporetech.com/productDetail/?id=cdna-pcr-sequencing-kit111">Oxford Nanopore Technologies cDNA-PCR sequencing kit SQK-PCS111</a></td>
        </tr>
        <tr>
            <td>10X-3prime.json</td>
            <td><a href="https://www.10xgenomics.com/support/single-cell-gene-expression/documentation/steps/library-prep/chromium-single-cell-3-reagent-kits-user-guide-v-3-1-chemistry">10x Genomics Chromium 3’ kit</a></td>
        </tr>
        <tr>
            <td>10X-5prime.json</td>
            <td><a href="https://www.10xgenomics.com/support/single-cell-immune-profiling/documentation/steps/library-prep/chromium-single-cell-5-reagent-kits-user-guide-v-2-chemistry-dual-index">10x Genomics Chromium 5’ kit</a></td>
        </tr>
        <tr>
            <td>NEBNext.json</td>
            <td><a href="https://www.nebiolabs.com.au/products/e6420-nebnext-single-cell-low-input-rna-library-prep-kit-for-illumina#Product%20Information">New England Biolabs NEBNext single-cell/low input RNA library prep kit</a></td>
        </tr>
        <tr>
            <td>trimmed.json</td>
            <td></td>
        </tr>
    </tbody>
</table>

#### Switching configurations

Our file `data/10x.fq.gz` is a slice of single-cell long-read RNA-seq data, prepared using the 10x Genomics Chromium 3' kit. It's important to use the appropriate config file for your library prep kit. If Restrander is configured incorrectly, the issue may not be obvious. For demonstration, let's try using the default `PCB109.json`:

**Input**
```bash
./restrander/restrander \
    data/10x.fq.gz \
    data/10x-restranded-wrong-primers.fq.gz \
    restrander/config/PCB109.json \
        > 10x-wrong-primers-stats.json
```
**Output**
```json
{
    "stats": {
        "artefactStats": {
            "RTP-RTP": 1,
            "TSO-TSO": 1,
            "no artefact": 19998
        },
        "strandStats": {
            "+": 6850,
            "-": 6434,
            "?": 6716
        },
        "totalReads": 20000
    }
}
```

Restrander runs without error, and looking at the output stats, we see that most reads have been successfully classified - this is due to the polyA/T tail presence common to reads across both protocols. However, there's a high number of unknown reads, and the number of primer artefacts is suspiciously low, since we've provided the wrong primers.

Now, run Restrander with the right config file, `10X-3prime.json`:

**Input**
```bash
./restrander/restrander \
    data/10x.fq.gz \
    data/10x-restranded-correct-primers.fq.gz \
    restrander/config/10X-3prime.json \
        > 10x-correct-primers-stats.json
```
**Output**
```json
{
    "stats": {
        "artefactStats": {
            "RTP-RTP": 129,
            "TSO-TSO": 2046,
            "no artefact": 17825
        },
        "strandStats": {
            "+": 8970,
            "-": 7005,
            "?": 4025
        },
        "totalReads": 20000
    }
}
```

More reads have been classified, and now we can see most of the unknown reads are explained by the high volume of primer artefacts present in the input file.

Since Restrander doesn't know which library preparation kit was used to produce your data, it won't fail when provided with the wrong config, but the results may be spurious. Always double-check that your config file matches your library preparation method!
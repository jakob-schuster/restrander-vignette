<p align="center">
    <img src="figures/logo.png" title="Restrander" alt="Restrander" width="300">
</p>

A vignette, tutorial and guide for the read orientation software, [Restrander](https://github.com/jakob-schuster/restrander).

## Table of Contents

- [Introduction](#introduction)
- [Usage](#usage)
    - [Installation & setup](#installation--setup)
    - [Configuration files](#configuration-files)
    - [Basic restranding](#basic-restranding)
        - [Output statistics](#output-statistics)
    - [Customising the configuration](#customising-the-configuration)
        - [Enabling artefact detection](#artefact-detection)
        - [Enabling artefact detection](#artefact-detection)
    - [Lower quality data](#lower-quality-data)
    - [Trimmed data](#trimmed-data)


## Introduction

In transcriptomic analyses, it is helpful to keep track of the strand of the RNA molecules. However, Oxford Nanopore long-read cDNA sequencing protocols generate reads that correspond to either the first or second-strand cDNA, therefore the strandedness of the initial transcript has to be inferred bioinformatically.

Library preparation for cDNA sequencing adds different oligonucleotide sequences on the ends of each read. The template-switching oligo (TSO, also known as Strand-Switching Primer SSP) is found on the 5’ end of forward reads, and on the 3’ end of reverse reads (as a reverse complement). The sequence of the reverse transcription primer (RTP, also known as oligo(dT) VN primer or VNP in some protocols), is conversely found at the 3’ end of forward reads (as a reverse complement) and the 5’ end of reverse reads. When reads are sequenced from the 5’ end, forward reads and reverse reads can be differentiated by the order in which the TSO and RTP appear. In addition to these primers, polyA tails are present near the end of forward reads, while complementary polyT tails are found near the start of reverse reads.

Restrander parses an input fastq, infers the orientation of each read, and prints to an output fastq. The strand of each read is recorded with the `strand` tag, either `+` or `-`. Each read from the reverse strand is replaced with its reverse-complement, ensuring all reads in the output have the same orientation as the original transcripts.

Only well-formed reads are included in the main output file; reads whose strand cannot be inferred are marked with `strand=?`, and optionally filtered out into an “unknown” fastq to be handled separately by the user. If Restrander is configured to detect artefacts, these artefactual reads will also be placed in the “unknown” fastq.

In a typical cDNA-seq analysis pipeline, Restrander would be applied after basecalling, and before mapping.

## Usage

In this vignette, we will go through some usage examples for Restrander. We'll use tiny slices of data to illustrate the features.

### Installation & Setup

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

### Configuration files

Since each sequencing protocol uses its own set of primers, you'll need to use the configuration file specific to your protocol. The Restrander download includes configurations for several protocols in the `config` directory. 

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
            <td></td>
        </tr>
        <tr>
            <td>PCB111.json</td>
            <td></td>
        </tr>
        <tr>
            <td>10X-3prime.json</td>
            <td></td>
        </tr>
        <tr>
            <td>10X-5prime.json</td>
            <td></td>
        </tr>
        <tr>
            <td>NEBNext.json</td>
            <td></td>
        </tr>
        <tr>
            <td>trimmed.json</td>
            <td></td>
        </tr>
    </tbody>
</table>

### Basic restranding

First, we'll try Restrander on some PCR-cDNAseq data, sequenced using SQK-PCS109 on PromethION. A tiny sample of 2000 reads is included with this vignette. The sample can be found in `data/PCB109.fq.gz`.

Run Restrander by giving it the input file to process, the output destination to write to, and a configuration file to use.

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

### Customising the configuration

The configuration file describes the full pipeline of operations used by Restrander to classify each read. By tweaking their config file, users can specialise Restrander's behaviour to their needs. The config file is in the `json` format:

```json
{
    "name": "PCB109",
    "description": "The default configuration. First applies PolyA/PolyT classification, then looks for the standard TSO (SSP) and RTP (VNP) used in PCB109 chemistry.",
    "pipeline": [
        {
            "type": "poly",
            "tail-length": 12,
            "search-size": 200
        },
        {
            "type": "primer",
            "tso": "TTTCTGTTGGTGCTGATATTGCTGGG",
            "rtp": "ACTTGCCTGTCGCTCTATCTTCTTTTTTTTTT",
            "report-artefacts": true
        }
    ],
    "silent": false,
    "exclude-unknowns": false,
    "error-rate": 0.25
}
```

When you customise your configuration, it's a good idea to create a named copy of your old config, rather than modify the default configuration directly.

#### Disabling artefact detection

In the first example, our output stats indicate that Restrander is searching for TSO-TSO and RTP-RTP artefacts in our input data. If we're not interested in this feature, we can disable it in the config.

Looking closely at the config:
- `"report-artefacts": true` specifies that, while searching for primers, we would like to quantify TSO and RTP artefacts as we parse the input file. 
- `"exclude-unknowns": true` means that any unknown reads (including primer artefacts) should be filtered out of the input, into a separate output file. If you check your output directory, you'll find them in an `unknowns.fq.gz` file.

Let's change these parameters. Create a copy of `PCB109.json`. Give it some new name like `my-custom-PCB109.json`, and tweak the lines so that `"report-artefacts": false` and `"exclude-unknowns": false`. Now, run Restrander with this new config:

```bash
./restrander/restrander \
    data/PCB109.fq.gz \
    data/PCB109-restranded-new-config.fq.gz \
    restrander/config/my-custom-PCB109.json \
        > output-stats.json
```

Looking at `output-stats.json`, we see that artefacts are no longer being found and quantified. Also, since `"exclude-unknowns": false`, no unknowns file was created - all successfully and unsuccessfully oriented reads are included in `PCB109-restranded-new-config.fq.gz`.

### Using different primers


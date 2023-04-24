<p align="center">
    <img src="figures/logo.png" title="Restrander" alt="Restrander" width="300">
</p>

A vignette and tutorial for the read orientation software, [Restrander](https://github.com/jakob-schuster/restrander).

## Introduction

In transcriptomic analyses, it is helpful to keep track of the strand of the RNA molecules. However, Oxford Nanopore long-read cDNA sequencing protocols generate reads that correspond to either the first or second-strand cDNA, therefore the strandedness of the initial transcript has to be inferred bioinformatically.

Library preparation for cDNA sequencing adds different oligonucleotide sequences on the ends of each read. The template-switching oligo (TSO, also known as Strand-Switching Primer SSP) is found on the 5’ end of forward reads, and on the 3’ end of reverse reads (as a reverse complement). The sequence of the reverse transcription primer (RTP, also known as oligo(dT) VN primer or VNP in some protocols), is conversely found at the 3’ end of forward reads (as a reverse complement) and the 5’ end of reverse reads. When reads are sequenced from the 5’ end, forward reads and reverse reads can be differentiated by the order in which the TSO and RTP appear. In addition to these primers, polyA tails are present near the end of forward reads, while complementary polyT tails are found near the start of reverse reads.


## Usage

### 0. Installation

Download and compile Restrander using the following command:

```bash
git clone https://github.com/jakob-schuster/restrander.git
cd restrander
make
```

### 1. Your first restrand

First, we'll try Restrander on some 

### 2. Lower quality data

### 3. Trimmed data

### 4. 
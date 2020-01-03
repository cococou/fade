![alt text](https://github.com/blachlylab/fade/raw/master/logo/fade_logo.png "FADE")

# **F**ragmentase **A**rtifact **D**etection and **E**limination

DNA shearing is a crucial first step in most NGS protocols for Illumina. Enzymatic fragmentation has shown in recent years to be a cost and time effective alternative to physical shearing (i.e. sonication). We discovered that enzymatic fragmentation leads to unexpected alteration of the original DNA solurce material. We provide fade as a method of identification and removal of enymatic fragmentation artifacts.

## Quick Start

### Installation

Install [htslib](http://www.htslib.org/download/) and download a binary release.

Or build from source:
```
git clone --recurse-submodules https://github.com/blachlylab/fade.git 
cd fade
dub build -b release

#LDC
#LIBRARY_PATH=/usr/local/lib/ dub build -b release
```

### Running FADE
```
./fade annotate sam1.bam ref.fa sam1.anno.bam
samtools sort -n sam1.anno.bam > sam1.anno.qsort.bam #recommended but not neccessary
./fade filter sam1.anno.qsort.bam sam1.filtered.bam
```

## Algorithm
FADE is written in D and uses [htslib](http://www.htslib.org/download/) via [dhtslib](https://github.com/blachlylab/dhtslib.git). FADE accepts SAM/BAM/CRAM files containing reads that have been mapped to a reference genome and filters or cleans up artifact-containing reads according to the following procedure. 

FADE is designed to determine a sequencing read’s enzymatic artifact status by employing aligner soft-clipping. Soft-clipping is an action performed by the aligner to improve the alignment score of a read to the reference by ignoring a portion on one end of the read. Soft-clipping can help an aligner correctly align a read that has sequencing error on one end of the read or has adapter contamination. FADE employs soft-clipping to identify potentially enzymatic artifact containing reads. 
1. It considers only those reads aligned with soft-clipping; alignments without soft-clips. 
2. The reference sequence that the alignment is mapped to is extracted such that there exists 300 nucleotides (nt) of padding on each end of the mapped read.\* 
3. The read is reverse-complemented and then aligned via a Smith-Waterman local alignment to the extracted region of reference sequence. We use a scoring matrix with a gap open penalty of 10, a gap extension penalty of 2, a mismatch penalty of 3, and a match score of 2.\*\* 

FADE makes available four subcommands that all rely on the algorithm described above. The **annotate** subcommand performs the initial analysis and adds BAM tags encoding information concerning artifact status to the alignments, used during filtration to remove the artifacts. The **filter** subcommand removes reads from the output BAM/SAM file completely if they or their mate contain an identified fragmentation artifact. After filtration, FADE reports statistics describing the total number of alignments, the percentage of soft-clipped alignments, and the percentage of enzymatic artifacts (EAs) found. The filtration step must be run on a queryname-sorted BAM file in order to fully filter out the read, its mate, and any other supplementary or secondary alignments. The **clip** subcommand operates similarly filter with the exception that artifact-containing reads are trimmed to remove extraneous sequence originating from the opposite strand, but the reads are not removed.

<sub><sup>\* The 300 nt padding on each end of the mapped region provides ample search space for artifact alignment search without being too computationally expensive; most artifacts originate very close to the mapped region and 300 nt was chosen as an optimal tradeoff, but could be adjusted.</sub></sup>

<sub><sup>\*\* Harsher gap penalties allows the algorithm to be strict in allowing gaps, since we expect the artifact sequences to directly match the reference, except for soft-clipped regions derived from sequencing error. A soft-clipped region is considered to be an artifact if there is a 90% or greater match to the opposite strand sequence. </sub></sup>

## Test Data
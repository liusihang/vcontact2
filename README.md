vConTACT : Viral CONtig Automatic Cluster Taxonomy 2
====================================================

[![Anaconda-Server Badge](https://anaconda.org/bioconda/vcontact2/badges/version.svg)](https://anaconda.org/bioconda/vcontact2)
[![Anaconda-Server Badge](https://anaconda.org/bioconda/vcontact2/badges/latest_release_date.svg)](https://anaconda.org/bioconda/vcontact2)
[![Anaconda-Server Badge](https://anaconda.org/bioconda/vcontact2/badges/platforms.svg)](https://anaconda.org/bioconda/vcontact2)
[![Anaconda-Server Badge](https://anaconda.org/bioconda/vcontact2/badges/license.svg)](https://anaconda.org/bioconda/vcontact2)
[![Anaconda-Server Badge](https://anaconda.org/bioconda/vcontact2/badges/downloads.svg)](https://anaconda.org/bioconda/vcontact2)
[![DOI:10.1038/s41587-019-0100-8](https://zenodo.org/badge/DOI/10.1038/s41587-019-0100-8.svg)](https://doi.org/10.1038/s41587-019-0100-8)

**Notice: vConTACT2 is no longer being actively developed due to end of funding (research grants often don't consider the 
perpetual nature of software support :( ), though we are still working on bug fixes 
and addressing other issues as they arise. We are working on a significant upgrade (hopefully some limited release 
soon) that already resolves longstanding issues with the current v2.**

vConTACT2 is a tool to perform guilt-by-contig-association classification of viral genomic sequence data. It's designed 
to cluster and provide taxonomic context of viral metagenomic sequencing data.

## Documentation

Please excuse our new documentation placement! The README was getting a bit on the longer side, so has been moved to 
[our new wiki](https://bitbucket.org/MAVERICLab/vcontact2/wiki/Home). We hope that this will minimize the README length 
and provide a much larger area for detailed descriptions of vConTACT2.

Please see below for an abbreviated installation and running instructions.

## Installation Requirements

vConTACT requires numerous python packages to function correctly, and each must be properly installed and working for 
vConTACT to also work.

 * python >=3.7 (not python 2.7 safe!)
 * networkx>=2.2
 * numpy>=1.20.1
 * scipy>=1.6.0
 * pandas>=1.0.5
 * scikit-learn>=0.24.1
 * biopython>=1.78
 * hdf5>=1.10.4
 * pytables>=3.4.0 (tables in pypi)
 * pyparsing>=2.4.6
 * psutil>=5.8.0

vConTACT also requires several executables, depending on use.

 * mcl>=14.137
 * blast>=2.6.0
 * diamond>=0.9.14
 * clusterone

Generally you want these tools to be in your system or user PATHs. vConTACT2 will use any user-provided paths before 
searching through system PATHs.

Hardware requirements *can be considerable* (exceeding 48 GB!), depending mainly on the size and complexity of the 
dataset. (Relationship between memory requirements and sequences analyzed forthcoming)

## Installation

Installing vConTACT dependencies may seem daunting, but the instructions below should work for the vast majority of 
users. While Windows is not officially supported, users have been able to run vConTACT on these machines (usually 
through some sort of virtual machine).

#### Singularity (all) (trivial if already have Singularity installed)

A singularity definitions file is provided accompanying this repository (vConTACT2.def). Please use this file to create 
and bootstrap the vConTACT2 container.

```bash
sudo singularity build vConTACT2.sif vConTACT2.def
```

The build process can take a significant amount of time depending on available hardware and network speed. Builds can 
take anywhere from 5 to 30 minutes. If you see *Finalizing Singularity container* at the end of bootstrapping, you're 
probably good to go.

Once built, the container can be run via:

```bash
singularity run vConTACT2.sif <args>
```

#### Conda-based installation (Mac/Linux) (recommended)

We highly recommend using a python environment when installing this software, as dependency versions can (and often can)
 conflict with each other.

For this, we'll be installing everything into a single directory that the user has access to. This is generally the 
user's home directory. In the example, we'll be installing to the "conda" directory under the user's home folder.

First, grab our favorite manager, Anaconda/Miniconda and install. If it offers the option of adding the installation to 
the user's $PATH, then do so. Otherwise, follow the instructions (at the end of the install) to ensure the install is 
activated.

```bash
wget https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
# Install into $HOME/conda
conda create --name vContact2 python=3
source activate vContact2
conda install -y -c bioconda vcontact2 mcl blast diamond
```

Note: DIAMOND is highly recommended over BLASTP for any large-scale analysis. It’s much faster and shows little/no 
difference in the final VCs. *This hasn't been officially benchmarked, but a sufficient number of in-house analyses have
 been performed to recommend.*

##### Bleeding edge

It is possible for the bioconda version to lag behind the most recent releaase. If you really want 
the most recent update, you'll need to install the dependencies and then manually install from source.

```bash
conda install -y -c conda-forge pytables biopython networkx numpy pandas scipy scikit-learn psutil pyparsing
conda install -y -c bioconda mcl blast diamond
```

Finally, install vConTACT2 from source file.

```bash
wget https://bitbucket.org/MAVERICLab/vcontact2/get/master.tar.gz
tar xvf MAVERICLab-vcontact2-XXXXXXX.tar.gz
cd MAVERICLab-vcontact2-XXXXXXX && python3 -m pip install .
```

(Some users have mentioned that their version of pip installs to a non-conda location. In this case, run "pip install 
--prefix=$HOME/conda/bin)

## Usage (for the impatient)

Singularity

```bash
singularity run vConTACT2.sif --raw-proteins [proteins file] --proteins-fp [gene-to-genome mapping file] --db 'ProkaryoticViralRefSeq211-Merged' --output-dir [target output directory]
```

Local installs

```bash
vcontact2 --raw-proteins [proteins file] --proteins-fp [gene-to-genome mapping file] --db 'ProkaryoticViralRefSeq211-Merged' --output-dir [target output directory]
```

## Input files and formats

vConTACT2 tries to alleviate some of the challenges created by the complex file format of the original vConTACT and 
provide a more seamless "pipeline" to go from raw data to a finished network. It also allows the user flexibility in 
providing files at various points along the processing pipeline. Generally speaking, if a user provides an intermediary 
file, vConTACT2 will skip the steps it would of taken to get to that file. This is useful because it allows partial 
recovery and re-start of any interrupted analysis. That said, *if there is an issue and you're trying to troubleshoot, 
always re-run and specify a new output directory*.

The only required files are:

1. A FASTA-formatted amino acid file.

```
>ref|NP_039777.1| ORF B-251 [Sulfolobus spindle-shaped virus 1]
MVRNMKMKKSNEWLWLGTKIINAHKTNGFESAIIFGKQGTGKTTYALKVAKEVYQRLGHE
PDKAWELALDSLFFELKDALRIMKIFRQNDRTIPIIIFDDAGIWLQKYLWYKEEMIKFYR
IYNIIRNIVSGVIFTTPSPNDIAFYVREKGWKLIMITRNGRQPDGTPKAVAKIAVNKITI
IKGKITNKMKWRTVDDYTVKLPDWVYKEYVERRKVYEEKLLEELDEVLDSDNKTENPSNP
SLLTKIDDVTR
>ref|NP_039778.1| ORF D-335 [Sulfolobus spindle-shaped virus 1]
MTKDKTRYKYGDYILRERKGRYYVYKLEYENGEVKERYVGPLADVVESYLKMKLGVVGDT
PLQADPPGFEPGTSGSGGGKEGTERRKIALVANLRQYATDGNIKAFYDYLMNERGISEKT
AKDYINAISKPYKETRDAQKAYRLFARFLASRNIIHDEFADKILKAVKVKKANADIYIPT
```

2. A "gene-to-genome" mapping file, in either tsv (tab)- or csv (comma)-separated format.

```
protein_id,contig_id,keywords
ref|NP_039777.1|,Sulfolobus spindle-shaped virus 1,ORF B-251
ref|NP_039778.1|,Sulfolobus spindle-shaped virus 1,ORF D-335
ref|NP_039779.1|,Sulfolobus spindle-shaped virus 1,ORF E-54
ref|NP_039780.1|,Sulfolobus spindle-shaped virus 1,ORF F-92
ref|NP_039781.1|,Sulfolobus spindle-shaped virus 1,ORF D-244
ref|NP_039782.1|,Sulfolobus spindle-shaped virus 1,ORF E-178
ref|NP_039783.1|,Sulfolobus spindle-shaped virus 1,ORF F-93
ref|NP_039784.1|,Sulfolobus spindle-shaped virus 1,ORF E-51
ref|NP_039785.1|,Sulfolobus spindle-shaped virus 1,ORF E-96
```

(Multiple keywords must be separated using ";":)

Note: These keywords have no effect on the calculations. However, vConTACT will aggregate these gene keywords and in 
the output files you can find how many keywords were found in the same VC or PC. So, for example, if there are 5 phage 
tail fiber proteins in the same PC, that could be indicative of something pretty interesting! Or perhaps (more 
likely) there are novel genomes in your analysis. 4 of them have keywords associated to tail fibers, 1 of them is from 
a novel virus - well, if they're in the same PC then there's a good chance that "unknown" ORF is a tail fiber. vConTACT 
can't replace existing tools for understanding protein homology or similar functions, but it can help guide you through 
looking at viruses from their VC point-of-view.

```
protein_id,contig_id,keywords
ref|NP_039777.1|,Sulfolobus spindle-shaped virus 1,Fuselloviridae;Alphafusellovirus;Sulfolobus spindle-shaped virus 1;ORF B-251
ref|NP_039778.1|,Sulfolobus spindle-shaped virus 1,Fuselloviridae;Alphafusellovirus;Sulfolobus spindle-shaped virus 1;ORF D-335
ref|NP_039779.1|,Sulfolobus spindle-shaped virus 1,Fuselloviridae;Alphafusellovirus;Sulfolobus spindle-shaped virus 1;ORF E-54
ref|NP_039780.1|,Sulfolobus spindle-shaped virus 1,Fuselloviridae;Alphafusellovirus;Sulfolobus spindle-shaped virus 1;ORF F-92
ref|NP_039781.1|,Sulfolobus spindle-shaped virus 1,Fuselloviridae;Alphafusellovirus;Sulfolobus spindle-shaped virus 1;ORF D-244
ref|NP_039782.1|,Sulfolobus spindle-shaped virus 1,Fuselloviridae;Alphafusellovirus;Sulfolobus spindle-shaped virus 1;ORF E-178
ref|NP_039783.1|,Sulfolobus spindle-shaped virus 1,Fuselloviridae;Alphafusellovirus;Sulfolobus spindle-shaped virus 1;ORF F-93
ref|NP_039784.1|,Sulfolobus spindle-shaped virus 1,Fuselloviridae;Alphafusellovirus;Sulfolobus spindle-shaped virus 1;ORF E-51
ref|NP_039785.1|,Sulfolobus spindle-shaped virus 1,Fuselloviridae;Alphafusellovirus;Sulfolobus spindle-shaped virus 1;ORF E-96
```

And the run command:

```
vcontact2 --raw-proteins [proteins file] --proteins-fp [gene-to-genome mapping file] --db 'ProkaryoticViralRefSeq211-Merged' --output-dir [target output directory]
```

**Instructions to use other input files/formats are available at the wiki!**

### Example files

Example files are provided in the test_data/ directory. These files contain either 10 genomes (VIRSorter_genomes*) or a 
single genome (VIRSorter_genome*). To use vConTACT2 with them, run the following command:

```
vcontact2 --raw-proteins test_data/VIRSorter_genomes.faa --proteins-fp test_data/VIRSorter_genomes_g2g.csv --db 'ProkaryoticViralRefSeq211-Merged' --output-dir vConTACT2_Results
```

You should find a large assortment of input, intermediate and final output files in the "vConTACT2_Results" directory. 
*Most important* are **viral_cluster_overview.csv** and **genome_by_genome_overview.csv** file. They contain a list of 
VC-by-VC and genome-by-genome processed, as well as any reference databases used.

Also included in the example files are intermediate files (vConTACT_pcs/profiles/proteins). These are generated after 
the Diamond analysis on the input proteins, the MCL run on the resulting Diamond results, and the parsing of the MCL 
clusters. This is the "1st stage" of vConTACT2 pre-processing. If, **at any time the job fails after this step**, you 
can restart the run using these 3 intermediate files.

```
vcontact2 --pcs test_data/vConTACT_pcs.csv --contigs test_data/vConTACT_contigs.csv --pc-profiles test_data/vConTACT_profiles.csv --db 'ProkaryoticViralRefSeq211-Merged' --output-dir vConTACT2_Results
```

## Output files

There are a lot of output files generated by vConTACT2, *most of these are temporary or intermediate files that are not 
useful to the general user*. The most important files are the network and annotation files.

**genome_by_genome_overview.csv**: Contains all the taxonomic information to *reference genomes*, as well as all the 
clustering information (initial VC (VC_22), refined VC (VC_22_1)), confidence metrics, and misc scores.

One important note is that the taxonomic information *is not included* for user sequences. This means that each user 
will need to find their genome(s) of interest and check to see if reference genomes are located in the same VC. If the 
user genome is within the same VC subcluster as a reference genome, then there's a very high probability that the user 
genome is part of the same genus. If the user genome is in the same VC *but not the same subcluster as a reference*, 
then it's highly likely the two genomes are related at roughly genus-subfamily level. If there are no reference genomes 
in the same VC or VC subcluster, then it's likely that they are not related at the genus level at all. That said (more 
below), it is possible they could be related at a higher taxonomic level (subfamily, family, order)

**c1.ntw**: Contains source / target / edge weight information for all genome pairs higher than the significance 
threshold as determined by the probability that those two genomes would share N genes. The lowest value in this file 
must be greater than the minimum significance threshold (default: 1). To create a network figure in Gephi or Cytoscape, 
the user will want to import *this file* into their favorite program.

Once imported, a user can add an "annotation file" - that will be the genome_by_genome file. The annotation information 
for each genome will be added to each node/genome in the network. Then, the user can color the network figure by any 
attribute in the annotation file. This will allow the user to "quickly" create a colorful image suitable for publication.

This is documented in the protocols.io protocol, available at https://dx.doi.org/10.17504/protocols.io.x5xfq7n

Note: Many times a user will notice that their genome is connected to another (possibly reference) genome in the 
network but those two genomes won't be in the same VC subcluster or even the same VC. This doesn't mean that they 
aren't related, it just means they did not share a sufficiently significant proportion of their genes to be of the same 
genus. *They could very much be related at the subfamily or family level.* However, that's for the researcher to 
decide.

## Publications/Citation

If you find vConTACT2 useful, please cite our newly published article in Nature Biotech!

Bin Jang, H., Bolduc, B., Zablocki, O., Kuhn, J. H., Roux, S., Adriaenssens, E. M., … Sullivan, M. B. (2019). Taxonomic assignment of uncultivated prokaryotic virus genomes is enabled by gene-sharing networks. Nature Biotechnology. https://doi.org/10.1038/s41587-019-0100-8

The theory and original code this work was based on:

Bolduc, B., Jang, H. Bin, Doulcier, G., You, Z., Roux, S., & Sullivan, M. B. (2017). vConTACT: an iVirus tool to classify double-stranded DNA viruses that infect Archaea and Bacteria. PeerJ, 5, e3243. https://doi.org/10.7717/peerj.3243


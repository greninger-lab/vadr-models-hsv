# <a name="documentation"></a>Herpes simplex virus (HSV) genome annotation

## [How to annotate HSV genomes with VADR](#howto)

## [HSV VADR model](#hsvmodel)

## [Additional VADR documentation](#docs)

## [References](#reference)

---
## <a name="howto"></a>How to annotate HSV genomes with VADR

Steps for using VADR for HSV annotation:

1. Download and install the latest version of VADR, following the
   instructions on this [page](https://github.com/ncbi/vadr/tree/master).
   Alternatively, you can use the StaPH-B VADR 1.6.3-hav-flu2
   docker image created by Curtis Kapsak (docker image names:
   `staphb/vadr:1.6.3-hav-flu2` and `staphb/vadr:latest`), available on 
   [dockerhub](https://hub.docker.com/r/staphb/vadr/tags) and
   [quay](https://quay.io/repository/staphb/vadr?tab=tags). A brief
   [README for the docker image is here](https://github.com/StaPH-B/docker-builds/tree/master/vadr/1.6.3-hav-flu2).
 
2. Clone the latest HSV VADR model from this repository (current release v1.0)<br/>
   `git clone git@github.com:greninger-lab/vadr-models-hsv.git`<br/>
   or download the current release from [here](https://github.com/greninger-lab/vadr-models-hsv/releases/tag/v1.0).</br>
   Note the path to the directory name created plus the /hsv2 or /hsv1
   subdirectory as `<hsv-models-dir-path>`
   for step 4.

4. Remove terminal ambiguous nucleotides from your
   input fasta sequence file using the `fasta-trim-terminal-ambigs.pl`
   script in `$VADRSCRIPTSDIR/miniscripts/`.

   To remove terminal ambiguous nucleotides from your sequence file
   `<input-fasta-file>` and to remove short and long sequences to create a new trimmed file
   `<trimmed-fasta-file>`, execute:

```
$VADRSCRIPTSDIR/miniscripts/fasta-trim-terminal-ambigs.pl --minlen 50 --maxlen 165000 <input-fasta-file> > <trimmed-fasta-file>
```        

4. Run the `v-annotate.pl` program on an input trimmed fasta file with
   HSV-2 sequences using the recommended command and options
   below (the command is long so you will likely have to scroll to the right to view the entire command).

   ***NOTE: The following command runs multithreaded on up to 4 CPUs,
   and so is only recommended if you have at least 4 CPUs and 16Gb RAM
   available. To run on `<n>` CPUs instead, replace `--cpu 4` with
   `--cpu <n>`. To run single threaded on a single CPU remove the
   `--cpu 4` option. The `--split` and `--cpu` options are
   incompatible with `-p`.***

```
v-annotate.pl --split --cpu 4 -s --glsearch -r --alt_pass dupregin,discontn,indfstrn,indfstrp --alt_mnf_yes insertnp --r_lowsimok --nmiscftrthr 10 -f --keep --mkey <NC_001806.vadr for hsv1 or NC_001798.vadr for hsv2> --mdir <hsv-model-dir-path> <fasta-file-to-annotate> <output-directory-to-create>
```

5. After running the `v-annotate.pl` command in step 4, there will be a number of files
   generated in the `<output-directory-to-create>`. Among these files, there are 5-column
   tab-delimited feature table files that end with the suffix `.tbl`. There is a separate
   file for passing (`XXXXX.vadr.pass.tbl`) and failing (`XXXXX.vadr.fail.tbl`) sequences.
   The format of the `.tbl` files is described here:
   https://www.ncbi.nlm.nih.gov/genbank/feature_table/

   More information about understanding failures and error alerts can be found in the VADR
   documentation here: https://github.com/ncbi/vadr/blob/master/documentation/annotate.md

   Error alerts related to features contained in long terminal and internal repeats:<br/>
   `misc_not_failure:"1"` was added to the .minfo file FEATURE entries contained in the long terminal
   and internal repeat regions (e.g., LAT, RL1/neurovirulence protein ICP34.5,
   RL2/ubiquitin E3 ligase ICP0, RS1/transcriptional regulator ICP4) and to
   UL36/large tegument protein (see explanation below in the [HSV VADR model](#hsvmodel) section).
   Any normally fatal error alerts for these features will NOT cause failure, and will NOT be
   seen in the `XXXXX.vadr.alt.list`. To assess and potentially resolve error alerts
   and corresponding `misc_feature` entries in the `.tbl` files for these features, one
   will need to look in the `XXXXX.vadr.alt` file.

   Example `misc_feature` in `XXXXX.vadr.pass.tbl` (snippet):
   ```
   ...
   80437	71102	misc_feature
			note	similar to large tegument protein
   ...
   ```
   Corresponding alert found in `XXXXX.vadr.alt`:
   ```
   1.5.1 SAMPLE1 NC_001798 CDS large_tegument_protein 81 deletinn no DELETION_OF_NT 71637..71637:- 1 72174..72136:- 39 too large of a deletion in nucleotide-based alignment of CDS feature [39>27]
   1.5.2 SAMPLE1 NC_001798 CDS large_tegument_protein 81 deletinp no DELETION_OF_NT 71636..71636:- 1 72173..72135:- 39 too large of a deletion in protein-based alignment [39>27]
   ```
   In this case, if one is confident in the sequence/assembly for the CDS large_tegument_protein region,
   the deletinn & deletinp alerts can be resolved by adding `--nmaxdel 39 --xmaxdel 39` command line
   options to the `v-annotate.pl` command in step 4. A list of additional command line options for
   controlling alert thresholds can be found [here](https://github.com/ncbi/vadr/blob/release-1.6.3/documentation/annotate.md#v-annotatepl-options-for-controlling-thresholds-related-to-alerts-).

   Error alerts related to gaps in start/stop codon due to alignment:<br/>
   In rare cases VADR will generate an error alert when trying to infer start/stop codons.
   VADR is dependent on the alignment in specific ways, and it can infer incorrectly depending on where
   insertions/deletions are placed. This is sometimes seen in error alerts related to tegument protein VP11/12.
   These alerts will cause failure and can be found in `XXXXX.vadr.alt.list`. Here is an
   example alert: 
   ```
   SAMPLE1  NC_001798  CDS  tegument protein VP11/12  MUTATION_AT_END 99277..99275:-  99468..99467:-  expected stop codon could not be identified, first in-frame stop codon exists 3' of predicted stop position [TAA]
   ```  
   Alignment snippet of disrupted stop codon (reverse orientation):
   ```
   GGGGCCGGCGGTCGCATTTTTTTTTAATGGCTCTGGTGTCGGCCGCGTTTGAGCTTCGT
   GGGGCCGGCGGTCGCATTTTTTT..AATGGCTCTGGTGTCGGCCGCGTTTGAGCTTCGT
   00000000000000000000000..0000000000000000000000000000000000
   99999999999999999999999..9999999999999999999999999999999999
   99999999999999999999999..9999999999999999999999999999999999
   44444444444444444444444..4444444444444444444444444444444455
   44444555555555566666666..6677777777778888888888999999999900
   56789012345678901234567..8901234567890123456789012345678901
   ```
   In cases where the alignment disrupts the identification of a stop/start codon, one will need
   to manually edit the `.tbl` file with accurate coordinates to resolve the alerts.

---
## <a name="hsvmodel"></a>HSV-2 VADR model

The VADR model library for HSV-1 annotation includes a single HSV-1
model based on the Refseq sequence [`NC_001806`](https://ncbi.nlm.nih.gov/nuccore/NC_001806.2)
and for HSV-2 annotation a single HSV-2 model based on the
Refseq sequence [`NC_001798`](https://www.ncbi.nlm.nih.gov/nuccore/NC_001798.2)

This model was initially created using the following command (using HSV-2 model as an example):
```
v-build.pl --forcelong --skipbuild -f --keep NC_001798 NC_001798
```
Due to sequencing and assembly difficulties in the long terminal and 
internal repeat regions of the HSV-2 genome, low similarity range 
exceptions were added in the MODEL entry in the .minfo file. The 
coordinates of these regions were extended by 101 to account for 
SPAdes insertion of 100 Ns when unable to join contigs separated 
by complex tandem repeats. 
```
MODEL NC_001798 blastdb:"NC_001798.vadr.protein.fa" length:"154675" lowsim_exc:"1..9401:+,117920..133809:+,147933..154675:+"
```
misc_not_failure:"1" was added to the features in these repeat regions 
(e.g., LAT, RL1, RL2, RS1).
misc_not_failure:"1" was also added to the large (9.5k) UL36 gene 
and CDS features due to significant variation seen in the published genomes.

An explanation of "misc_not_failure" can be found [here](https://github.com/ncbi/vadr/blob/vadr-1.6.3/documentation/annotate.md#mnf).

Additionally, using the available HSV-2 genomes in GenBank (as of 5/2024), 
verified alternative features were added to the model. These 
alternative features were identified and confirmed by mapping reads 
available in the SRA or identifying common alternatives found in multiple 
genomes (or both). 

---

## <a name="docs"> Additional VADR documentation

* [VADR README](https://github.com/ncbi/vadr/blob/master/README.md#top)
* [VADR installation instructions](https://github.com/ncbi/vadr/blob/master/documentation/install.md#top)
  * [Installation using `vadr-install.sh`](https://github.com/ncbi/vadr/blob/master/documentation/install.md#install)
  * [Setting environment variables](https://github.com/ncbi/vadr/blob/master/documentation/install.md#environment)
  * [Verifying successful installation](https://github.com/ncbi/vadr/blob/master/documentation/install.md#tests)
  * [Further information](https://github.com/ncbi/vadr/blob/master/documentation/install.md#further)
* [`v-build.pl` example usage and command-line options](https://github.com/ncbi/vadr/blob/master/documentation/build.md#top)
  * [`v-build.pl` example usage](https://github.com/ncbi/vadr/blob/master/documentation/build.md#exampleusage)
  * [`v-build.pl` command-line options](https://github.com/ncbi/vadr/blob/master/documentation/build.md#options)
  * [Building a VADR model library](https://github.com/ncbi/vadr/blob/master/documentation/build.md#library)
  * [How the VADR 1.0 model library was constructed](https://github.com/ncbi/vadr/blob/master/documentation/build.md#1.0library)
* [`v-annotate.pl` example usage, command-line options and alert information](https://github.com/ncbi/vadr/blob/master/documentation/annotate.md#top)
  * [`v-annotate.pl` example usage](https://github.com/ncbi/vadr/blob/master/documentation/annotate.md#exampleusage)
  * [`v-annotate.pl` command-line options](https://github.com/ncbi/vadr/blob/master/documentation/annotate.md#options)
  * [Basic Information on `v-annotate.pl` alerts](https://github.com/ncbi/vadr/blob/master/documentation/annotate.md#alerts)
  * [Additional information on `v-annotate.pl` alerts](https://github.com/ncbi/vadr/blob/master/documentation/annotate.md#alerts2)
* [Explanations and examples of `v-annotate.pl` detailed alert and error messages](https://github.com/ncbi/vadr/blob/master/documentation/alerts.md#top)
  * [Output fields with detailed alert and error messages](https://github.com/ncbi/vadr/blob/master/documentation/alerts.md#files)
  * [Explanation of sequence and model coordinate fields in `.alt` files](https://github.com/ncbi/vadr/blob/master/documentation/alerts.md#coords)
  * [`toy50` toy model used in examples of alert messages](https://github.com/ncbi/vadr/blob/master/documentation/alerts.md#toy)
  * [Examples of different alert types and corresponding `.alt` output](https://github.com/ncbi/vadr/blob/master/documentation/alerts.md#examples)
  * [Posterior probability annotation in VADR output Stockholm alignments](https://github.com/ncbi/vadr/blob/master/documentation/alerts.md#pp)
* [VADR output file formats](https://github.com/ncbi/vadr/blob/master/documentation/formats.md#top)
  * [VADR output files created by all VADR scripts](https://github.com/ncbi/vadr/blob/master/documentation/formats.md#generic)
  * [`v-build.pl` output files](https://github.com/ncbi/vadr/blob/master/documentation/formats.md#build)
  * [`v-annotate.pl` output files](https://github.com/ncbi/vadr/blob/master/documentation/formats.md#annotate)
  * [VADR `coords` coordinate string format](https://github.com/ncbi/vadr/blob/master/documentation/formats.md#coords)
  * [VADR sequence naming conventions](https://github.com/ncbi/vadr/blob/master/documentation/formats.md#seqnames)


## Reference <a name="reference"></a>
* The recommended citation for using VADR is:
  Alejandro A Schäffer, Eneida L Hatcher, Linda Yankie, Lara Shonkwiler,
  J Rodney Brister, Ilene Karsch-Mizrachi, Eric P Nawrocki; *VADR:
  validation and annotation of virus sequence submissions to
  GenBank.* BMC Bioinformatics 21, 211
  (2020). https://doi.org/10.1186/s12859-020-3537-3

* This page was adapted for HSV from [Mpox virus annotation](https://github.com/ncbi/vadr/wiki/Mpox-virus-annotation)

---

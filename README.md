# <a name="documentation"></a>HSV-2 annotation

## [How to annotate HSV-2 virus sequences with VADR v1.6.3-hav-flu2](#howto)

## [HSV-2 VADR model](#hsv2model)

## [Example annotation of HSV-2 sequences](#example)
 * [How to interpret VADR alert and error messages](#howtointerpret)
 * [Explanation of options used in example annotation](#exampleoptions)

## [Additional VADR documentation](#docs)

## [References](#reference)

---
## <a name="howto"></a>How to annotate HSV-2 sequences with VADR v1.6.3-hav-flu2:

Steps for using VADR for HSV-2 annotation:

1. Download and install the latest version of VADR (v1.6.3-hav-flu2), following the
   instructions on this [page](https://github.com/ncbi/vadr/blob/vadr-1.6.3/documentation/install.md).
   Alternatively, you can use the StaPH-B VADR 1.6.3-hav-flu2
   docker image created by Curtis Kapsak (docker image names:
   `staphb/vadr:1.6.3-hav-flu2` and `staphb/vadr:latest`), available on 
   [dockerhub](https://hub.docker.com/r/staphb/vadr/tags) and
   [quay](https://quay.io/repository/staphb/vadr?tab=tags). A brief
   [README for the docker image is here](https://github.com/StaPH-B/docker-builds/tree/master/vadr/1.6.3-hav-flu2).
 
2. Download the latest HSV-2 VADR model from the top of this page
   (click button "Code" > "Download ZIP"). Note the path to the directory name
   created (`<mpxv-models-dir-path>`) for step 4.

4. <a name="trim">Remove terminal ambiguous nucleotides from your
   input fasta sequence file using the `fasta-trim-terminal-ambigs.pl`
   script in `$VADRSCRIPTSDIR/miniscripts/`.

   To remove terminal ambiguous nucleotides from your sequence file
   `<input-fasta-file>` and to remove short and long sequences to create a new trimmed file
   `<trimmed-fasta-file>`, execute:

```
$VADRSCRIPTSDIR/miniscripts/fasta-trim-terminal-ambigs.pl --minlen 50 --maxlen 165000 <input-fasta-file> > <trimmed-fasta-file>
```        

4. Run the `v-annotate.pl` program on an input trimmed fasta file with
   MPXV sequences using the recommended command and options
   below (the command is long so you will likely have to scroll to the right to view the entire command).

   ***NOTE: The following command runs multithreaded on up to 4 CPUs,
   and so is only recommended if you have at least 4 CPUs and 16Gb RAM
   available. To run on `<n>` CPUs instead, replace `--cpu 4` with
   `--cpu <n>`. To run single threaded on a single CPU remove the
   `--cpu 4` option. The `--split` and `--cpu` options are
   incompatible with `-p`.***

```
v-annotate.pl --split --cpu 4 -s --glsearch -r --alt_pass dupregin,discontn,indfstrn,indfstrp --r_lowsimok --nmiscftrthr 10 -f --keep --mdir <mpxv-models-dir-path> <fasta-file-to-annotate> <output-directory-to-create>
```

---
## <a name="hsv2model"></a>HSV-2 VADR model

The VADR model library for HSV-2 annotation includes a single HSV-2
model based on the
[`NC_001798`](https://www.ncbi.nlm.nih.gov/nuccore/NC_001798.2)
RefSeq sequence of length 154,675 nt.

This model was initially created using the following command:
```
v-build.pl --forcelong --skipbuild -f --keep NC_001798 NC_001798
```
Notes about the model:
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

---
## <a name="example">Example annotation of HSV-2 sequences

|option|explanation|
|------|-----------|
|`--split` | split input file into chunks of about 300Kb and run each chunk separately (300Kb can be changed to `<n>` by adding option `--nkb <n>`| 
|`--cpu 4` | for input sequence files > 300Kb, run multi-threaded by parallelizing across up to 4 CPU workers (4 can be changed to `<n1>` with `--cpu <n1>`), requires `--split`|
|`--glsearch` | use the glsearch program from the FASTA package for the alignment stage, required for HSV-2 annotation |
|`-s`| turn on the seed acceleration heuristic: use the max length ungapped region from blastn to seed the alignment, required for HSV-2 annotation |
|`-r`| turn on the replace-N strategy: replace stretches of Ns with expected nucleotides, where possible |
|`--minimap2` | use the minimap2 program to derive seeds and use them instead of blastn-based seeds if they are longer |
|`--nomisc` | specify that features for failing sequences not be changed to misc_features in the output .tbl file |
| `--mkey mpxv` | use the model files with prefix `mpxv` in the directory from `--mdir` |
|`--r_lowsimok` | do not report LOW_SIMILARITY* errors due to N-rich regions |
|`--r_lowsimxl 2000` | for `--r_lowsimok`, LOW_SIMILARITY* errors will not be reported only for low similarity regions of at most 2000nt |
|`--r_lowsimxd 100` | for `--r_lowsimok`, LOW_SIMILARITY* errors will not be reported only for low similarity regions in which the difference in expected length compared to the model RefSeq is at most 100nt |
| `--alt_pass discont,dupregin` | specify that discontn and dupregion alerts that occur due to repetitive regions, which are common in HSV-2 sequences, do NOT cause a sequence to fail |
| `--s_overhang 150` | specify that the number of nucleotides overlap between the seed and flanking regions aligned with glsearch be 150 nt |
| `--mdir /usr/local/vadr-models-hsv2` | specify that the models to use are in directory /usr/local/vadr-models-hsv2 |

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

* This page was adapted for HSV-2 from [Mpox virus annotation](https://github.com/ncbi/vadr/wiki/Mpox-virus-annotation)

---

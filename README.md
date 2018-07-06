# VCF2SM
Python script that integrates sequencing depth information of polymorphisms in variant call format (VCF) files and SuperMASSA software for quantitative genotype calling in polyploids.

## Dependencies

Besides [Python](https://www.python.org/) (v.2.7+) and [Matplotlib](https://matplotlib.org/), you will need the SuperMASSA source code, which is available [here](https://bitbucket.org/orserang/supermassa). SuperMASSA implements a Bayesian network to address the dosage calling problem taking genetic models into consideration, such as full-sib family and Hardy-Weinberg Equilibrium expected frequencies. Please see the original [paper](http://journals.plos.org/plosone/article?id=10.1371/journal.pone.0030906) for details and cite it together with [this one](https://bmcbioinformatics.biomedcentral.com "VCF2SM paper") if you use it in a study that ends up published.

Please note that VCF2SM is not compatible with Python 3.

## Usage

```
[-h]
-i INPUT -o OUTPUT -S SMSCRIPT -I INFERENCE
[-g GENO_PATTERN [GENO_PATTERN ...] | -r GENO_RANGE [GENO_RANGE ...]]
[-1 PAR1_PATTERN [PAR1_PATTERN ...] | -k PAR1_RANGE [PAR1_RANGE ...]]
[-2 PAR2_PATTERN [PAR2_PATTERN ...] | -l PAR2_RANGE [PAR2_RANGE ...]]
[--sF SF] [--eF EF] [-a ALLELE_DEPTH]
[-d MINIMUM_DEPTH] [-D MAXIMUM_DEPTH] [-e] [-M PLOIDY_RANGE]
[-V SIGMA_RANGE] [-f PLOIDY_FILTER] [-p POST]
[-n NAIVE_REPORTING] [-c CALLRATE] [-t THREADS]
```

The mandatory arguments include: *i)* the input/output files; *ii)* path to the SuperMASSA script; and *iii)* the inference mode.

## Input

VCF file with exact allele depth counts generated by software such as [TASSEL4-Poly](https://github.com/gramarga/tassel4-poly), [FreeBayes](https://github.com/ekg/freebayes) and [GATK](https://software.broadinstitute.org/gatk/).

## Output

VCF file with quantitative genotype calls, i.e., depicting reference and alternative allele dosages.

## Example command

Data from [this paper](http://journals.plos.org/plosone/article?id=10.1371/journal.pone.0062355) is available [here](http://journals.plos.org/plosone/article/file?type=supplementary&id=info:doi/10.1371/journal.pone.0062355.s007). You can download it and run it straight from your terminal as follows (no compilation needed):

```
python VCF2SM.py -i NewPlusOldCalls.headed.vcf -o NewPlusOldCalls.headed_poly.vcf -S ./src/SuperMASSA.py -I hw -a RA/AA -r 1:84 -d 15 -D 500 -M 4:6 -f 4 -p 0.80 -n 0.90 -c 0.75 -t 1
```

A more detailed explanation of each option is provided below:

- Inference modes for SuperMASSA include the Hardy-Weinberg model (`-I hw`, as used above), F1 biparental crosses (`-I f1`) and a general ploidy model, with no assumptions about segregation proportions or allele frequency (`-I ploidy`).

- By default, `VCF2SM` searches for allele depth information in the `AD` field, which is used by the TASSEL-GBS pipeline. If this information is present in other field(s), the argument `-a` can be used to change this behavior. In the example above, the `RA` and `AA` fields contain information about the reference and alternative allele depths, respectively.

- It is common in genotyping-by-sequencing projects to include samples from different populations in the same sequencing run. If separate inference is intended for each population, the `-r` argument specifies a range of the indices to be used by SuperMASSA. As an example, we run the script for the first 84 individuals. Alternatively, one can specify a string pattern of the sample names to match. For instance, if the VCF file contains sample names starting with either **PRJ1** or **PRJ2**, the option `-g PRJ1` will use genotypes from the first project only.

- The minimum and maximum average allele depths per variant site are specified with the `-d` and `-D` options, respectively. Polymorphisms that do not meet these criteria are filtered out.

- By default, ploidies from 2 to 16 are tested with SuperMASSA. This can be modified with the `-M` argument. Because the example dataset is from autotetraploid potato, we fit ploidies 4 and 6 (in this case, a higher than expected ploidy level is useful to capture variants that do not fit well with the tetraploid model).

- Combined with the argument above, we choose to only keep sites with an estimated ploidy of four, using the `-f 4` option.

- A posterior probability filter for each site is applied with the argument `-p 0.80`. Variants with lower probabilities are filtered out.

- SuperMASSA assigns a naive posterior probability to each individual in the population. In the example, we use `-n 0.90` to only keep samples with an associated probability of 0.90 or higher.

- To exclude loci with excessive missing data, we require a call rate of 0.75 or higher (`-c 0.75`). Note that only samples passing the naive reporting probability filter are considered as valid calls, *i.e.*, samples that have allele depth information but do not fit well to any genotype group are treated as missing.

- Finally, the number of threads to be used in the parallel processing of sites is defined with `-t`.

## More Options

VCF2SM provides some additional arguments that were not used in the previous example. These are briefly described next:

- By default, SuperMASSA uses a greedy algorithm to fit the various models. If exact inference is desired, the `-e` option can be used, but please note that this increases runtime substantially.

- The range of variances to test can be changed with the `-V` argument, which is simply passed verbatim to SuperMASSA.

- Similarly to the sample range or pattern argument, the options `-1` and `-2`, or `-k` and `-l`, can be used to specify index ranges or name patterns corresponding to both parents in F1 segregating progenies.

- If the variants are split in multiple input VCF files, the arguments `--sF` and `--eF` specify the starting and ending file indices. In this case, the input and output file paths must contain a `+` character, such as: `-i in_path/input_chr+.vcf -o out_path/output_chr+.vcf --sF 1 --eF 10` (ten VCF files are expected).

## Arguments

|Short|Long|Description|Details
|--- | --- | --- | ---
|`-i`|`--input`|Path to VCF input file|
|`-o`|`--output`|Path to VCF output file|
|`-S`|`--SMscript`|Path to the SuperMASSA script|
|`-I`|`--inference`|Inference mode for SuperMASSA|`f1` or `hw` for respective F1 segregant or Hardy-Weinberg model; `ploidy` for non-model based ploidy estimation
|`-g`|`--geno_pattern`|Pattern(s) of genotype IDs to include as samples|
|`-r`|`--geno_range`|Indices of genotypes to include as samples|List or range in the form `1:150`
|`-1`|`--par1_pattern`|Pattern(s) of genotype IDs for parent 1|
|`-k`|`--par1_range`|Indices of genotypes to include as parent 1|List or range in the form `1:5`|
|`-2`|`--par2_pattern`|Pattern(s) of genotype IDs for parent 2|
|`-l`|`--par2_range`|Indices of genotypes to include as parent 2|List or range in the form `1:5`|
||`--sF`|Starting file|
||`--eF`|Ending file|
|`-a`|`--allele_depth`|VCF field(s) to get allele depths from|Default = `AD`. It also supports two fields in the `RA/AA` format
|`-d`|`--minimum_depth`|Minimum average depth per sample for variant site to be processed (not including parents, for F1 progenies)|Default = `0`
|`-D`|`--maximum_depth`|Maximum average depth per sample for variant site to be processed (not including parents, for F1 progenies)|Default = `inf`
|`-e`|`--exact`|Perform exact inference|Default is `FALSE` (performs approximate inference)
|`-M`|`--ploidy_range`|Ploidy range to test with SuperMASSA|Default = `2:16`
|`-V`|`--sigma_range`|Sigma range for SuperMASSA|Default = `0.01:1:0.05` (range in the form low:high:step)
|`-f`|`--ploidy_filter`|Range of ploidies to include in the output|Default = NULL (keep all ploidy levels tested)
|`-p`|`--post`|Minimum posterior probability to keep variant|Default = `0.0`
|`-n`|`--naive_reporting`|Naive posterior reporting threshold|Default = `0.0`
|`-c`|`--callrate`|Minimum call rate to keep variant|Default = `0.0`
|`-t`|`--threads`|Maximum number of threads to use|Default = `1`

## Cite

Pereira GS, Garcia AAF, Margarido GRA. (2018) A fully automated pipeline for quantitative genotype calling from next generation sequencing data in polyploids. *BMC Bioinformatics* (submitted).

Serang O, Mollinari M, Garcia AAF. (2012) Efficient Exact Maximum a Posteriori Computation for Bayesian SNP Genotyping in Polyploids. *PLoS ONE* 7(2): e30906. https://doi.org/10.1371/journal.pone.0030906

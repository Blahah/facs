FACS (Fast and Accurate Classification of Sequences) C implementation
======================================================================

[![Build Status](https://travis-ci.org/SciLifeLab/facs.png?branch=master)](https://travis-ci.org/SciLifeLab/facs)

WARNING: This program is under active development and this documentation might not reflect reality.
Please file a GitHub issue and we will take care of it as soon as we can.

Introduction
------------

FACS is the C implementation of a previous Perl module, please select the
<a href="https://github.com/SciLifeLab/facs/tree/perl">perl branch</a> if
you want to have a look at the old (unsupported) implementation.

Overview
--------

* 'build' is for building a bloom filter from a reference file.
It supports large genome files (>4GB), human genome, for instance.
* 'query' is for querying a fastq/fasta file against the bloom filter.
* 'remove' is for removing contamination sequences from a fastq/fasta file.


Quickstart
----------

In order to fetch the source code and compile, run:

```
$ git clone https://github.com/SciLifeLab/facs && cd facs && make -j8
```

Please note that python's <a href="https://github.com/brainsik/virtualenv-burrito">virtualenv</a>
is needed to run the tests.

If you are compiling on MacOSX or with <a href='http://clang.llvm.org/'>LLVM's clang</a>, please note that FACS it'll run
in single core mode since <a href="http://www.phoronix.com/scan.php?page=news_item&px=MTI2MjU">OpenMP is still not ported to clang</a>.

Also, `wget` is required to run the testsuite, so please run:

```
brew install wget
```

Or use <a href="http://fink.sf.net/">whatever</a> <a href="http://www.macports.org/">packaging</a>
<a href='http://mxcl.github.com/homebrew/'>means</a> you have in MacOSX.

Usage
------

Facs uses a similar commandline structure to the one found in the popular <a href="https://github.com/lh3/bwa">bwa</a>.
There are three main commands: build, query and remove. Each of them might have slightly different flags but should
behave similarly.

```
$ ./facs -h

Program: facs - Sequence analysis using bloom filters
Version: 2.0 
Contact: Enze Liu <enze.liu@scilifelab.se>

Usage:   facs <command> [options]

Command: build         build a bloom filter from a FASTA/FASTQ reference file
         query         query a bloom filter given a FASTA/FASTQ file
         remove        remove (contamination) sequences from FASTQ/FASTA file
```

For example, to build a bloom filter out of a FASTA reference genome, one should type:

```
$ ./facs build -r ecoli.fasta -o ecoli.bloom
```

That would generate a ecoli bloom filter that could be used to query a FASTQ file:

```
$ ./facs query -r ecoli.bloom -q contaminated_sample.fastq.gz -f "json"
```

Note that both plaintext fastq files and gzip-compressed files are supported transparently
to the user.

Which would return some metrics, in json format, indicating how many reads might
be contaminated with ecoli in that particular sample:

```
{
    "timestamp": "2013-03-27T11:16:21.809+0100"
    "organism": "test200.fastq"
    "bloom_filter": "eschColi_K12.bloom"
    "total_read_count": 201,
    "contaminated_reads": 1,
    "total_hits": 36,
    "contamination_rate": 0.004975,
}
```

If one wishes to get `tsv` format to easily import in 
<a href="http://www.libreoffice.org/">LibreOffice.org</a> or Excel, indicate
`-f "tsv"` in the commandline, and a tsv file will be written in the local directory:

```
$ cat test200.fastq.tsv
organism    bloom_filter    total_read_count    contaminated_reads  contamination_rate
test200.fastq   eschColi_K12.bloom  201 1   0.004975
```


Finally, if one wants to remove those reads from the sample, one should run the following
command:

```
$ ./facs remove -r ecoli.bloom -q contaminated_sample.fastq
```

Two output files will be generated:

`contaminated_sample_ecoli_contam.fastq`
`contaminated_sample_ecoli_clean.fastq`


Python interface
----------------

A python C-Extension provides a very simple API to build, query and remove sequences,
just as described above with the plain C-based commandline.

```
$ python
Python 2.6.6 (r266:84292, Jun 18 2012, 09:57:52) 
[GCC 4.4.6 20110731 (Red Hat 4.4.6-3)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import facs
>>> facs.build("ecoli.fasta", "ecoli.bloom")
>>> facs.query("contaminated_sample.fastq.gz", "ecoli.bloom")
>>> facs.remove("contaminated_sample.fastq", "ecoli.bloom")
```

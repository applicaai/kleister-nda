Extract key information from Edgar NDA documents
=====================================================================================

Extract the information from NDAs (Non-Disclosure Agreements) about the involved parties,
jurisdiction, contract term, etc.

Note that this an information extraction task, you are given keys
(labels, attribute names) and you are expected to guess their
respective values. It is not a NER task, we are not interested in
where the information or entity is to be found, just the information
itself.

The metric used is F1 score calculated on upper-cased values. As an
auxiliary metric, also F1 on true-cased values is calculated.

It should not be assumed that for each key, a corresponding value is
to be extracted from a document. There might be some “decoy” keys, for
which no value should be given.

There might be more than value given for a given key. In such cases,
more than one value should be given. You are allowed to give more than one value, even if one is expected
(e.g. if you have two options, but you are not sure which is right), though, of course,
the metric will be lower than just guessing the right value.

Evaluation
----------

You can carry out evaluation using the [GEval](https://gitlab.com/filipg/geval),
when you generate `out.tsv` files (in the same format as `expected.tsv` files):

```
wget https://gonito.net/get/bin/geval
chmod u+x geval
./geval -t dev-0
```

Textual and graphical features
------------------------------

1D (textual) and/or 2D (graphical) features can be considered, as both
the generated PDF documents and the extracted text is available. PDF files were generated using
[Puppeteer package](https://developers.google.com/web/tools/puppeteer/) from the original
HTML files. We provide 4 different text outputs based on:
* pdf2djvu/djvu2hocr tools, ver. `0.9.8`,
* tesseract tool, ver. `4.1.1-rc1-7-gb36c`, ran with `--oem 2 -l eng --dpi 300` flags
(meaning both new and old OCR engines were used simultaneously, and language and pixel
density were forced for better results),
* textract tool, ver. `March 1, 2020`,
* combination of pdf2djvu/djvu2hocr and tesseract tools. Documents are processed with both tools, by
default we take the text from pdf2djvu/djvu2hocr, unless the text returned by tesseract is 1000
characters longer.

It should not be assumed that the OCR-ed text layer is perfect.
You are free to use alternative OCR software.

The texts are not tokenized nor pre-processed in any manner.


Directory structure
-------------------

* `README.md` — this file
* `config.txt` — GEval configuration file
* `in-header.tsv` — one-line TSV file with column names for input data (features),
* `train/` — directory with training data
* `train/in.tsv.xz` — input data for the train set
* `train/expected.tsv` — expected (reference) data for the train set
* `dev-0/` — directory with dev (test) data from the same sources as the train set
* `dev-0/in.tsv.xz` — input data for the dev set
* `dev-0/expected.tsv` — expected (reference) data for the dev set
* `test-A` — directory with test data
* `test-A/in.tsv.xz` — input data for the test set
* `test-A/expected.tsv` — expected (reference) data for the test set (hidden)
* `documents/` — all documents (for train, dev-0 and test-A), they are references in TSV files

Note that we mean TSV, *not* CSV files. In particular, double quotes
are not considered special characters here! In particular, set
`quoting` to `QUOTE_NONE` in the Python `csv` module:

    import csv
    with open('file.tsv', 'r') as tsvfile:
        reader = csv.reader(tsvfile, delimiter='\t', quoting=csv.QUOTE_NONE)
        for item in reader:
            ...

The files are sorted by MD5 sum hashes.

Structure of data sets
----------------------

The original dataset was split into train, dev-0 and test-A subsets in
a stable pseudorandom manner using the hashes (fingerprints) of the
document contents:

* the train set contains 254 items,
* the dev-0 set contains 83 items,
* the test-A set contains 203 items.


Format of the test sets
-----------------------

The input file (`in.tsv.xz`) consists of 6 TAB-separated columns:

* the file name of the document (MD5 sum for binary contents with the
  right extension), to be taken from the `documents/' subdirectory,
* list of keys in alphabetical order to be considered during
  prediction, keys are given in English with underscores in place of
  spaces and are separated with spaces,
* the plain text extracted by pdf2djvu/djvu2hocr tools from the document with the end-of-lines
  TABs and non-printable characters replaced with spaces (so that they
  would not be confused with TSV special characters),
* the plain text extracted by tesseract tool from the document with the end-of-lines
  TABs and non-printable characters replaced with spaces (so that they
  would not be confused with TSV special characters),
* the plain text extracted by textract tool from the document with the end-of-lines
  TABs and non-printable characters replaced with spaces (so that they
  would not be confused with TSV special characters),
* the plain text extracted by combination of pdf2djvu/djvu2hocr and tesseract tools
  from the document with the end-of-lines TABs and non-printable characters replaced
  with spaces (so that they would not be confused with TSV special characters).

The `expected.tsv` file is just a list of key-value pairs sorted
alphabetically (by keys). Pairs are separated with spaces, value is
separated from a key with the equals sign (`=`). The spaces and colons in values are
replaced with underscores.

In case of “decoy” keys (with no expected values), they are omitted in
`expected.tsv` files (they are *not* given with empty value).

Escaping special characters
---------------------------

The following escape sequences are used for the OCR-ed text:

* `\f` — page break (`^L`)
* `\n` — end of line,
* `\t` — tabulation
* `\\` — literal backslash

Information to be extracted
---------------------------

There are up to 6 attributes to be extracted from each document:

* `effective_date` - date in `YYYY-MM-DD` format, at which point the contract is legally binding,
* `jurisdiction` - under which state _or_ country jurisdiction is the contract signed,
* `party` - party or parties of the contract,
* `term` - length of the legal contract as expressed in the document.

Note that `party` usually occur more than once.

Normalization
-------------

The expected pieces of information were normalized to some degree:

* in attribute values, all spaces ` ` and colons `:` were replaced with an underscores `_`,
* all expected dates should be returned in `YYYY-MM-DD` format,
* values for attribute `term` are normalized with the same original units e.g. `eleven months`
is changed to `11_months`; all of them are in the same format: `{number}_{units}`.

Format of the output files for test sets
----------------------------------------

The format of the output is the same as the format of
`expected.tsv` files. The order of key-value pairs does not matter.

Format of the train set
-----------------------

The format of the train set is the same as the format of a test set.

Sources
-------

Original data was gathered from the [Edgar Database](https://www.sec.gov/edgar.shtml).


# bicleaner-hardrules

![License](https://img.shields.io/badge/License-GPLv3-blue.svg)


Bicleaner hard-rules (`bicleaner-hardrules`) is a pre-filtering step for obvious noise based on rules, poor language based on general language modelling and vulgar language based on specific language modelling.
It is part of [Bicleaner](https://github.com/bitextor/bicleaner).

## Citation

If you find Bicleaner useful, please consider citing the following papers:

> V. M. Sánchez-Cartagena, M. Bañón, S. Ortiz-Rojas and G. Ramírez-Sánchez,\
> "[Prompsit's submission to WMT 2018 Parallel Corpus Filtering shared task](http://www.statmt.org/wmt18/pdf/WMT116.pdf)",\
>in *Proceedings of the Third Conference on Machine Translation, Volume 2: Shared Task Papers*.\
>Brussels, Belgium: Association for Computational Linguistics, October 2018

```latex
@InProceedings{prompsit:2018:WMT,
  author    = { V\'{i}ctor M. S\'{a}nchez-Cartagena and Marta Ba{\~n}\'{o}n and Sergio Ortiz-Rojas and Gema Ram\'{i}rez-S\'{a}nchez},
  title     = {Prompsit's submission to WMT 2018 Parallel Corpus Filtering shared task},
  booktitle = {Proceedings of the Third Conference on Machine Translation, Volume 2: Shared Task Papers},
  month     = {October},
  address   = {Brussels, Belgium},
  publisher = {Association for Computational Linguistics}
}
```


> Gema Ramírez-Sánchez, Jaume Zaragoza-Bernabeu, Marta Bañón and Sergio Ortiz Rojas \
> "[Bifixer and Bicleaner: two open-source tools to clean your parallel data.](https://eamt2020.inesc-id.pt/proceedings-eamt2020.pdf#page=311)",\
>in *Proceedings of the 22nd Annual Conference of the European Association for Machine Translation*.\
>Lisboa, Portugal: European Association for Machine Translation, November 2020

```latex
@InProceedings{prompsit:2020:EAMT,
  author    = {Gema Ram\'{i}rez-S\'{a}nchez and Jaume Zaragoza-Bernabeu and Marta Ba{\~n}\'{o}n and Sergio Ortiz-Rojas},
  title     = {Bifixer and Bicleaner: two open-source tools to clean your parallel data.},
  booktitle = {Proceedings of the 22nd Annual Conference of the European Association for Machine Translation},
  pages	    = {291--298},
  isbn      = {978-989-33-0589-8},
  year	    = {2020},
  month     = {November},
  address   = {Lisboa, Portugal},
  publisher = {European Association for Machine Translation}
}
```

## Installation & Requirements

Bicleaner hard-rules is written in Python and can be installed using `pip`:

```bash
python3.7 -m pip install bicleaner-hardrules
```

Bicleaner hard-rules requires the [KenLM](https://github.com/kpu/kenlm) Python bindings with support for 7-gram language models. You can easily install it by running the following commands:

```bash
git clone https://github.com/kpu/kenlm
cd kenlm
python3.7 -m pip install . --install-option="--max_order 7"
mkdir -p build && cd build
cmake .. -DKENLM_MAX_ORDER=7 -DCMAKE_INSTALL_PREFIX:PATH=/your/prefix/path
make -j all install
```

The remaining modules will be automatically downloaded and installed/upgraded (if required) with the first command.

After installation, a binary file (`bicleaner-hardrules`) will be located in your `python/installation/prefix/bin` directory. This is usually `$HOME/.local/bin` or `/usr/local/bin/`.

## Cleaning

`bicleaner-hardrules` aims at detecting obvious noisy sentence pairs in a parallel corpus using a pre-defined set of rules.
Sentences that are considered noisy will be tagged with a `0` and the rest will be tagged with a `1`.

By default, the input file (the parallel corpus to be classified) must contain at least two columns, being:

* col1: Source sentence
* col2: Target sentence

but the source and target sentences column index can be customized by using the `--scol` and `--tcol` flags, in case you have more columns.

The generated output file will contain the same lines and columns that the original input file had, adding an extra column containing the Bicleaner hard-rules tag.

This tool can be run with

```bash
bicleaner-hardrules [-h]
                    [--annotated_output]
                    -s SOURCE_LANG
                    -t TARGET_LANG
                    [--tmp_dir TMP_DIR]
                    [-b BLOCK_SIZE]
                    [-p PROCESSES]
                    [--disable_lang_ident]
                    [--disable_minimal_length]
                    [--scol SCOL]
                    [--tcol TCOL]
                    [--disable_lm_filter]
                    [--disable_porn_removal]
                    [--metadata METADATA]
                    [--lm_threshold LM_THRESHOLD]
                    [-q]
                    [--debug]
                    [--logfile LOGFILE]
                    [input]
                    [output]
```

### Parameters

* positional arguments:
  * `input`: Tab-separated files to be classified (default line format: `URL1 URL2 SOURCE_SENTENCE TARGET_SENTENCE [EXTRA_COLUMNS]`, tab-separated). When input is -, reads standard input.
  * `output`: Output of the classification (default: standard output). When output is -, writes standard output.
* Optional:
  * `--metadata METADATA`: Training metadata (YAML file), generated by `bicleaner-train` or [downloaded](https://github.com/bitextor/bicleaner-data/releases/latest) as a part of a language pack. You just need to `untar` the language pack for the pair of languages of the file you want to clean. The tar file contains the YAML metadata file.
  There's a script that can download and unpack it for you, use:
  ```bash
  $ ./utils/download-pack.sh en cs ./models
  ```
  to download English-Czech language pack to the ./models directory and unpack it.
  * `-S SOURCE_TOKENIZER_COMMAND`: Source language tokenizer full command (including flags if needed). If not given, Sacremoses tokenizer is used (with `escape=False` option).
  * `-T TARGET_TOKENIZER_COMMAND`: Target language tokenizer full command (including flags if needed). If not given, Sacremoses tokenizer is used (with `escape=False` option).
  * `--scol SCOL`: Source sentence column (starting in 1) (default: 3)
  * `--tcol TCOL`: Target sentence column (starting in 1) (default: 4)
  * `--tmp_dir TMP_DIR`: Temporary directory where creating the temporary files of this program (default: default system temp dir, defined by the environment variable TMPDIR in Unix)
  * `-b BLOCK_SIZE, --block_size BLOCK_SIZE`: Sentence pairs per block (default: 10000)
  * `-p PROCESSES, --processes PROCESSES`: Number of processes to use (default: all CPUs minus one)
  * `--lm_threshold LM_THRESHOLD`: Threshold for language model fluency scoring. All sentence pairs whose LM fluency score falls below the threshold are removed (classifier score set to 0), unless the option --keep_lm_result is set. (default: 0.5)
  * `--disable_hardrules`: Disables the bicleaner_hardrules filtering (only bicleaner_classify is applied) (default: False)
  * `--disable_lm_filter`: Disables LM filtering.
  * `--disable_porn_removal`: Disables porn removal.
  * `--disable_minimal_length` : Don't apply minimal length rule (default: False).
  * `-h, --help`: show this help message and exit

* Logging:
  * `-q, --quiet`: Silent logging mode (default: False)
  * `--debug`: Debug logging mode (default: False)
  * `--logfile LOGFILE`: Store log to a file (default: `stderr`)
  * `-v, --version`: show version of this script and exit

### Example

```bash
bicleaner-hardrules  \
        corpus.en-es.raw  \
        corpus.en-es.classifed
```

This will read the "`corpus.en-es.raw`" file, tag it and write the resul in `corpus.classified`.
Each line of the new file will contain the same content as the input file, adding a column with the tag given by the Bicleaner hard-rules.

### Automatic test

We included a small test corpus and a script to check that your Bicleaner classifier is working as expected. 
In order to use it, just run:

```bash
python3.7 -m pytest -s tests/bicleaner_test.py
```

This will download the required language pack, classify the provided test corpus, and check the resulting classification scores. If everything went as expected, the output will be "1 passed in XX.XX seconds". All downloaded data will be removed at the end of the testing session.

## Training classifiers

In case you need to train a new classifier (i.e. because it is not available in the language packs provided at [bicleaner-data](https://github.com/bitextor/bicleaner-data/releases/latest)), you can use `bicleaner-train` .
`bicleaner-train` is a Python3 tool that allows you to train a classifier which predicts 
whether a pair of sentences are mutual translations or not and discards too noisy sentence pairs. Visit our [Wiki](https://github.com/bitextor/bicleaner/wiki/How-to-train-your-Bicleaner) for a detailed example on Bicleaner training.

___

![Connecting Europe Facility](https://www.paracrawl.eu/images/logo_en_cef273x39.png)

All documents and software contained in this repository reflect only the authors' view. The Innovation and Networks Executive Agency of the European Union is not responsible for any use that may be made of the information it contains.

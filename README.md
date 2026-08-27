# Extracting (some) Knowledge Graph out of a tiny text<!-- omit from toc -->

## Table of contents<!-- omit from toc -->

- [Introduction](#introduction)
- [Running the full data workflow](#running-the-full-data-workflow)

## Introduction

This repository holds a very brief snippet of text (less than ten sentences), known in Buddhism as the ["Four Noble Truths"](https://en.wikipedia.org/wiki/Four_Noble_Truths), to be used as input to test various knowledge graph extraction data workflows.
Because we expect quite reduced execution runtime for the workflows (and thus speed up development) the only researched feature of the considered text is its brevity.

The [chosen english version of the Four Noble Truths](250_BCE_-_Dhammacakkappavattana_Sutta_Four_Noble_Truths_Wikipedia_translation.md) was extracted from [Wikipedia's "Four Noble Truths" article](https://en.wikipedia.org/wiki/Four_Noble_Truths).

![Resulting Knowledge Graph](./Doc/neo4j_UI_graph_visualisation.png)

and if we remove the document structure information (Document type nodes) we get
![Resulting Knowledge Graph without Document](./Doc/neo4j_UI_graph_visualisation_without_Document.png)

## Running the full data workflow

### Prerequisites

Install and configure [`jejune_cli`](https://github.com/EricBoix/jejune_cli), then run `jejune doctor` to verify the configuration. This boils down to

```bash
uv tool install git+https://github.com/EricBoix/jejune_cli
jejune configuration doc-steward init     
# Proceed with the configuration of the files located in .jejune/ dir
jejune doctor
```

Define a convenience variable for the results directory:

```bash
export RESULTS_DIR=`pwd`/result_data
```

### pdf conversion

This is a special case where the is no need for `pdf` conversion since the original data is here in a markdown format.

### Preparation: splitting original text in chunks

```bash
jejune graph split --splitter headers
jejune graph split --splitter paragraphs
jejune graph split --splitter sentences
```

where the resulting `json` files end up is the same directory as the `markdown_file` entry (implicit input) pointed by the `catalog.yaml` document catalog file.

### Extracting the Knowledge Graph at markdown header level

Run the (Knowledge Graph) extraction (starting a neo4j database is a prerequisite)

```bash
jejune neo4j start $RESULTS_DIR
jejune graph extract `pwd` \
  --load_json_document /data/original_data/250_BCE_-_Dhammacakkappavattana_Sutta_Four_Noble_Truths_Wikipedia_translation_-_Headers_as_LangChain_document.json
jejejune neo4j stats --assert 12/15
```

Extract knowledge graph in [Turtle](https://en.wikipedia.org/wiki/Turtle_(syntax)) format

```bash
jejune neo4j dump-turtle $RESULTS_DIR Four-Noble-Truths-Wikipedia-translation-Headers_chunking.ttl
jejune neo4j stop
```

#### Optional: dump the database content for later usage

```bash
jejune neo4j dump $RESULTS_DIR neo4j.Four-Noble-Truths-Wikipedia-translation.Headers_chunking.dump
```

Assert the integrity/validity of the dump by restoring the database out of it:

```bash
WARNING: dump restoration WIPES OUT the existing database content
jejune neo4j restore $RESULTS_DIR neo4j.Four-Noble-Truths-Wikipedia-translation.Headers_chunking.dump
jejune neo4j start $RESULTS_DIR
jejune neo4j stats --assert 12/15
jejune neo4j stop
```

### Extracting the Knowledge Graph at the sentence level

Instead of defaulting to markdown document structure chunking, we can use an additional sentence based chunking as follows

```bash
jejune neo4j delete $RESULTS_DIR
jejune neo4j stats --assert 0/0

jejune graph extract `pwd` \
  --load_json_document /data/original_data/250_BCE_-_Dhammacakkappavattana_Sutta_Four_Noble_Truths_Wikipedia_translation_-_Headers_as_LangChain_document.json \
  --load_json_document /data/original_data/250_BCE_-_Dhammacakkappavattana_Sutta_Four_Noble_Truths_Wikipedia_translation_-_Sentences_as_LangChain_document.json

jejune neo4j stats --assert 24/53
```

Extract knowledge graph in [Turtle](https://en.wikipedia.org/wiki/Turtle_(syntax)) format

```bash
jejune neo4j dump-turtle $RESULTS_DIR \
  Four-Noble-Truths-Wikipedia-translation-HeadersAndSentences_chunking.ttl
jejune neo4j stop
```

which displayed with neo4j UI yields something like

![KG with sentences](./Doc/neo4j_UI_graph_visualisation_added_sentences.png)

#### Optionally dump the database

```bash
jejune neo4j dump $RESULTS_DIR neo4j.Four-Noble-Truths-Wikipedia-translation.HeadersAndSentences_chunking.dump
```

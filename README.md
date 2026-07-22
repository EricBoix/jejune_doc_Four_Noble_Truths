# Extracting (some) Knowledge Graph out of a tiny text<!-- omit from toc -->

## Table of contents<!-- omit from toc -->

- [Introduction](#introduction)
- [Running the full data workflow](#running-the-full-data-workflow)
- [Sentence-based Knowledge Graph extraction](#sentence-based-knowledge-graph-extraction)

## Introduction

This repository holds a very brief snippet of text (less than ten sentences), known in Buddhism as the ["Four Noble Truths"](https://en.wikipedia.org/wiki/Four_Noble_Truths), to be used as input to test various knowledge graph extraction data workflows.
Because we expect quite reduced execution runtime for the workflows (and thus speed up development) the only researched feature of the considered text is its brevity.

The [chosen english version of the Four Noble Truths](250_BCE_-_Dhammacakkappavattana_Sutta_Four_Noble_Truths_Wikipedia_translation.md) was extracted from [Wikipedia's "Four Noble Truths" article](https://en.wikipedia.org/wiki/Four_Noble_Truths).

![Resulting Knowledge Graph](./Doc/neo4j_UI_graph_visualisation.png)

and if we remove the document structure information (Document type nodes) we get
![Resulting Knowledge Graph without Document](./Doc/neo4j_UI_graph_visualisation_without_Document.png)

## Running the full data workflow

Install and configure [`jejune_cli`](https://github.com/EricBoix/jejune_cli), then run `jejune doctor` to verify the configuration. This boils down to

```bash
uv tool install git+https://github.com/EricBoix/jejune_cli
jejune configuration init     
# Proceed with the configuration of the files located in .jejune/ dir
jejune doctor
```

Define a convenience variable for the results directory:

```bash
export RESULTS_DIR=`pwd`/result_data
```

Note: no need for `pdf` conversion since the original data is here in a markdown format.

Run the (Knowledge Graph) extraction (launch a neo4j database is a prerequisite)

```bash
jejune neo4j start $RESULTS_DIR
jejune graph extract `pwd`/original_data \
  --load_markdown_document \
  250_BCE_-_Dhammacakkappavattana_Sutta_Four_Noble_Truths_Wikipedia_translation.md
jejune neo4j stats --assert 12/15
```

Dump the database content for later usage (optional)

```bash
jejune neo4j stop
jejune neo4j dump $RESULTS_DIR neo4j.Four-Noble-Truths-Wikipedia-translation.Markdown.dump
# Restore the database out of the dump (just to make sure)
# WARNING: restoring DELETEs the existing database
jejune neo4j restore $RESULTS_DIR neo4j.Four-Noble-Truths-Wikipedia-translation.Markdown.dump
jejune neo4j start $RESULTS_DIR
jejune neo4j stats --assert 12/15
```

Extract knowledge graph in [Turtle](https://en.wikipedia.org/wiki/Turtle_(syntax)) format (initial restoration validates the previous dump):

```bash
jejune neo4j dump-turtle $RESULTS_DIR Four-Noble-Truths-Wikipedia-translation-Markdown.ttl
jejune neo4j stop
```

## Sentence-based Knowledge Graph extraction

Instead of defaulting to document structure aware chunking, we can use an additional sentence based chunking as follows

```bash
jejune neo4j delete $RESULTS_DIR
jejune neo4j stats --assert 0/0
jejune graph extract `pwd`/original_data \
  --load_markdown_document \
  250_BCE_-_Dhammacakkappavattana_Sutta_Four_Noble_Truths_Wikipedia_translation.md \
  --load_json_document \
  Four_Noble_Truth_-_Sentences_as_LangChain_document.json
jejune neo4j stats --assert 28/56
```

Eventually dump the database for later usage

```bash
jejune neo4j stop
jejune neo4j dump $RESULTS_DIR neo4j.Four-Noble-Truths-Wikipedia-translation-added-sentences.Markdown.dump
```

which yields

![KG with sentences](./Doc/neo4j_UI_graph_visualisation_added_sentences.png)

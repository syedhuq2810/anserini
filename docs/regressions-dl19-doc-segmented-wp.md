# Anserini Regressions: TREC 2019 Deep Learning Track (Document)

**Models**: various bag-of-words approaches on segmented documents with WordPiece tokenization

This page describes experiments, integrated into Anserini's regression testing framework, on the [TREC 2019 Deep Learning Track document ranking task](https://trec.nist.gov/data/deep2019.html).
Here we are using **WordPiece tokenization** (i.e., from BERT) on passages from the documents.
At retrieval time, we select the score of the highest-scoring passage from a document as the score for that document to produce a document ranking; this is known as the MaxP technique.
In general, effectiveness is lower than with "standard" Lucene tokenization for two reasons: (1) we're losing stemming, and (2) some terms are chopped into less meaningful subwords.

Note that the NIST relevance judgments provide far more relevant documents per topic, unlike the "sparse" judgments provided by Microsoft (these are sometimes called "dense" judgments to emphasize this contrast).

The exact configurations for these regressions are stored in [this YAML file](../src/main/resources/regression/dl19-doc-segmented-wp.yaml).
Note that this page is automatically generated from [this template](../src/main/resources/docgen/templates/dl19-doc-segmented-wp.template) as part of Anserini's regression pipeline, so do not modify this page directly; modify the template instead.

From one of our Waterloo servers (e.g., `orca`), the following command will perform the complete regression, end to end:

```
python src/main/python/run_regression.py --index --verify --search --regression dl19-doc-segmented-wp
```

## Indexing

Typical indexing command:

```
target/appassembler/bin/IndexCollection \
  -collection JsonCollection \
  -input /path/to/msmarco-doc-segmented-wp \
  -index indexes/lucene-index.msmarco-doc-segmented-wp/ \
  -generator DefaultLuceneDocumentGenerator \
  -threads 16 -storePositions -storeDocvectors -storeRaw -pretokenized \
  >& logs/log.msmarco-doc-segmented-wp &
```

The directory `/path/to/msmarco-doc-segmented/` should be a directory containing the segmented corpus in Anserini's jsonl format.
See [this page](experiments-msmarco-doc-doc2query-details.md) for how to prepare the corpus.

For additional details, see explanation of [common indexing options](common-indexing-options.md).

## Retrieval

Topics and qrels are stored in [`src/main/resources/topics-and-qrels/`](../src/main/resources/topics-and-qrels/).
The regression experiments here evaluate on the 43 topics for which NIST has provided judgments as part of the TREC 2019 Deep Learning Track.
The original data can be found [here](https://trec.nist.gov/data/deep2019.html).

After indexing has completed, you should be able to perform retrieval as follows:

```
target/appassembler/bin/SearchCollection \
  -index indexes/lucene-index.msmarco-doc-segmented-wp/ \
  -topics src/main/resources/topics-and-qrels/topics.dl19-doc.wp.tsv.gz \
  -topicreader TsvInt \
  -output runs/run.msmarco-doc-segmented-wp.bm25-default.topics.dl19-doc.wp.txt \
  -bm25 -pretokenized -hits 10000 -selectMaxPassage -selectMaxPassage.delimiter "#" -selectMaxPassage.hits 1000 &
```

Evaluation can be performed using `trec_eval`:

```
tools/eval/trec_eval.9.0.4/trec_eval -c -M 100 -m map src/main/resources/topics-and-qrels/qrels.dl19-doc.txt runs/run.msmarco-doc-segmented-wp.bm25-default.topics.dl19-doc.wp.txt
tools/eval/trec_eval.9.0.4/trec_eval -c -m ndcg_cut.10 src/main/resources/topics-and-qrels/qrels.dl19-doc.txt runs/run.msmarco-doc-segmented-wp.bm25-default.topics.dl19-doc.wp.txt
tools/eval/trec_eval.9.0.4/trec_eval -c -m recall.100 src/main/resources/topics-and-qrels/qrels.dl19-doc.txt runs/run.msmarco-doc-segmented-wp.bm25-default.topics.dl19-doc.wp.txt
tools/eval/trec_eval.9.0.4/trec_eval -c -m recall.1000 src/main/resources/topics-and-qrels/qrels.dl19-doc.txt runs/run.msmarco-doc-segmented-wp.bm25-default.topics.dl19-doc.wp.txt
```

## Effectiveness

With the above commands, you should be able to reproduce the following results:

| **AP@100**                                                                                                   | **BM25 (default)**|
|:-------------------------------------------------------------------------------------------------------------|-----------|
| [DL19 (Doc)](https://trec.nist.gov/data/deep2019.html)                                                       | 0.2269    |
| **nDCG@10**                                                                                                  | **BM25 (default)**|
| [DL19 (Doc)](https://trec.nist.gov/data/deep2019.html)                                                       | 0.5206    |
| **R@100**                                                                                                    | **BM25 (default)**|
| [DL19 (Doc)](https://trec.nist.gov/data/deep2019.html)                                                       | 0.3708    |
| **R@1000**                                                                                                   | **BM25 (default)**|
| [DL19 (Doc)](https://trec.nist.gov/data/deep2019.html)                                                       | 0.6476    |
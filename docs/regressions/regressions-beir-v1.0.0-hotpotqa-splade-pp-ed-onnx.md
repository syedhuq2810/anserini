# Anserini Regressions: BEIR (v1.0.0) &mdash; HotpotQA

**Model**: [SPLADE++ (CoCondenser-EnsembleDistil)](https://arxiv.org/abs/2205.04733) (using ONNX for on-the-fly query encoding)

This page describes regression experiments, integrated into Anserini's regression testing framework, using [SPLADE++ (CoCondenser-EnsembleDistil)](https://arxiv.org/abs/2205.04733) on [BEIR (v1.0.0) &mdash; HotpotQA](http://beir.ai/).
The model itself can be download [here](https://huggingface.co/naver/splade-cocondenser-ensembledistil).
See the [official SPLADE repo](https://github.com/naver/splade) and the following paper for more details:

> Thibault Formal, Carlos Lassance, Benjamin Piwowarski, and Stéphane Clinchant. [From Distillation to Hard Negative Sampling: Making Sparse Neural IR Models More Effective.](https://dl.acm.org/doi/10.1145/3477495.3531857) _Proceedings of the 45th International ACM SIGIR Conference on Research and Development in Information Retrieval_, pages 2353–2359.

In these experiments, we are using ONNX to perform query encoding on the fly.

The exact configurations for these regressions are stored in [this YAML file](../../src/main/resources/regression/beir-v1.0.0-hotpotqa-splade-pp-ed-onnx.yaml).
Note that this page is automatically generated from [this template](../../src/main/resources/docgen/templates/beir-v1.0.0-hotpotqa-splade-pp-ed-onnx.template) as part of Anserini's regression pipeline, so do not modify this page directly; modify the template instead and then run `bin/build.sh` to rebuild the documentation.

From one of our Waterloo servers (e.g., `orca`), the following command will perform the complete regression, end to end:

```
python src/main/python/run_regression.py --index --verify --search --regression beir-v1.0.0-hotpotqa-splade-pp-ed-onnx
```

All the BEIR corpora, encoded by the SPLADE++ CoCondenser-EnsembleDistil model, are available for download:

```bash
wget https://rgw.cs.uwaterloo.ca/pyserini/data/beir-v1.0.0-splade-pp-ed.tar -P collections/
tar xvf collections/beir-v1.0.0-splade-pp-ed.tar -C collections/
```

The tarball is 42 GB and has MD5 checksum `9c7de5b444a788c9e74c340bf833173b`.
After download and unpacking the corpora, the `run_regression.py` command above should work without any issue.

## Indexing

Sample indexing command:

```
target/appassembler/bin/IndexCollection \
  -collection JsonVectorCollection \
  -input /path/to/beir-v1.0.0-hotpotqa-splade-pp-ed \
  -generator DefaultLuceneDocumentGenerator \
  -index indexes/lucene-index.beir-v1.0.0-hotpotqa-splade-pp-ed/ \
  -threads 16 -impact -pretokenized \
  >& logs/log.beir-v1.0.0-hotpotqa-splade-pp-ed &
```

The important indexing options to note here are `-impact -pretokenized`: the first tells Anserini not to encode BM25 doclengths into Lucene's norms (which is the default) and the second option says not to apply any additional tokenization on the pre-encoded tokens.
For additional details, see explanation of [common indexing options](../../docs/common-indexing-options.md).

## Retrieval

Topics and qrels are stored [here](https://github.com/castorini/anserini-tools/tree/master/topics-and-qrels), which is linked to the Anserini repo as a submodule.

After indexing has completed, you should be able to perform retrieval as follows:

```
target/appassembler/bin/SearchCollection \
  -index indexes/lucene-index.beir-v1.0.0-hotpotqa-splade-pp-ed/ \
  -topics tools/topics-and-qrels/topics.beir-v1.0.0-hotpotqa.test.tsv.gz \
  -topicReader TsvString \
  -output runs/run.beir-v1.0.0-hotpotqa-splade-pp-ed.splade-pp-ed.topics.beir-v1.0.0-hotpotqa.test.txt \
  -impact -pretokenized -removeQuery -hits 1000 -encoder SpladePlusPlusEnsembleDistil &
```

Evaluation can be performed using `trec_eval`:

```
target/appassembler/bin/trec_eval -c -m ndcg_cut.10 tools/topics-and-qrels/qrels.beir-v1.0.0-hotpotqa.test.txt runs/run.beir-v1.0.0-hotpotqa-splade-pp-ed.splade-pp-ed.topics.beir-v1.0.0-hotpotqa.test.txt
target/appassembler/bin/trec_eval -c -m recall.100 tools/topics-and-qrels/qrels.beir-v1.0.0-hotpotqa.test.txt runs/run.beir-v1.0.0-hotpotqa-splade-pp-ed.splade-pp-ed.topics.beir-v1.0.0-hotpotqa.test.txt
target/appassembler/bin/trec_eval -c -m recall.1000 tools/topics-and-qrels/qrels.beir-v1.0.0-hotpotqa.test.txt runs/run.beir-v1.0.0-hotpotqa-splade-pp-ed.splade-pp-ed.topics.beir-v1.0.0-hotpotqa.test.txt
```

## Effectiveness

With the above commands, you should be able to reproduce the following results:

| **nDCG@10**                                                                                                  | **SPLADE++ (CoCondenser-EnsembleDistil)**|
|:-------------------------------------------------------------------------------------------------------------|-----------|
| BEIR (v1.0.0): HotpotQA                                                                                      | 0.6868    |
| **R@100**                                                                                                    | **SPLADE++ (CoCondenser-EnsembleDistil)**|
| BEIR (v1.0.0): HotpotQA                                                                                      | 0.8177    |
| **R@1000**                                                                                                   | **SPLADE++ (CoCondenser-EnsembleDistil)**|
| BEIR (v1.0.0): HotpotQA                                                                                      | 0.8951    |
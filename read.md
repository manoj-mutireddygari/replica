MANDATORY: ROW-LEVEL WORD/PHRASE AND SENTENCE EMBEDDING GENERATION AND STORAGE

This is a core architectural requirement. Do not omit or simplify it.

The master dataset must be converted into a searchable embedding-based representation during offline index construction.

For every master record and every available language description (English, French and Spanish), generate and store two separate types of representations:

A. Sentence-level embeddings

* Generate one embedding representing the complete description in each available language.
* Store English, French and Spanish description embeddings separately.
* Link every vector to its original master record ID and language.
* Use these vectors for whole-description semantic retrieval.

B. Word/phrase-level embeddings

* Extract meaningful individual terms and distinctive phrases from each description.
* Generate separate embeddings for selected terms and phrases.
* Prioritize product names, technical terminology, features, functions and distinctive descriptive phrases.
* Store each embedding with its master record ID, language, original phrase and source-field information.
* Build a separate searchable phrase-vector index.

Do not generate or store every token vector indiscriminately. Use meaningful phrase extraction and benchmark whether individual-word vectors provide additional value.

C. Embedding storage

Maintain separate indexes:

1. Sentence embedding index.
2. Word/phrase embedding index.

Both must preserve references to the same master records.

The original master dataset must remain read-only.

Store embedding metadata, model version, embedding dimensions, language, record reference, source field and preprocessing version.

D. Runtime query matching

When OCR processing is complete:

1. Generate a sentence embedding from the recognized product description.
2. Generate embeddings for selected distinctive words and phrases.
3. Search the precomputed sentence embedding index using GPU acceleration.
4. Search the precomputed word/phrase embedding index using GPU acceleration.
5. Search BM25 indexes across English, French and Spanish.
6. Run character n-gram OCR-resistant retrieval.
7. Run multilingual terminology retrieval using curated translation and synonym dictionaries.
8. Run structured feature and specification matching.

Execute these retrieval channels in parallel wherever practical.

E. Candidate fusion and reranking

Combine the retrieval results using Reciprocal Rank Fusion (RRF).

Deduplicate by master record ID while retaining the scores and ranks from every retrieval channel.

Pass approximately 100–200 candidates initially to the LightGBM reranking model, with actual limits determined by Recall@K and latency benchmarks.

The reranker must consider:

* Sentence similarity.
* Word/phrase similarity.
* BM25 relevance.
* Character similarity.
* Terminology agreement.
* Feature agreement.
* Specification agreement.
* OCR confidence.
* Contradictions.
* Retrieval-channel evidence.

Use calibrated confidence and ambiguity checks before returning one master record.

F. Important distinction

Sentence embeddings and word/phrase embeddings are not interchangeable.

Sentence embeddings capture overall description meaning.

Word/phrase embeddings capture distinctive local semantic evidence.

Both must be independently generated, stored, indexed, retrieved and evaluated.

Do not replace one with the other.
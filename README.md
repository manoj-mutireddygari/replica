# replica
MASTER PROJECT PROMPT — FORMAID OFFLINE PRODUCT IDENTIFICATION SDK

1. Project Overview

I want to independently design and develop a professional, high-performance, fully offline Product Identification SDK using Python.

The project is called FormaID Product Identification Engine.

The objective is to identify a physical product by scanning its packaging, extracting the printed information using YOLO-based text detection and OCR, and matching the extracted information against a confidential master dataset containing more than 60,000 product records.

The dataset may eventually grow to 600,000 records or more.

The SDK must identify the correct master record even when:

* The product box contains extensive printed information.
* The OCR output contains spelling or character recognition errors.
* The box language differs from the master description language.
* Different products have very similar descriptions.
* Multiple records share common product names and technical terminology.
* Product records belong to parent projects containing multiple child product IDs.

The system should aim for at least 95% identification accuracy, validated using real product-box images.

The matching engine should target sub-second execution after OCR has completed, using an NVIDIA GPU.

Do not assume this accuracy or latency is guaranteed. Design the architecture and evaluation process to measure and improve both.

⸻

2. Strict Project Scope

We are developing the AI identification SDK and its supporting indexing infrastructure.

Do not develop the Flutter application or any UI.

The Flutter application will be developed separately and integrated with this SDK later.

Your responsibility is to design and implement:

1. YOLO text-region detection integration.
2. Offline OCR recognition integration.
3. Image and OCR preprocessing.
4. Multilingual text processing.
5. Master dataset ingestion.
6. Offline embedding generation.
7. Word/phrase and sentence embedding storage.
8. Traditional lexical retrieval.
9. OCR-resistant retrieval.
10. Multilingual terminology retrieval.
11. Structured attribute extraction and matching.
12. Vector indexing and GPU search.
13. Candidate fusion.
14. LightGBM reranking.
15. Confidence calibration.
16. Product hierarchy handling.
17. Local catalog and model management interfaces.
18. Python SDK packaging.
19. Benchmarking, testing and performance optimization.

The implementation must be modular, reusable, maintainable and suitable for integration into an independent Flutter application.

⸻

3. System Constraints and Important Facts

These requirements are mandatory.

3.1 Product code is NOT printed on the physical box

The master dataset contains an internal product code, usually 12–15 characters, potentially containing hyphens and special characters.

However, this code is not printed on the physical product packaging.

Therefore:

* Never depend on OCR to extract this code.
* Never use the internal product code as the primary search query.
* Never assume the code can be recovered from the box image.
* First identify the correct master record using printed product information.
* Retrieve its internal code only after identification.

The internal code acts as the final output identifier.

3.2 Confidential master dataset

The master dataset is confidential.

Requirements:

* Read the original master dataset through an authorized read-only adapter.
* Never modify the original dataset.
* Never expose the complete dataset through public APIs.
* Create separate derived local indexes.
* Treat embeddings, indexes, terminology dictionaries and derived metadata as confidential.
* Implement appropriate local access controls and data protection.
* Do not upload confidential data to external services.

3.3 Fully offline operation

The complete identification process must work without an internet connection.

Internet access may be used during authorized installation or initial preparation to obtain model files.

After the required model files, dependencies and catalog indexes are installed:

* No cloud APIs.
* No external inference services.
* No network dependency for identification.
* No online embedding generation.
* No external database requirement.

3.4 Model restrictions

Allowed:

* YOLO .pt detection models.
* Offline OCR models.
* Small multilingual embedding models.
* Local sentence and phrase embedding inference.
* LightGBM.
* Traditional machine-learning and information-retrieval algorithms.

Not allowed:

* Cloud-based embedding APIs.
* LLM-based product identification.
* Cloud-based OCR.
* External inference services.
* A requirement to connect to the internet during identification.

The embedding model must be compact and locally executable.

⸻

4. Master Dataset Structure

The dataset contains more than 60K records and may scale to 600K records.

The known columns include:

Column	Description
code	Internal product identifier
description_en	English description
description_fr	French description
description_es	Spanish description

There may be additional structured columns. Inspect the actual dataset before finalizing the schema.

Important characteristics:

* English, French and Spanish descriptions.
* Different languages may appear on the physical packaging and in the master record.
* Some translations may be incomplete or inconsistent.
* Records may have similar descriptions.
* Some projects contain one product ID.
* Other projects contain multiple product IDs representing different products, features or components.

Do not assume that every project has the same hierarchy.

Inspect the real data relationships and preserve their meaning.

⸻

5. Complete System Architecture

Implement the following architecture.

PRODUCT IMAGE
|
v
IMAGE PREPROCESSING
|
v
YOLO TEXT DETECTION (.pt)
|
v
TEXT REGION EXTRACTION AND RECTIFICATION
|
v
OFFLINE OCR RECOGNITION
|
v
OCR STRUCTURED OUTPUT
|
v
TEXT NORMALIZATION AND LANGUAGE PROCESSING
|
v
FEATURE AND ATTRIBUTE EXTRACTION
|
v
PARALLEL RETRIEVAL ENGINES
|
+–> SENTENCE EMBEDDING SEARCH
|
+–> WORD/PHRASE EMBEDDING SEARCH
|
+–> BM25 SEARCH
|
+–> OCR-RESISTANT CHARACTER N-GRAM SEARCH
|
+–> MULTILINGUAL TERMINOLOGY SEARCH
|
+–> STRUCTURED ATTRIBUTE SEARCH
|
v
RECIPROCAL RANK FUSION (RRF)
|
v
CANDIDATE DEDUPLICATION AND ENRICHMENT
|
v
FINE-GRAINED CANDIDATE COMPARISON
|
v
LIGHTGBM RERANKING
|
v
CONFIDENCE CALIBRATION AND AMBIGUITY CHECKS
|
+–> IDENTIFIED
|
+–> NEEDS VERIFICATION
|
+–> NOT IDENTIFIED
|
v
RETRIEVE INTERNAL PRODUCT CODE FROM MASTER RECORD

Each component must be independently implemented and testable.

⸻

6. Image Processing and YOLO Integration

Use OpenCV for image processing.

Use the provided YOLO .pt model for text-region detection, provided its architecture and training configuration are compatible.

Important distinction:

YOLO text detection is not OCR recognition.

YOLO identifies the locations of text regions. A separate OCR recognizer must recognize the characters inside those regions.

Required workflow:

1. Load the product image.
2. Correct orientation.
3. Resize appropriately.
4. Apply perspective correction when required.
5. Improve contrast when necessary.
6. Run YOLO text detection.
7. Extract bounding boxes.
8. Crop detected regions.
9. Rectify and preprocess the crops.
10. Pass them to the OCR recognizer.
11. Reconstruct reading order.
12. Return recognized text and confidence.

Evaluate suitable offline OCR engines such as PaddleOCR, EasyOCR or Tesseract.

Do not select an OCR engine solely based on popularity. Benchmark recognition quality on actual English, French and Spanish product packaging.

Retain:

* Original recognized text.
* Corrected or normalized text.
* Bounding boxes.
* OCR confidence.
* Region order.
* Language information, when available.

Do not blindly correct ambiguous characters such as O/0, I/1 or S/5. Preserve alternatives when uncertainty matters.

If OCR confidence is low, consider alternative preprocessing or recognition passes rather than blindly trusting one result.

⸻

7. Text Processing and Feature Extraction

After OCR, organize the extracted text into meaningful groups.

Recommended categories:

1. Product name.
2. Brand.
3. Product description.
4. Technical specifications.
5. Features.
6. Keywords.
7. Compatibility information.
8. Measurements and quantities.
9. Manufacturer information.
10. General marketing text.

Not every detected word should receive equal importance.

Distinctive product features and technical specifications should carry more identification value than generic marketing phrases.

Implement:

* Unicode normalization.
* Accent-preserving original text.
* Separate normalized search text.
* Language-specific tokenization.
* Stop-word handling.
* Phrase extraction.
* Number and unit extraction.
* Synonym handling.
* OCR confidence propagation.
* Duplicate phrase removal.

Use language-aware processing for English, French and Spanish.

Do not translate all records into English as the main strategy.

Preserve the original language descriptions.

⸻

8. Multilingual Embedding Architecture

Embeddings are allowed, but they must be generated locally.

Use a compact multilingual model as the starting point.

Initial benchmark candidates:

* intfloat/multilingual-e5-small
* sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2

Consider LaBSE only if its additional computational cost provides a meaningful accuracy improvement.

Do not automatically assume one model is superior. Benchmark them on actual product descriptions and cross-language queries.

8.1 Sentence embeddings

Generate sentence embeddings for:

* English master descriptions.
* French master descriptions.
* Spanish master descriptions.

Each language description should have its own vector representation associated with the same master record ID.

At query time, generate an embedding for the OCR description and compare it against the combined multilingual description index.

This allows an English product box to match a French or Spanish master description when the model recognizes the semantic relationship.

8.2 Word and phrase embeddings

Do not restrict the architecture to one sentence vector.

Generate additional representations for important product phrases, including:

* Product names.
* Distinctive features.
* Technical phrases.
* Functional descriptions.
* Important product terminology.

For example:

Product description:

“Wireless temperature monitoring system with Bluetooth connectivity.”

Possible phrase representations:

* Wireless temperature monitoring.
* Bluetooth connectivity.
* Temperature sensor.
* Remote monitoring.

Compare these phrase representations independently.

Individual token-level embeddings may be explored if they demonstrably improve retrieval, but do not automatically store every token vector for every record.

Prefer selected word/phrase representations initially because they reduce storage and noisy matches.

8.3 Important E5 model handling

Follow the selected model’s actual input conventions.

For example, if using E5 retrieval conventions:

* Queries must use the appropriate query prefix.
* Master descriptions must use the appropriate passage prefix.

Apply consistent preprocessing and inference conventions during both index building and runtime.

Do not change embedding preprocessing after creating the catalog index without rebuilding or versioning the index.

⸻

9. Traditional Retrieval Engines

Embeddings alone are insufficient.

Implement the following independent retrieval engines.

9.1 BM25 search

Create three language-specific search indexes:

* English BM25.
* French BM25.
* Spanish BM25.

Search all three languages for every product scan.

This is necessary because the packaging language may differ from the language of the matching master description.

Recommended technologies:

* SQLite FTS5 for a prototype.
* Tantivy for a specialized high-performance implementation.

Compare their retrieval performance and operational complexity.

9.2 OCR-resistant character n-gram search

This is an essential component.

OCR may recognize:

* Bluetooth as Blueto0th.
* Controller as ControIIer.
* Temperature as Tempereture.
* Model numbers with incorrect characters.

Use character n-gram TF-IDF or another indexed character-level retrieval implementation.

The purpose is to identify records even when the OCR text is not perfectly recognized.

Benchmark n-gram sizes and preprocessing configurations.

Do not depend exclusively on word-level matching.

9.3 Fuzzy matching

Use RapidFuzz for candidate-level comparison.

It should compare:

* Product names.
* Important phrases.
* Technical terminology.
* Recognized keywords.

Do not execute expensive fuzzy comparisons against the complete 600K catalog.

Use it after initial candidate generation.

9.4 Multilingual terminology search

This must be an independent retrieval component.

Build a curated multilingual terminology dictionary containing:

* English terms.
* French equivalents.
* Spanish equivalents.
* Technical synonyms.
* Domain-specific vocabulary.
* Common abbreviations.

Example:

English	French	Spanish
Wireless	Sans fil	Inalámbrico
Temperature	Température	Temperatura
Monitoring	Surveillance	Monitorización
Controller	Contrôleur	Controlador
Sensor	Capteur	Sensor

When OCR detects a term in one language, expand the search query using verified equivalents.

This is particularly important when semantic embeddings are not sufficiently accurate for technical vocabulary.

Do not blindly translate every phrase.

Use a controlled and versioned terminology dictionary.

9.5 Structured attribute matching

Extract technical attributes from both the OCR text and master records.

Examples:

* Voltage.
* Capacity.
* Dimensions.
* Connectivity.
* Material.
* Operating range.
* Supported standards.
* Functional features.

Normalize equivalent units.

For example:

* 1 kg = 1000 g.
* 12 volts = 12 V.

Compare normalized values directly.

Do not use embeddings as a replacement for exact numerical and technical comparisons.

Treat missing evidence as unknown, not as a contradiction.

⸻

10. Vector Database and GPU Retrieval

The production hardware will have an NVIDIA GPU.

Use GPU acceleration for embedding generation and vector retrieval when the hardware and available VRAM justify it.

Benchmark FAISS:

* GPU Flat index for exact nearest-neighbour search.
* GPU IVF for approximate retrieval.
* HNSW as an alternative CPU search configuration.

Start with exact search as a baseline.

Then measure whether approximate retrieval materially improves latency without unacceptable recall loss.

Use cosine similarity or normalized inner-product search consistently.

All master embeddings must be generated during offline index construction.

Never generate embeddings for the entire catalog during a product scan.

Only generate embeddings for the current OCR query and its selected phrases.

Keep the embedding model loaded in memory.

Keep vector indexes resident in GPU memory when possible. If VRAM is insufficient, evaluate CPU-resident or hybrid indexes.

Storage estimation

For 600K records, three language descriptions, 384-dimensional vectors and float32 storage:

600,000 × 3 × 384 × 4 bytes ≈ 2.58 GB.

Float16 reduces the raw sentence-vector storage to approximately 1.29 GB.

These estimates exclude:

* Phrase vectors.
* ANN index overhead.
* Metadata.
* Lexical indexes.
* Attribute indexes.

Benchmark FP16 and FP32 retrieval quality before choosing the final storage format.

⸻

11. Candidate Generation and Fusion

Run retrieval channels in parallel.

Initial candidate-generation targets:

Retrieval method	Starting target
Sentence embedding search	Top 100
Phrase embedding search	Top 100
BM25	Top 100
Character n-gram	Top 100
Terminology-expanded retrieval	Top 100
Attribute retrieval	Top 100

These are experimental starting values, not fixed limits.

Merge candidates by master record ID.

Do not treat different language descriptions of the same record as separate final products.

Use Reciprocal Rank Fusion (RRF):

RRF(d) = Σ 1 / (k + rank_r(d))

RRF combines retrieval rankings without directly mixing raw cosine similarity and BM25 scores.

Preserve the original retrieval scores and ranks as evidence for the next stage.

The main requirement is high candidate recall: the correct master record must enter the candidate pool before reranking.

⸻

12. Fine-Grained Reranking with LightGBM

This is one of the most important components.

Frequently similar descriptions make simple cosine ranking insufficient.

Use LightGBM as a locally trained reranking model.

It is a traditional gradient-boosted decision-tree model, not an LLM.

Generate candidate-level features such as:

1. Sentence embedding similarity.
2. Phrase embedding similarity.
3. BM25 score.
4. Character similarity.
5. Product-name similarity.
6. Feature agreement.
7. Specification agreement.
8. Language compatibility.
9. Terminology agreement.
10. OCR confidence.
11. Retrieval ranks.
12. Number of retrieval channels supporting the candidate.
13. Explicit contradiction features.
14. Parent-child relationship information, where relevant.

Train the reranker using labelled examples containing:

* Actual product-box OCR results.
* Correct master record IDs.
* Hard negative candidates.
* Similar product descriptions.
* Cross-language matches.
* OCR-corrupted examples.

Do not train only on easy negative examples.

The model must learn to distinguish closely related products.

Contradiction handling

Use three different evidence states:

* Match.
* Unknown or missing.
* Explicit contradiction.

Example:

OCR detects 12V with high confidence, while the candidate explicitly specifies 24V.

This is a potential contradiction.

If OCR did not detect voltage, do not assume the candidate is wrong.

If OCR confidence is low, avoid applying a strong contradiction penalty.

Do not allow generic semantic similarity to override strong technical contradictions.

⸻

13. Product Hierarchy

The master data may contain projects with one or multiple product IDs.

Preserve the actual hierarchy.

Recommended identification:

1. Identify the likely parent project.
2. Retrieve the associated child products.
3. Compare the printed evidence against those child records.
4. Select a child only when sufficient distinguishing evidence exists.

Do not automatically choose a child simply because its parent matches.

If the parent is identifiable but the specific child is not, return a parent-level or verification status as appropriate.

Do not invent project relationships.

Inspect the actual master schema first.

⸻

14. Confidence Calibration and Decision Engine

A top-ranked result is not necessarily correct.

The SDK must evaluate:

* Top candidate’s ranking evidence.
* Difference between first and second candidates.
* Agreement across retrieval channels.
* Distinctive feature agreement.
* Technical specification agreement.
* OCR confidence.
* Contradictions.
* Hierarchical consistency.

Use a calibrated confidence model.

Consider logistic calibration or isotonic regression.

Do not treat raw cosine similarity, RRF scores or LightGBM outputs as probabilities without calibration.

The SDK must support three outcomes:

IDENTIFIED

A sufficiently reliable individual master record has been identified.

Return its record reference and internal product code.

NEEDS_VERIFICATION

Multiple candidates remain plausible or the evidence is insufficient.

Do not force a potentially incorrect ID.

NOT_IDENTIFIED

No reliable matching record is available.

The application may later request another image or another product-box surface.

⸻

15. Performance Requirements

The matching engine must target less than one second after OCR has completed.

The target includes:

* Text normalization.
* Query embedding generation.
* Phrase embedding generation.
* Vector retrieval.
* BM25 retrieval.
* Character retrieval.
* Terminology retrieval.
* Attribute retrieval.
* Candidate fusion.
* LightGBM reranking.
* Confidence evaluation.

Optimize for an NVIDIA GPU.

Recommended techniques:

1. Load embedding models once.
2. Precompute all catalog embeddings.
3. Use FP16 where accuracy permits.
4. Batch phrase embedding generation.
5. Keep vector indexes in GPU memory when possible.
6. Run CPU lexical retrieval concurrently.
7. Avoid full-catalog scans.
8. Rerank only a small candidate pool.
9. Warm up the GPU before serving queries.
10. Avoid unnecessary CPU-GPU transfers.
11. Cache repeated queries where appropriate.
12. Avoid rebuilding indexes during runtime.

Benchmark actual latency rather than claiming the target is achieved.

Report:

* p50 latency.
* p95 latency.
* p99 latency.
* GPU memory.
* CPU memory.
* Cold-start latency.
* Warmed-up latency.

The complete image-to-result latency is a separate measurement because OCR time is excluded from the one-second matching target.

⸻

16. Offline Index Builder

The index builder must be separate from the runtime SDK.

Responsibilities:

1. Read the authorized master dataset.
2. Validate the source schema.
3. Preserve original descriptions.
4. Normalize English, French and Spanish text.
5. Generate sentence embeddings.
6. Generate selected phrase embeddings.
7. Build FAISS indexes.
8. Build BM25 indexes.
9. Build character n-gram indexes.
10. Build terminology indexes.
11. Extract structured attributes.
12. Build product hierarchy metadata.
13. Save versioned index files.
14. Generate a catalog manifest.
15. Validate index integrity.

Support:

* Complete index rebuilds.
* Authorized incremental updates.
* Index versioning.
* Compatibility checks.
* Integrity verification.
* Rollback to an earlier approved index.

The original master dataset must remain unchanged.

⸻

17. Python SDK Architecture

Create a modular Python package.

Suggested structure:

product_identifier/
│
├── init.py
├── client.py
├── config.py
│
├── vision/
│   ├── detector.py
│   ├── recognizer.py
│   ├── preprocessing.py
│   └── text_regions.py
│
├── text/
│   ├── normalization.py
│   ├── language.py
│   ├── tokenization.py
│   ├── terminology.py
│   └── attributes.py
│
├── embeddings/
│   ├── model_loader.py
│   ├── sentence_encoder.py
│   └── phrase_encoder.py
│
├── retrieval/
│   ├── vector_search.py
│   ├── bm25.py
│   ├── character_search.py
│   ├── fuzzy_search.py
│   ├── terminology_search.py
│   ├── attribute_search.py
│   └── fusion.py
│
├── ranking/
│   ├── features.py
│   ├── reranker.py
│   └── calibration.py
│
├── catalog/
│   ├── loader.py
│   ├── index_builder.py
│   ├── index_loader.py
│   ├── metadata.py
│   └── relationships.py
│
└── evaluation/
├── metrics.py
└── benchmark.py

The SDK must expose simple public APIs.

Example:

from product_identifier import ProductIdentifier

identifier = ProductIdentifier(
index_path=”./product_index”
)

result = identifier.identify(
image_path=”./product_box.jpg”
)

The SDK should also support receiving OCR text directly:

result = identifier.identify_text(
text=[
“Wireless temperature monitoring system”,
“Bluetooth connectivity”,
“12V operating voltage”
]
)

Return structured results containing:

* Status.
* Master record reference.
* Internal product code, only after identification.
* Confidence.
* Matched language.
* Evidence summary.
* Distinctive matched features.
* Specification agreements.
* Contradictions.
* Processing latency.

The public SDK must not expose unnecessary internal dataset information.

⸻

18. Model and Catalog Packaging

The model files and catalog indexes will be downloaded or installed separately from the application.

Do not assume that a .pt file alone contains everything required for inference.

Model packaging should include:

* Model weights.
* Compatible architecture or runtime.
* Preprocessing configuration.
* Class names for detection models.
* Required dependencies.
* Model version.
* Integrity hashes.

The catalog package should contain:

* Sentence embedding indexes.
* Phrase embedding indexes.
* FAISS index files.
* BM25 indexes.
* Character indexes.
* Terminology dictionary.
* Attribute metadata.
* Hierarchy metadata.
* Catalog manifest.

Keep the catalog and model packages versioned and separate.

The SDK should validate compatibility before loading them.

Do not embed the confidential master dataset in a publicly distributed application package.

⸻

19. Evaluation and Benchmarking

Build the benchmark system before claiming 95% accuracy.

Use real product-box photographs with manually verified master record IDs.

Include:

* English packaging.
* French packaging.
* Spanish packaging.
* Cross-language matches.
* OCR errors.
* Similar product descriptions.
* Different product variants.
* Parent-child project examples.
* Low-quality photographs.
* Incomplete printed information.

Metrics:

Metric	Purpose
OCR CER/WER	Recognition quality
Recall@10	Candidate retrieval quality
Recall@50	Candidate retrieval quality
Recall@100	Candidate retrieval quality
Top-1 accuracy	First-ranked identification
Accepted accuracy	Accuracy among automatically accepted results
Coverage	Percentage automatically identified
False acceptance rate	Incorrect confident identifications
MRR	Candidate ranking quality
Per-language accuracy	English/French/Spanish performance
Cross-language accuracy	Language mismatch performance
p95 latency	Runtime performance

Avoid data leakage.

Split evaluation data by physical product or product family rather than randomly splitting near-identical images.

Use separate training, validation and test sets.

The 95% target must be reported alongside coverage and false acceptance rate.

⸻

20. Mandatory Cautions

These cautions must be followed throughout development.

1. Never use the internal product code as OCR search evidence because it is not printed on the box.
2. Never modify the confidential master dataset.
3. Never send product data or images to cloud services.
4. Never assume YOLO performs OCR recognition.
5. Never assume all three language descriptions are complete.
6. Never assume different product IDs have sufficiently different descriptions.
7. Never rely only on sentence cosine similarity.
8. Never rely only on BM25 or keyword search.
9. Never treat character n-grams as a complete multilingual translation solution.
10. Never use embeddings as a replacement for exact technical specification matching.
11. Never confuse RRF scores with probabilities.
12. Never treat missing OCR evidence as a negative match.
13. Never force an identification when candidates are indistinguishable.
14. Never generate all catalog embeddings during a scan.
15. Never compare every query against the entire catalog using an unindexed full scan.
16. Never load and unload GPU models for every identification.
17. Never claim sub-second performance without measuring it.
18. Never claim 95% accuracy without a properly constructed independent test set.
19. Never assume the parent-child hierarchy without inspecting the master data.
20. Never build the Flutter application as part of this task.

⸻

21. Implementation Methodology

Do not immediately generate the entire project in one large code response.

Work in controlled phases.

Phase 1: Requirements and dataset audit

Inspect the actual dataset schema and available YOLO/OCR model files.

Identify missing fields, language coverage, duplicates and hierarchy information.

Produce the final internal data schema.

Phase 2: OCR pipeline

Implement and evaluate YOLO text detection and offline OCR.

Produce structured OCR output.

Phase 3: Index builder

Implement multilingual preprocessing, sentence embeddings, selected phrase embeddings, BM25, character n-grams, terminology indexes and structured attributes.

Phase 4: Retrieval engine

Implement parallel candidate retrieval and RRF.

Measure Recall@K.

Phase 5: Reranking

Create candidate features, train LightGBM and evaluate difficult near-duplicate examples.

Phase 6: Confidence engine

Calibrate confidence and implement identified, needs-verification and not-identified outcomes.

Phase 7: Performance optimization

Optimize GPU retrieval, inference, memory and concurrency.

Verify the sub-second matching target on actual hardware.

Phase 8: Python SDK packaging

Build a reusable, installable Python SDK with documentation, examples and tests.

Phase 9: Final validation

Run multilingual, accuracy, latency, security and offline tests.

⸻

22. How I Want You to Work as the Technical Architect

Act as a senior AI/ML architect and Python systems engineer.

Your responsibilities:

* Explain important design decisions.
* Identify architectural weaknesses before implementation.
* Challenge unnecessary complexity.
* Avoid blindly accepting my assumptions.
* Recommend practical alternatives when justified.
* Use modular, production-quality Python.
* Include error handling, logging and type hints.
* Avoid placeholder implementations being presented as completed features.
* Clearly distinguish implemented, simulated and pending functionality.
* Test each phase before moving to the next.
* Provide reproducible installation and execution instructions.

Do not start by writing thousands of lines of code.

First, review the architecture, identify unresolved technical questions and establish the implementation plan.

If a required detail is unknown, ask a focused clarification rather than inventing an answer.

FINAL OBJECTIVE

Develop a robust, fully offline, GPU-accelerated multilingual product identification SDK that can search a confidential catalog of 60K–600K records, tolerate OCR errors, distinguish products with highly similar descriptions, preserve parent-child product relationships and return the correct internal product code after reliable identification.

The target is at least 95% validated identification accuracy and sub-second matching latency after OCR, subject to real-world benchmarking.

The Flutter application is a separate project and must not be developed in this task.
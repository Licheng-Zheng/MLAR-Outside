## Agentic Researcher Pipeline
Meant to be a complete project where the user provides a prompt, and the program writes up an in-depth report. Uses confidence scoring based on metrics such as  number of corroborating (eheheh big word) sources and source quality. 

#### Rationale 
Research models and typical AI models online are very compute expensive (and money expensive!) to run and do not provide a very broad view of topics. This is to say, in order to do research on a domain, you need to already know what you're looking for, which is not always the case for me (the highest biology education I have is grade 10 science). This project provides something that provides both depth and breadth. 

Research models such as Gemini Research (which is the only model I have access to), filters through sources for every research request, leading to the same sources being read and reread over and over again. This project will lead to the creation of a database, similar to a FAISS database, preventing rereading of infromation for different queries. The items beign stored are condensed claims (after merging similar claims), so this is much more memory efficient than storing everything. 

Less true for research models which use sources, typical LLMs can hallucinate facts. This is not as important for applications where this project would be used (hopefully), but it is never fun to find out you've learned something wrong. This project only writes things that have been backed up by multiple sources (dependent on the confidence function being used). The distilled model that performs the writing is only provided information that comfortably fits its attention (a couple sentences), and only writes based on what it is provided. As long as the upstream information is accurate, the result will be accurate. 

#### sample text analysis
This pdf is the high quality output (the normal file is way too large, and it contains all the lower confidence claims that were found online which did not make it into the final report) of the writers. 
- It is the first draft (it needs ot go through lots of evaluation to ensure the information is good quality) 
- Currently, the pipeline can only run on biomedical text (or biomedical adjacent), because that's the only domain that I've created a custom NER model for

#### Cost Analysis 
(A very crude analysis, but I don't have the exact numbers, all in CAD) 
- Local compute costs not counted 
- All GPU use is on the Modal cloud provider, using what I calculated to be the most cost efficient GPU

Final Overall Cost: About 5.78 dollars. 
(I used the freely provided credits for all costs, costs may be in USD, but until I accidentally use too many tokens and it shows up on my bank statements, I'll never now) 

Cost to train a domain specific NER model (If there is a domain specific BERT model): About 3.30 dollars 
- Data acquisition - Web scraping (free)
- Data labelling/Synthetic Data generation: 2.75  dollars
  - Using OpenAI API GPT 5.4 Mini ($0.375 in / $2.25 out for batch)
  - Assuming input accounts for about 90% of the tokens used 
- Training on A10G GPUs: 0.55 
  - Using Modal A10G GPUs: 1.10 per hour per GPU 
  - Training for about half an hour (unsure about this, had to retrain multiple times to clean up the data) 
- Model runs well on CPU, so future inference costs 

Cost to run semantic similarity calculations: 1.83 dollars
- Includes Bi-Encoder Embedding, FAISS Indexing, Cross-encoder verification (Graph clustering performed on CPU) 
- Trainign on 10 A10G GPUs:
  - Using Modal A10G GPUs: 1.10  per hour per GPU 
  - 10 minutes for all computations: 1.83 dollars
- When new sources are found and added to the database, will also require GPUs

Cost to run multiple distilled Qwen2.5 7B writers to write to markdown: 
- After initial setup time (I will ignore this cost because it is subcent) 
  - Using Modal H100 GPUs: 3.95 per hour per GPU 
  - Runs for 1 minute: 0.65
- Large part of the operation takes place on local CPU, which is not counted   

#### Architecture
Phase 0: Data Ingestion & NER (Create_Claim.py)
- Scraping & Chunking: Ingests target URLs (e.g., Wikipedia, PubMed) and chunks the text into manageable sentences.
- Named Entity Recognition (NER): Deploys PubMedBERT-CRF (custom model which you can find out more about in the model information folder in this repo)  on Modal (T4 GPUs) to extract specific biological entities (Cell Types, Pathogens, Treatments, etc.).

**Output: claims.json**

Phase 1: Semantic Deduplication & Scoring (semantic_similarity.py)
- Bi-Encoder Embedding: Converts all text claims into 768-dimensional float32 vectors using S-PubMedBert-MS-MARCO.
- FAISS Indexing: Builds a high-speed HNSW index to quickly find nearest-neighbor candidate pairs.
- Cross-Encoder Verification: Uses nli-deberta-v3-small to verify textual entailment and remove false positives.
- Graph Clustering: Uses NetworkX and Louvain community detection to group identical claims, calculate Medoids, and assign a confidence score based on Domain Authority.

**Output: scored_claims.jsonl**

Phases 2 & 3: Architect & Draft Composer (drafting_pipeline.py)
- The Architect: Makes chunked, highly targeted API calls to GPT-4o-mini to semantically roll up topics and generate a textbook-style document outline.
- Token Guardrails: Slices the top-confidence facts (max 150-300 per section) to prevent context overflow.
- The Writers: Deploys Qwen2.5-7B-Instruct on Modal H100 GPUs using vLLM. Batches the drafted sections in parallel for blazingly fast generation.
- The Collator: Stitches the final Markdown document together and appends a "Requires Further Research" Appendix for low-confidence data.

**Output: Agentic_Report_First_Draft.md**

Installation & Setup
**Not yet finalized, so not included here** (also theres no code here anyways) 



#### Create_Claim.py 
**Created Files** 
- scraped_sentences_checkpoint.json: After scraping through all the provided sources and chunking has been performed, data is stored here. 
- output_flat.json: The final output in a jsonl format 
- output_grouped.json: The final output in a json format 
- **claims.json**: The claim data objects in their final form

#### Semantic_Similarity.py 
- pipeline_embeddings.npy: An array of all the vectors for each claim (each claim gets its own vector, computed based on its semantic meaning) 
- pipeline_hnsw.index: An index created by FAISS for searching for ismilar claims to group them togehter 
- pipeline_verified_edges.json: The final check if two claims are similar to one another (the FAISS method is not very accurate, but is very fast at getting rid of things that definitely aren't related) 
- **scored_claims.jsonl**: The final deduplicated claims with the dataclaim updated with argreement count and confidence scoring calculated based on a function

#### drafting_pipeline.py 
- packaged_claims_checkpoint.json: Figures out what headers are being returned by OpenAI API (Used for debugging) 
- **Report_First_Draft.md**: The final first draft after it has been written 


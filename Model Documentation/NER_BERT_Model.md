Model On Hugging Face: https://huggingface.co/Licheng-Zheng/NER-BIO-BERT

**Intended Use & Limitations:** For extracting entities from biomedical literature, should not be used for clinical purposes as there are many flaws in the training data. 

#### How to Use 
**PubMedBERT CRF Custom Class** (required for loading in the CRF layer on top of the BERT model)
```py

class PubMedBertCRF(nn.Module):
    def __init__(self, model_checkpoint, num_labels):
        super().__init__()
        self.bert = AutoModel.from_pretrained(model_checkpoint)
        self.config = self.bert.config

        self.dropout = nn.Dropout(0.1)
        self.classifier = nn.Linear(self.config.hidden_size, num_labels)
        self.crf = CRF(num_tags=len(label2id), batch_first=True)

    def forward(self, input_ids, attention_mask, labels=None, **kwargs):
        outputs = self.bert(input_ids=input_ids, attention_mask=attention_mask)
        sequence_output = self.dropout(outputs.last_hidden_state)
        emissions = self.classifier(sequence_output)

        if labels is not None:
            crf_mask = attention_mask.bool()
            crf_mask[:, 0] = True 
            
            safe_labels = torch.where(labels == -100, torch.tensor(0, device=labels.device), labels)
            log_likelihood = self.crf(emissions, safe_labels, mask=crf_mask, reduction='mean')
            loss = -log_likelihood
            return (loss, emissions)
        else:
            return emissions
```

**Test Script** 
```py
import torch
from transformers import AutoTokenizer
# Assuming PubMedBertCRF and label2id/id2label are defined in your script...

def predict_entities(text, model, tokenizer, device="cuda"):
    model.eval()
    model.to(device)
    
    # 1. Tokenize the input text
    inputs = tokenizer(text, return_tensors="pt", truncation=True, max_length=512)
    inputs = {k: v.to(device) for k, v in inputs.items()}
    
    with torch.no_grad():
        # 2. Get emissions from BERT + Linear layer
        emissions = model(input_ids=inputs["input_ids"], attention_mask=inputs["attention_mask"])
        
        # 3. Decode using the CRF
        mask = inputs["attention_mask"].bool()
        mask[:, 0] = True # Safety catch
        predictions = model.crf.decode(emissions, mask=mask)[0]
        
    # 4. Map back to words
    tokens = tokenizer.convert_ids_to_tokens(inputs["input_ids"][0])
    results = []
    
    for token, pred_id in zip(tokens, predictions):
        if token not in ["[CLS]", "[SEP]", "[PAD]"]:
            label = id2label[pred_id]
            if label != "O":
                results.append((token, label))
                
    return results

# --- Test it out ---
tokenizer = AutoTokenizer.from_pretrained("microsoft/BiomedNLP-PubMedBERT-base-uncased-abstract-fulltext")
model = PubMedBertCRF("microsoft/BiomedNLP-PubMedBERT-base-uncased-abstract-fulltext", num_labels=len(label2id))

# Load your trained weights
model.load_state_dict(torch.load("best_pubmedbert_crf_model.pt"))

test_sentence = "Patients with severe rheumatoid arthritis were treated with infliximab to target TNF-alpha."
entities = predict_entities(test_sentence, model, tokenizer)

for token, label in entities:
    print(f"{token:<15} : {label}")
```

#### Model Description 
1. Architecture: 
A NER (Named Entity Recognition) Model that is fine tuned for medical texts. 
File Size: 417.9 MB 

PubMedBERT (microsoft/BiomedNLP-PubMedBERT-base-uncased-abstract-fulltext) because it was trained on medical text. 
Custom CRF (Conditional Random Fields) (layer): Models structural dependencies between output labels during the labelling sequence. 
- This is done in order to consider transitions from one label to the next 
- Without CRF, everything is predicted independently, with CRF, the model is forced to look at the previous label(s) to label the current one 
Tokenizer (microsoft/BiomedNLP-PubMedBERT-base-uncased-abstract-fulltext)

2. Model Labels and Outputs 
These are the different labels that the model provides to each token. 
- O - Not an Entity 
- Possible Entities: Cell Function, Cell Type, Condition, Location, Molecule, Pathogen, Process Receptor, Treatment, Visual Property
- Possible Prefixes (BILOU): Beginning, Inside, Last, Outside (Not and Entity), Unique 
```
    "O": 0,
    "B-CELL_FUNCTION": 1, "I-CELL_FUNCTION": 2, "L-CELL_FUNCTION": 3, "U-CELL_FUNCTION": 4,
    "B-CELL_TYPE": 5, "I-CELL_TYPE": 6, "L-CELL_TYPE": 7, "U-CELL_TYPE": 8,
    "B-CONDITION": 9, "I-CONDITION": 10, "L-CONDITION": 11, "U-CONDITION": 12,
    "B-LOCATION": 13, "I-LOCATION": 14, "L-LOCATION": 15, "U-LOCATION": 16,
    "B-MOLECULE": 17, "I-MOLECULE": 18, "L-MOLECULE": 19, "U-MOLECULE": 20,
    "B-PATHOGEN": 21, "I-PATHOGEN": 22, "L-PATHOGEN": 23, "U-PATHOGEN": 24,
    "B-PROCESS": 25, "I-PROCESS": 26, "L-PROCESS": 27, "U-PROCESS": 28,
    "B-RECEPTOR": 29, "I-RECEPTOR": 30, "L-RECEPTOR": 31, "U-RECEPTOR": 32,
    "B-TREATMENT": 33, "I-TREATMENT": 34, "L-TREATMENT": 35, "U-TREATMENT": 36,
    "B-VISUAL_PROPERTY": 37, "I-VISUAL_PROPERTY": 38, "L-VISUAL_PROPERTY": 39, "U-VISUAL_PROPERTY": 40
```
#### Training Procedure 
1. Hardware Details:
    - GPU: 24GB A10G 
    - Precision: bf16 (Required as the CRF layer cannot work with fp16 of T4s) 
    - Batch Size: 110 (RAM can fit a maximum of 118, 110 for safety) 

2. Metrics for Loss Calculation: 
    - Precision and recall for each label, then weighted depending on the number of entries for each label. 
      - Precision: Out of all the positive predictions, how many were positive? (Found and Correct over Found) 
      - Recall: Out of all the positives in the dataset, how many were found? (Found and Correct over Total Positives) 
    - These values combine to produce F1 Loss (harmonic mean of precision and recall), which is the metric used for training
    
3. Training Tactics Used 
- Differential Learning Rates 
  - Also called discriminative learning rates. CRF layer trained much faster than BERT. (BERT is already somewhat pre-trained, we don't want the errors of CRF messing up the BERT model)
  - Typical momentum learning rate variation 
- Focal Loss 
  - Force the model to focus more on more difficult entries to hopefully learn more than using standard BCE (very slight improvement unfortunately) 

#### Dataset Details 
1. Amount of each entity: 
Entity Type        | Count
------------------------------------------------------------
CELL_FUNCTION      | 37
CELL_TYPE          | 1040
CONDITION          | 283
LOCATION           | 159
MOLECULE           | 949
PATHOGEN           | 124
PROCESS            | 257
RECEPTOR           | 230
TREATMENT          | 69
(There is one visual property entry, so I got rid of that category. However, there is a single datapoint that is labelled visual property, so I had to include the labels to prevent the model from breaking when it encountered it during training) 

- Source of Data: 30 thousand sentences scraped from biology and medical texts, deduplicated. 
- Labelling: Ground truth provided by **OpenAI API GPT5.4 mini 2026-03-17** (not labelled by humans) 
- Tokenizer from the PubMedBERT Model. 



#### Quantitative and Qualitative Evaluation
1. Quantitative Details 
**OVERALL F1**:  0.7488 (when compared to the ground truth) 
- Diagnostics across 1500 Sentence
Structural Errors (BILOU violations): 23 - Previously without the CRF layer, this was in the 400 - 500 range, so CRF drastically reduces it 
Semantic Errors (Type mismatches)   : 122 - An issue with the model or the ground truth (it guesses one category when it is another, not much I can do with these) 

- Statistics that I think are important are bolded and explained below (epoch 4 had the best F1 score): 
{'eval_loss': 3.8826029300689697, 'eval_precision': 0.7255661501787842, 'eval_recall': 0.7735069885641678, 'eval_f1': 0.748769987699877, **'eval_accuracy'**: 0.959349593495935, 'eval_structural_errors': 23, 'eval_semantic_errors': 122, 'eval_CELL_FUNCTION_f1': 0.7042253521126761, **'eval_CELL_TYPE_f1'**: 0.7440180586907448, 'eval_CONDITION_f1': 0.7910189982728844, 'eval_LOCATION_f1': 0.7781155015197568, 'eval_MOLECULE_f1': 0.7900466562986003, 'eval_PATHOGEN_f1': 0.7341772151898733, **'eval_PROCESS_f1'**: 0.5378486055776893, 'eval_RECEPTOR_f1': 0.7615230460921845, 'eval_TREATMENT_f1': 0.7692307692307693, 'eval_runtime': 4.7462, 'eval_samples_per_second': 316.042, 'eval_steps_per_second': 2.95, 'epoch': 4.0}
1. eval_accuracy: highlighting this one to show that it is not important - most of the tags are "O", so the model would probably get a higher accuracy if it just always guessed "not an entity", which is why this is not the metric we are optimizing for 
2. eval_celltype: Despite having the most data points, it doesn't perform the best, may be due to the low quality of data? Not very confusable with the other data labels, unsure why this one is as low as it is 
3. eval_process: very low accuracy, this is likely because it is easy to confuse this with the other ones, for example, cell function. There is probably a large issue with the teacher model for this as well, so we can't improve this much without getting much better data (is it time to label it myself?). Despite there being a large amount of data for process (relatively), it still performs far worse than cell function. Will need to figure out why this is the case in the future. 

2. Qualitative Details 
Even though the model does not have a very high F1 score, it is qualitatively very good, and the low F1 score is (in my very humble opinion) due to the teacher model not providing strict enough labels. 

Here are 4 random samples that the model produced in which it was marked as incorrect: 

**Sample 1:**
Seems like there are a lot more entities in this text than either model detected. 
- Text: moreover, studies have also found that mir - 34a secreted by adipocytes can inhibit m2 polarization by inhibiting the expression of kruppel - like factor 4 ( klf4 ), thus promoting obesity - induced inflammation ( ( https : / / www. frontiersin. org / journals / immunology / articles / 10. 3389 / fimmu. 2024. 1352946 / full # b175 ) ).
  - FALSE POSITIVES (Model Hallucinated or Got Boundaries Wrong): [CONDITION] -> 'obesity - induced inflammation'

**Sample 2: Inconsistent teacher model**
There is quite literally not a better example of an inconsistent teacher model. There are also many many other entities that both models fail to recognize (possible due to the model being penalized in the past for detecting similar items) -> a much more stringent labelling procedure needs to be followed for future dataset creation. 
- Text: stewart i, radtke d, phillips b, mcgowan sj, bannard o. germinal center b cells replace their antigen receptors in dark zones and fail light zone entry when immunoglobulin gene mutations are damaging.
  - FALSE NEGATIVES (Model Missed or Got Boundaries Wrong): [LOCATION] -> 'light zone'
  - FALSE POSITIVES (Model Hallucinated or Got Boundaries Wrong): [LOCATION] -> 'dark zones'

**Sample 3: A less context dependent approach with a context dependent ground truth**
The teacher model likely saw the link and realized that this sentence wasn't focused on the actual medical value of the sentence, and thus didn't assign any entities to the text (the Chat GPT model is likely much more focused on the entire text rather than evaluating it little chunk by chunk). The BERT model focuses on each token independently (and with the CRF layer, it also considers the labels assigned to the previous tokens), however, it is much closer to evaluating each token independently compared to the teacher model, leading to the false positive (penalty for precision) 
- Text: ( https : / / www. dovepress. com / mechanistic - research - on - the - crosstalk - between - macrophage - polarization - - peer - reviewed - fulltext - article - jhc # cit0008 ) consequently, additional research is needed to develop effective therapeutic strategies to improve outcomes for individuals with hepatic malignancies.
  - FALSE POSITIVES (Model Hallucinated or Got Boundaries Wrong): [CONDITION] -> 'hepatic malignancies'

**Sample 4: Harsh Penalization for an Overacheiving Student** 
This entry resulted in two errors for the model (it failed on both recall and precision), because it came up with a response that was better than that of its teachers. The seqeval requires exact output to be identical to the ground truth, so small deviations led this to be marked incorrect. (it is incorrect because the metric only accepts one less correct answer)
- Text: with regard to the treatment and prevention of diabetes mellitus and its complications, the method of causing tissue macrophages to repolarize from a pro - inflammatory m1 phenotype to an anti - inflammatory healing m2 phenotype offers substantial potential.
- Results
  - FALSE NEGATIVES (Model Missed or Got Boundaries Wrong): [CELL_TYPE] -> 'macrophages'
  - FALSE POSITIVES (Model Hallucinated or Got Boundaries Wrong): [CELL_TYPE] -> 'tissue macrophages'

Through these samples, it is clear that even though the distilled model is likely not performing at its best, the dataset is contributing a large part ot eh low F1 scores. The model is more sophisticated than the teacher model in many cases (as seen in the samples), and further training to cram the model into the teacher model's image would reduce the model's use in the real world. (it will be interesting to see how much further F1 can be reduced, or if the BERT model refuses to learn from the teacher model in some cases) 
- The F1 score does not tell a complete picture and is not a very accurate picture of how well the model performs. It is important to note the F1 score is not based on what is true, but what the GPT model (which is no where near the frontier models) says is true, which are very different things. 

#### Flaws of the Model 
1. Trained on faulty data with many inconsistencies 
2. Is only trained on a very particular set of labels (will require more fine-tuning for different labels) 
3. Along with low quality data, it is also lacking in data for many of the labels 

#### Future Possible Improvements to the Model 
- Probably many fixes can be implemented, but this is by far the largest one: get better data. 
  - Use better models with more stringent prompts 
  - Get domain specific teachers to label the data to distil their more extensive knowledge into a dataset 
    - There are not very many (free and open source) models that do what is required 
  - Human labelled data to be distilled into a not domain specific (but frontier model) to perform mass labelling. 

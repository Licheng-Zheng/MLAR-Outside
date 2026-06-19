**Dataset Information**:
**Epochs (Not Consistent)**: 
**Epoch Performance**:
**Final Performance** 
**Evaluation**: 

#### Metric Information 
**Dataset Information**:
**Epochs (Not Consistent)**: 
**Epoch Performance**:
**Final Performance** 
**Evaluation**: 
^ That is the structure that we want to use for this stuff

**Accuracy**  Just correct over incorrect. In normal next, most tokens are just labelled "0" (not important words), so the accuracy will be really high if they always just respond with "0". This metric is not a very good metric to use
-> More important Metrics
**Precision** Total correct guessed over total guessed. This is a much better metric than accuracy (it tells us more) 
-  It doesn't make many false alarms (False Positives).
- Low Precision means the model is randomly guessing
**Recall**  Out of all the labelled ones, how many did it find. 
- High Recall means the model is thorough. It rarely misses an entity that should have been labeled (False Negatives).
- Low Recall means the model is leaving many terms completely unlabelled as O.
-> Most Important Metric 
**F1 Score** The Balanced Score: The harmonic mean of Precision and Recall.
- If your Precision is 99% but your Recall is 10%, your F1-score will be a terrible ~18%.
- High F1-score requires both high Precision and high Recall.
- You want to increase F1 as faras you can (this is what you need to show that your model is not cooked) 

#### Training Run 1
(Not as extensive documentation as with the other ones, I'll just put an empty header here for the ones not present) 
**Dataset Information** 333 Elements 
**Epochs (Not Consistent)**: 5 (still learning, has not plateaued yet))
**Epoch Performance**: Not provided for this run, didn't track it 
**Final Performance** 
Training Loss: 0.567 
Validation Loss: 0.339 
Precision: 0.432
Recall: 0.429 
F1: 0.430 
Accuracy: 0.92 (Always very high because its typically just "O"s

**Evaluation**: This run is just to try everything out, and see if I can get everything working (it works :D) Very bad scores so hopefully the reason it does not work is the limited dataset. 

#### Training Run 2
**Dataset Information** 10k Elements 
**Epochs (Not Consistent)**: 5
**Epoch Performance**: Not provided for this run, didn't track it 
**Final Performance** 

Entity Type | Precision | Recall | F1-Score | Count
CELL_FUNCTION | 0.6667 | 0.2857 | 0.4000 | 7
CELL_TYPE | 0.7029 | 0.7500 | 0.7257 | 388
CONDITION | 0.7901 | 0.6957 | 0.7399 | 92
LOCATION | 0.8000 | 0.9167 | 0.8544 | 48
MOLECULE | 0.7312 | 0.7969 | 0.7626 | 256
PATHOGEN | 0.7917 | 0.7037 | 0.7451 | 27
PROCESS | 0.4194 | 0.5532 | 0.4771 | 47
RECEPTOR | 0.6875 | 0.8049 | 0.7416 | 82
TREATMENT | 0.5294 | 0.4286 | 0.4737 | 21
VISUAL_PROPERTY | 0.0000 | 0.0000 | 0.0000 | 1
OVERALL | 0.7032 | 0.7482 | 0.7250 |
- Much better performance with the larger dataset 

**Evaluation** : Not performing very well with the visual property, because there is very very limited visual property data points (I'm going to leave it in so that the model training is consistent across runs) 
- Very good at location, molecule reciptor and condition. The process is very hard to train because it sounds similar to the other entities, so it gets confused easily. (maybe get something other than cross-entropy so that it doesn't get completely penalized when it guesses something similar) 
  - Increase the loss by changing how loss is evaluated :D how could this go wrong


#### Training Run 3
**Dataset Information** 20k Elements 
**Epochs (Not Consistent)**: 20
**Epoch Performance**:
OVERALL | 0.6603 | 0.7279 | 0.6924 |
OVERALL | 0.6351 | 0.7567 | 0.6906 |
**Loss absolutely plateaus :( not sure why** 
OVERALL | 0.6779 | 0.7577 | 0.7156 |
OVERALL | 0.6535 | 0.8014 | 0.7200 |
OVERALL | 0.6271 | 0.8081 | 0.7062 |
OVERALL | 0.6819 | 0.7346 | 0.7073 |
OVERALL | 0.6654 | 0.7917 | 0.7230 |
OVERALL | 0.6688 | 0.7634 | 0.7129 |
OVERALL | 0.6750 | 0.7798 | 0.7236 |
OVERALL | 0.6831 | 0.7485 | 0.7143 |
OVERALL | 0.6646 | 0.7634 | 0.7106 |
OVERALL | 0.6680 | 0.7845 | 0.7216 |
OVERALL | 0.7023 | 0.7464 | 0.7237 |
OVERALL | 0.6869 | 0.7608 | 0.7220 |
OVERALL | 0.6935 | 0.7567 | 0.7237 |
OVERALL | 0.7009 | 0.7510 | 0.7251 |
OVERALL | 0.6895 | 0.7665 | 0.7259 |
OVERALL | 0.6796 | 0.7737 | 0.7236 |

**Final Performance** 
Entity Type | Precision | Recall | F1-Score | Count
CELL_FUNCTION | 0.6364 | 0.7000 | 0.6667 | 10
CELL_TYPE | 0.6801 | 0.7623 | 0.7188 | 753
CONDITION | 0.7214 | 0.6733 | 0.6966 | 150
LOCATION | 0.6786 | 0.8482 | 0.7540 | 112
MOLECULE | 0.7213 | 0.8039 | 0.7603 | 515
PATHOGEN | 0.8222 | 0.7255 | 0.7708 | 102
PROCESS | 0.3511 | 0.4182 | 0.3817 | 110
RECEPTOR | 0.6902 | 0.8089 | 0.7449 | 157
TREATMENT | 0.6471 | 0.6471 | 0.6471 | 34
VISUAL_PROPERTY | 0.0000 | 0.0000 | 0.0000 | 1
OVERALL | 0.6794 | 0.7510 | 0.7134 |
- Very similar performance as the other model 

**Evaluation**: 
- The preicision and recall bounce around, precision lags behind typically
- Plateaus very early on -> maybe this is the best and furthest I can get with my current architecture (hopefully not because that's all the thinking I can do) 

#### Training Run 4
**Dataset Information** 20k Elements 
**Epochs (Not Consistent)**: 9
Training is slightly different this time, using learning rate schedules to hopefully get into the optima and settle in and not jump out. 
**Epoch Performance**:
Entity Type        | Precision | Recall    | F1-Score  | Count
OVERALL            | 0.4608    | 0.3807    | 0.4169    |
OVERALL            | 0.5789    | 0.6816    | 0.6260    |
OVERALL            | 0.6400    | 0.7654    | 0.6971    |
OVERALL            | 0.6284    | 0.7855    | 0.6982    |
OVERALL            | 0.6364    | 0.8086    | 0.7123    |
OVERALL            | 0.6745    | 0.7269    | 0.6997    |
OVERALL            | 0.6220    | 0.8194    | 0.7072    |
OVERALL            | 0.6634    | 0.7572    | 0.7072    |
OVERALL            | 0.6727    | 0.7665    | 0.7165    |

**Final Performance** 
Entity Type        | Precision | Recall    | F1-Score  | Count
CELL_FUNCTION      | 0.7778    | 0.7000    | 0.7368    | 10
CELL_TYPE          | 0.6701    | 0.7716    | 0.7173    | 753
CONDITION          | 0.7569    | 0.7267    | 0.7415    | 150
LOCATION           | 0.6642    | 0.8125    | 0.7309    | 112
MOLECULE           | 0.7277    | 0.8252    | 0.7734    | 515
PATHOGEN           | 0.8058    | 0.8137    | 0.8098    | 102
PROCESS            | 0.3397    | 0.4818    | 0.3985    | 110
RECEPTOR           | 0.6685    | 0.7707    | 0.7160    | 157
TREATMENT          | 0.5882    | 0.5882    | 0.5882    | 34
VISUAL_PROPERTY    | 0.0000    | 0.0000    | 0.0000    | 1
OVERALL            | 0.6727    | 0.7665    | 0.7165    |

**Evaluation**: 
- It trains a lot slower, but it also plateaus very on: this quick fix is not gonna work :( 
- Here is the other stuff that I can try to improve and hopefully push it up a bit more 
  - Data Audit to make sure the model is working 
  - Focal Loss (Algorithmic Fix): Using a different loss function 
  - CRF Layer 
    - Prevents the model from making structural mistakes so it only needs to figure out the class, and not the BILOU classifications. 
    
    
#### Training Run 5
**Dataset Information** 30k Elements 
**Epochs (Not Consistent)**: 11
Using Focal Loss with a gamma of 2 instead of Binary Cross Entropy Loss 
**Epoch Performance**:
OVERALL            | 0.5014    | 0.5502    | 0.5247    |
OVERALL            | 0.6031    | 0.7433    | 0.6659    |
OVERALL            | 0.6212    | 0.7605    | 0.6838    |
OVERALL            | 0.6288    | 0.7776    | 0.6954    |
OVERALL            | 0.6098    | 0.7964    | 0.6907    |
OVERALL            | 0.6550    | 0.7592    | 0.7033    |
OVERALL            | 0.5902    | 0.7557    | 0.6628    |
OVERALL            | 0.6211    | 0.7786    | 0.6910    |
OVERALL            | 0.6786    | 0.6820    | 0.6803    |
OVERALL            | 0.6518    | 0.7481    | 0.6966    |
OVERALL            | 0.6321    | 0.7579    | 0.6893    |
OVERALL            | 0.6633    | 0.7665    | 0.7112    |
OVERALL            | 0.6781    | 0.7208    | 0.6988    |

**Final Performance** 
Entity Type        | Precision | Recall    | F1-Score  | Count
CELL_FUNCTION      | 0.5909    | 0.7027    | 0.6420    | 37
CELL_TYPE          | 0.6523    | 0.7288    | 0.6885    | 1040
CONDITION          | 0.7645    | 0.7456    | 0.7549    | 283
LOCATION           | 0.7576    | 0.7862    | 0.7716    | 159
MOLECULE           | 0.7670    | 0.7355    | 0.7509    | 949
PATHOGEN           | 0.7295    | 0.7177    | 0.7236    | 124
PROCESS            | 0.4257    | 0.5019    | 0.4607    | 257
RECEPTOR           | 0.6285    | 0.7870    | 0.6988    | 230
TREATMENT          | 0.6842    | 0.7536    | 0.7172    | 69
OVERALL            | 0.6781    | 0.7208    | 0.6988    |

**Evaluation**: 
- Very similar performance to the other training runs. Got stuck at about the same place -> does using focal loss for evaluation mean its a better model if their performances are similar? 
- Plateaus very early on as well 
- Loss was not the issue, might need ot try using the CRF layer - are there any diagnostics that I can use to see that it will be effective before actually trying it (seeing what kinds of predictions its making -> is it making structural errors? or is it putting it into the wrong buckets (which would make it not a structural error, and would require a different fix) 

# Training Run 6 + Diagnostics 
I have created a new diagnostic suite, and it is used in order to see what kinds of errors are being made by the model. 
- Are they semantic errors? In this case, it is the model that is messing up, and the only way to improve is to learn better. (whether through higher quality data, or through allowing more neurons to learn the more difficult things, by using a secondary model to find the nouns) 
- Are they structural errors? In this case, the model is putting things into the correct bucket, HOWEVER, it is labelling the BILOU incorrectly, making it wrong. To reduce this, we can include a CRF structural head, penalizing correct categorization, but messing up on a task much simpler (In this case, it does not know if it was incorrect semantically, and leads to a large increase in noise, clouding its learning) 

**Dataset Information**: 30k entries
**Epochs (Not Consistent)**: 6
**Epoch Performance**:
============================================================
Entity Type        | Precision | Recall    | F1-Score  | Count
------------------------------------------------------------
CELL_FUNCTION      | 0.0000    | 0.0000    | 0.0000    | 37
CELL_TYPE          | 0.5411    | 0.8288    | 0.6548    | 1040
CONDITION          | 0.6452    | 0.7774    | 0.7051    | 283
LOCATION           | 0.5410    | 0.6226    | 0.5789    | 159
MOLECULE           | 0.6019    | 0.7903    | 0.6834    | 949
PATHOGEN           | 0.7667    | 0.5565    | 0.6449    | 124
PROCESS            | 0.1561    | 0.1245    | 0.1385    | 257
RECEPTOR           | 0.4742    | 0.6783    | 0.5581    | 230
TREATMENT          | 0.0000    | 0.0000    | 0.0000    | 69
------------------------------------------------------------
OVERALL            | 0.5488    | 0.6950    | 0.6133    |
Structural Errors (BILOU violations): 833 (affected 500 sentences)
Semantic Errors (Type mismatches)   : 197 (affected 149 sentences)

OVERALL            | 0.5969    | 0.7300    | 0.6568    |
NER DIAGNOSTICS (Across 1500 Sentences)
Structural Errors (BILOU violations): 751 (affected 464 sentences)
Semantic Errors (Type mismatches)   : 135 (affected 117 sentences)

OVERALL            | 0.6118    | 0.7294    | 0.6654    |
NER DIAGNOSTICS (Across 1500 Sentences)
Structural Errors (BILOU violations): 818 (affected 470 sentences)
Semantic Errors (Type mismatches)   : 113 (affected 102 sentences)

============================================================
Entity Type        | Precision | Recall    | F1-Score  | Count
------------------------------------------------------------
CELL_FUNCTION      | 0.5778    | 0.7027    | 0.6341    | 37
CELL_TYPE          | 0.6016    | 0.8000    | 0.6868    | 1040
CONDITION          | 0.7134    | 0.8445    | 0.7735    | 283
LOCATION           | 0.7167    | 0.8113    | 0.7611    | 159
MOLECULE           | 0.6803    | 0.8072    | 0.7383    | 949
PATHOGEN           | 0.6074    | 0.7984    | 0.6899    | 124
PROCESS            | 0.2903    | 0.1751    | 0.2184    | 257
RECEPTOR           | 0.6087    | 0.8522    | 0.7101    | 230
TREATMENT          | 0.6944    | 0.7246    | 0.7092    | 69
------------------------------------------------------------
OVERALL            | 0.6300    | 0.7567    | 0.6875    |
------------------------------------------------------------
NER DIAGNOSTICS (Across 1500 Sentences)
Structural Errors (BILOU violations): 543 (affected 335 sentences)
Semantic Errors (Type mismatches)   : 120 (affected 99 sentences)

OVERALL            | 0.5998    | 0.7916    | 0.6825    |
NER DIAGNOSTICS (Across 1500 Sentences)
Structural Errors (BILOU violations): 591 (affected 406 sentences)
Semantic Errors (Type mismatches)   : 125 (affected 116 sentences)

**Final Performance** 
============================================================
Entity Type        | Precision | Recall    | F1-Score  | Count
------------------------------------------------------------
CELL_FUNCTION      | 0.5000    | 0.7838    | 0.6105    | 37
CELL_TYPE          | 0.6487    | 0.7529    | 0.6969    | 1040
CONDITION          | 0.7382    | 0.7173    | 0.7276    | 283
LOCATION           | 0.6116    | 0.8616    | 0.7154    | 159
MOLECULE           | 0.7123    | 0.8272    | 0.7655    | 949
PATHOGEN           | 0.6644    | 0.7984    | 0.7253    | 124
PROCESS            | 0.4108    | 0.4747    | 0.4404    | 257
RECEPTOR           | 0.6718    | 0.7565    | 0.7117    | 230
TREATMENT          | 0.6092    | 0.7681    | 0.6795    | 69
------------------------------------------------------------
OVERALL            | 0.6520    | 0.7576    | 0.7009    |
------------------------------------------------------------
NER DIAGNOSTICS (Across 1500 Sentences)
Structural Errors (BILOU violations): 418 (affected 290 sentences)
Semantic Errors (Type mismatches)   : 118 (affected 105 sentences)

**Evaluation**: 
- After performing the diagnostics script on our model, we can see that it is mostly structural errors that are holding us back. 
  - Pretty consistenty after it plateaus, there is a 36% overall error out of the 1500 samples. 
  - About 78% of the errors are structural errors (this means that about 0.23 of the F1 score is lost due to structural errors: this means we should go with the CRF head to reduce these errors. 
    - Best case scenario (we get every single structural error), we will get an F1 score of about 0.93, even if we acheive half of this improvemnet, it is already a very good model. 
    - We can use a noun detection script as well to reduce the learning capcity wasted on finding "0"s 

# Training Run 7
**Dataset Information**:30k 
**Epochs (Not Consistent)**:5 
**Epoch Performance**:
Trying out a CRF layer to see if it would lead to large increases in the F1 score. (I think it's pretty cooked) 
| Epoch | Training Loss | Validation Loss | Precision |  Recall  |   F1     | Accuracy |
|-------|---------------|-----------------|-----------|----------|----------|----------|
| 1     | 13.118667     | 6.230810        | 0.636196  | 0.767154 | 0.695565 | 0.956815 |
| 2     | 12.295437     | 6.236152        | 0.662551  | 0.767154 | 0.711026 | 0.957826 |
| 3     | 11.346749     | 8.436106        | 0.654683  | 0.717281 | 0.684554 | 0.957918 |
| 4     | 10.665707     | 7.310059        | 0.677864  | 0.750000 | 0.712110 | 0.958811 |
| 5     | 7.191811      | 6.493928        | 0.672090  | 0.774142 | 0.719516 | 0.958325 |
**Final Performance** 
**Evaluation**: 



OVERALL F1         | 0.7162   
NER DIAGNOSTICS (Across 1500 Sentences)
Structural Errors (BILOU violations): 90
Semantic Errors (Type mismatches)   : 116
============================================================

# Training Run 8 
{'eval_loss': 4.7263665199279785, 'eval_precision': 0.7180715197956578, 'eval_recall': 0.7144218551461246, 'eval_f1': 0.7162420382165605, 'eval_accuracy': 0.9578391583592735, 'eval_structural_errors': 90, 'eval_semantic_errors': 116, 'eval_CELL_FUNCTION_f1': 0.65625, 'eval_CELL_TYPE_f1': 0.7235023041474654, 'eval_CONDITION_f1': 0.8015122873345937, 'eval_LOCATION_f1': 0.7407407407407407, 'eval_MOLECULE_f1': 0.7748414376321352, 'eval_PATHOGEN_f1': 0.6457399103139013, 'eval_PROCESS_f1': 0.4229885057471265, 'eval_RECEPTOR_f1': 0.6563106796116503, 'eval_TREATMENT_f1': 0.703125, 'eval_runtime': 9.4874, 'eval_samples_per_second': 158.105, 'eval_steps_per_second': 19.816, 'epoch': 1.0}

OVERALL F1         | 0.7269   
NER DIAGNOSTICS (Across 1500 Sentences)
Structural Errors (BILOU violations): 2
Semantic Errors (Type mismatches)   : 140

OVERALL F1         | 0.7216   
NER DIAGNOSTICS (Across 1500 Sentences)
Structural Errors (BILOU violations): 3
Semantic Errors (Type mismatches)   : 104

OVERALL F1         | 0.7426   
NER DIAGNOSTICS (Across 1500 Sentences)
Structural Errors (BILOU violations): 7
 Semantic Errors (Type mismatches)   : 134

OVERALL F1         | 0.7377   
NER DIAGNOSTICS (Across 1500 Sentences)
Structural Errors (BILOU violations): 4
Semantic Errors (Type mismatches)   : 151
{'eval_loss': 3.625276803970337, 'eval_precision': 0.6896932854379663, 'eval_recall': 0.7928843710292249, 'eval_f1': 0.737697650362051, 'eval_accuracy': 0.9554093279220353, 'eval_structural_errors': 4, 'eval_semantic_errors': 151, 'eval_CELL_FUNCTION_f1': 0.6410256410256411, 'eval_CELL_TYPE_f1': 0.7339688041594454, 'eval_CONDITION_f1': 0.7651006711409395, 'eval_LOCATION_f1': 0.7705382436260623, 'eval_MOLECULE_f1': 0.797735460627895, 'eval_PATHOGEN_f1': 0.7200000000000001, 'eval_PROCESS_f1': 0.5284280936454849, 'eval_RECEPTOR_f1': 0.7429718875502008, 'eval_TREATMENT_f1': 0.7272727272727273, 'eval_runtime': 9.392, 'eval_samples_per_second': 159.71, 'eval_steps_per_second': 20.017, 'epoch': 5.0}

OVERALL F1         | 0.7358   
NER DIAGNOSTICS (Across 1500 Sentences)
Structural Errors (BILOU violations): 0
Semantic Errors (Type mismatches)   : 118

OVERALL F1         | 0.7379   
NER DIAGNOSTICS (Across 1500 Sentences)
Structural Errors (BILOU violations): 0
Semantic Errors (Type mismatches)   : 106

OVERALL F1         | 0.7374   
NER DIAGNOSTICS (Across 1500 Sentences)
Structural Errors (BILOU violations): 1
Semantic Errors (Type mismatches)   : 121

OVERALL F1         | 0.7460   
NER DIAGNOSTICS (Across 1500 Sentences)
Structural Errors (BILOU violations): 0
Semantic Errors (Type mismatches)   : 124

OVERALL F1         | 0.7411   
NER DIAGNOSTICS (Across 1500 Sentences)
Structural Errors (BILOU violations): 2
Semantic Errors (Type mismatches)   : 130
{'eval_loss': 7.104794502258301, 'eval_precision': 0.7118197776477472, 'eval_recall': 0.77287166454892, 'eval_f1': 0.7410904660371612, 'eval_accuracy': 0.9575633397691005, 'eval_structural_errors': 2, 'eval_semantic_errors': 130, 'eval_CELL_FUNCTION_f1': 0.6666666666666667, 'eval_CELL_TYPE_f1': 0.720897615708275, 'eval_CONDITION_f1': 0.8075601374570447, 'eval_LOCATION_f1': 0.790273556231003, 'eval_MOLECULE_f1': 0.7920691408235891, 'eval_PATHOGEN_f1': 0.749034749034749, 'eval_PROCESS_f1': 0.5502645502645502, 'eval_RECEPTOR_f1': 0.7520325203252032, 'eval_TREATMENT_f1': 0.7051282051282052, 'eval_runtime': 9.4248, 'eval_samples_per_second': 159.154, 'eval_steps_per_second': 19.947, 'epoch': 10.0}

OVERALL F1         | 0.7391   
NER DIAGNOSTICS (Across 1500 Sentences)
Structural Errors (BILOU violations): 1
Semantic Errors (Type mismatches)   : 127

OVERALL F1         | 0.7430   
NER DIAGNOSTICS (Across 1500 Sentences)
Structural Errors (BILOU violations): 3
Semantic Errors (Type mismatches)   : 124

OVERALL F1         | 0.7441   
NER DIAGNOSTICS (Across 1500 Sentences)
Structural Errors (BILOU violations): 4
Semantic Errors (Type mismatches)   : 130

OVERALL F1         | 0.7424   
NER DIAGNOSTICS (Across 1500 Sentences)
Structural Errors (BILOU violations): 6
Semantic Errors (Type mismatches)   : 127

OVERALL F1         | 0.7423   
NER DIAGNOSTICS (Across 1500 Sentences)
Structural Errors (BILOU violations): 6
Semantic Errors (Type mismatches)   : 129
{'eval_loss': 10.66079330444336, 'eval_precision': 0.7206102303320371, 'eval_recall': 0.7652477763659467, 'eval_f1': 0.7422585117855492, 'eval_accuracy': 0.9581675138237651, 'eval_structural_errors': 6, 'eval_semantic_errors': 129, 'eval_CELL_FUNCTION_f1': 0.6499999999999999, 'eval_CELL_TYPE_f1': 0.7249070631970259, 'eval_CONDITION_f1': 0.8055555555555556, 'eval_LOCATION_f1': 0.7797619047619048, 'eval_MOLECULE_f1': 0.7905301080802881, 'eval_PATHOGEN_f1': 0.784313725490196, 'eval_PROCESS_f1': 0.5239005736137667, 'eval_RECEPTOR_f1': 0.760914760914761, 'eval_TREATMENT_f1': 0.7172413793103448, 'eval_runtime': 10.5311, 'eval_samples_per_second': 142.435, 'eval_steps_per_second': 17.852, 'epoch': 15.0}
{'train_runtime': 3356.2347, 'train_samples_per_second': 60.331, 'train_steps_per_second': 3.772, 'train_loss': 5.594796442733651, 'epoch': 15.0}

# Training 9 
**Dataset Information**: 30k 
**Epochs (Not Consistent)**: 5 
**Epoch Performance**:
OVERALL F1         | 0.7063   
NER DIAGNOSTICS (Across 1500 Sentences)
Structural Errors (BILOU violations): 217
Semantic Errors (Type mismatches)   : 152
{'eval_loss': 5.356651782989502, 'eval_precision': 0.6636871508379888, 'eval_recall': 0.7547649301143583, 'eval_f1': 0.7063020214030915, 'eval_accuracy': 0.9577866214849547, 'eval_structural_errors': 217, 'eval_semantic_errors': 152, 'eval_CELL_FUNCTION_f1': 0.5671641791044775, 'eval_CELL_TYPE_f1': 0.7054880603267699, 'eval_CONDITION_f1': 0.7731958762886597, 'eval_LOCATION_f1': 0.7624633431085045, 'eval_MOLECULE_f1': 0.7479905437352247, 'eval_PATHOGEN_f1': 0.7172995780590717, 'eval_PROCESS_f1': 0.44036697247706424, 'eval_RECEPTOR_f1': 0.6774193548387096, 'eval_TREATMENT_f1': 0.6356589147286822, 'eval_runtime': 4.8509, 'eval_samples_per_second': 309.221, 'eval_steps_per_second': 2.886, 'epoch': 1.0}

OVERALL F1         | 0.7347   
NER DIAGNOSTICS (Across 1500 Sentences)
Structural Errors (BILOU violations): 65
Semantic Errors (Type mismatches)   : 133
{'eval_loss': 4.3260626792907715, 'eval_precision': 0.7136867325546571, 'eval_recall': 0.7569885641677255, 'eval_f1': 0.734700169569909, 'eval_accuracy': 0.9577603530477954, 'eval_structural_errors': 65, 'eval_semantic_errors': 133, 'eval_CELL_FUNCTION_f1': 0.6052631578947368, 'eval_CELL_TYPE_f1': 0.726457399103139, 'eval_CONDITION_f1': 0.7950530035335689, 'eval_LOCATION_f1': 0.7648902821316613, 'eval_MOLECULE_f1': 0.7818844351900053, 'eval_PATHOGEN_f1': 0.7468879668049793, 'eval_PROCESS_f1': 0.5104602510460252, 'eval_RECEPTOR_f1': 0.7351778656126482, 'eval_TREATMENT_f1': 0.72, 'eval_runtime': 4.7115, 'eval_samples_per_second': 318.368, 'eval_steps_per_second': 2.971, 'epoch': 2.0}

OVERALL F1         | 0.7448   
NER DIAGNOSTICS (Across 1500 Sentences)
Structural Errors (BILOU violations): 38
Semantic Errors (Type mismatches)   : 118
{'eval_loss': 4.032294750213623, 'eval_precision': 0.7128339140534262, 'eval_recall': 0.7798602287166455, 'eval_f1': 0.7448422330097088, 'eval_accuracy': 0.9588110905341687, 'eval_structural_errors': 38, 'eval_semantic_errors': 118, 'eval_CELL_FUNCTION_f1': 0.6578947368421052, 'eval_CELL_TYPE_f1': 0.7336907953529938, 'eval_CONDITION_f1': 0.8111888111888113, 'eval_LOCATION_f1': 0.7547169811320754, 'eval_MOLECULE_f1': 0.7880964597229347, 'eval_PATHOGEN_f1': 0.7626459143968872, 'eval_PROCESS_f1': 0.5468164794007491, 'eval_RECEPTOR_f1': 0.7490196078431371, 'eval_TREATMENT_f1': 0.782608695652174, 'eval_runtime': 4.7219, 'eval_samples_per_second': 317.666, 'eval_steps_per_second': 2.965, 'epoch': 3.0}

OVERALL F1         | 0.7488   
NER DIAGNOSTICS (Across 1500 Sentences)
Structural Errors (BILOU violations): 23
Semantic Errors (Type mismatches)   : 122
{'eval_loss': 3.8826029300689697, 'eval_precision': 0.7255661501787842, 'eval_recall': 0.7735069885641678, 'eval_f1': 0.748769987699877, 'eval_accuracy': 0.959349593495935, 'eval_structural_errors': 23, 'eval_semantic_errors': 122, 'eval_CELL_FUNCTION_f1': 0.7042253521126761, 'eval_CELL_TYPE_f1': 0.7440180586907448, 'eval_CONDITION_f1': 0.7910189982728844, 'eval_LOCATION_f1': 0.7781155015197568, 'eval_MOLECULE_f1': 0.7900466562986003, 'eval_PATHOGEN_f1': 0.7341772151898733, 'eval_PROCESS_f1': 0.5378486055776893, 'eval_RECEPTOR_f1': 0.7615230460921845, 'eval_TREATMENT_f1': 0.7692307692307693, 'eval_runtime': 4.7462, 'eval_samples_per_second': 316.042, 'eval_steps_per_second': 2.95, 'epoch': 4.0}

OVERALL F1         | 0.7480   
NER DIAGNOSTICS (Across 1500 Sentences)
Structural Errors (BILOU violations): 22
Semantic Errors (Type mismatches)   : 125
{'eval_loss': 3.9066359996795654, 'eval_precision': 0.7162790697674418, 'eval_recall': 0.7827191867852605, 'eval_f1': 0.7480267152398299, 'eval_accuracy': 0.958797956315589, 'eval_structural_errors': 22, 'eval_semantic_errors': 125, 'eval_CELL_FUNCTION_f1': 0.6493506493506493, 'eval_CELL_TYPE_f1': 0.7452745274527453, 'eval_CONDITION_f1': 0.7917383820998278, 'eval_LOCATION_f1': 0.7682926829268293, 'eval_MOLECULE_f1': 0.791476407914764, 'eval_PATHOGEN_f1': 0.7509881422924901, 'eval_PROCESS_f1': 0.5430210325047802, 'eval_RECEPTOR_f1': 0.7505070993914807, 'eval_TREATMENT_f1': 0.7571428571428571, 'eval_runtime': 4.6934, 'eval_samples_per_second': 319.598, 'eval_steps_per_second': 2.983, 'epoch': 5.0}
{'train_runtime': 350.8536, 'train_samples_per_second': 192.374, 'train_steps_per_second': 1.753, 'train_loss': 9.650341077354865, 'epoch': 5.0}

**Evaluation**: 
- Model hits its best performance basically right at the first epoch, with very little growth afterwards. 
- Lots of different data, but I'm gonna put it into the model information rather than here so I don't have to write it here.

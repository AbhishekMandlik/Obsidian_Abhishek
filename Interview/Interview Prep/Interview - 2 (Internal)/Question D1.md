
1. Prompting vs RAG vs Fine-tuning....
	1. Prompting - It is done when we want to change:
		1. Behavior
		2. Format
		3. Tone
		4. Reasoning style
		It had limitations that it can't teach new knowledge. If LLMs don't know the data then they will never be able to answer the question correctly.
	2. RAG - It gives LLM external knowledge:
		1. Architecture:
		   User Question->Embedding Model->Vector DB->Top-k documents->LLM.
		   So LLM has got external knowledge.
		   >Reduces hallucination gives correct information to the LLM.
		   
		2. Advantages
			1. Uses latest data.
			2. No retraining.
		3. Limitations 
			1. Can't change model behaviour fundamentally.
			2. It only provides the context.
	3. Fine Tuning: Change weights, change model behaviour.
		1. Advantages
			1. No long prompts
			2. Consistent output
			3. Learns pattern
		2. Disadvantages
			1. Expensive
			2. Needs GPUs
			3. Needs training data
			4. Can't Easily update knowledge



2. LoRA/ QLoRA / PEFT
   During Full Fine-tuning: Every parameter must be updated. 
	1. PEFT: Parameter Efficient Fine-Tuning: Only update a tiny percentage of parameters instead of updating all.
		1. Benefits
			1. Much lower GPU memory
			2. Faster training
			3. Smaller checkpoints
			4. Easy to switch between tasks
	2. LoRA: Low Rank Adaptation
	   Original Weight matrix W, (Freeze it and learn two much smaller matrices.)
	   Learn A x B and update the weight. Both low rank(less param) matrices.
	   If W is (4096 x 4096) choose a rank 8. { A=4096 x 8;B=8 x 4096 } 
	   Y=(W+BA)X;
	   as W is always frozen backpropagation upgrades only B,A. 
	   ```W˚=W + A x B``` 
		1. Advantages
			1. Much lower GPU memory
			2. Faster training
		How do we choose Rank:
		- Small rank - lower memory, may underfit.
		- Large rank - more memory, can overfit but captures meaning well.
			  ```
			  W′=W+(α/r)​BA 
			  r=rank
			  alpha controls how strongly does the adapter influence the frozen 
			  weights. Scaling factor for the learned update
			  ```
		  
	3. QLoRA: Quantisation LoRA:
		1. Instead of storing each weight as FP16 store it using 4 bits.
		   eg. 0.8347234 represent it as 0.83.
		   4-bit quantisation map 16 bit to 4 bit 0.87 round it to 0.75 or 0.88 round it to 1.
		2. NF4 uses levels optimised for weights that approximately follow a normal distribution (Gaussian).
		3. Double Quantisation: It requires storing scale values, QLoRA also compresses these scales => Double Quantisation.
		   #### Why is QLoRA more memory-efficient?
			Because the base model is stored in 4-bit precision while only the small LoRA adapters are trained in higher precision. This dramatically reduces GPU memory requirements without retraining billions of parameters.


3. LLM Evaluation:
	1. BLEU: (Bilingual Evaluation Understudy)Used for machine translation.
	   It measure how many n-grams overlap with the reference answer, Higher the better. Only rewards if wording is similar.
	2. ROUGE: (Recall-Oriented Understudy for Gisting Evaluation)
	   It breaks text down into n-grams.
		1. ROUGE - 1: (Unigrams)Measures the overlap of individual words.
		2. ROUGE - 2: (Bigrams)Measures the overlap of two-word pairs.
		   (checks if the word combination are in order)
		3. ROUGE - L (Longest Common Subsequence) Evaluates the longest sequence of words that appear in both the texts and in same order even if they aren't next to each other.
	3. Perplexity:  Metric used to evaluate how well a LLM predicts a sample of text, measuring models internal confusion or uncertainty.![[Pasted image 20260702180800.png]]
	   W=(w1,w2,w3,w4,....,wN) is the sequence of text.
	   Perplexity of 10 means it is confused between 10 words.
	4. Hallucination Detection: It means model invents facts.
	   Detecting hallucination
		1. M1: Compare against ground truth.
		2. M2: RAG Faithfulness: Check if every claim is supported by retrieved document.
		3. LLM-as-a-Judge: Validate against response from other model based on specific prompts and expected_answer, answer, input question and rag parameters. 
	5. Human Evaluation: Gold standard, HITL used most of the times.



4. What is a context window: A **context window** is the maximum amount of text (measured in tokens) an AI model can process and remember at one single time during a conversation.


5. Evaluation Metrics
	1. Precision(P) =TP/(TP + FP)...whenever false positive are expensive.
	2. Recall(R) = TP/(TP+FN)...whenever missing positive is expensive.
	3. F1 Score = 2(P * R)/(P+R)
	4. Accuracy = (TP +TN)/Total
	5. False positive Rate=FP/(FP+TN)
	6. True positive Rate=TP/(TP+FN)
	7. ROC Curve: X axis: FPR, Y axis: TPR;
   **ROC-AUC**: Use to compare classifiers or evaluate how well a model separates classes across different decision thresholds. For heavily imbalanced datasets, I would also examine the **Precision-Recall curve**, since it better reflects performance on the positive class.
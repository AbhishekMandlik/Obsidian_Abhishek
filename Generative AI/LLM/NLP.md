Traditional NLP Pipeline.
Text -> Tokenisation -> Stopword Removal -> Stemming/Lemmatisation -> Feature Extraction -> ML Model.
1. Tokenisation - Split text into tokens. (It Depends on the Tokeniser).
2. Stopword Removal - remove words like the, is, a, etc.
3. Stemming - Reduce word to root form.
   eg. running becomes run. It uses simple rule, so sometimes studies becomes "studi".
4. Lemmatisation - Dictionary based word improvement. Better becomes good.


### Traditional Text Representation
1. Bag of Words (BoW):
   Documents:
	Doc1:
	I love AI
	Doc2:
	I love NLP
	
	Vocabulary: I, love, AI, NLP.
	Doc1 [1,1,1,0], Doc 2 [1,1,0,1]. Based on which word is present in the doc and vocabulary.
2. TF - IDF:
   Term Frequency (In doc).
   Inverse Document Frequency (In all docs)
3. Word Embeddings:
   Semantic Meaning.
	1. Word2Vec: Learn meaning from context.
	   Training Approaches
		1. Center word prediction
		2. Skip - Gram (Neighbouring words)
	2. GloVe: global word statistics
	3. FastText
	   split playing into pla, lay, ayi, yin, ing...
	   Now player, played and playing share subword information. It can handle
		1. Rare Words.
		2. Typos.
		3. Morphological variations.

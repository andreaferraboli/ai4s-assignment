AI PROJECT IDEAS & PROPOSALS

TASK 1

I would personally merge the 2 datasets before performing any evaluation. They both have the exact same characteristics (just the prompt and the label) plus I don’t believe there is anything to be gained by splitting our investigation on both: we might actually miss some characteristics or overfit on one dataset peculiarity if we keep them separated, plus having to do the same evaluations twice.

**1.1) INVESTIGATE BASE CHARACTERISTICS**  
Investigate some base characteristics for each prompt type and evaluate differences. Some examples may be:

- Average length of genuine vs. jailbreak prompts.  
- Words/terms heavily present in jailbreak prompts but close to or not at all in genuine prompts.  
- Type of request being made.  
- Other…

Build some *naive classifiers* based on these characteristics after investigation (the professor always does it) to be later compared to trained classifiers.

**1.2) FIRST CLUSTERING AND INVESTIGATION OF THE SEPARABILITY OF GENUINE VS JAILBREAK PROMPTS BY FEATURE SPACE ALONE**  
We could investigate which representation of the text divides genuine prompts from jailbreak ones just in their feature space representation alone.   
To do so we could run k-means clustering algorithm with parameter k=2 and then for each representation evaluate how well the formed clusters separate the classes using *homogeneity, completeness & v-measure* (capire bene quale di queste metriche usare, probabile che l’approccio migliore sia usarle tutte).

VISUALIZE all the clusterings we perform and accompany metric evaluation with visual one: do the 2 classes look separated? Do they overlap? If they do, where is the overlapping happening?

Perform *lowercasing,* *tokenization, stop word removal & lemmatization/stemming* (io preferisco sempre la lemmatization) before using any kind of encoding/representation to remove noise.

POSSIBLE DIFFERENT REPRESENTATIONS TO BE TESTED:

- Simple word count vectors (CountVectorizer).  
- Tf-idf word vectors.  
- Use text embedding model to embed prompts into shared feature space.  
- Use a the embedding layer of a fine-tuned classifier for prompt jailbreak detection?

HYPOTHESIS: the representation that best divides genuine and jailbreak prompts into 2 different clusters will also result in the best classifier performance (tested in section **2** where we train and test each classifier on each different representation).

POSSIBLE EXTRA IDEA: perform k-means clustering at different k values or use DBScan to investigate how many general types of prompts exist. Bit redundant and overlapping with the next section **1.3**.

**1.3) EVALUATE DIFFERENT JAILBREAK TYPES USING CLUSTERING ALGORITHMS**  
We could run clustering algorithms ONLY ON JAILBREAK PROMPTS to classify different jailbreaks into different types.

- k-means using the elbow method (which metric? Probabile BSS, WSS or Variance Ratio Criterion for best cluster separation).  
- Hierarchical clustering, keep clusters that result in best separation (which metrics?).  
- DBScan.

VISUALIZE THE MOST PROMISING RESULT AND CHOOSE THE BEST CLUSTERS BASED ON METRICS \+ VISUAL EVALUATION.

Once we have decided the best separation of jailbreak types we can investigate the prompts that fall in each cluster and classify them based on their characteristics (ex. roleplaying jailbreak attempt) → useful later in section   for model evaluation and to test which jailbreak an LLM is most vulnerable to. Also test which defenses work best against specific jailbreak types.

**1.4) ADDITIONAL TESTING AND EVALUATIONS USING SAMPLES FROM DATASETS C & D**  
To better evaluate our clusterings and divisions we can introduce samples from the other 2 datasets and investigate if the divisions we found are still valid.

POSSIBLE INVESTIGATIONS TO BE MADE HERE:

- OVERALL REPRESENTATION: does our best representation (the one that separates genuine vs. jailbreak prompts) correctly separate also examples coming from the C and D datasets? If not where do the extra datapoints land? Does one representation become better or worse with the introduction of these new prompts?  
- JAILBREAK TYPE CLUSTERING DIVISION: do the newly introduced jailbreak prompts fall inside the clusters that define the different jailbreak types? Yes: the classes of jailbreak types we found by clustering samples from datasets A & B cover most of the jailbreak feature space, classification is good and can be used. No: the classes of jailbreak types we found by clustering samples from datasets A & B do not sufficiently cover the jailbreak feature space, we need to reperform the clustering and jailbreak classification including examples from datasets C & D to have better and complete classification of jailbreak types.

TASK 2

I would train and test each single classifier on [all of the different text representations](#bookmark=id.cs0ldlkfq8v5): for each classifier evaluate which representation leads to the best performance and why that representation is tailored to that specific classifier. The [main hypothesis](#bookmark=id.j8pta7ghpu6g) is that the representation that best separates genuine and jailbreak prompts will also lead to better classifier performance.

Goes without saying but perform train-validation-test split on the dataset and use k-cross validation (k to be determined) to evaluate each single classifier. SET THE SEED SO EACH EXPERIMENT IS REPRODUCIBLE

Try simple linear classifiers first then move to non-linear classifiers. 

**COMPARISONS**  
We could compare each classifier’s performance with the [naive classifiers](#bookmark=id.s1c9e3eev2rl) and with fine-tuned models just for comparison purposes.

**MOST APPROPRIATE EVALUATION METHODS**  
*Recall* is in my opinion the most important metric for this task: we don’t want to miss any jailbreak attempt. 

**DETECTION OF DIFFERENT JAILBREAK TYPES**  
Evaluate which [jailbreak types](#bookmark=id.o6o074ih29nk) our best classifiers easily detect and which they miss.

EVALUATE BY APPLYING CLASSIFICATION METRICS TO EACH SINGLE CLASS OF JAILBREAK PROMPTS.

Are there types that all models find easy to detect/struggle with? Are there classifiers who perform best on different types? If yes why?

**CALIBRATION**  
Let’s put it in just because the professor always puts it in the tutorials. Get our best classifiers (I don’t know the best 3? 5?) and test their calibration. → Other parameter we can use to judge which classifier is the best.

**LIME EXPLANATIONS**  
Can ONLY BE USED ON CLASSIFIERS TRAINED WITH THE WORD COUNT OR TF-IDF WORD VECTORS (only ones where the features are the words). Take the best performing word count/tf-idf based classifiers and explain some predictions:

- Use it on prompts classified as jailbreak to understand what words drove the decision. Are they the same words we based our [naive classifiers](#bookmark=id.s1c9e3eev2rl) on? Are they the same words that [we found to be mainly in jailbreak prompts](#bookmark=id.6faazdp93ce)?  
- What are the features that drive the classifier to classify a prompt as genuine?

**PREDICTION ON DATASETS C & D**  
Run our best classifier on datasets C & D → test the performance with [different text representations](#bookmark=id.cs0ldlkfq8v5) here as well.

On which jailbreak types (of datasets C & D) does the classifier work best? On which ones does it work the worst? Do performances coincide with the ones [coming from datasets A & B](#bookmark=id.zflvq4stk2wk)?

**LLM JAILBREAK TESTING**  
Try to jailbreak an LLM using prompts coming from datasets C & D. Try to evaluate the difference in results obtained by using the [different jailbreak types](#bookmark=id.o6o074ih29nk) we found during clustering.

TASK 3

**USING BEST CLASSIFIER AS JAILBREAK GUARD**  
Try to use our best classifier as guard against jailbreaks.   
POSSIBLE IMPLEMENTATIONS:

- Define a predicted probability of being a jailbreak threshold over which the LLM does not respond/gives automated response.  
- Use regex strings based on the [most common jailbreak words](#bookmark=id.6faazdp93ce) and the [features that drive positive jailbreak classification the most](#bookmark=id.yltovuezv20l) discovered through the LIME explanator.  
- Other ideas?

During this process investigate and keep track of which jailbreak types tend to work against our classifier.

**IMPROVE OUR BEST CLASSIFIER**  
Evaluate what are the weaknesses of our best classifier.

SPECIFIC JAILBREAK TYPE AVOIDS DETECTION → Perform further fine-tuning on just that type of jailbreak prompt. Other ideas?

**CHANGING SYSTEM PROMPT TO AVOID JAILBREAK ATTACKS**  
Possible system prompt approaches to try:

- Provide examples of jailbreak attempts inside the system prompt. → Investigate if providing examples of different types of jailbreaks helps the LLM detect only some type of attempts.  
- Provide very specific instructions to enforce that secrets should never be revealed.  
- Other ideas?

**POSSIBLE SMALL MODELS TO TEST**

- Qwen family models.  
- Gemma family models.  
- Llama family models.  
- Mistral family models.  
- Deepseek family models.

TASK 4

**SECOND GUARDRAIL LLM**  
We can try using a large language model as safeguard. Plenty of pre-trained fine tuned models for this. 

POSSIBLE GUARDRAIL LLMs:

- Fine-tuned models transformer models for this task: an example (and best one in my opinion) is [meta-llama/Llama-Prompt-Guard-2-86M · Hugging Face](https://huggingface.co/meta-llama/Llama-Prompt-Guard-2-86M) .  
- We can also use a more general purpose LLM with a specific system prompt that tells it to detect prompt injection attempts. → System prompt should be very detailed and contain examples of ALL JAILBREAK TYPES.  
- Text encoders that embed the prompt and raise a flag if its embedding representation is similar to the ones of jailbreak prompts.

**ATTACKER LLM**  
LLM which actively tries to jailbreak another.  
POSSIBLE ATTACKER LLMs:

- Give very detailed system prompt. Provide examples for ALL JAILBREAK TYPES.  
- Connect the LLM to a small knowledge base containing information and examples of all different jailbreak types.

**DATABASE AGENT (SQL INJECTION)**  
Set up the SQL database agent using Langchain and Langgraph, give it access to SQL tools (we can code them or use the pre-made ones from *langchain\_community SQLDatabaseToolkit*).

POSSIBLE DIFFERENT SAFEGUARDS TO TEST:

- [ ] Put in system prompts that nobody should access a certain table and then trying to make the agent access it.  
- [ ] Put in system prompts that it should only perform SELECT operations and then try to make the agent inject some data inside the database.  
- [ ] Put safeguards (our classifier, naive classifiers, regex strings, other guardrail LLMs) on top of the agent and then try to make it return data it should not or make it modify sensitive data.


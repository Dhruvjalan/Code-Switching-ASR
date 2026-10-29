# ASR for Code-Switched Speech: An Experimental Study

## 1. Introduction

Automatic Speech Recognition (ASR) aims to convert spoken language into text. However, recognizing speech becomes more challenging when speakers switch between languages within an utterance, a phenomenon known as *code-switching*. Such speech is common in multilingual communities, including speakers who combine Hindi and English in everyday conversations.

Code-switched ASR must handle changes in vocabulary, pronunciation, language-specific phonetics, and writing systems within the same utterance. For example, a speaker may use Hindi grammatical structures while incorporating English technical terms, or switch between Hindi and English multiple times in a single sentence. A recognition system must identify these transitions while producing a consistent textual representation.

This project investigates pretrained ASR models and fine-tuning strategies for code-switched speech, with a focus on Hindi-English (Hinglish). Rather than training large neural networks from scratch, the project adopts an experimental approach using pretrained multilingual and speech recognition models. Different model architectures, training strategies, decoding configurations, and evaluation methods are studied to identify practical approaches for improving recognition of code-switched speech under limited computational resources.

## 2. Problem Statement

The objective is to develop and evaluate an ASR pipeline capable of recognizing Hindi-English code-switched speech while preserving words from both languages in the transcription.

The project focuses on the following questions:

- How effectively do pretrained multilingual ASR models recognize code-switched speech without additional training?
- Does fine-tuning on code-switched data improve recognition accuracy compared with zero-shot or baseline inference?
- How do model architecture, training data composition, and decoding strategies affect recognition performance?
- Can parameter-efficient fine-tuning reduce computational requirements while retaining useful recognition accuracy?
- Does fine-tuning on code-switched data affect performance on speech that is predominantly Hindi or English?

The primary goal is to establish a reproducible experimental framework that supports systematic comparison of models and training configurations.

## 3. Background and Research Motivation

Existing approaches to code-switched ASR include multilingual encoder-decoder models, shared multilingual acoustic representations, language identification, multitask learning, and language-model-assisted decoding.

Pretrained models such as Whisper and multilingual speech encoders provide a practical starting point because they have already learned useful speech representations from large datasets. Fine-tuning these models on domain-specific or code-switched speech may improve recognition without the cost of training a model from scratch.

However, multilingual pretraining does not guarantee accurate recognition of code-switched utterances. Models may normalize unfamiliar words into a dominant language, omit English terms, produce inconsistent scripts, or generate repetitive output after unstable fine-tuning.

This project therefore emphasizes empirical validation. Baseline performance, data quality, training stability, and held-out evaluation are treated as essential components of the experimental process.

## 4. Dataset and Preprocessing

### 4.1 Dataset

The project uses the Hugging Face dataset `agarwalayushi/hinglish` as an initial source of Hindi-English speech and transcripts. The dataset is accessed through streaming where appropriate, reducing the need to download the entire collection before experimentation.

The usable dataset size depends on audio availability, successful decoding, transcript quality, and filtering criteria. The following properties are recorded during dataset preparation:

- Number of usable audio-transcript pairs.
- Duration and sampling rate of the audio recordings.
- Distribution of speakers or source identifiers.
- Distribution of utterance lengths.
- Language and script composition of the transcripts.
- Missing audio, decoding errors, and malformed transcripts.

### 4.2 Preprocessing

Audio is converted to a consistent 16 kHz mono representation when required by the selected model. Transcripts are cleaned to remove dataset-specific markup while preserving the original spoken content.

Preprocessing must avoid unintentionally translating Hindi into English, English into Hindi, or converting the entire transcript into a single script. Such transformations could change the recognition target and make comparisons between systems unreliable.

Duplicate recordings and near-duplicate transcripts should also be identified where possible.

### 4.3 Data Splitting

The dataset is divided into training, validation, and test partitions. Whenever speaker or source metadata is available and sufficiently diverse, splitting is performed by speaker or source rather than randomly by utterance.

This reduces the risk of evaluating a model on speech that is highly similar to recordings it encountered during training. The test set is reserved for final comparisons and is not used for hyperparameter selection.

## 5. Experimental Methodology

The project follows an incremental experimental framework. Each experiment changes a limited number of variables so that observed differences can be attributed more reliably to the methodology being tested.

### 5.1 Experiment 1: Pretrained Model Baselines

The first experiment evaluates pretrained ASR models without additional fine-tuning. Whisper and other suitable pretrained multilingual speech recognition models can be compared using the same held-out recordings.

The purpose is to establish the performance achievable without task-specific training and identify common error patterns.

**Variables investigated:**
- Pretrained model and model size.
- Language and transcription-task configuration.
- Greedy decoding versus beam search, where supported.
- Output normalization and transcription format.

**Evaluation:** Word Error Rate (WER), transcription examples, and qualitative analysis of language-switching errors.

### 5.2 Experiment 2: Full Fine-Tuning

The second experiment fine-tunes a pretrained model using the available code-switched training data.

Training parameters such as learning rate, batch size, number of epochs, and gradient accumulation are controlled. Validation performance is monitored to identify overfitting and unstable generation.

The fine-tuned model is compared against its original pretrained checkpoint using the same evaluation examples. Training loss alone is not considered evidence of improved recognition.

**Objective:** Determine whether supervised adaptation to Hinglish improves recognition over the pretrained baseline.

### 5.3 Experiment 3: Parameter-Efficient Fine-Tuning

Full fine-tuning may be computationally expensive for larger models. Parameter-efficient methods, such as Low-Rank Adaptation (LoRA), provide an alternative by updating a smaller number of trainable parameters.

This experiment compares a parameter-efficient approach against full fine-tuning using the same base model and data split.

**Variables investigated:**
- Number of trainable parameters.
- Learning rate and adapter configuration.
- Training time and GPU memory consumption.
- Validation WER and qualitative transcription quality.

The goal is to determine whether parameter-efficient adaptation offers a useful trade-off between computational cost and recognition accuracy.

### 5.4 Experiment 4: Training Data Composition

The composition of the training data may strongly influence code-switched recognition. This experiment investigates the effect of changing the amount and type of training data.

Possible configurations include:

- Small, medium, and larger subsets of code-switched speech.
- Hindi-dominant, English-dominant, and mixed-language utterances.
- Balanced and naturally occurring language distributions.
- Different proportions of monolingual and code-switched data, where suitable datasets are available.

The experiments should keep the evaluation set fixed and report the actual number of utterances and audio hours used.

**Objective:** Understand how training data quantity and composition affect generalization to unseen speakers and different patterns of language switching.

### 5.5 Experiment 5: Language and Script Handling

Hindi-English speech can contain Hindi words written in Devanagari, English words written in Roman script, and transcripts that mix both scripts. Recognition errors may arise from incorrect words as well as inconsistent script choices.

This experiment examines the effect of transcript normalization and target representation on measured performance.

The primary evaluation preserves a consistent reference transcription policy. Alternative normalization strategies are evaluated separately so that script normalization does not conceal genuine recognition errors.

**Objective:** Distinguish acoustic recognition errors from differences caused by spelling, transliteration, and text normalization.

### 5.6 Experiment 6: Decoding and Language-Model Assistance

Greedy decoding and beam search are compared where supported by the selected model. If a suitable text corpus and sufficient resources are available, an external language model or hypothesis-rescoring method can be explored as a separate experiment.

Any external language model must be evaluated carefully because it may improve grammatical plausibility while incorrectly replacing code-switched words with more common monolingual alternatives.

This experiment is optional and follows the establishment of a reliable baseline and fine-tuning pipeline.

## 6. Methodologies Explored

The project investigates the following methodological directions:

1. **Pretrained multilingual ASR:** Evaluating existing speech recognition models without additional training.
2. **Supervised fine-tuning:** Adapting pretrained model weights to code-switched speech.
3. **Parameter-efficient fine-tuning:** Exploring methods such as LoRA to reduce memory and training requirements.
4. **Data composition experiments:** Studying the effects of dataset size, language distribution, and source diversity.
5. **Text representation and normalization:** Examining the handling of mixed scripts, spelling variation, and language-specific vocabulary.
6. **Decoding strategies:** Comparing supported decoding configurations and investigating language-model assistance if resources permit.

These methods form an experimental framework rather than a claim that every approach has already been implemented or evaluated. Each method is recorded with its configuration, computational cost, and measured outcome.

## 7. Experiments and Results

### 7.1 Experimental Setup

The project investigates Hinglish code-switched automatic speech recognition (ASR) using the `agarwalayushi/hinglish` dataset and the pretrained `openai/whisper-small` model. The experiments were conducted in two phases.

- **Phase A – ASR Fine-Tuning:** The pretrained Whisper model was fine-tuned on Hinglish speech data to improve transcription performance. The resulting checkpoint was reused in Phase B.
- **Phase B – Joint ASR and Language Identification:** A language identification (LID) classification head was added to the Phase A model. The encoder and LID head were trained jointly, while the decoder remained frozen.

Phase B used 5,085 training examples, 6,054 validation examples, and 4,168 held-out test examples. LID labels were derived from transcript script composition and grouped into Hindi-script dominant, English-script dominant, and mixed-script categories. These labels represent script categories rather than independently verified spoken-language labels.

### 7.2 Phase A: Baseline vs. Fine-Tuned Whisper

An initial comparison was performed on 415 validation examples to measure the effect of fine-tuning.

| Model | Validation WER |
|---|---:|
| Pretrained Whisper (Baseline) | 1.4116 |
| Fine-tuned Whisper | **0.4890** |

Fine-tuning reduced WER by approximately **65.4%**, indicating a substantial improvement over the pretrained baseline in this evaluation.

This comparison used a smaller evaluation subset and is reported separately from the larger Phase B evaluation below.

### 7.3 Phase B: Joint ASR and LID Results

The best Phase B checkpoint was evaluated on the full validation and held-out test sets. The Whisper weights were used for transcription, while the trained encoder and LID head generated script-category predictions.

| Metric | Validation | Test |
|---|---:|---:|
| Number of examples | 6,054 | 4,168 |
| Word Error Rate (WER) | 0.5894 | 0.6763 |
| Character Error Rate (CER) | 0.4164 | 0.4773 |
| LID Accuracy | 82.59% | 76.82% |
| LID Macro-F1 | 0.7429 | 0.7407 |
| LID Weighted-F1 | 0.8155 | 0.7572 |

The test LID macro-F1 of 0.7407 was close to the validation score of 0.7429, suggesting consistent overall classification performance across the two splits. However, the higher test WER indicates that transcription errors remain substantial and that ASR performance varies across the evaluated data.

### 7.4 Class-Wise LID Performance

The test-set results reveal differences in the model's ability to identify the three script categories.

| Script category | Precision | Recall | F1-score |
|---|---:|---:|---:|
| Hindi-script dominant | 0.68 | 0.48 | 0.56 |
| English-script dominant | 0.82 | 0.84 | 0.83 |
| Mixed-script | 0.78 | 0.88 | 0.83 |

The model performed better on English-script and mixed-script examples than on Hindi-script dominant examples. In particular, 582 of the 1,132 Hindi-dominant test examples were classified as mixed-script, indicating difficulty distinguishing these categories. The predominance of mixed-script examples also makes it important to consider macro-F1 and per-class results alongside overall accuracy.

### 7.5 Observations and Future Work

- **Fine-tuning improved ASR performance:** The initial baseline comparison showed a 65.4% reduction in WER on the 415-example evaluation subset.
- **Joint ASR and LID evaluation:** Phase B produced both transcriptions and three-class script predictions, with a test WER of 0.6763 and LID macro-F1 of 0.7407.
- **Code-switching remains challenging:** Language transitions, pronunciation differences, spelling conventions, and decoding errors continue to affect transcription quality.
- **Hindi-script recognition needs improvement:** The low recall for Hindi-script dominant examples suggests confusion with mixed-script utterances.
- **Evaluation labels have limitations:** Since LID labels were inferred from transcript scripts, the reported scores measure agreement with heuristic labels rather than independently annotated language-identification ground truth.

Future work will focus on detailed transcription error analysis, improving Hindi-dominant script classification, refining LID labels, and exploring alternative training and decoding configurations. A direct comparison between the Phase A ASR-only model and the Phase B joint model should also be performed on the same held-out test set with identical normalization and decoding settings.

### 7.6 Saved Outputs

The Phase B checkpoint and evaluation results were saved for reuse and further analysis.

- **Best checkpoint:** `outputs/phase_b_joint/best`
- **Evaluation directory:** `outputs/phase_b_joint/evaluation`
- **Metrics summary:** `final_metrics.csv`
- **Validation predictions:** `validation_predictions.csv`
- **Test predictions:** `test_predictions.csv`

The prediction files contain reference and predicted transcriptions alongside true and predicted LID labels, enabling further error analysis without rerunning inference.

## 8. Evaluation Metrics

The evaluation uses complementary metrics to assess transcription accuracy and script-category classification. ASR metrics measure the difference between predicted and reference transcripts, while LID metrics evaluate the agreement between predicted and heuristic script labels.

### 8.1 Word Error Rate (WER)

Word Error Rate is the primary metric for evaluating transcription accuracy.

\[
\mathrm{WER}=\frac{S+D+I}{N}
\]

where \(S\) is the number of substitutions, \(D\) is the number of deletions, \(I\) is the number of insertions, and \(N\) is the number of words in the reference transcription.

Lower WER indicates better performance. WER can exceed 1.0 when the total number of word errors exceeds the number of reference words. For Hinglish, consistent text normalization is particularly important because spelling conventions and script usage can affect the measured error rate.

### 8.2 Character Error Rate (CER)

Character Error Rate measures transcription errors at the character level:

\[
\mathrm{CER}=\frac{S_c+D_c+I_c}{N_c}
\]

where \(S_c\), \(D_c\), and \(I_c\) represent character substitutions, deletions, and insertions, respectively, and \(N_c\) is the number of characters in the reference text.

CER complements WER by capturing spelling differences and partial word errors that may be obscured by word-level evaluation. Lower CER indicates greater character-level similarity.

### 8.3 Language Identification Metrics

LID performance is evaluated using the following metrics:

- **Accuracy:** The proportion of examples assigned the correct script category.
- **Precision:** The proportion of predictions for a class that are correct.
- **Recall:** The proportion of actual examples of a class that are correctly identified.
- **Macro-F1:** The unweighted average of the F1-scores across the three classes, giving each class equal importance.
- **Weighted-F1:** The average of class-wise F1-scores weighted by each class's support.
- **Confusion matrix:** Shows how often examples from each true class are assigned to each predicted class.

Because the dataset is imbalanced, macro-F1 and class-wise recall are especially useful for identifying weaknesses that overall accuracy may conceal. Since the labels are generated heuristically from transcript scripts, these metrics should be interpreted as agreement with the assigned script categories.

### 8.4 Additional Evaluation Considerations

Further analysis can provide a more detailed picture of model performance, including:

- Hindi- and English-word recognition accuracy.
- Errors near language-switching boundaries.
- Substitution, deletion, insertion, repetition, and hallucination patterns.
- Inference latency and computational requirements.

Qualitative inspection is also valuable because systems with similar WER may differ in how they handle English technical terms, Hindi grammatical structures, and language changes within an utterance.


## 9. Challenges

### 9.1 Limited Effective Training Data

Although a dataset may contain many records, not every record necessarily has usable audio and reliable transcripts. The effective dataset size is determined after decoding, quality checks, deduplication, and filtering.

### 9.2 Accents and Pronunciation Variation

Speakers may pronounce the same word differently depending on regional accent, speaking style, or the language being used. English words embedded in Hindi sentences may also be pronounced differently from the same words in English-only speech.

### 9.3 Language-Switching Boundaries

Language switches may occur within a sentence or phrase. A model can recognize most of an utterance correctly but fail around the transition between Hindi and English, particularly when words are acoustically similar.

### 9.4 Mixed Scripts and Inconsistent Transcripts

The same spoken content may be represented using Devanagari, Roman script, or a mixture of both. Transcript inconsistencies can affect model learning and evaluation metrics even when the underlying pronunciation is similar.

### 9.5 Overfitting and Unstable Fine-Tuning

Small training subsets and inappropriate optimization settings can cause a model to memorize examples or generate repetitive text. Improvements in training loss do not necessarily translate into better recognition on unseen speech.

### 9.6 Computational Constraints

Training large speech recognition models from scratch is outside the practical scope of this project. The work therefore prioritizes pretrained checkpoints, manageable model sizes, parameter-efficient methods, mixed precision, gradient accumulation, and controlled experiments.

### 9.7 Evaluation Reliability

A small or poorly separated test set may produce misleading results. Reliable comparisons require consistent evaluation data, clear normalization rules, and inspection of individual predictions alongside aggregate metrics.

## 10. Conclusion and Future Work

This project investigates practical approaches to code-switched Hindi-English ASR through pretrained models and controlled fine-tuning experiments. The emphasis is on identifying configurations that improve recognition accuracy without requiring the computational resources needed to train large models from scratch.

The initial Whisper-Tiny pilot demonstrated that fine-tuning on a very small subset can degrade transcription quality, motivating a more rigorous pipeline based on larger training subsets, correct label padding, held-out evaluation, and reproducible configurations.

Future work includes systematic comparison of pretrained ASR models, full and parameter-efficient fine-tuning, training data scaling, and language-aware error analysis. If resources permit, language identification, multitask learning, and language-model-assisted decoding may be explored as extensions.

The intended outcome is an experimental framework that makes it possible to compare models fairly, understand their failure modes, and identify effective strategies for code-switched speech recognition under limited computational resources.

## 10. References

1. *Unsupervised Cross-lingual Representation Learning for Speech Recognition.* arXiv:2006.13979 [cs.CL]. https://arxiv.org/abs/2006.13979

2. Hugging Face. *Whisper Fine-Tuning Guide and Transformers Documentation.* https://huggingface.co/docs/transformers

3. *Code-Switching in End-to-End Automatic Speech Recognition: A Systematic Literature Review.* arXiv:2507.07741 [cs.CL]. https://arxiv.org/abs/2507.07741

4. Agarwal, A. *Hinglish Concatenated Audio Dataset.* Hugging Face, 2026. Aggregated from Mozilla Common Voice, AI4Bharat (Kathbath, Mann Ki Baat, Spoken Tutorial, IndicTTS), SPRINGLab IndicTTS, and custom recordings. https://huggingface.co/datasets/agarwalayushi/hinglish

5. Ganji, S., Dhawan, K., and Sinha, R. *IITG-HingCoS Corpus: A Hinglish Code-Switching Database for Automatic Speech Recognition.* Speech Communication, 110, 76–89, 2019. https://doi.org/10.1016/j.specom.2019.04.007
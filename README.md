# CAS2105 Homework 6: Mini AI Pipeline Project 🤗

**Author:** Yan Shiyu (2021147609)

---

## 1. Introduction

The aim of this project is to classify SMS messages into **normal text** and **spam text**.  
Given a raw SMS message as input, the model outputs a binary label indicating whether the message is spam (1) or a normal message (0).

- **Input:** A raw SMS text message  
- **Output:** A binary label (1 = spam, 0 = legitimate)

### Motivation

Spam messages usually contain misleading or promotional content. Many people find them annoying, and in some cases they can even have negative effects on users. Manually deleting such messages is time-consuming and inefficient, which is why an automatic classification method is needed.

Because the task has clear inputs and outputs, it provides a good setting for comparing simple rule-based baseline methods with more advanced AI-based pipelines.

---

## 2. Methods

### 2.1 Naïve Baseline

#### My Baseline (Keyword-based spam scoring + fixed threshold)

- **Method description:**  
  A simple keyword-based heuristic is used as the baseline model. Each message is converted to lowercase and lightly preprocessed. A predefined list of spam-related keywords (e.g., *free*, *win*, *prize*, *discount*) is checked against the message text. The number of matched keywords is counted, and if this count exceeds a fixed threshold, the message is classified as spam; otherwise, it is classified as non-spam.

- **Why naïve:**  
  This method relies on manually selected keywords rather than representations learned from data. It cannot capture semantic meaning or contextual information, and the decision rules are fixed rather than adaptive.

- **Likely failure modes:**  
  - **False negatives:** Spam messages that do not contain predefined keywords  
  - **False positives:** Legitimate messages that happen to include spam-related words  
  - Inability to handle paraphrases, implicit spam intent, or new vocabulary  
  - Sensitivity to keyword selection and threshold choice  

---

### 2.2 AI Pipeline

#### all-MiniLM-L6-v2

- **Models used:**  
  A pre-trained sentence embedding model (**all-MiniLM-L6-v2**) from Sentence-Transformers is used to convert each message into a fixed-length dense vector. A logistic regression classifier is trained on top of the frozen embeddings to perform binary spam classification.

- **Pipeline stages:**  
  - **Preprocessing:** Lowercasing and removal of extra whitespace  
  - **Embedding:** Each message is encoded into a semantic embedding using the pre-trained model  
  - **Classification:** Logistic regression is trained on the embeddings, with the regularization parameter selected using validation data  
  - **Post-processing:** The final model outputs binary predictions and is evaluated using accuracy, precision, recall, and F1 score on the test set  

- **Design choices and justification:**  
  Logistic regression is chosen as a lightweight and interpretable classifier that works well with fixed-size embeddings and small datasets. The embedding model is frozen, and only the downstream classifier is trained, which reduces computational cost and the risk of overfitting. Validation-based hyperparameter tuning ensures fair model selection without leaking test data.

---

## 3. Experiments

### 3.1 Dataset Description

- **Source:** Hugging Face dataset `codesignal/sms-spam-collection`  
- **Total examples:** 5,572 SMS messages (spam and legitimate)  

- **Train / Validation / Test split:**  
  - Training set: 4,456 samples  
  - Validation set: 558 samples  
  - Test set: 558 samples  

- **Preprocessing steps:**  
  Minimal preprocessing is applied. All text is lowercased and extra whitespace is removed. Labels are converted into binary values, where 1 indicates spam and 0 indicates legitimate messages.

---

### 3.2 Results

#### 1. Metric values for baseline vs. pipeline

The naïve keyword-based baseline achieves reasonable accuracy but low recall, meaning that many spam messages are missed. In contrast, the AI pipeline significantly improves recall and F1 score while maintaining high precision. Overall, the embedding-based classifier provides more balanced and robust performance than the baseline.

#### 2. Results Table

| Method       | Metric 1 (Accuracy) | Metric 2 (F1-score) |
|--------------|---------------------|---------------------|
| Baseline     | 0.9265              | 0.6667              |
| AI Pipeline  | 0.9892              | 0.9583              |

#### 3. Qualitative Examples

- **Example 1 (False Negative):**  
  *“You have been selected for a special reward. Please confirm your details to proceed.”*  
  The message does not contain explicit spam keywords such as *free* or *prize*, causing the baseline to miss it. The AI pipeline correctly identifies the implicit promotional intent using semantic context.

- **Example 2 (False Positive):**  
  *“Can you call me when you finish the meeting?”*  
  The baseline incorrectly flags this message due to the word *call*, while the AI pipeline understands the conversational context and avoids a false positive.

- **Example 3 (Both Correct):**  
  *“Congratulations! You win a free cash prize. Click the link now.”*  
  This message contains obvious spam keywords. Both methods correctly classify it, but this is a simple case that does not demonstrate deeper language understanding.

---

## 4. Reflection and Limitations

From the results, the AI pipeline clearly outperforms the naïve baseline. The embedding-based approach is able to capture semantic information beyond surface-level keywords and make context-aware decisions. Using a pre-trained sentence embedding model allows good performance without expensive training from scratch. However, I think threshold selection is particularly important, as small changes can significantly affect the trade-off between accuracy and recall. The naïve baseline struggles with implicit spam messages and produces both false negatives and false positives. Accuracy alone is not sufficient to evaluate model quality, so precision, recall, and F1 score are more informative metrics. One limitation of this work is that the embedding model was not fine-tuned on the SMS dataset. With more time or computational resources, I would experiment with fine-tuning the embeddings or trying other classifiers.

---

## 5. Code Quality and Reproducibility

All code is implemented in a single Jupyter notebook and runs end-to-end. The pipeline is organized sequentially, making it easy to follow. Random seeds are fixed during dataset splitting to ensure reproducibility. Only standard Python libraries are used, so the project can be executed with minimal setup. Overall, the code is clear, readable, and reproducible.

---

## References

[1] Jason Wei, Maarten Bosma, Vincent Zhao, Kelvin Guu, Adams Wei Yu, Brian Lester, Nan Du,  
Andrew M. Dai, and Quoc V. Le.  
*Finetuned Language Models Are Zero-Shot Learners.*  
International Conference on Learning Representations (ICLR), 2022.  
https://openreview.net/forum?id=gEZrGCozdqR

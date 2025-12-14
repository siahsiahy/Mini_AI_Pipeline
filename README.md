# CAS2105 Homework 6: Mini AI Pipeline Project 🤗

**Name:** YAN SHIYU

**Student ID:** 2021147609

Report_PDF : *https://ko.overleaf.com/read/fjshtzqdqmzt#c8c0ba*

---

## 1. Introduction

The aim of this project is to classify SMS messages into **normal text** and **spam text**.  
Given a raw SMS message as input, the model outputs a binary label indicating whether the message is spam (1) or a normal message (0).

- **Input:** A raw SMS text message  
- **Output:** A binary label (1 = spam, 0 = legitimate)

### Motivation

Modern spam messages or emails usually contain misleading or promotional contents. I think most people find these fancy messages annoying. And in some cases,they may even have a negative impact on users. However, delete such messages one by one is both time-consuming and inefficient. That is why the classification methods are needed.

The task has clear input and output, which provides a good setting for simple rule-based baseline methods and more advanced artificial intelligence-based pipelines.

---

## 2. Methods

### 2.1 Naïve Baseline

#### My Baseline (Keyword-based spam scoring + fixed threshold)

- **Method description:**  
  A simple keyword-based method is used as the baseline. The text of each message is first converted to lowercase and cleaned with basic preprocessing. Then, the model checks whether some common spam-related words (such as *free*, *win*, or *prize*) appear in the message. If enough of these keywords are found, the message is treated as spam; otherwise, it is considered a normal message.


- **Why naïve:**  
  This method depends on the manually selected keywords, not the characteristics learned from the data. It does not understand the actual meaning or context of the message, and the rules used for classification will not change according to the data.

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
  - **Preprocessing:** Lowercasing, removal of extra whitespace  
  - **Embedding:** Each message is encoded into semantic embedding with a pre-trained model. 
  - **Classification:** Logical regression is trained on embedding and uses verification data to select regularization parameters.
  - **Post-processing:** The final model outputs binary predictions and evaluates them using the accuracy, precision, recall and F1 score on the test set.

- **Design choices and justification:**  
  We use logistic regression as a lightweight and interpretable classifier for fixed-size embeddings and small data. The embedding model is frozen and only the downstream classifier is trained for simplicity and not for overfitting. The hyperparameter tuning ensures fair model selection without leaking test data.


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

The keyword-based baseline has achieved good accuracy, but its recall is still low. This means that some spam has not been detected. Compared with this baseline, the performance of the artificial intelligence pipeline in terms of recall and F1 scores shows better results, while still maintaining high accuracy. In general, embedded-based classifiers show more stable and balanced performance than the baseline.

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

From the results, the AI pipeline clearly outperforms the naïve baseline. The embedding-based approach is able to capture semantic information beyond surface-level keywords and make context-aware decisions. Using a pre-trained sentence embedding model allows good performance without expensive training from scratch. However, I think threshold selection is particularly important, as small changes can significantly affect the trade-off between accuracy and recall. The naïve baseline works for implicit spam messages and shows false negatives and false positives. Accuracy is not sufficient to measure the quality of a model, so precision, recall and F1 score should be better indicators. A limitation of this work is that it was not manually tuned for the SMS data. With more time or more computational resources, I could work on fine-tuning the embeddings or the other classifiers.

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

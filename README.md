\documentclass[11pt, a4paper]{article}  % Set default font and page size


% Load packages and configure them via options
\usepackage[utf8]{inputenc}
\usepackage[left=22mm, right=22mm, top=20mm, bottom=20mm, headheight=40pt, headsep=10pt]{geometry}  % Set margin of page
\usepackage[parfill]{parskip}  % Set spacing between paragraphs
\usepackage{amsmath}  % For the equation environment
\usepackage{graphicx}  % Required for inserting images
\usepackage[dvipsnames]{xcolor}  % Access color codes
\usepackage[colorlinks=true]{hyperref}  % Enable hyperlink
\usepackage{cleveref}  % Enable easy references to sections, figures, and tables
\usepackage{minted}  % Enable code formatting in LaTeX
\usepackage{changepage}  % Required for \adjustwidth
\usepackage{caption}
\usepackage{enumitem}  % Requried for changing configurations for list
\usepackage{lmodern}  % For \texttt font style
\usepackage{booktabs}  % For table formatting
\usepackage{multirow}  % Required for \multirow in tables
\usepackage{emoji}  % Required for Hugging Face emoji
\usepackage[numbers, sort&compress]{natbib}  % For citations and references
\bibliographystyle{unsrtnat}
\usepackage[utf8]{inputenc}

% Define macros, including color codes and macros
\definecolor{YonseiBlue}{HTML}{183772}

% TODO: Replace "CAS2105 Homework 6: Mini AI Pipeline Project \emoji{hugging-face}" with your report title
\title{CAS2105 Homework 6: Mini AI Pipeline Project \emoji{hugging-face}}  % Title


% TODO: Replace "Your Name (Your Student ID)" with your name and student ID
\author{YAN SHIYU(2021147609)}


% Redefine title style
\makeatletter
  \def\@maketitle{%
  \newpage
  \null
  \vskip 2em%
  \begin{flushleft}%
  {\color{YonseiBlue}\rule{\textwidth}{2pt}}
  \vskip 5pt%
  \let \footnote \thanks
    {\Large \bf \@title \par}%
    \vskip 1.5em%
    {\normalsize \bf \lineskip .5em% 
        \@author
        \vskip 2pt%
        \par}
    {\color{YonseiBlue}\rule{\textwidth}{2pt}}
    \vskip 1.5em
  \end{flushleft}%
  }
\makeatother



\begin{document}
\maketitle


% TODO: Replace the following sections with your own report!


\section{Introduction}
\label{sec:introduction}

% --- TODO: Students fill this in ---
% Describe your chosen task and why it is interesting.
% Keep the explanation clear enough for a classmate to understand.


The aim of this project is to classify SMS messages normal text and spam text .
Given an raw text message as input, the model outputs a binary label. This is used to indicate whether the text message is spam (1) or normal text message (0).

\begin{itemize}
    \item Input: A raw SMS text message
    \item Output: A binary label (1 = spam, 0 = legitimate)
\end{itemize}
\subsection{Motivation}
Spam messages usually contain misleading or promotional content. Many people find them annoying, and in some cases they can even have negative effects on users. Manually deleting such messages is both waste time and inefficient, which is why an automatic classification method is needed.

Because the task has clear input and output, it provides a good environment for simple rule-based baseline methods and more advanced artificial intelligence-based processes.




%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
\section{Methods}
\label{sec:methods}
\subsection{Na\"ive Baseline}
\subsubsection*{My Baseline (Keyword-based spam scoring + fixed threshold)}
\begin{itemize}
    \item \textbf{Method description:} Here I use a simple heuristic method based on keywords as the baseline model.
First, perform lowercase processing and some simple preprocessing for each message. Check the predefined spam-related keywords in the message text (for example, free, win, prize, discount)
This model can calculate the number of these keywords in the text. If the count exceeds the fixed threshold, the message is classified as spam; otherwise, it is classified as non-spam.
    \item \textbf{Why na\"ive:} It relies on manually selected keywords, not on the representation obtained through data learning. This method cannot capture the semantic meaning or context information of the text, and the decision-making rules are fixed and will not be adaptively adjusted with the data.
    \item \textbf{Likely failure modes:}  \begin{itemize}
    \item False negatives: spam that does not contain predefined keywords.
    \item False positives: legitimate messages that happen to include spam-related words.
    \item Inability to handle paraphrases, implicit spam intent, or new vocabulary.
    \item Sensitivity to keyword selection and threshold choice.
  \end{itemize}
\end{itemize}


\subsection{AI Pipeline}
\label{subsec:pipeline}

\subsubsection*{all-MiniLM-L6-v2}
\begin{itemize}
    \item \textbf{Models used:} Pre-trained sentence embedding model all-MiniLM-L6-v2 from Stence-Transformers. It is used to convert each text message into a dense vector representation of fixed length. The logical regression classifier is trained at the top of the frozen embedding to perform binary spam classification.
    \item \textbf{Pipeline stages:} \begin{itemize}
    \item Preprocessing : The pipeline first preprocesss SMS messages with lower-casing, and removing extra whitespace.
    \item Embedding : Every message is encoded into a semantic embedding by a pre-trained all-MiniLM-L6-v2 model.
    \item Classification : A logistic regression classifier is trained on these embeddings and the regularization parameter chosen based on validation.
    \item Post-processing : Finally, the final model outputs binary spam predictions and uses accuracies, accuracy, recall, and F1 scores on test set.
    \end{itemize}
    \item \textbf{Design choices and justification:} Logistic Regression is chosen as a lightweight and interpretable classifier that works well with fixed-size embeddings and small datasets. Freezing the pre-trained embedding model and train only the downstream classifier tor educes computational cost and overfitting risk. Verification-based hyperparameter adjustment ensures fair model selection without leaking test data.
\end{itemize}


%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
\section{Experiments}
\label{sec:experiments}


\subsection{Datasets}
\label{sec:experiments:datasets}

% --- TODO: Students fill this in ---
% Describe your dataset clearly.
\subsection*{Dataset Description}
\begin{itemize}
    \item \textbf{Source:} Hugging Face dataset codesignal/sms-spam-collection.
    \item \textbf{Total examples:} The dataset contains 5,572 SMS messages in total, including both spam and legitimate messages.
    \item \textbf{Train/Test split:} \begin{itemize}
    \item Training set: 4,456 samples
    \item Validation set: 558 samples
    \item Test set: 558 samples
    \end{itemize}
    \item \textbf{Preprocessing steps:} Only minimal preprocessing is applied to the SMS messages.  The text is all lowercase, and extra spaces have been removed to reduce surface inconsistencies.The labels are converted into a binary format, with 1 indicating spam and 0 indicating legitimate messages.
\end{itemize}

\subsection{Results}
\label{sec:experiments:results}

1. Metric values for baseline vs. pipeline
\begin{itemize}
The na\"ive keyword based baseline gets reasonable accuracy but low recall (many spam messages missed), while the AI pipeline improves recall and F1-score while maintaining high precision. Overall the embedding based classifier performs more balanced and robust than baseline.
\end{itemize}

2. A results table
\begin{center}
\begin{tabular}{lcc}
\toprule
\textbf{Method} & \textbf{Metric 1} & \textbf{Metric 2 (optional)} \\
\midrule
Baseline & 0.9265 & 0.6667 \\
AI Pipeline & 0.9892 & 0.9583 \\
\bottomrule
\end{tabular}
\end{center}
Metric 1 corresponds to classification accuracy, and Metric 2 corresponds to the F1-score.


3. At least three qualitative examples
\begin{itemize}
    \item \textbf{Example 1 (FN):}

''You have been selected for a special reward. Please confirm your details to proceed.'' 
    \item The message does not contain explicit spam keywords such as “free” or “prize”, causing the keyword-based baseline to miss it. The AI pipeline correctly identifies the implicit promotional intent using semantic context.
    
    \item \textbf{Example 2 (FP):}

    “Can you call me when you finish the meeting?”
    \item The baseline falsely flags the message due to the presence of the word “call” and AI pipeline understands the conversational context and avoids a false positive.

    \item \textbf{Example 3 (Both Correct):}

''Congratulations! You win a free cash prize. Click the link now.''
\item The email contains several obvious spam keywords.
Both methods can correctly identify it, but this is just a simple example and does not demonstrate a deeper level of language comprehension.
    
\end{itemize}

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
\section{Reflection and Limitations}
\label{sec:reflection}

Write approximately 6--10 sentences reflecting on:
% Students write freely here.
From this we can say that our AI pipeline provides more efficient performance than the naive baseline, that our generic AI pipeline is able to learn semantic information than the surface keywords and make decisions based on context. With our pre-trained sentence embedding model, our system will not need much computational time or training from scratch. I think it is important to decide threshold because even a small change can impact the tradeoff between accuracy and recall. The naive baseline is only using keyword matching, which fails to handle implicit spam messages, and thus makes false negatives and false positives. Accuracy alone is not sufficient to understand model quality, and precision, recall, and F1-score are more informative for this task. One limitation, however, was that the embedding was not tuned on the SMS dataset. If I had more time or computational space, I would try tuning the embeddings or experimenting with other classifiers.

\section{Code Quality and Reproducibility}
The code is written in a clear and structured way inside a single Jupyter notebook. Each step of the pipeline is implemented sequentially, which makes the workflow easy to follow. Random seeds are fixed during dataset splitting to ensure reproducible results. Common Python libraries are used, so the project can be run with minimal setup. Overall, the code is readable and reproducible.

\begin{thebibliography}{9}

\bibitem{wei2022zeroshot}
Jason Wei, Maarten Bosma, Vincent Zhao, Kelvin Guu, Adams Wei Yu, Brian Lester, Nan Du, 
Andrew M. Dai, and Quoc V. Le.
\newblock Finetuned language models are zero-shot learners.
\newblock In \emph{International Conference on Learning Representations (ICLR)}, 2022.
\newblock \url{https://openreview.net/forum?id=gEZrGCozdqR}

\end{thebibliography}

\end{document}

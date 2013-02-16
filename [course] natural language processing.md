# Natural Language Processing

Columbia University, via Coursera

## Readings policy

There are excellent readings assigned to the class. They're explicitly inlined into the respective lecture, to save typing stuff out twice.

Other readings (papers, textbooks, other courses) are explicitly inlined as well.

## Rendering

In order to use pandoc run (need to include custom LaTeX packages for some symbols):

        pandoc \[course\]\ natural\ language\ processing.md -o pdf/nlp.pdf --include-in-header=latex.template

or, for Markdown + LaTex to HTML + MathJax output:

        pandoc \[course\]\ natural\ language\ processing.md -o pdf/nlp.html --include-in-header=latex.template --mathjax

and, for the ultimate experience, after `pip install watchdog`:

        watchmedo shell-command --patterns="*.md" --ignore-directories --recursive --command='pandoc \[course\]\ natural\ language\ processing.md -o pdf/nlp.html --include-in-header=latetemplate --mathjax' .

## Week 1 - Introduction to Natural Language Processing

### Introduction (Part 1)

-   What is NLP?
    -   Computers using natural language as input and/or output.
    -   NLU: understanding, input
    -   NLG: generation, output.

Tasks

-   Oldest task: **machine translation**. Convert between two languages.
-   **Information extraction**
    -   Text as input, structure of key content as output.
    -   e.g. job posting into industry, position, location, company, salary.
    -   Complex searches ("jobs in Boston paying XXX").
    -   Statistical queries ("how has jobs changed in IT changed over time?")
-   **Text summarization**
    -   Condense one or many documents into a summary.
    -   [*Columbia Newsblaster*](http://newsblaster.cs.columbia.edu/) is an example.
-   **Dialogue systems**
    -   Humans can interact with a computer to ask questions and achieve tasks.

Basic NLP problems

-   **Tagging**
    -   Map strings to tagged sequences (each word is lexed and tagged with an appropriate label).
    -   **Part-of-speech tagging**: noun, verb, preposition, ...
        -   Profits (N) soared (V) at (P) Boeing (N)
    -   **Named Entity Recognition**: companies, locations, people
        -   Profits (NA) soared (NA) at (NA) Boeing (C)

-   **Parsing**
    -   e.g. "Boeing is located in Seattle" into a parse tree.

### Introduction (Part 2)

Why is NLP hard?

-   **Ambiguity**
    -   "At last, a computer that understands you like your mother"; three intrepretations at the *syntactic* level.
    -   But also occurs at an *acoustic* level: "like your" sounds like "lie cured".
        -   One is *more likely* than the other, but without this information difficult to tell.
    -   At *semantic* level, words often have more than one meaning. Need context to disambiguate.
        -   "I saw her duck with a telescope".
    -   At *discourse* (multi-clause) level.
        -   "Alice says they've built a computer that understands you like your mother"
        -   If you start a sentence saying "but she...", who is she referring to?

What will this course be about

-   **NLP subproblems**: tagging, parsing, disambiguation.
-   **Machine learning techniques**: probabilistic CFGs, HMMs, EM algorithm, log-linear models.
-   **Applications**: information extraction, machine translation, natural language interfaces.

#### Syllabus

-   Language modelling, smoothed estimation
-   Tagging, hidden Markov models
-   Statistical parsing
-   Machine translation
-   Log-linear models, discriminative methods
-   Semi-supervised and unsupervised learning for NLP

## Week 1 - The Language Modeling Problem

### Introduction to the Language Modeling Problem (Part 1)

-   We have some finite vocabulary, i.e.

$$V = \{the, a, man telescope, Beckham, two, ...\}$$

-   We have countably infinite set of strings, which are the set of possible sentences in the language:

$$V^+ = \{"the\:STOP", "a\:STOP", "the\:fan\:STOP", ...\}$$

-   STOP is a stop symbol at the end of a sentence. Convenient later on.
-   Sentences don't have to make sense, just every sequence of words.
-   Also a sentence could just be {"STOP"}, empty.

-   We have a *training sample* of example sentences in English.
    -   Sentences from the New York Times in the last 10 years.
    -   Sentences from a large set of web pages.
    -   In the 1990's 20 million words common, by the end of the 90's 1 billion words common.
    -   Nowadays 100's of billions of words.

-   With this training sample we want to "learn" a probabiliy distribution p, i.e. p is a function that satisfies:

$$\sum_{x \in V^+} p(x) = 1, \quad p(x) \ge 0 \; \forall \; x \in V^+$$

-   For any sentence x in language, p(x) >= 0.
-   If we sum over all sentences x in language, p(x) sums to 1.
-   A good language model assigns high probabilities to likely sentences in English (the fan saw Beckham STOP), low probabilities to unlikely sentences in English (Beckham fan saw the STOP)

### Introduction to the Language Modeling Problem (Part 2)

-   But...why do we want to do this?!
    -   **Speech recognition** was original motivation; related problems are optical character recognition and handwriting recognition.
    -   Input: sound wave time series.
    -   Preprocess: split into relatively short time periods, e.g. 10ms.
    -   For each frame do a Fourier transform, get energies of frequencies.
    -   Problem is to output recognised speech, sequence of words.
    -   Main course notes: it's useful to have prior probabilities so that if we can choose between alternatives we can ask "which is most likely?".  
        -   "recognise speech" vs "wreck a nice beach"
    -   The estimation techniques developed for this problem will be very useful for other problems in NLP.

-   Naive method of language modelling
    -   We have N training sentences.
    -   For any sentence $x_1, ..., x_n$, define $c(x_1, ..., x_n)$ as the number of times the sentences is seen in our training data.
    -   Naive estimate:

$$p(x_1, \ldots, x_n) = \frac{c(x_1, \ldots, x_n)}{N}$$

-   This is a valid, well-formed language model (p(x) sums to 1, they're all >= 0).
-   However, they'll assign a probabiliy of 0 to any unseen sentences; no ability to generalise to new sentences.
-   How can we build language models that generalise beyond the test sentences?

### Markov Processes (Part 1)

-   Markov Processes
    -   Consider a sequence of random variables $X_1, X_2, \ldots, X_n$.
    -   Each random variable can take any value in a finite set V.
    -   For now assume n is fixed, e.g. = 100. Every sequence is the same length.
    -   Our goal: model the joint probability distribution of the values of these n variables:

$$P(X_1 = x_1, X_2 = x_2, \ldots, X_n = x_n)$$

-   This is huge: for vocabulary V, number of sequences of length n is $|V|^n$.

-   First-Order Markov Processes
-   Going to use the chain rule of probabilities to decompose the expression into a product of expressions.
-   For two expressions, this rule is:

$$P(A,B) = P(A) \times P(B|A)$$
$$P(A,B,C) = P(A) \times P(B|A) \times P(C|A,B)$$

-   Hence:

$$P(X_1 = x_1, X_2 = x_2) = P(X_1 = x_1) \times P(X_2 = x_2 | X_1 = x_1)$$
$$P(X_1 = x_1, X_2 = x_2, X_3 = x_3) = ... P(X_3 = x_3 | P(X_2 = x_2, X_1 = x_1)$$

-   This kind of decomposition is *exact*: this is always true, and no assumptions are involved.
-   Hence the general decomposition:

$$P(X_1 = x_1, X_2 = x_2, \ldots, X_n = x_n)$$
$$=P(X_1 = x_1) \prod_{i=2}^{n} P(X_i = x_i\;|\;X_1 = x_1, \dots, X_{i-1} = x_{i-1})$$

-   Continuing on, with first-order Markov assumption:

$$= P(X_1 = x_1) \prod_{i=2}^{n} P(X_i = x_i\;|\; X_{i-1} = x_{i-1})$$

-   The first-order Markov assumption: for any $i \in \{2, \dots, n\}$, for any $x_1, \dots, x_n$:

$$P(X_i=x_i|X_1=x_1, \ldots, X_{i-1} = x_{i-1}) = P(X_i=x_i | X_{i-1} = x_{i-1})$$

-   Random variable at position i depends on just the previous value, on the variable at position (i-1).
    -   $X_i$ is conditionally independent of all the other random variables once you condition on $X_{i-1}$.

### Markov Processes (Part 2)

-   What about Second-Order Markov Processes?
-   Again, the problem is to model the joint distribution over $n$ random variables:

$$P(X_1 = x_1, X_2 = x_2, \ldots, X_n = x_n)$$
$$=P(X_1 = x_1) P(X_2 = x_2 | X_1 = x_1) \prod_{i=3}^{n} P(X_i = x_i | X_{i-2} = x_{i-2}, X_{i-1} = x_{i-1})$$

-   For elements further along in the sequence the value for the i'th random variable depends on the previous *two* random variables.
-   This is a bit awkward, so for convenience we assume $x_0 = x_{-1} = *$, where $*$ is a special "start" symbol.

$$= \prod_{i=1}^{n} P(X_i = x_i | X_{i-2} = x_{i-2}, X_{i-1} = x_{i-1})$$

-   For example, $x_{-1} = *,\;x_0 = *,\;x_1 = the,\;\ldots$,

#### Modelling Variable Length Sequences

-   Want $n$ to also be a random variable.
-   Simple solution: always define $X_n = STOP$, where $STOP$ is a special symbol.
-   Use a Markov process as before, but assume $X_n = STOP$.

### Trigram Language Models

-   A trigram language model consists of:
    1.  A finite set $V$ (the words, the vocabulary).
    2.  A parameter $q(w|u,v)$ for each trigram $u,v,w$ such that $w \in V \bigcup \{STOP\}$, and $u,v \in V \bigcup \{*\}$.
        -   For each *trigram* $u,v,w$, a sequence of three words, we have a parameter $q(w|u,v)$.
        -   $w$ could be any element of V or STOP, and
        -   $u,v$ could be any element of V or START.

-   For any sentence $x_1, \ldots, x_n$ where $x_i \in V$ for $i = 1 \ldots (n-1)$, and $x_n = STOP$, the probability of the sentence under the trigram model is:

$$p(x_1, \dots, x_n) = \prod_{i=1}^{n}q(x_i\;|\;x_{i-2},x_{i-1})$$

-   where we define $x_0 = x_{-1} = *$.
-   i.e. for any sentence the probability of it is the product of second-order Markov probabilities of its constituent trigrams.

An example. For the sentence

        the dog barks STOP

we could have

$p(\textrm{the dog barks STOP}) =$  
$q(\textrm{the | *, *})$  
$\times q(\textrm{dog | *, the})$  
$\times q(\textrm{barks | the, dog})$  
$\times q(\textrm{STOP | dog, barks})$  

-   This is still a naive language model. It's easy to find problems.
-   PCFGs, explored later, are much superior.
-   Having said that, trigram language models are extremely useful.
    -   They are very hard to improve upon.
    -   Considerable simplicity.

- - -

-   Quiz: say we have a language model with $V = \{\textrm{the, dog, runs}\}$, and the following parameters:

$q(\textrm{the | *, *}) = 1$  
$q(\textrm{dog | *, the}) = 0.5$  
$q(\textrm{STOP | *, the}) = 0.5$  
$q(\textrm{runs | the, dog}) = 0.5$  
$q(\textrm{STOP | the, dog}) = 0.5$  
$q(\textrm{STOP | dog, runs}) = 1$  

-   There are **three** sentences with non-zero probability under this model. Draw out a graph, where nodes are words and edge labels denote probabilities, to see this.

- - -

#### The Trigram Estimation Problem

-   But what are the values of parameters q?
-   This turns out to be a challenging problem.
-   A natural estimate: the **maximum likelihood estimate (ML)**.
-   Recall that we assume that we have a training set, some example sentences in our language, typically, as you recall, millions or billions of sentences.
-   From these sentences we can derive counts; how often do trigrams occur?

$$q(w_i\;|\;w_{i-2},w_{i-1}) = \frac{\textrm{Count}(w_{i-2},w_{i-1},w_{i})}{\textrm{Count}(w_{i-2},w_{i-1})}$$

-   For example:

$$q(\textrm{laughs | the, dog}) = \frac{\textrm{Count(the, dog laughs)}}{\textrm{Count(the, dog)}}$$

-   This is intuitive. For instances of a particular bigram how often are they followed by the particular third word of our trigram?

- - -

-   Quiz: consider the following corpus of sentences:
    -   the dog walks STOP
    -   walks the dog STOP
    -   dog walks fast STOP
-   Let $q_{ML}$ by the maximum-likelihood parameters of a trigram langauge model trained on this corpus. Which of the following parameters have a value that is both well-defined and non-zero?

Correct:

$q_{ML}({\textrm{walks | *, dog}})$  
$q_{ML}({\textrm{dog | walks, the}})$  
$q_{ML}({\textrm{walks | the, dog}})$  


Incorrect:

$q_{ML}({\textrm{walks | dog, the}})$  
$q_{ML}({\textrm{fast | dog, the}})$  
$q_{ML}({\textrm{STOP | walks, dog}})$  

- - -

-   ML is a useful starting point, but has serious problems.

Spare Data problems

-   Say our vocabulary size is $N = |V|$, then there are $N^3$ parameters in our model.
-   e.g. $N = 20,000\;\implies\;20,000^3 = 8 \times 10^{12}$ parameters. 
-   Most parameters will be zero; most possible trigrams will not appear.
-   But does that mean all trigrams we haven't seen are necessarily impossible to *ever* see? No.
-   Worse still, the bigram denominator may be zero, and the ML ratio is undefined.

### Evaluating Language Models: Perplexity

-   We have some test data, $m$ sentences, i.e. $s_1, s_2, s_3, \ldots, s_m$. Each of these is a sentence in the language, e.g. {the dog laughs STOP}.
-   Additionally, assume that use some *development* data to determine the language model parameters, but hold out some additional *test data* to evaluate the language model.
-   Natural to look at the probability that our language model gives to sentences in the test data $\prod_{i=1}^{m}p(s_i)$; it's never seen it before.

$$\textrm{log}\;\prod_{i=1}^{m} p(s_i) = \sum_{i=1}^{m} \textrm{log}\;p(s_i)$$

-   (the above is a basic rule of logarithms; log of product = sum of logs).
-   recall that e.g.:

$$p(s_i) = q(\textrm{the | *, *}) \times q(\textrm{dog | *, the}) \times \ldots$$

-   Naturally we'd expect better languages models to assign higher probabilities to sentences in the test data.
-   And log is a monotonically increasing function, so expect the sum of logs to correspondingly be higher for better language models.

-   In fact, the usual evaluation measure is **perplexity**:

$$\textrm{Perplexity} = 2^{-l},\;\textrm{where}$$
$$l = \frac{1}{M} \sum_{i=1}^{m} \textrm{log}\;p(s_i)$$

-   and M is the total number of *words* in the test data. In some sense with (1/M) the perplexity is now stable with respect to the size of the test data.
-   The *lower* the perplexity the *better the fit* of the language model to the test data.

Some Intuition about Perplexity

-   Say we have vocabulary $V$, and $N = |V| + 1$, and the dumbest possible model predicts:

$$q(w|u,v) = \frac{1}{N},\;\forall\;w \in V \cup \{\textrm{STOP}\},\;\forall\;u,v \in V \cup \{\textrm{*}\}$$.

-   This dumbest model assigns the uniform distribution over all possible words in each possible. Ignores previous words, doesn't measure relative frequency.
-   Easy to calculate perplexity:

$$\textrm{Perplexity} = 2^{-l},\;\textrm{where}\;l=\textrm{log}\;\frac{1}{N}$$
$$\implies\; \textrm{Perplexity} = N$$

-   !!AI implying all these calculations use log base 2.
-   Perplexity is a measure of effective "branching factor".

- - -

Quiz: define a trigram language model with the following parameters:

-   q(the | *, *) = 1
-   q(dog | *, the) = 0.5
-   q(cat | *, the) = 0.5
-   q(walks | the, cat) = 1
-   q(STOP | cat, walks) = 1
-   q(runs | the, dog) = 1
-   q(STOP | dog, runs) = 1

Now consider a test corpus with the following sentences:

-   the dog runs STOP
-   the cat walks STOP
-   the dog runs STOP

Note that the number of words in this corpus, M, is 12.

What is the perplexity of the language model, to 3dp?

$$P = 2^{-l}$$
$$l = \frac{1}{M} \sum \textrm{log}_2\{p(s_i)\}$$

$p(\textrm{the dog runs STOP}) = q(\textrm{the | *, *}) \times q(\textrm{dog | *, the}) \times q(\textrm{runs | the, dog}) \times q(\textrm{STOP | dog, runs})$  
$= 1 \times 0.5 \times 1 \times 1 = 0.5$

$p(\textrm{the cat walks STOP}) = q(\textrm{the | *, *}) \times q(\textrm{cat | *, the}) \times q(\textrm{walks | the, cat}) \times q(\textrm{STOP | cat walks})$  
$= 1 \times 0.5 \times 1 \times 1 = 0.5$

$l = \frac{1}{12} \{ 3 \times \textrm{log}_2(0.5) \}$
$=\frac{1}{12}(-3) = \frac{-1}{4}$  
$p=2^{-\frac{1}{4}} = \sqrt[4]{2} = 1.189\;\textrm{(3dp)}$

- - -

#### Typical values of perplexity (Goodman)

-   $|V| = 50,000$.
-   Trigram model, second-order Markov process, $p(x_1 \dots x_n) = \prod_{i=1}^{n} q(x_i|x_{i-2},x_{i-1})$ gave perplexity = 74.
-   This is vastly smaller than the vocabulary size, so this is vastly superior to the uniform distribution.
-   Bigram model, a first-order Markov process, $p(x_1 \ldots x_n) = \prod_{i=1}^{n}q(x_i|x_{i-1})$ gave perplexity = 137.
-   Unigram model, $p(x_1 \ldots x_n) = \prod_{i=1}^{n} q(x_i)$, gave perplexity = 955.
    -   Predicting each word without using context of previous words.

#### Some history

-   Shannon conducted experiments on entropy of English. See "Prediction and entropy of printed English", 1951.
-   Chomsky, in "Syntactic Structures", 1957
    -   "Colorless green ideas sleep furiously"
    -   "Furiously sleep ideas green colorless"
    -   Argues probability has little to offer for semantic sense and grammatical validity.
    -   Very much against Shannon's experiments with Markov processes and language.
    -   Later in the course we'll look at PCFGs that capture long-range dependencies.

## Week 1 - Parameter Estimation in Language Models

### Linear Interpolation (Part 1)

-   Recall the "Sparse Data Problems" section before.

#### The Bias-Variance Trade-Off

-   Trigram ML estimate

$$q_{ML}(w_i\;|\;w_{i-2},w_{i-1}) = \frac{\textrm{Count}(w_{i-2},w_{i-1},w_i)}{\textrm{Count}(w_{i-2},w_{i-1})}$$

-   Bigram ML estimate

$$q_{ML}(w_i\;|\;w_{i-1}) = \frac{\textrm{Count}(w_{i-1},w_i)}{\textrm{Count}(w_{i-1})}$$

-   Unigram ML estimate

$$q_{ML} = \frac{\textrm{Count}(w_i)}{\textrm{Count}()}$$

-   The trigram MLE's advantage is that it conditions on a lot of context, so given sufficient training data these counts will be high and it will converge to the "true value".
    -   This has **relatively low bias**. It is able to generalise from one particular training set to other unknown data.
-   The unigram MLE completely ignores context, and so it will converge to a less-good estimator as the number of training samples increases.
    -   This has **relatively high bias**.

-   The trigram MLE's disadvantage is that many counts will be equal to zero, so we need many samples to get a good estimate.
    -   This has **relatively high variance**. It needs far more data to be able to generalise; if it has insufficient data it will not learn / generalise.
-   The unigram MLE's count will converge relatively quickly to their expected value, and so don't need many samples.
-   The bigram MLE is in between the trigram MLE and unigram MLE.

### Linear Interpolation (Part 2)

#### Linear Interpolation

-   Take our estimate $q(w_i\;|\;w_{i-2},w_{i-1})$ to be:

$= \lambda_1 \times q_{ML}(w_i\;|\;w_{i-2},w_{i-1})$  
$+ \lambda_2 \times q_{ML}(w_i\;|\;w_{i-1})$  
$+ \lambda_3 \times q_{ML}(w_i)$

-   where $\lambda_1 + \lambda_2 + \lambda_3 = 1$ and $\lambda_i \ge 0\;\forall\; i$.
-   New estimate is a weighted average of the three MLEs.
-   For example, assuming $\lambda_1 = \lambda_2 = \lambda_3 = \frac{1}{3}$

$q(\textrm{laughs | the, dog})$  
$= \frac{1}{3} \times q_{ML}(\textrm{laughs | the, dog})$  
$+ \frac{1}{3} \times q_{ML}(\textrm{laughs | dog})$  
$+ \frac{1}{3} \times q_{ML}(\textrm{laughs})$  

- - -

Quiz: we are given the following corpus:

-   the green book STOP
-   my blue book STOP
-   his green house STOP
-   book STOP

Assume we compute a language model based on this corpus using linear interpolation with $\lambda_i = \frac{1}{3}\;\forall\;i \in \{1,2,3\}$.

What is the value of the parameter $q_{LI}(\textrm{book | the, green})$ in this model to 3dp? (Note: please include STOP words in your unigram model).

$q_{LI}(\textrm{book | the, green})$  
$= \frac{1}{3} \times q_{ML}(\textrm{book | the, green})$  
$+ \frac{1}{3} \times q_{ML}(\textrm{book | green})$  
$+ \frac{1}{3} \times q_{ML}(\textrm{book})$  

$= \frac{1}{3} \times \frac{\textrm{Count(the, green, book)}}{\textrm{Count(the, green)}}$  
$+ \frac{1}{3} \times \frac{\textrm{Count(green, book)}}{\textrm{Count(green)}}$  
$+ \frac{1}{3} \times \frac{\textrm{Count(book)}}{\textrm{Count()}}$  

$= \frac{ \frac{1}{3}(1) }{(1)} + \frac{ \frac{1}{3}(1) }{(2)} + \frac{ \frac{1}{3}(3) }{(14)}$  
$= 0.571\;\textrm(3dp)$  

- - -

Our estimate correctly defines a distribution. Define $V^{'} = V \cup \{STOP\}.$

$\sum_{w \in V^{'}} q(w|u,v)$  
$=\sum_{w \in V^{'}} [\lambda_1 \times q_{ML}(w|u,v) + \lambda_2 \times q_{ML}(w|v) + \lambda_3 \times q_{ML}(w)]$  

move out the constant lambdas:

$=\lambda_1 \sum_w q_{ML}(w|u,v) + \lambda_2 \sum_w q_{ML}(w|v) + \lambda_3 \sum_w q_{ML}(w)$

By definition the maximum likelihood estimates in a given trigram, bigram, or unigram model sum to 1. Intuitively, the probability of each given trigram, bigram, or unigram probability in the model sums to 1.

$= \lambda_1 + \lambda_2 + \lambda_3 = 1$  

(Can also show that $q(w|u,v) \ge 0\;\forall\;w \in V^{'}$).

- - -

Quiz: say we have $\lambda_1 = -0.5, \lambda_2 = 0.5, \lambda_3 = 1.0$. Note that these satisfy the constraint $\sum_i \lambda_i = 1$, but violate the constraint that $\lambda_i \ge 0$.

Recalling our definition of $q$ above within: $\sum_{w \in V^{'}} q(w|u,v)$, it's hence true that there might be a trigram $u,v,w$ such that $q(w|u,v) \lt 0$ or $q(w|u,v) \gt 1$.

It is not true that we may have a bigram $u,v$ such that $\sum_{w \in V} q(w|u,v) \neq 1$.

- - -

#### How to estimate the $\lambda$ values?

-   Hold out part of the training set as "validation" data.
-   Define $c^{'}(w_1,w_2,w_3)$ to be the number of times the trigram $(w_1,w_2,w_3)$ is seen in the validation set.
-   Take some small portion of all of our sentences, say 5%, as validation.
-   We train on the 95% bigger portion.
-   Define $c^{'}$ as the number of times we see the training data in the smaller, other set.
-   Choose $\lambda_1, \lambda_2, \lambda_3$ to maximize:

$$L(\lambda_1,\lambda_2,\lambda_3) = \sum_{w_1,w_2,w_3} c^{'}(w_1,w_2,w_3)\;\textrm{log}\;q(w_3|w_1,w_2)$$

such that $\lambda_1 + \lambda_2 + \lambda_3 = 1$ and $\lambda_i \ge 0\;\forall\;i$ and where:

$q(w_i|w_{i-2},w_{i-1}) =$  
$\lambda_1  \times q_{ML}(w_i|w_{i-2},w_{i-1})$  
$+\lambda_2 \times q_{ML}(w_i|w_{i-1})$  
$+\lambda_3 \times q_{ML}(w_i)$  

-   Many of the $c^{'}(w_1,w_2,w_3)$ counts will of course be zero.
-   Optimization problem to maximize L, under the contraints that the lambdas are positive and sum to one.
-   If you maximize L it is easy to show that you minimize the perplexity of the language model with respect to the validation data.

#### Allowing the $\lambda$'s to vary

-   Take a function $\Pi$ that partitions histories, e.g. for some bigram:

$$
\begin{equation}
    \Pi(w_{i-2},w_{i-1}) = \begin{cases}
        1, & \textrm{If Count}(w_{i-1},w_{i-2}) = 0\\
        2, & \textrm{If 1} \le \textrm{Count}(w_{i-1},w_{i-2}) \le 2\\
        3, & \textrm{If 3} \le \textrm{Count}(w_{i-1},w_{i-2}) \lt 5\\
        4, & \textrm{Otherwise}
    \end{cases}
\end{equation}
$$

-   Introduce a dependence of the $\lambda$'s on the partition:

$$
\begin{align}
    &\begin{aligned}
        q(w_i\;|\;w_{i-2},w_{i-1}) & = \lambda_1^{\Pi(w_{i-2},w_{i-1})} \times q_{ML}(w_i\;|\;w_{i-2},w_{i-1}) \\
        &\; + \lambda_2^{\Pi(w_{i-2},w_{i-1})} \times q_{ML}(w_i\;|\;w_{i-1}) \\
        &\; + \lambda_3^{\Pi(w_{i-2},w_{i-1})} \times q_{ML}(w_i)
    \end{aligned}
\end{align}
$$

-   where $\lambda_1^{\Pi(w_{i-2},w_{i-1})} + \lambda_2^{\Pi(w_{i-2},w_{i-1})} + \lambda_3^{\Pi(w_{i-2},w_{i-1})} = 1$, and $\lambda_i^{\Pi(w_{i-2},w_{i-1})} \ge 0\;\forall\;i$.
-   Instead of just 3 lambdas now we have 3 * 4 = 12 lambdas, one per MLE per partition, and we determine which parition to use based on the bigram count.
    -   We condition on the bigram counts.
    -   $\lambda_1^1, \lambda_2^1, \lambda_3^1$. These counts are used if the bigram count is 0.
    -   $\lambda_1^2, \lambda_2^2, \lambda_3^2$. These counts are used if the bigram count is [1, 2].
    -   $\lambda_1^3, \lambda_2^3, \lambda_3^3$. These counts are used if the bigram count is [3, 5).
    -   $\lambda_1^4, \lambda_2^4, \lambda_3^4$. These counts are used if the bigram count is [5, $\infty$).
-   Partitions are generally chosen by hand, but this one is a typical definition.
-   These 12 lambdas are optimized according to L as before using validation data.
-   If this bigram count is 0 then parameter $\lambda_1$ will also be equal to 0, else it is undefined.
    -   Recall that $\lambda_1$ is for the trigram MLE, and the bigram count is in the denominator.

### Discounting Methods (Part 1)

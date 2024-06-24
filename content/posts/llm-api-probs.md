---
layout: post
title: Another derivation for "Obtaining logprobs from an LLM API"
date: 2024-06-17 01:06:04
tags: 
    - LLM
---

> OpenAI has modified their API to return the log probabilities before any logit bias is applied. Hence the methods described in this article are no longer applicable.

The aim of this article is to provide an alternative derivation of the results in Mattew Finlayson's article [Obtaining logprobs from an LLM API](https://mattf1n.github.io/openlogprobs.html).

Most LLM APIs return the logprobs of only the top-$ k $ predictions. The value of $ k $ is often small, in the order of 10. The goal is to extract the logprobs of all tokens in the vocabulary.
Fortunately, some APIs allow adding a logit bias to tokens.
The idea is to use the logit bias to control which tokens are returned by the API. 

The contribution of this article is to unify the derivations in the original article under a single lemma, providing a clearer and more comprehensible derivation.

[Delete this line and the following lines before publishing the article.]



### Preliminaries
Most language models are trained to predict the next token. They do so by computing logits $ l_i $ for each token $ i $ in the vocabulary. The logits $(l_i)_{i=1}^v$ are then converted to probabilities $ (p_i)_{i=1}^v$ by the softmax function:
$$
p_i = \frac{e^{l_i}}{\sum_{j=1}^v e^{l_j}},
$$
where $ v $ is the size of the vocabulary. The normalizing constant $ Z = \sum_{j=1}^v e^{l_j} $ ensures that the probabilities sum to 1.

**Definition** (Change in logits).
Given the original logits $ ( l_1, l_2, \ldots, l_v) $ and the modified logits $ ( l_1', l_2', \ldots, l_v') $, we define the change in logits as 
$$
( \Delta l_1, \Delta l_2, \ldots, \Delta l_v) = ( l_1' - l_1, l_2' - l_2, \ldots, l_v' - l_v) .
$$

We prove the following lemma which relates the probability distributions, before and after a change in logits.

**Lemma**

For any $ i \in \{1, 2, \dots, v\} $, we have
$$
p_i = p_i' \left(\frac{e^{-\Delta l_i}}{\sum_{j=1}^v p_j' e^{-\Delta l_j}}\right). \tag{1}
$$ 
The significance of the equation is that the original probability $ p_i $ is written in terms of the modified probabilities $(p_j')_{j=1}^v$ and the change in logits $ (\Delta l_j)_{j=1}^v $.
Similarly, we have
$$
p_i' = p_i \left(\frac{e^{\Delta l_i}}{\sum_{j=1}^v p_j e^{\Delta l_j}}\right). \tag{2}
$$

_proof_.
To begin, we write down the probabilities in terms of the logits: 
$$
p_i = \frac{e^{l_i}}{\sum_{j=1}^v e^{l_j}} = \frac{e^{l_i}}{Z} \quad \text{and} \quad p_i' = \frac{e^{l_i'}}{\sum_{j=1}^v e^{l_j'}} = \frac{e^{l_i'}}{Z'},
$$
where $Z$ and $Z'$ are the normalizing constants for the original and modified logits, respectively.

Since $ l_i' - l_i = \Delta l_i $, we have
$$
e^{l_i} = e^{l_i'} e^{-\Delta l_i}.
$$
Thus, we can write the original probability in terms of the modified probabilities and the logit differences:
$$
p_i = \frac{e^{l_i}}{\sum_{j=1}^v e^{l_j}} = \frac{e^{l_i'} e^{-\Delta l_i}}{\sum_{j=1}^v e^{l_j'} e^{-\Delta l_j}}.
$$
Dividing the numerator and the denominator by the normalizing constant $ Z' $, we eliminate the dependence on the logits:
$$
p_i = \frac{(e^{l_i'}/Z') e^{-\Delta l_i}}{\sum_{j=1}^v (e^{l_j'}/Z') e^{-\Delta l_j}} = \frac{p_i' e^{-\Delta l_i}}{\sum_{j=1}^v p_j' e^{-\Delta l_j}}.
$$
The symmetric result of Equation (2) can be obtained similarly.
$ \square $

## Extracting probabilities of any $k$ tokens in a single API call
The following result is used to extract the probabilities of an arbitrary set of $ k $ tokens in a single API call by exploiting the logit bias option of the API.
We assume that the API returns the probabilities of the top-$ k $ tokens, and allows adding logit biases to the tokens.

We denote the set of indices of the desired $k$ tokens as $ B $.  To expose the probabilities of the tokens in $ B $, we add a sufficiently large logit bias $ b $ to the tokens in $ B $ so that they appear in the top-$ k $ predictions. 
(If not all desired tokens appeared in the top-$k$ predictions from the API call, we can try again with a larger bias.)

After getting the results from the API, we know the probabilities $ p_i' $ for $ i \in B $ but not for $ i \notin B $.
The aim is to find the original probabilities $ p_i $ for $ i \in B $.

In this case, the changes in logits are
$$
\Delta l_i = l_i' - l_i = \begin{cases}
b & \text{if } i \in B, \\
0 & \text{if } i \notin B.
\end{cases}
$$

By Equation (1) in the lemma, we have for any $i \in B$:
$$
p_i = \frac{p_i' \exp(-\Delta l_i)}{\sum_{j=1}^v p_j' \exp(-\Delta l_j)}.
$$
We can split the summation in the denominator into two parts: one for $ j \in B $ and the other for $ j \notin B $:
$$
p_i =  \frac{p_i' \exp(-b)}{\sum_{j \in B} p_j' \exp(-b) + \sum_{j \notin B} p_j'}.
$$

<!-- To eliminate this dependence, we can use the fact that the probabilities sum to 1: -->
To eliminate the dependence on unknown probabilities, we use the fact that the probabilities sum to 1:
$$
\sum_{i \in B} p_i + \sum_{i \notin B} p_i = 1,
$$
which gives
$$
p_i = \frac{p_i' \exp(-b)}{\sum_{j \in B} p_j' \exp(-b) + 1 - \sum_{j \in B} p_j'}.
$$

Here we have obtained a formula for the original probabilities $ p_i $ for $ i \in B $ in terms of only observed probabilities and the bias $ b $ known to us. Hence we can extract the probabilities of the $k$ tokens in $ B $ in a single API call.

### Full probability distribution in $ v / k$ API calls
To extract the full probability distribution, we can repeat the above process $ \lceil v/k \rceil $ times, each time exposing the probabilities of another $ k $ tokens. 

## Extracting the top-$n$ probabilities in $ \lceil n/k \rceil $ API calls
In most cases, the probability mass is concentrated on a small number of tokens. It is more cost-effective to extract just the top-$n$ probabilities instead of the full distribution. We can assume the rest of the probabilities to be negligible.

To this end, we apply a sufficiently large negative bias $ -b $ to the tokens with already known probabilities, exposing the next $k$ tokens. Repeat the process until the top-$ n $ logprobs are obtained. The required number of API calls is $ \lceil n/k \rceil $.

Denote the set of indices of the known tokens as $ C $. In this case the changes in logits are
$$
\Delta l_i = \begin{cases}
-b & \text{if } i \in C, \\
0 & \text{otherwise}.
\end{cases}
$$

Using Equation (2) of the lemma, we have for any $ i \notin C $:
$$
% p_i' = \frac{p_i \exp(\Delta l_i)}{\sum_{j=1}^v p_j \exp(\Delta l_j)} = \frac{p_i}{\sum_{j \in C} p_j \exp(-b) + \sum_{j \notin C} p_j}.
p_i' = p_i \left(\frac{e^{\Delta l_i}}{\sum_{j=1}^v p_j e^{\Delta l_j}}\right).
$$
Splitting the summation in the denominator into two parts: one for $ j \in C $ and the other for $ j \notin C $, we get
$$
p_i' = p_i \left(\frac{e^{-b}}{\sum_{j \in C} p_j e^{-b} + \sum_{j \notin C} p_j}\right). 
$$

Recall that $ C $ is the set of known indices, $ p_j $ where $ j \notin C $ are unknown to us. To eliminate the dependence on unknown probabilities, we use the fact that the probabilities sum to 1:
$$
\sum_{i \in C} p_i + \sum_{i \notin C} p_i = 1,
$$
which gives
$$
p_i' = p_i \left( \frac{e^{-b}}{\sum_{j \in C} p_j e^{-b} + 1 - \sum_{j \in C} p_j} \right).
$$
Rearranging the terms, we get
$$
p_i = p_i' \left( \frac{\sum_{j \in C} p_j e^{-b} + 1 - \sum_{j \in C} p_j}{e^{-b}} \right).
$$
which gives a formula for the original probabilities $ p_i $ for $ i \notin C $ in terms of only observed probabilities, except $p_i'$ which is only known to us if it is one of the next $ k $ tokens exposed by the API.

To extract the top-$ n $ probabilities, we can repeat the above process $ \lceil n/k \rceil $ times, each time exposing the probabilities of another $ k $ tokens.
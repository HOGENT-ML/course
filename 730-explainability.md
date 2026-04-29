---
theme : "night"
transition: "slide"
highlightTheme: "monokai"
logoImg: "logo.png"
slideNumber: true
title: "VSCode Reveal intro"
---
<style>
.reveal {
  font-size: 24px;
}

li
{
    margin: 1.5em 0;
}
</style>

# Explainability - Interpretability

--

## 1. Introduction

- A subfield of machine learning that tries to make models more understandable by humans. 
- Often opposed to the concept of "black box models" where it is hard to justify why a model makes a certain prediction. 
- This field is referred to as **explainable artificial intelligence (XAI)**. 

--

- This interest comes from the fact that machine learning is becoming an integral part of a lot of decision support systems.  
- As this importance keeps increasing, questions arise about the justification that can be obtained regarding these models, and researchers start to focus on more interpretable models.

--

- for certain problems, getting the prediction from our models is not enough, an explanation is also required.
- In fraud detection for example, models are usually complex. Mostly, human investigators will process the machine-learned alarms. Knowing why a transaction leads to an alarm is clearly an added value. 

--

Interpretability also provides helpful tools for 
- debugging,
- increasing model acceptance,
- providing justifications during audits.

---

### 1.1 Taxonomy
#### 1.1.1 Global vs. local explainability/interpretability

A first property of interpretabilty methods is their scope: 
- **global methods** focus on the model level: "What features contribute to the model?"
- **local methods** focus on a specific prediction (for example a transaction) or a group of predictions: "How do the values of the features influence the model prediciton for a particular observation?"
- Both are equally important.

---

#### 1.1.2 Pre-Model vs. In-Model vs. Post-Model

Interpretability methods can take place 
- before (pre-model)
- during (in-model)
- after (post-model) 

building the ML model.

---

**Pre-model interpretability** techniques are 
- independent of the model
- take place before model selection (during the data exploration phase): They focus on the data level. 
- For example, carefully selecting a low number of intuitive features can help to achieve data interpretability. As this approach requires more a-priori knowledge and is problem dependant, it was less studied than the other two.

--

The most popular pre-model techniques are visualization methods: 
- PCA (Principal Component Analysis)
- t-SNE (t-Distributed Stochastic Neighbor Embedding)
- UMAP (Uniform Manifold Approximation and Projection for Dimension Reduction)
- clustering methods.

All except UMAP are discussed in the chapters about unsupervised learning.

---

**In-model interpretability** 
- models that are already sufficiently explainable by themselves: the so-called **white-box models**. 
- Tree-based classifiers, for example are easily interpretable (ref. Gini feature importance (Global)).
- regression models, for example the Lasso regression model. 

--

**Post-model interpretability** 
- often built on top of a black-box model
- a post-hoc explainer is added to improve the interpretability of the black-box model. 
- It allows, for instance, the interpretation of models that are already running in production. 

---

#### 1.1.3 Model-specific vs. model-agnostic

Can a post-model interpreter handle all possible ML models? 

- If no, this interpreter is said to be **model-specific**.
- Otherwise, it is said to be **model-agnostic**.

--

- Model-specific interpretations are limited to certain model classes because they are based on some specific model’s internals  
- For example, SHAP TreeExpainer is only devoted to decision trees-based algorithms, as it uses their properties to build its post-hoc explainer.
- Also, a large part of model-specific interpreters is devoted to Deep Neural Networks and computer vision (for example "saliency mapping")

--

On the contrary, model-agnostic methods 
- can be applied to any ML model (black box or not)
- are applied after the model has been trained (post hoc)
- rely on analyzing pairs of features' input and output. 
- Usually, model-agnostic explainers are also post-models, sometimes referred to as surrogate models.

--

- Suppose a black box model is too complex to be interpreted. 
- In that case, a workaround solution can be to build a white box model, called the surrogate model, which tries to globally approximate the black-box model as faithfully as possible. It is then easier to interpret the surrogate model. 

---

#### 1.1.4 white box models and black box models

**white box models**

- like logistic regression models and decision trees
- outcome easy to understand by decision makers
- used in credit risk modelling, medical diagnosis

--

**black box models**

- like random forests and neural networks
- more complex models
- the focus is on estimating the target as accurately, without necessarily wondering how the model came to that prediction
- example: fraud detection, response modeling

---

## 2. Global explainability
 
Different techniques exist to estimate which features are important according to the machine learning model:
 
### 2.1 Inspecting the mean decrease in impurity

- comes directly from tree-based models such as decision trees, random forests, and gradient boosted trees. 
- Each time a feature is used to split a node, the model reduces an impurity criterion such as the Gini impurity or the entropy. 
- The total reduction in impurity produced by a feature, averaged over all trees, is used as its importance score.

--

- gives a quick global view of which variables were most useful to build the trees.
- is model-specific
- can be biased
    - features with many possible split points, such as continuous variables or high-cardinality categorical variables, can receive artificially high importance. 
    - In addition, if two features carry similar information, the model may split on one of them more often and underestimate the importance of the other.

See `feature_importances_` member variable for tree-based models. 

---

### 2.2 Drop feature importance

- also called drop-column importance
- measures how much model performance decreases when one feature is completely removed from the training process. 

--

The procedure is simple:

1. Train a reference model with all features.
2. Measure its predictive performance on a validation set.
3. Remove one feature from the dataset.
4. Retrain the model from scratch.
5. Measure the new performance.
6. The performance loss indicates the importance of the removed feature.

-- 

- intuitive because it answers a practical question: how much do we lose if this variable is not available at all?

- Its main drawback is computational cost. 
    - A new model must be trained for every feature
    - becomes expensive for large datasets or complex models
    - when predictors are strongly correlated, dropping one feature may have only a small impact because the remaining correlated features can compensate for it.

---

### 2.3 Permutation based feature importance

- model-agnostic alternative. 
- After a model has been trained, the values of one feature are randomly shuffled in a validation or test set while all other columns stay unchanged. 
- This destroys the relationship between that feature and the target. 
- If model performance drops strongly, the feature was important. 
- If performance changes little, the feature was less important for the model.

--

- Compared with drop-column importance, permutation importance is much faster because the model does not need to be retrained. 
- It can be applied to almost any predictive model.
- Its main limitation is that the shuffled dataset may contain unrealistic combinations of feature values, especially when variables are correlated. 
- In that case, the measured importance can be misleading. 
- The result also depends on the chosen evaluation metric and on the dataset used for the permutation.

---

### 2.4 Partial dependence plots

- Partial dependence plots (PDPs) do not only tell us whether a feature is important, but also how that feature influences the model prediction on average. 
- The idea is to vary one feature, or a pair of features, over a range of values while averaging the model predictions over all other features in the dataset.
- The resulting curve shows the global relationship learned by the model. 
- For example, a PDP can reveal whether the predicted risk increases linearly with age, reaches a plateau, or changes only after a threshold.

--

- PDPs are especially useful to interpret nonlinear models. 
- They provide a visual explanation of the direction and shape of the effect of a feature. We explore this method in the next notebook. 

--

- The main limitation: PDPs average over the full dataset and therefore hide individual differences. 
- They can become unreliable when features are strongly correlated, because the plot may evaluate feature combinations that are rare or impossible in the real data.

---

## 3. Local explainability
- 3.1 Individual conditional expectation (ICE) plots  
    - display one line per instance that shows how the instance’s prediction changes when a feature changes.
- 3.3 Shapley values
    - See further

## 4. reading (optional)
https://christophm.github.io/interpretable-ml-book/pdp.html

















 
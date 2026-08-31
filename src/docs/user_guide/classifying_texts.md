# Classifying Texts

The Lexos `classification` module provides a public `Classifier` API for accessing more complex backend logic. The `Classifier` takes input data, labels, and a classification pipeline. The pipeline implements different classification strategies with settings specific to each backend. Lexos ships with two built-in pipelines: `SpaCyTextCategorizerPipeline` wraps spaCy's text categorizer and `SklearnClassifierPipeline` wraps scikit-learn's classification models.

Typically, you will begin your classification workflow by configuring a pipeline and creating a `Classifier` instance. Here is an example usage:

```python
from lexos.classification import Classifier, SpaCyTextCategorizerPipeline

# Configure a spaCy TextCategorizer Pipeline with options
pipeline = SpaCyTextCategorizerPipeline(
    language="en",
    epochs=10,
)

# Create a Classifier instance with texts, labels, and pipeline
classifier = Classifier(
    data=texts, # A list of text strings or spaCy Docs
    labels=labels, # A list of labels corresponding to the data items
    pipeline=pipeline,
)
```

Next, you will split your data into training and test sets.

```python
# Split the data into training and test sets
split = classifier.train_test_split(test_size=0.2, random_state=42)
```

The `test_size` parameter specifies the proportion of the dataset to include in the test split, and `random_state` ensures reproducibility of the split. In the example above, we set `test_size` to `0.2`, meaning 20% of the data will be used for testing. If we need to reproduce the exact split, we should use the same `random_state` value.

Now we will train ("fit") the model using the training data and then make predictions on the test data.

```
# Train the model
classifier.fit(
    data=split["train"]["data"],
    labels=split["train"]["labels"],
)

# Predict labels for the test data
predictions = classifier.predict(split["test"]["data"])
```

The example above assumes that there is one label per document. Multiple label classification is also supported and will be discussed below.

Each document's active categories are converted internally to the spaCy `cats` score map where every positive label is set to `1.0` and all others remain `0.0`. Predicted categories are passed back to the pipeline. This allows you to pass a new text to that pipeline's model to see the predicted categories for that text:

```python
doc = pipeline.model("Some text from outside the training data.")
```

---

## `Classifier` Constructor Options

- `data`: Normally a list of text strings or spaCy `Docs`, but a list containing a list of tokens or a Lexos `DTM` object can also be used.
- `labels`: For single-label classification (one label per document), a list of strings. For multiple labels per document, the input should be a list of lists with one list per document.
- `pipeline`: An instance of `BaseClassificationPipeline`. Lexos ships with one of two possibilities: `SpaCyTextCategorizerPipeline` or `SklearnClassifierPipeline`.

By default, when `data` contains spaCy `Doc` objects or a Lexos `DTM`, the classifier preserves the document's tokenization. Otherwise, the text will be tokenized on the fly using the logic defined in the pipeline. Pipeline objects can be instantiated without configuration, but their settings can also be configured, as discussed below.

## The SpaCy Backend

The spaCy backend is invoked by supplying an instance `SpaCyTextCategorizerPipeline`. It has the following configuration settings:

- `language`: If the model is not already available, spaCy will create a blank language object for that setting. This mainly determines which tokenizer and language pipeline are used for text preprocessing if the data is supplied as a wrong string. The default language is English ("en").
- `nlp`: This setting allows you to pass a specific spaCy language model like "en_core_web_sm". Note that you must first load the model with something like `nlp = spacy.load("en_core_web_sm")`, then specify it with `SpaCyTextCategorizerPipeline(nlp=nlp)`. If omitted, the class creates a blank spaCy model for the selected language.
- `exclusive_classes`: Controls whether a document is treated as single-label or multi-label. If set to `True` (the default), the spaCy component uses the single-label `textcat` pipeline. If set to `False`, it uses the multi-label `textcat_multilabel` pipeline. In normal use, Lexos automatically switches to multi-label mode when the labels are supplied as a list of lists, so you usually do not need to set this manually.
- `score_ranking`: Controls how output labels are ranked for multi-label predictions. Use `"document"` (default) to rank labels within each document, or `"global"` to rank from the overall score distribution across the full batch.
- `architecture`: Specifies a bag-of-words categorizer ("bow"), a convolutional text categorizer ("cnn"), an "ensemble" categorizer, or a dictionary containing custom architecture settings. See below for further notes on architecture configuration.
- `epochs`: Sets how many training passes the spaCy model performs over the examples. Higher values can improve performance, but also increase runtime. Small datasets often benefit from a modest number of epochs. The default is 10.

### SpaCy `TextCategorizer` Architectures

SpaCy's `TextCategorizer` supports a number of different classification models. Supported string values in the current Lexos wrapper are:

- `"bow"` — linear bag-of-words text categorizer
- `"cnn"` — convolutional text categorizer using spaCy's CNN reduce configuration
- `"ensemble"` — spaCy's ensemble-style text categorizer configuration

SpaCy’s ensemble-style text categorizer is a stacked ensemble model architecture which combines the other two types. It performs well over different data lengths and is good for short texts. `"bow"` is the simplest and most interpretable choice for small Lexos experiments and teaching examples. `"cnn"` and `"ensemble"` are useful when you want a more expressive model, but they often require more memory and longer training time.

Lexos uses preset configurations when setting up the classification architecture, but you can modify them by passing a custom spaCy architecture dictionary. The dictionary is validated and merged with Lexos defaults such as `exclusive_classes` and `no_output_layer` if they are not already supplied. Here is an example:

```python
custom_pipeline = SpaCyTextCategorizerPipeline(
    exclusive_classes=False,
    architecture={
        "@architectures": "spacy.TextCatBOW.v3",
        "exclusive_classes": False,
        "ngram_size": 2,
        "length": 262144,
    },
)
```

For information on spaCy text classification architectures and their settings, see [https://spacy.io/api/architectures#textcat](https://spacy.io/api/architectures#textcat){target="_blank"}.

### Multi-Label Classification

When a document can legitimately belong to more than one category, provide labels as a list of lists, where each sublist contains the categories for one document. Lexos automatically detects this pattern and switches the spaCy backend to `textcat_multilabel` without requiring any manual `exclusive_classes` configuration. The `exclusive_classes` field is retained as an explicit override when you want to force single-label behavior, but most users can leave it at its default value.

Here is an example of a multi-label classification setup:

```python
texts = [
    "medieval cathedral architecture and stonework",
    "industrial factories and workers",
    "medieval history and political change",
]
labels = [
    ["history", "architecture"],
    ["economics", "industry"],
    ["history", "politics"],
]

pipeline = SpaCyTextCategorizerPipeline()

classifier = Classifier(data=texts, labels=labels, pipeline=pipeline)
classifier.fit(data=texts, labels=labels)
```

### Inspecting Raw Scores

The public classifier API exposes `predict_scores()` so you can inspect the underlying confidence values before the final label selection step. Scores are ranked according to the configured `score_ranking` mode, and the returned labels reflect the selected top-scoring categories for each document.

```python
scores = classifier.predict_scores(texts)
```

This is especially helpful for multi-label tasks when you want to see which labels are strongly activated for a document and how the rank order is decided.

!!! note "How Score Ranking Works"
    The `score_ranking` parameter determines how the predicted labels are ordered for multi-label classification tasks.
    - `"document"`: Ranks labels within each document based on their confidence scores. This is the default behavior.
    - `"global"`: Ranks labels according to their scores across the entire batch of predictions.

    Document ranking is the default and generally aligns better with human intuition for multi-label classification tasks. Each document gets its own top labels, so it preserves the document-specifi signal and avoids corpus-wide bias from dominaing a single document's prediction.

    Global ranking, on the other hand, can be useful for understanding label importance across the entire dataset. It favors labels that are consistently strong across the dataset and is useful when you want corpus-level priors to dominate. However, it can sometimes suppress labels that are important for individual documents, thereby obscuring document-specific nuances.

---

## The scikit-learn Backend

Lexos also includes a `SklearnClassifierPipeline`, which allows you to use scikit-learn vectorizers and estimators for textual classification. It has the following configuration settings:

- `vectorizer`: Optional text vectorizer to transform raw text before fitting. The default `TfidfVectorizer` is not mandatory: users can pass a custom scikit-learn `vectorizer`, or provide a precomputed Lexos `DTM` whose weighting has already been configured (that will override the default).
- `estimator`: Underlying scikit-learn estimator (for example `RandomForestClassifier` or `LinearSVC`).
- `max_iter`: Maximum number of optimization iterations for iterative solvers such as `LogisticRegression` and `LinearSVC`. An iterative solver repeatedly updates the model coefficients until it converges or reaches this limit. The default is 1000.
- `score_ranking`: Controls the ordering of labels for multi-label predictions. Use `"document"` (default) to rank labels within each document, or `"global"` to rank labels according to their scores across the full prediction batch.
- `multi_label_wrapper`: Optional factory that wraps a base estimator for multi-label tasks. Receives a base estimator and returns a multi-label-compatible estimator.

Here is an example of basic usage:

```python
from lexos.classification import Classifier, SklearnClassifierPipeline

pipeline = SklearnClassifierPipeline()
classifier = Classifier(data=texts, labels=labels, pipeline=pipeline)
classifier.fit(data=texts, labels=labels)
predictions = classifier.predict(texts)
```

!!! note "Technical Note"
    Internally, `SklearnClassifierPipeline` performs single-label tasks using [`TfidfVectorizer`](https://scikit-learn.org/stable/modules/generated/sklearn.feature_extraction.text.TfidfVectorizer.html#tfidfvectorizer){target="_blank"} for feature extraction and [`LogisticRegression`](https://scikit-learn.org/stable/modules/generated/sklearn.linear_model.LogisticRegression.html){target="_blank"} for classification. However, `TfidfVectorizer` can be overridden by configuring a different vectorizer or passing the data as a Lexos `DTM`. Multi-label taks use [`MultiLabelBinarizer`](https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.MultiLabelBinarizer.html){target="_blank"} to encode the labels and [`OneVsRestClassifier`](https://scikit-learn.org/stable/modules/generated/sklearn.multiclass.OneVsRestClassifier.html){target="_blank"} (with a default `LogisticRegression` base estimator) for classification.

### Weighting Through a DTM vs a scikit-learn Vectorizer

A Lexos `DTM` can also be used to control the feature weighting before the classifier is called. When you build a `DTM` with parameters such as `tf_type`, `idf_type`, `dl_type`, or `norm`, you are choosing the upstream weighting scheme in advance. If you then pass the fitted DTM to `Classifier` without setting a `vectorizer` in `SklearnClassifierPipeline`, the pipeline will consume the already weighted matrix directly and not apply a new scikit-learn vectorizer at prediction time.

This is functionally similar to setting a custom scikit-learn vectorizer on the pipeline, but the weighting is fixed earlier in the `DTM`-building step rather than inside the classifier itself. In other words: a `DTM` gives you a way to bake the normalization policy into the feature matrix, while a custom scikit-learn `vectorizer` lets you choose the tokenization and weighting policy inside the classifier backend.

!!! note "Tokenization and Doc Preservation"
  If the input data consists of spaCy `Doc` objects, or a Lexos `DTM` carrying spaCy docs, the scikit-learn backend preserves the original spaCy tokenization by default. However, if a scikit-learn vectorizer is configured explicitly, the text is passed to that vectorizer and the tokenization policy is effectively overridden by the scikit-learn pipeline. You should supply a custom vectorizer only when you want to replace the spaCy-derived tokenization with a different downstream tokenization strategy.

### Single- and Multi-Label Training

This backend supports both single-label and multi-label training.

Example multi-label usage:

```python
texts = [
    "medieval architecture and history",
    "industrial labor and economics",
]
labels = [
    ["history", "architecture"],
    ["economics", "industry"],
]

classifier = Classifier(
    data=texts,
    labels=labels,
    pipeline=SklearnClassifierPipeline(),
)

classifier.fit(data=texts, labels=labels)
print(classifier.predict(texts))
```

When multi-label predictions are returned, the pipeline keeps each document's predicted labels as a Python list of strings. This makes the user-facing output easier to read and matches the underlying per-document label structure used during training and scoring.

### Using a Specialized Estimator

You can swap in a different scikit-learn estimator by passing it to `SklearnClassifierPipeline` when you construct the pipeline. The following examples use a `RandomForestClassifier` and then substitute a linear SVM for `LogisticRegression`.

```python
from sklearn.ensemble import RandomForestClassifier
from sklearn.svm import LinearSVC

# Single-label example: use a Random Forest instead of LogisticRegression
rf_pipeline = SklearnClassifierPipeline(
    vectorizer=TfidfVectorizer(ngram_range=(1, 2)),
    estimator=RandomForestClassifier(
        n_estimators=200,
        random_state=42,
    ),
)

# Single-label example: use a linear SVM instead of LogisticRegression
svc_pipeline = SklearnClassifierPipeline(
    vectorizer=TfidfVectorizer(),
    estimator=LinearSVC(C=1.0),
)

classifier = Classifier(data=texts, labels=labels, pipeline=rf_pipeline)
classifier.fit(data=texts, labels=labels)
```

The `vectorizer` controls how raw text is converted into features, and the `estimator` controls the actual classification model. This is the cleanest way to plug in a specialized model for ordinary single-label tasks.

!!! note
    Choosing a custom estimator is task specific. For literary, historical, and archival text analysis, the most useful classifiers are usually the ones that handle sparse, high-dimensional text features well. A practical shortlist would include:

    - [`LogisticRegression`](https://scikit-learn.org/stable/modules/generated/sklearn.linear_model.LogisticRegression.html){target="_blank"} (easy to interpret and good for genre, period, authorship, topic, and sentiment tasks)
    - [`LinearSVC`](https://scikit-learn.org/stable/modules/generated/sklearn.svm.LinearSVC.html){target="_blank"} (good for large corpora with sparse feature space)
    - [`MultinomialNB`](https://scikit-learn.org/stable/modules/generated/sklearn.naive_bayes.MultinomialNB.html){target="_blank"} (fast and simple, particularly useful for short texts or smaller exploratory corpora)
    - [`ComplementNB`](https://scikit-learn.org/stable/modules/generated/sklearn.naive_bayes.ComplementNB.html){target="_blank"} (good alternative to MultinomialNB for texts of different lengths)
    - [`OneVsRestClassifier`](https://scikit-learn.org/stable/modules/generated/sklearn.multiclass.OneVsRestClassifier.html){target="_blank"} with LogisticRegression or LinearSVC (best choice for multi-label DH tasks such as multiple genres, themes, or subject tags per document)

### Custom Multi-label Wrappers (Advanced Usage)

For multi-label data, the default behavior is to build a `OneVsRestClassifier` around a logistic regression base estimator. If you want to customize that wrapper, you can pass a configured estimator with the `multi_label_wrapper`.

```python
from sklearn.ensemble import RandomForestClassifier

multi_pipeline = SklearnClassifierPipeline(
    vectorizer=TfidfVectorizer(),
    multi_label_wrapper=lambda base_estimator: OneVsRestClassifier(
        RandomForestClassifier(n_estimators=200, random_state=42)
    ),
)
```

## Saving and Loading Trained Models

Both built-in backend pipelines expose `save()` and `load()` methods, so you can persist a trained classifier and reuse it later without re-fitting the model. This is useful for repeated experiments, long-running training jobs, and deployment pipelines where you want a trained model to be reusable across sessions.

The `save()` and `load()` methods live on the pipeline object itself and preserve both the fitted model state and the configuration used to create it.

!!! tip "When to Save a Model"
    Use `save()` when you want to reuse a trained model later without retraining, especially when you are working with a fixed corpus or when the model will be applied to new texts across multiple sessions. Retraining is still the right choice when your labels, training data, or pipeline configuration have changed and you want to update the model to reflect the newest source material.

Here are some examples:

```python
from lexos.classification import SpaCyTextCategorizerPipeline

pipeline = SpaCyTextCategorizerPipeline(
    language="en",
    epochs=10,
    score_ranking="document",
)

# fit the model as usual
classifier = Classifier(data=texts, labels=labels, pipeline=pipeline)
classifier.fit(data=texts, labels=labels)

pipeline.save("./saved_models/my_spacy_pipeline")
restored = SpaCyTextCategorizerPipeline.load("./saved_models/my_spacy_pipeline")
print(restored.predict(texts))
```

The same pattern works for the scikit-learn backend:

```python
from lexos.classification import SklearnClassifierPipeline

pipeline = SklearnClassifierPipeline(
    max_iter=2000,
    score_ranking="document",
)

classifier = Classifier(data=texts, labels=labels, pipeline=pipeline)
classifier.fit(data=texts, labels=labels)

pipeline.save("./saved_models/my_sklearn_pipeline")
restored = SklearnClassifierPipeline.load("./saved_models/my_sklearn_pipeline")
print(restored.predict(texts))
```

Lexos saves the backend configuration together with the trained model. For spaCy, this means the textual classification component configuration and model weights are preserved on disk. For scikit-learn, the fitted estimator, vectorizer, label encoder, and pipeline settings are serialized together. In both cases, reloading restores the same classification behavior and ranking mode without requiring the original training script.

!!! important
    For the spaCy backend, use a directory path like `my_model/`. The scikit-learn backend saves a single `joblib` file. No file extension is not required, but common extensions are `.joblib` and `.pkl`.

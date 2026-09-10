
# Mallet

Topic modeling is a statistical method for discovering abstract themes or "topics" within a collection of documents. MALLET is a mature tool for topic modeling used widely in the Humanities. It is a Java package that needs to be installed separately from Lexos. The Lexos `mallet` module provides a straightforward wrapper for running MALLET, managing outputs, and creating visualizations of your topic model.

The current public API is exported from `lexos.topic_modeling.mallet` and includes both the Java-backed `Mallet` implementation and the optional `PyRMallet` backend.

## Public API

### ::: lexos.topic_modeling.mallet.MALLET_BINARY_PATH
    rendering:
      show_root_heading: true
      heading_level: 3

### ::: lexos.topic_modeling.mallet.read_file
    rendering:
      show_root_heading: true
      heading_level: 3

### ::: lexos.topic_modeling.mallet.read_dirs
    rendering:
      show_root_heading: true
      heading_level: 3

### ::: lexos.topic_modeling.mallet.import_files
    rendering:
      show_root_heading: true
      heading_level: 3

### ::: lexos.topic_modeling.mallet.import_docs
    rendering:
      show_root_heading: true
      heading_level: 3

## Model classes

### ::: lexos.topic_modeling.mallet.Mallet
    rendering:
      show_root_heading: true
      heading_level: 3

### ::: lexos.topic_modeling.mallet.PyRMallet
    rendering:
      show_root_heading: true
      heading_level: 3

## Model workflow methods

### ::: lexos.topic_modeling.mallet.Mallet.import_data
    rendering:
      show_root_heading: true
      heading_level: 3

### ::: lexos.topic_modeling.mallet.Mallet.train
    rendering:
      show_root_heading: true
      heading_level: 3

### ::: lexos.topic_modeling.mallet.Mallet.infer
    rendering:
      show_root_heading: true
      heading_level: 3

### ::: lexos.topic_modeling.mallet.Mallet.get_keys
    rendering:
      show_root_heading: true
      heading_level: 3

### ::: lexos.topic_modeling.mallet.Mallet.get_top_docs
    rendering:
      show_root_heading: true
      heading_level: 3

### ::: lexos.topic_modeling.mallet.Mallet.get_topic_term_probabilities
    rendering:
      show_root_heading: true
      heading_level: 3

### ::: lexos.topic_modeling.mallet.Mallet.plot_termite
    rendering:
      show_root_heading: true
      heading_level: 3

## Backend implementation notes

The default `Mallet` class uses the Java MALLET backend, while the optional `PyRMallet` backend is exposed as `PyRMallet` and can be selected by passing `backend="pyrmallet"` to `Mallet` or by instantiating `PyRMallet` directly.

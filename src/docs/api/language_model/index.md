# Language Model

The `language_model` module is a wrapper around spaCy's training workflow for fine-tuning language models on custom corpora. It handles directory setup, config generation (including fine-tuning via component sourcing and transformer-based training via recipes), data conversion, training, evaluation, and packaging without requiring the user to edit config files or use the command line.

For a user-friendly overview, see [Training Language Models](../../user_guide/language_model/index.md) in the User Guide. For hands-on walkthroughs, see the tutorial notebooks listed in [Tutorials](../../tutorials/index.md).

## Constants

### ::: lexos.language_model.FULL_UD_PIPELINE

    rendering:
      show_root_heading: true
      heading_level: 3

## CONLL-U Utilities

Standalone functions for preparing and managing CONLL-U training data.

### ::: lexos.language_model.split_conllu

    rendering:
      show_root_heading: true
      heading_level: 3

### ::: lexos.language_model.export_to_conllu

    rendering:
      show_root_heading: true
      heading_level: 3

### ::: lexos.language_model.combine_conllu

    rendering:
      show_root_heading: true
      heading_level: 3

## The LanguageModel Class

The main entry point for the module is `LanguageModel`, a Pydantic-based configuration object that manages the model directory, training config, and workflow lifecycle. It is intended to be used as follows:

```python
from lexos.language_model import LanguageModel

model = LanguageModel(
    model_dir="./model",
    lang="en",
    gpu=False,
    components=["tok2vec", "tagger", "morphologizer", "trainable_lemmatizer", "parser"],
)
model.copy_assets(train="train.conllu", dev="dev.conllu")
model.convert_assets()
model.train()
```

Public workflow methods exposed by the class include:

- `LanguageModel.copy_assets()`
- `LanguageModel.convert_assets()`
- `LanguageModel.validate()`
- `LanguageModel.train()`
- `LanguageModel.evaluate()`
- `LanguageModel.package()`
- `LanguageModel.config_path`
- `LanguageModel.save_config()`
- `LanguageModel.load_config()`

This page intentionally avoids rendering the full Pydantic class with `mkdocstrings` because the generated schema for the model field defaults is not currently compatible with the installed `griffe-pydantic` template stack. The public methods above are the stable contract for the training workflow.

## Debugging Utilities

Wrappers around spaCy's debugging commands for inspecting a model's config and data before training.

### ::: lexos.language_model.debug_config

    rendering:
      show_root_heading: true
      heading_level: 3

### ::: lexos.language_model.debug_data

    rendering:
      show_root_heading: true
      heading_level: 3

### ::: lexos.language_model.debug_model

    rendering:
      show_root_heading: true
      heading_level: 3

### ::: lexos.language_model.fill_config

    rendering:
      show_root_heading: true
      heading_level: 3

## Internal Helpers

The following helpers are used internally by the training workflow and are not part of the public API contract:

- `_has_nvidia_gpu()`
- `_get_tok2vec_width()`
- `_patch_tok2vec_width()`

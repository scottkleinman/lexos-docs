# Installation

The information below describes how to install Lexos as a user. If you are interested in contributing to the Lexos source code or documentation, see the separate [Development](development/index.md) documentation.

## Installing Python

Lexos requires Python 3.12 or greater. Our development environment is [`uv`](https://docs.astral.sh/uv/){target="_blank"}, and Lexos should work in a Python virtual environment created using that tool. If you are using a different Python environment, you can install Lexos using `pip`.

## Installing the Lexos Package

### Install with `uv`:

In your project folder, run the following command in your terminal:

```bash
uv add lexos
```

### Install with `pip`:

You can install Lexos globally or in a virtual environment by running the following command in your terminal:

```bash
pip install lexos
```

By default, `uv` installs the latest version of Lexos. To update to the latest version with `pip`, use

```bash
pip install -U lexos
```

## Download Language Models

Many features of Lexos use language models created for the Python [`spaCy`](https://spacy.io/){target="_blank"} natural language processing library.

!!! note
    For information on how Lexos uses language models, see [Tokenizing Texts](user_guide/tokenizing_texts.md).

To ensure all features work correctly, it is recommended to download spaCy's multi-language model [`xx_sent_ud_sm`](https://spacy.io/models/xx#xx_sent_ud_sm){target="_blank"} and, if possible, a small model for your chosen language (e.g. [`en_core_web_sm`](https://spacy.io/models/en#en_core_web_sm){target="_blank"}) *before* using Lexos. If you try to use a Lexos module that requires a language model without performing this step, Lexos will attempt to download `xx_sent_ud_sm` model automatically.

To download the language models in advance, run the following additional command:

```bash
uv run download-spacy-models xx_sent_ud_sm en_core_web_sm
```

or

```bash
python download-spacy-models xx_sent_ud_sm en_core_web_sm
```

You can download as many models as you want by separating their names with spaces.

The `xx_sent_ud_sm` model is a minimal model that can be used for sentence and token segmentation in a variety of languages, while the `en_core_web_sm` model is specifically for English text. If you are working in another language or need a larger language, you may need to download additional language models. You can find information on available models on the [`spaCy` models](https://spacy.io/models){target="_blank"} page.

!!! note
    There are a number of alternative ways to install spaCy models from the command line. For instance, you can run the download command as a Python module: `python -m lexos.download_spacy_models xx_sent_ud_sm` or `uv run python -m lexos.download_spacy_models xx_sent_ud_sm`.

    The native spaCy download command should also work:  `uv run python -m spacy download xx_sent_ud_sm` or `python -m spacy download xx_sent_ud_sm`.

### Downloading Language Models Using Python

If you are working in Python, Lexos has a helper function to download models programmatically. You can use the `lexos.download_spacy_model` function to download any spaCy model. For instance, to download the small Chinese model `zh_core_web_sm`, you can use the following code:

```python
from lexos.util import download_spacy_model

download_spacy_model("zh_core_web_sm")
```

## Verify Installation

To verify that Lexos is installed correctly, you can run the following command in your terminal:

If you are using `uv`:

```bash
uv run python -m lexos --info
```

or, if you are not using `uv`:

```bash
python -m lexos --info
```

You can also check the installation by running the following code:

```python
import lexos

lexos.get_info()
```

This should display the Lexos package information, including the version number. If you see an error, please check your installation steps or refer to the [Troubleshooting](#troubleshooting) section below.

## Troubleshooting

Below are some common issues and solutions you may encounter during installation:

### 1. Python Version Error

**Issue:** You see an error like `ModuleNotFoundError` or `SyntaxError` when installing or running Lexos.

**Solution:**
Lexos requires Python 3.12 or greater. Check your Python version with:

```bash
python --version
```

If your version is lower than 3.12, please install a compatible version and create a new virtual environment.

### 2. `uv` or `pip` Not Found

**Issue:** The terminal says `uv: command not found` or `pip: command not found`.

**Solution:**
Make sure you have installed [`uv`](https://docs.astral.sh/uv/){target="_blank"} or [`pip`](https://pip.pypa.io/en/stable/installation/){target="_blank"}. If not, follow the official installation instructions for your platform.

### 3. Permission Denied Errors

**Issue:** You see `Permission denied` when installing packages.

**Solution:**
Always use a virtual environment for Lexos. It is not recommended that you install Lexos globally, but, if you must do so, you may need to use `sudo` on Mac and Linux systems to avoid permission issues.

### 4. spaCy Model Not Found

**Issue:** You see an error like `OSError: [E050] Can't find model 'en_core_web_sm'` or similar when running Lexos features that use spaCy.

**Solution:**
Install the required spaCy model using one of the following commands:

```bash
uv run python download_spacy_models.py en_core_web_sm
# or, if not using uv:
python download_spacy_models.py en_core_web_sm
```

### 5. Lexos Not Found After Installation

**Issue:** Running `python -m lexos --version` or `import lexos` fails with `ModuleNotFoundError`.

**Solution:**
Ensure you are in the correct virtual environment where Lexos was installed. Activate your environment and try again. If the problem persists, reinstall Lexos using `uv add lexos` or `pip install lexos`.

### 6. Outdated `pip` or `uv`

**Issue:** Installation fails with errors about incompatible or missing dependencies.

**Solution:**
Update your package manager:

```bash
pip install --upgrade pip
# or
uv pip install --upgrade uv
```

If you encounter any other problems not covered here, please consider reaching out to the Lexos community Discussion forum on GitHub or checking the [GitHub Issues page](https://github.com/scottkleinman/lexos/issues){target="_blank"} for assistance.

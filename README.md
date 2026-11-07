# NLBSE'27 Tool Competition: Multitask Code Classification

This repository provides the dataset, baseline results, and supporting resources
for the NLBSE'27 Tool Competition on multitask code classification.

Competition website: https://nlbse2027.github.io/tool/

## Competition Overview

Automated vulnerability detection can help developers identify security issues
earlier and improve software quality. However,
their computational demands can hinder real-world deployment, making
**predictive performance, execution time, and resource consumption** equally
important considerations.

This competition challenges participants to balance these objectives by
developing **a single ML model that performs two classification tasks jointly**:

1. **Vulnerability Detection** — determine whether a source-code function contains a vulnerability.
2. **SATD Detection** — determine whether the function is positive for the **SATD** representing a self-admitted technical debt, e.g., `//FIXME: bad implementation`

In other words, **one function goes in, and one model produces two predictions**:

```text
                 ┌─> Vulnerability
Source Function ─┤
                 └─> SATD
```

Models will be evaluated on: 
- **predictive performance**
- **Execution time**
- **Computational cost**

All submissions will be evaluated in a common environment to ensure a fair comparison.

## Contents

- [Competition Task](#competition-task)
- [Dataset](#dataset)
- [Loading the Dataset](#loading-the-dataset)
- [Dataset Preparation](#dataset-preparation)
- [Evaluation](#evaluation)
- [CodeBERT Baseline](#codebert-baseline)
- [Participation](#participation)
- [Citing Related Work](#citing-related-work)

## Competition Task

The competition is formulated as a **supervised multi-label classification problem**.
Given a formatted source-code function as input, the model must predict two binary labels:

| Task | Prediction |
|---|---|
| **Weakness/Vulnerability Detection** | Whether the function contains a weakness or vulnerability |
| **MAT Detection** | Whether the function is positive for the MAT annotation |

Participants should therefore develop **one multitask model** capable of learning both classification tasks jointly.

## Dataset

The competition dataset is derived from
[MADE-WIC](https://doi.org/10.1145/3691620.3695348) and combines instances from
the OSPR, Big-Vul, and Devign collections.

The [Hugging Face repository](https://huggingface.co/datasets/NLBSE/nlbse27-code-comment-classification) provides two participant-facing splits:

| Hub split | Instances | Labels | Intended use |
| --- | ---: | --- | --- |
| `train` | **214,586** | Available | Model training |
| `test` | **30,655** | Available | Local evaluation and result reporting |


An additional **61,311-instance hidden test split** is retained by the organisers and will be used for the final evaluation and ranking. 

Each observation in the dataset is compose by 4 attribts:

| Column | Description |
| --- | --- |
| `Projectname` | Name of the source project |
| `Filepath` | Original source-file path |
| `Function` | Source-code function after formatting |
| `Label` | Binary vector in `[weakness, MAT]` order |

For the classification tasks, the **`Function`** column contains the source code that should be provided to the model.
The corresponding targets are stored in **`Label`** as a two-dimensional binary vector.
The label order is `[weakness, MAT]`:

| Label | Meaning |
| --- | --- |
| `[0, 0]` | Neither weakness-positive nor MAT-positive |
| `[1, 0]` | Weakness-positive and not MAT-positive |
| `[0, 1]` | Not weakness-positive and MAT-positive |
| `[1, 1]` | Both weakness-positive and MAT-positive |

Participants do **not** need to train separate classifiers for the two labels, individual projects, or programming languages. 
The objective is to build **one model capable of performing both classification tasks jointly** across the dataset.

## Loading the dataset

Install the Datasets library and load both public splits:

```bash
python -m pip install datasets
```

```python
from datasets import load_dataset

dataset = load_dataset("NLBSE/nlbse27-code-comment-classification")
train_dataset = dataset["train"]
test_dataset = dataset["test"]

print(train_dataset)
print(test_dataset)
print(train_dataset[0])
print(train_dataset[0].keys())
```

## Dataset preparation

The dataset was prepared as follows:

1. Combine the OSPR, Big-Vul, and Devign MADE-WIC datasets.
2. Format every function with `clang-format` using the LLVM style.
3. Combine the original `Devign`, `Big-Vul`, and `W` annotations with a Boolean
   OR to obtain the weakness label.
4. Remove functions containing more than 500 `cl100k_base` tokens.
5. Downsample overrepresented joint `[weakness, MAT]` classes to restore the
   distribution that existed before token filtering.
6. Create deterministic, jointly stratified training, public test, and hidden
   test partitions using a 70/10/20 ratio.

## Evaluation

Precision, recall, accuracy, and F1-score are calculated independently for the weakness and MAT outputs. 
The primary predictive metric is the mean of the two task-specific F1-scores.

For the final competition ranking, mean F1 contributes 60% of the score.
Average execution time and average GFLOPs each contribute 20%. The organisers
will run every submission on the hidden test split in the same evaluation
environment. The complete scoring procedure is provided in the
competition notebook.

[Competition notebook](https://colab.research.google.com/drive/1b_EiXx5woyCrDntwbljfi-BwS9Dwvhoa?usp=sharing)

## CodeBERT baseline

The baseline is based on
[`microsoft/codebert-base`](https://huggingface.co/microsoft/codebert-base). It
uses a shared CodeBERT encoder and a two-output multi-label classification head.

Validation data is not consulted during fine-tuning. The final trained model is
evaluated once on the complete public test split.

| Output | Accuracy | Precision | Recall | F1 |
| --- | ---: | ---: | ---: | ---: |
| Weakness/vulnerability | 0.960 | 0.943 | 0.935 | 0.939 |
| MAT | 0.996 | 0.978 | 0.802 | 0.881 |
| Unweighted average | 0.978 | 0.961 | 0.869 | 0.910 |

The trained baseline is available from the
[NLBSE'27 CodeBERT baseline repository](https://huggingface.co/NLBSE/nlbse27-code-comment-classification-baseline).

## Participation

Participants must train and evaluate one model that predicts both labels. A
submission must include:

- A description of the model architecture and preprocessing;
- The training and tuning procedure;
- Results on the published test split; and
- Open-source code with instructions for reproducing the results.

Detailed rules and executable instructions are provided in competition website [https://nlbse2027.github.io/tool/](https://nlbse2027.github.io/tool/).

- [Colab instructions](https://colab.research.google.com/drive/1b_EiXx5woyCrDntwbljfi-BwS9Dwvhoa?usp=sharing)
- [CoteBERT baseline notebook](https://colab.research.google.com/drive/1q1fGPpU7uxaK1mUVHa3b2xJOtEwhWwLG?usp=sharing)

## Citing related work

General Competition Citation:

```bibtex
@inproceedings{nlbse2027,
@inproceedings{nlbse2027,
  author = {Mock, Moritz and Borsani, Thomas and Russo, Barbara and Santos, Fabio and and Penney, Jacob and Valenzuela{-}Toledo, Pablo and Kehrer, Timo and Panichella, Sebastiano },
  title={The NLBSE'27 Tool Competition and Challenge},
  booktitle={Proceedings of The 6th International Workshop on Natural Language-based Software Engineering (NLBSE'27)},
  year={2027}
} (bearbeitet)
```

Please also cite the work underlying the dataset and baseline for the code comment competition:

```bibtex
@inproceedings{MockEtAl2024MadeWIC,
  author    = {Mock, Moritz and Melegati, Jorge and Kretschmann, Max and Diaz Ferreyra, Nicolas E. and Russo, Barbara},
  title     = {MADE-WIC: Multiple Annotated Datasets for Exploring Weaknesses In Code},
  booktitle = {Proceedings of the 39th IEEE/ACM International Conference on Automated Software Engineering},
  pages     = {2346--2349},
  year      = {2024},
  doi       = {10.1145/3691620.3695348}
}
```

```bibtex
@inproceedings{RussoEtAl2025VulSATD,
  author    = {Russo, Barbara and Melegati, Jorge and Mock, Moritz},
  title     = {Leveraging Multi-Task Learning to Improve the Detection of SATD and Vulnerability},
  booktitle = {2025 IEEE/ACM 33rd International Conference on Program Comprehension (ICPC)},
  pages     = {1--12},
  year      = {2025},
  doi       = {10.1109/ICPC66645.2025.00017}
}
```

```bibtex
@inproceedings{feng-etal-2020-codebert,
    title = "{C}ode{BERT}: A Pre-Trained Model for Programming and Natural Languages",
    author = "Feng, Zhangyin  and
      Guo, Daya  and
      Tang, Duyu  and
      Duan, Nan  and
      Feng, Xiaocheng  and
      Gong, Ming  and
      Shou, Linjun  and
      Qin, Bing  and
      Liu, Ting  and
      Jiang, Daxin  and
      Zhou, Ming",
    editor = "Cohn, Trevor  and
      He, Yulan  and
      Liu, Yang",
    booktitle = "Findings of the Association for Computational Linguistics: EMNLP 2020",
    month = nov,
    year = "2020",
    address = "Online",
    publisher = "Association for Computational Linguistics",
    url = "https://aclanthology.org/2020.findings-emnlp.139/",
    doi = "10.18653/v1/2020.findings-emnlp.139",
    pages = "1536--1547"
}
```

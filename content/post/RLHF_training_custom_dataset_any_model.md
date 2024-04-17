+++
title = 'Reinforcement Learning with Human Feedback Building a Custom Dataset for Any Model'
date = 2024-02-02T18:31:22+05:30
draft = false
+++

Reinforcement Learning with Human Feedback (RLHF) is a powerful technique in machine learning that combines the strengths of supervised learning with reinforcement learning. It enables models to learn from both structured data and iterative feedback, enhancing their predictive accuracy and adaptability. In this guide, we'll take you through setting up RLHF for any model using a custom dataset. By the end of this post, you'll know how to configure, train, and deploy your model efficiently.

**Setup and Installation**
To get started, you need to install the necessary Python libraries. Run the following command in your environment:

```bash
!pip install transformers trl
```

This command installs the `transformers` library, which provides us with state-of-the-art machine learning models, and `trl` (Transformers for Reinforcement Learning), which is essential for applying RL techniques with transformers.

**Importing Libraries**
Next, let's import the required libraries:

```python
import torch
import transformers
from transformers import pipeline, AutoTokenizer, AutoModelForCausalLM, DataCollatorForLanguageModeling
from trl import RewardTrainer, SFTTrainer
from datasets import Dataset
import json
import pandas as pd
```

These imports are crucial for our project as they provide the foundational components needed to manipulate datasets, train models, and utilize natural language processing tools effectively.

**Downloading and Preparing the Dataset**
We'll use a custom dataset from Hugging Face's datasets library. First, download the dataset:

```bash
!wget https://huggingface.co/datasets/CarperAI/openai_summarize_comparisons/resolve/main/data/test-00000-of-00001-0845e2eec675b16a.parquet
# Rename the downloaded file for easier access
!mv test-00000-of-00001-0845e2eec675b16a.parquet test.parquet
```

Once downloaded, read the data into a DataFrame and convert it into a format suitable for our models:

```python
df = pd.read_parquet("test.parquet")
df = df[:10]  # Limit the data for initial testing
raw_dataset = Dataset.from_pandas(df)
```

**Tokenization**
Tokenization is the process of converting text into a format that's amenable to machine learning models:

```python
Model_name = "bigcode/tiny_starcoder_py"
tokenizer = AutoTokenizer.from_pretrained(Model_name)
model = AutoModelForCausalLM.from_pretrained(Model_name)
tokenizer.add_special_tokens({'pad_token': '[PAD]'})
```

Here, we prepare the tokenizer and model from a pre-trained mini model called `bigcode/tiny_starcoder_py`, which is specially suited for coding language tasks.

**Formatting the Dataset**
Next, we define a function to format our dataset properly:

```python
def formatting_func(examples):
    kwargs = {"padding": "max_length", "truncation": True, "max_length": 256, "return_tensors": "pt"}
    prompt_plus_chosen_response = examples["prompt"] + "\n" + examples["chosen"]
    prompt_plus_rejected_response = examples["prompt"] + "\n" + examples["rejected"]
    tokens_chosen = tokenizer.encode_plus(prompt_plus_chosen_response, **kwargs)
    tokens_rejected = tokenizer.encode_plus(prompt_plus_rejected_response, **kwargs)
    return {
        "input_ids_chosen": tokens_chosen["input_ids"][0], "attention_mask_chosen": tokens_chosen["attention_mask"][0],
        "input_ids_rejected": tokens_rejected["input_ids"][0], "attention_mask_rejected": tokens_rejected["attention_mask"][0]
    }
raw_dataset = raw_dataset.map(formatting_func)
```

This function appends each response to its corresponding prompt and tokenizes the combined text, preparing it for the model.

**Training the Model**
To train the model with RLHF, we set up a `RewardTrainer` and specify our training arguments:

```python
training_args = TrainingArguments(
    output_dir="tinystarcoder-rlhf-model",
    num_train_epochs=1,
    logging_steps=10,
    gradient_accumulation_steps=1,
    save_strategy="steps",
    evaluation_strategy="steps",
    per_device_train_batch_size=2,
    per_device_eval_batch_size=1,
    eval_accumulation_steps=1,
    eval_steps=500,
    save_steps=500,
    warmup_steps=100,
    logging_dir="./logs",
    learning_rate=1e-5,
    save_total_limit=1,
    no_cuda=True
)
trainer = RewardTrainer(model=model, tokenizer=tokenizer, train_dataset=formatted_dataset['train'], eval_dataset=formatted_dataset['test'], args=training_args)
trainer.train()
```

This section defines the training parameters and initializes the training process. The `

RewardTrainer` takes in both training and evaluation datasets and performs the training over one epoch.

**Evaluating and Saving the Model**
After training, it's essential to evaluate and save the model:

```python
trainer.evaluate()
trainer.save_model("tinystarcoder-rlhf-model")
```

**Conclusion**
Implementing RLHF with a custom dataset is a robust way to enhance your models' performance by integrating human-like understanding and feedback. This guide provides a comprehensive walkthrough from setup to deployment, ensuring you have a solid foundation to build upon for your projects.

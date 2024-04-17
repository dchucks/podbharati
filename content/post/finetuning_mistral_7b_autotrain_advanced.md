+++
title = 'Finetuning Mistral-7b FineTuning Model using Autotrain-advanced'
date = 2024-02-02T18:31:22+05:30
draft = false
+++

In the rapidly advancing field of artificial intelligence, leveraging pre-trained models by finetuning them for specific tasks can drastically improve performance. This tutorial takes an in-depth look at how to finetune the Mistral-7b model using Autotrain-advanced, providing a clear explanation for each line of code and practical steps to deploy the project effectively.

### Step 1: Setting Up Your Environment

Before diving into the world of model finetuning, it's crucial to prepare your working environment. This involves installing necessary libraries and setting up the workspace. Start by running the following commands in your Python environment (like Jupyter notebook or Google Colab):

```bash
!pip install pandas autotrain-advanced -q
```

These commands install pandas for data manipulation and Autotrain-advanced for model training and deployment.

```bash
!autotrain setup --update-torch
```

This command ensures your PyTorch installation is up to date, which is vital for compatibility with the latest models and training procedures.

### Step 2: Authentication with Hugging Face

To upload and use models for inference on Hugging Face's platform, authentication is necessary. Here’s how you can authenticate:

#### Logging to Hugging Face

1. **Navigate to this URL**: [Hugging Face Tokens](https://huggingface.co/settings/tokens).
2. **Create a write `token`** and copy it to your clipboard.
3. **Authenticate** by running the following Python code:

```python
from huggingface_hub import notebook_login
notebook_login()
```

This code snippet will prompt you to enter the token you copied earlier, establishing a secure connection with Hugging Face.

### Step 3: Preparing Your Dataset

The effectiveness of your finetuned model depends significantly on the quality and relevance of your training data. Here’s how to add your dataset:

```bash
!git clone https://github.com/joshbickett/finetune-llama-2.git
%cd finetune-llama-2
%mv train.csv ../train.csv
%cd ..
```

These commands clone a repository containing an example dataset and move the `train.csv` file to your current working directory.

```python
import pandas as pd
df = pd.read_csv("train.csv")
print(df['text'][15])
```

This Python code loads your dataset using pandas, a powerful data manipulation library, allowing you to inspect the data, particularly the text data at index 15.

### Step 4: Running the Fine-Tuning Process

The core of this tutorial is finetuning the model using specific command flags that configure the training process. Here’s an explanation of the key flags:

- `--train`: Initiates the training process.
- `--project_name`: Assign a name to your project.
- `--model`: Define the base model to start from.
- `--data_path`: Specify the dataset location.
- `--use_int4`: Opt for INT4 quantization to balance speed and precision.
- `--learning_rate`: Set the pace of model learning during training.
- `--train_batch_size`: Define how many examples are processed together.
- `--num_train_epochs`: Set the number of times the training algorithm will work through the entire dataset.

Adjust these flags to fit your project requirements and run the command:

```bash
!autotrain llm --train --project_name mistral-7b-mj-finetuned --model bn22/Mistral-7B-Instruct-v0.1-sharded --data_path . --use_peft --use_int4 --learning_rate 2e-4 --train_batch_size 12 --num_train_epochs 3 --trainer sft --target_modules q_proj,v_proj --push_to_hub --repo_id ashishpatel26/mistral-7b-mj-finetuned
```

### Step 5: Uploading and Verifying the Model

Upon completion of the training process, your model is automatically uploaded to Hugging Face under your specified repository. You can verify the upload by visiting your Hugging Face profile and navigating to the repository.

### Step 6: Inference and Utilization

To use your finetuned model, install additional libraries for efficient model loading and inference:

```bash
!pip install -q peft accelerate bitsandbytes safetensors
```

Load the model using the following Python code:

```python
import torch
from peft import PeftModel
from transformers import AutoModelForCausalLM, AutoTokenizer
import transformers

model_name = "bn22/Mistral-7B-Instruct-v0.1-sharded"
adapters_name = "ashishpatel26/mistral-7b-mj-finetuned"

bnb_config = transformers.BitsAndBytesConfig(
    load_in_

4bit=True,
    bnb_4bit_use_double_quant=True,
    bnb_4bit_quant_type="nf4",
    bnb_4bit_compute_dtype=torch.bfloat16
)

model = AutoModelForCausalLM.from_pretrained(
    model_name,
    load_in_4bit=True,
    torch_dtype=torch.bfloat16,
    quantization_config=bnb_config,
    device_map='auto'
)

model = PeftModel.from_pretrained(model, adapters_name)

tokenizer = AutoTokenizer.from_pretrained(model_name)
tokenizer.bos_token_id = 1

stop_token_ids = [0]

print(f"Successfully loaded the model {model_name} into memory")

text = "[INST] generate a midjourney prompt for A person walks in the rain [/INST]"

encoded = tokenizer(text, return_tensors="pt", add_special_tokens=False)
model_input = encoded
model.to(device)
generated_ids = model.generate(**model_input, max_new_tokens=200, do_sample=True)
decoded = tokenizer.batch_decode(generated_ids)
print(decoded[0])
```

This code loads the model and tokenizer, sets up necessary configurations for efficient processing, and runs a sample inference to generate text based on a prompt.

### Conclusion

By following this guide, you have not only finetuned a powerful language model but also learned how to deploy it for practical use. Whether for enhancing your projects or developing new AI-driven applications, the skills you've acquired here are invaluable.

We hope this tutorial empowers you to dive deeper into the world of AI and machine learning. Happy coding!
---
title: "Finetuning Mistral-7b FineTuning Model using Autotrain-advanced"
description: Welcome to this in depth guide on fine-tuning the Mistral-7b model using Autotrain-Advanced. In this tutorial, we'll explore how to adapt a powerful pre-trained large language model to your specific domain. Fine-tuning allows you to customize a general-purpose model, making it perform better on tasks relevant to your particular dataset.
date: 2024-05-26T18:31:22+05:30
featured: true
authors: [admin]
image: "https://img.freepik.com/free-vector/pixel-art-rural-landscape-background_23-2150592944.jpg?w=740&t=st=1716826695~exp=1716827295~hmac=b96dfbc3983c8e63bb61216ce6aa2ec22c1a6c908160557338fdec69f1562b5d"
---

The Mistral-7b model is a robust language model known for its performance and effefficiency. By leveraging Autotrain-Advanced, we can streamline the fine-tuning process, making it more efficient and accessible.

We'll cover everything from setting up your environment, preparing your data, and running the fine-tuning process, to evaluating the performance of your custom model.

You can view the full notebook [here](https://colab.research.google.com/drive/1bicxnSG-TPm-asLtvwyeuIbgOJOQQNgB?usp=sharing#scrollTo=ZbOOX8cve0lR) or follow along with the following steps.

### Step 1: Getting started

We will be using google colab or Jupyter notebook to create and filetune this model. The first step involves installing and setting up your enviroment.

```python
!pip install pandas autotrain-advanced -q
!autotrain setup --update-torch
```

Here, we are doing the following:

1. Installing libraries like `pandas` which is used for data manipulatoin and analysis, `autotrain-advanced` which simplifies the process of training and fine-tuning models.
2. Updating `PyTorch` which is a framework that is crucial for training our model.
   These commands install pandas for data manipulation and Autotrain-advanced for model training and deployment.

### Step 2: Authentication with Hugging Face

To make sure the model can be uploaded and be used for Inference, it's necessary to log in to the Hugging Face hub. The steps of getting the Hugging Face token are:

1. Navigating to [Hugging Face settings](https://huggingface.co/settings/tokens)
2. Create a write `token` and copy it to your clipboard.
3. Run the next cell and enter yout `token`

```python
from huggingface_hub import notebook_login
notebook_login()
```

### Step 3: Preparing and Uploading Your Dataset

Add your data set to the root directory in the Colab under the name train.csv. The AutoTrain command will look for your data there under that name.

If you don't have a dataset and want to try finetuning on an example dataset, you can run these commands below to get an example data set and save it to train.csv

```python
!git clone https://github.com/joshbickett/finetune-llama-2.git
%cd finetune-llama-2
%mv train.csv ../train.csv
%cd ..
```

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

```python
!autotrain llm --train --project_name mistral-7b-mj-finetuned --model bn22/Mistral-7B-Instruct-v0.1-sharded --data_path . --use_peft --use_int4 --learning_rate 2e-4 --train_batch_size 12 --num_train_epochs 3 --trainer sft --target_modules q_proj,v_proj --push_to_hub --repo_id ashishpatel26/mistral-7b-mj-finetuned
```

### Steps needed before running the above code block

Go to the !autotrain code cell above and update it by following the steps below:

After `--project_name` replace `_enter-a-project-name_` with the name that you'd like to call the project. After `--repo_id` replace `_username_/_repository_`. Replace `_username_` with your Hugging Face username and `_repository_` with the repository name you'd like it to be created under. You don't need to create this repository before hand, it will automatically be created and uploaded once the training is completed.

Confirm that train.csv is in the root directory in the Colab. The `--data_path .` flag will make it so that AutoTrain looks for your data there.

Make sure to add the LoRA Target Modules to be trained `--target-modules q_proj, v_proj`

**after making those changes and running the code cell, you should now have successfully uploaded the model to Hugging Face.**

### Step 5: Uploading and Verifying the Model

Upon completion of the training process, your model is automatically uploaded to Hugging Face under your specified repository. You can verify the upload by visiting your Hugging Face profile and navigating to the repository.

### Step 6: Inference and Utilization of the Model

By installing these libraries and setting up the model, tokenizer, and device, we're preparing the environment for the fine-tuning process. These steps ensure we have the necessary tools and configurations to modify the Mistral-7b model to suit our specific requirements.

```python
!autotrain llm -h
!pip install -q peft accelerate bitsandbytes safetensors
import torch
from peft import PeftModel
from transformers import AutoModelForCausalLM, AutoTokenizer
import transformers
adapters_name = "{enter usename}/mistral-7b-mj-finetuned"
model_name = "bn22/Mistral-7B-Instruct-v0.1-sharded" #"mistralai/Mistral-7B-Instruct-v0.1"


device = "cuda" # the device to load the model onto
```

The command `!autotrain llm -h` shows help information for Autotrain's Large Language Model functionality, helping you familiarize yourself with available options.

The `peft` library stands for "Parameter-Efficient Fine-Tuning" and optimizes memory and computational efficiency during fine-tuning. The `accelerate` library by Hugging Face streamlines and speeds up the training of large models, while `bitsandbytes` provides efficient quantization and memory optimization, particularly useful for large models.

The `safetensors` library handles tensor data securely and efficiently. The `torch` library (PyTorch) is essential for handling tensors and running deep learning models. `PeftModel` from `peft` allows for parameter-efficient fine-tuning techniques.

`AutoModelForCausalLM` and `AutoTokenizer` from `transformers` are used to load the pre-trained model and tokenizer. The `adapters_name` variable holds the path to the adapter configuration for fine-tuning, and `model_name` specifies the pre-trained model, here the Mistral-7B. The `device` variable indicates using a GPU (`"cuda"`) for faster processing; if a GPU is unavailable, it can be set to `"cpu"` for slower training.

**Configuring Bits and Bytes for Efficient Model Loading**

```python
bnb_config = transformers.BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_use_double_quant=True,
    bnb_4bit_quant_type="nf4",
    bnb_4bit_compute_dtype=torch.bfloat16
)
```

To efficiently load the Mistral-7B model, we use BitsAndBytesConfig from the transformers library. This configuration enables 4-bit precision (load_in_4bit=True), applies double quantization (bnb_4bit_use_double_quant=True),

uses non finite 4-bit quantization (bnb_4bit_quant_type="nf4"), and sets the computation type to bfloat16 (bnb_4bit_compute_dtype=torch.bfloat16). These settings significantly reduce memory usage and maintain performance, making it feasible to handle large models on limited-resource hardware like GPUs.

**Loading the Model with Quantazation**

We load the Mistral-7B model using AutoModelForCausalLM.from_pretrained, applying the previously defined BitsAndBytesConfig for efficient memory usage. The model is loaded in 4-bit precision (load_in_4bit=True), uses bfloat16 for computation (torch_dtype=torch.bfloat16), and automatically maps to available devices (device_map='auto').

```python
model = AutoModelForCausalLM.from_pretrained(
    model_name,
    load_in_4bit=True,
    torch_dtype=torch.bfloat16,
    quantization_config=bnb_config,
    device_map='auto'
)
```

This step initializes the Mistral-7B model with the specified quantization settings to optimize memory usage and performance, automatically distributing the model across available hardware resources.

### Step 7: Applying Parameter-Efficient Fine-Tuning and loading the Tokenizer\*\*

In this step, we apply the parameter-efficient fine-tuning (PEFT) configuration to our model and load the tokenizer. We also set the beginning-of-sequence token ID and define stop token IDs.

```python
model = PeftModel.from_pretrained(model, adapters_name)
tokenizer = AutoTokenizer.from_pretrained(model_name)
tokenizer.bos_token_id = 1

stop_token_ids = [0]

print(f"Successfully loaded the model {model_name} into memory")
```

We enhance our model with PEFT (PeftModel.from_pretrained), which applies the fine-tuning configuration specified by adapters_name to the pre-trained model. We then load the tokenizer (AutoTokenizer.from_pretrained) for handling text input and output, setting its beginning-of-sequence token ID (tokenizer.bos_token_id = 1).

We also define stop token IDs (stop_token_ids = [0]), which are tokens indicating where the model should stop generating text. Finally, we print a confirmation message to indicate successful model loading.

**Generating Text with Fine-Tuned Model**

Now, we will use the fine-tuned Mistral-7B model to generate text based on a given prompt. We'll encode the input text, generate a response, and decode the output.

```python
text = "[INST] generate a midjourney prompt for A person walks in the rain [/INST]"

encoded = tokenizer(text, return_tensors="pt", add_special_tokens=False)
model_input = encoded
model.to(device)
generated_ids = model.generate(**model_input, max_new_tokens=200, do_sample=True)
decoded = tokenizer.batch_decode(generated_ids)
print(decoded[0])
```

In this step, we prompt the fine-tuned Mistral-7B model to generate text. First, we define an input prompt (text), encode it using the tokenizer, and prepare it for the model. We then move the model to the GPU (device) for efficient processing.

The model generates text based on the prompt, allowing up to 200 new tokens and using sampling for diversity (`model.generate`). The generated token IDs are decoded back into readable text (`tokenizer.batch_decode`), and the result is printed. This showcases how to use the fine-tuned model to create customized outputs from specific prompts.

+++
title = 'Fine-tuning Microsoft Phi 1.5b on Custom Dataset with DialogStudio'
date = 2024-02-02T18:31:22+05:30
draft = false
+++

Fine-tuning Microsoft's Phi 1.5b model on a specialized dataset like DialogStudio presents a unique opportunity to create a powerful AI tool tailored for specific tasks, such as summarization or conversational analysis. In this extended guide, we delve deeper into the process, uncovering advanced strategies and providing a thorough walkthrough of each step, from the initial setup to the final deployment of the fine-tuned model.

### In-Depth Exploration of Fine-Tuning Microsoft Phi 1.5b

#### Comprehensive Setup and Dependency Management

The journey begins with a robust setup, ensuring all necessary tools are ready:

```bash
!pip install accelerate transformers einops datasets peft bitsandbytes trl
```

We use these libraries for their specific roles in the fine-tuning pipeline: `accelerate` for distributed training, `transformers` and `trl` for model handling and optimization, `datasets` for data management, `peft` for efficient parameter tuning, and `bitsandbytes` for memory-efficient training.

#### Detailed Data Engineering with DialogStudio

We select the DialogStudio dataset for its rich conversational content, ideal for training models on summarization tasks:

```python
from datasets import load_dataset

dataset = load_dataset("Salesforce/dialogstudio", "TweetSumm")
```

This dataset provides a foundation for our fine-tuning, offering real-world conversational data that can be used to train the model in generating concise and relevant summaries.

#### Advanced Preprocessing and Tokenization

To prepare our data for fine-tuning, we implement a series of preprocessing steps:

```python
def process_dataset(data: Dataset):
    return (
        data.shuffle(seed=42)
        .map(generate_text)
        .remove_columns([...])
    )

dataset["train"] = process_dataset(dataset["train"])
```

This function not only shuffles and processes the data but also ensures that it is in the optimal format for training, with all unnecessary columns removed and the text tokenized effectively.

#### Model Configuration and Optimization

Loading the Phi 1.5b model requires careful configuration to suit our fine-tuning needs:

```python
from transformers import AutoModelForCausalLM, BitsAndBytesConfig

model, tokenizer = create_model_and_tokenizer()
```

Here, `create_model_and_tokenizer` is a custom function that initializes the model and tokenizer with settings that facilitate efficient training, including memory and computation optimizations.

#### Fine-Tuning Strategy with PEFT and TRL

Utilizing PEFT (Parameter Efficient Fine-tuning) and TRL (Token-level Reinforcement Learning), we fine-tune the model efficiently:

```python
from trl import SFTTrainer

trainer = SFTTrainer(
    model=model,
    train_dataset=dataset["train"],
    ...
)
trainer.train()
```

The `SFTTrainer` class allows for an efficient fine-tuning process, leveraging token-level optimization to improve the quality of the model’s output while maintaining computational efficiency.

#### Evaluating Model Performance

After training, it's crucial to evaluate how well the model has adapted to the new dataset:

```python
trainer.evaluate()
```

This step assesses the fine-tuned model's summarization capabilities, ensuring it meets the desired performance benchmarks.

#### Saving and Deploying the Fine-Tuned Model

Once fine-tuned and evaluated, the model is saved and can be deployed for practical applications:

```python
trainer.save_model("phi-1_5-finetuned-dialogstudio")
```

Saving the model allows for its reuse in various applications, from automated summarization tools to conversational analysis systems.

#### Advanced Inference Techniques

Deploying the fine-tuned model for inference involves using it to generate summaries or responses based on new data:

```python
model = AutoModelForCausalLM.from_pretrained("username/phi-1_5-finetuned-dialogstudio")
inputs = tokenizer(f'''{dataset["test"]['text'][0]}''', return_tensors="pt")
outputs = model.generate(**inputs, max_length=512)
```

This step demonstrates the practical application of the fine-tuned model, showcasing its ability to generate coherent and contextually relevant text based on the training it received.

### Conclusion

Fine-tuning Microsoft's Phi 1.5b model on the DialogStudio dataset is a comprehensive process that involves meticulous setup, data preparation, and model optimization. Through this detailed guide, you have gained insights into the intricacies of each step, from preprocessing and tokenization to training, evaluation, and deployment. This journey not only enhances the model's performance on specific tasks but also provides a blueprint for leveraging advanced AI tools in real-world applications, paving the way for innovative solutions in AI-driven industries.
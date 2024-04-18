+++
title = 'MPT-30B Chatbot with LangChain A Comprehensive Guide to Building Advanced Conversational AI'
date = 2024-02-02T18:31:22+05:30
draft = false
+++

Are you intrigued by the potential of artificial intelligence in enhancing conversational experiences? In this guide, we will delve into creating a high-level chatbot using the MPT-30B model integrated with LangChain. This tutorial is designed to empower tech enthusiasts, developers, and AI researchers with the tools and knowledge to build a state-of-the-art chatbot.

### Preparing Your Development Environment

To kick things off, you'll need to prepare your environment. This involves installing several crucial libraries that will enable the creation and operation of your chatbot. Here’s the command to get everything set up:

```bash
!pip install -qU transformers accelerate einops langchain xformers bitsandbytes
```

**Explanation of Libraries**:
- **Transformers**: This library is at the forefront of providing easy-to-use APIs for pretrained models like MPT-30B.
- **Accelerate**: Helps manage model training and inference on multiple devices, making your experiments smoother and faster.
- **Einops**: A versatile tool for tensor manipulation, essential for processing inputs and outputs in model pipelines.
- **Langchain**: Simplifies the integration of language models into applications, offering tools for building and managing conversational agents.
- **Xformers & Bitsandbytes**: These libraries enhance performance through efficient memory usage and advanced transformer operations.

### Setting Up the MPT-30B Model

The MPT-30B model from MosaicML represents a significant advancement in language model capabilities, enabling nuanced and context-aware conversations. Let’s set up the model:

```python
from torch import cuda
import transformers

device = f'cuda:{cuda.current_device()}' if cuda.is_available() else 'cpu'
model = transformers.AutoModelForCausalLM.from_pretrained(
    'mosaicml/mpt-30b-chat',
    trust_remote_code=True,
    load_in_8bit=True,
    max_seq_len=8192,
    init_device=device
)
model.eval()
print(f"Model loaded on {device}")
```

**Deep Dive**:
- **AutoModelForCausalLM**: Chooses the right causal language model for generating text based on the input configuration.
- **device**: Utilizes GPU if available for better performance, otherwise falls back to CPU.
- **load_in_8bit**: Utilizes the `bitsandbytes` library to reduce model size and accelerate inference by loading the model in 8-bit precision.

### Configuring the Tokenizer

Tokenization is critical as it converts raw text into a format that the model can understand, involving splitting text into tokens and encoding these tokens into numbers.

```python
tokenizer = transformers.AutoTokenizer.from_pretrained("mosaicml/mpt-30b")
```

### Implementing Stopping Criteria

To control the model’s verbosity and ensure it stops generating text at logical points, we implement custom stopping criteria:

```python
import torch
from transformers import StoppingCriteria, StoppingCriteriaList

stop_token_ids = [tokenizer.convert_tokens_to_ids(x) for x in [['Human', ':'], ['AI', ':']]]
stop_token_ids = [torch.LongTensor(x).to(device) for x in stop_token_ids]

class StopOnTokens(StoppingCriteria):
    def __call__(self, input_ids: torch.LongTensor, scores: torch.FloatTensor, **kwargs) -> bool:
        for stop_ids in stop_token_ids:
            if torch.eq(input_ids[0][-len(stop_ids):], stop_ids).all():
                return True
        return False

stopping_criteria = StoppingCriteriaList([StopOnTokens()])
```

**Insights**:
- **StoppingCriteriaList**: Combines multiple stopping conditions to fine-tune where the model ceases text generation.
- **StopOnTokens**: Custom condition that halts generation when predefined token sequences are detected, preventing rambling outputs.

### Creating the Text Generation Pipeline

With the model and tokenizer configured, we establish a pipeline for generating text:

```python
generate_text = transformers.pipeline(
    'text-generation',
    model=model,
    tokenizer=tokenizer,
    stopping_criteria=stopping_criteria,
    temperature=0.1,
    top_p=0.15,
    top_k=0,
    max_new_tokens=128,
    repetition_penalty=1.1
)
```

### Integrating the Pipeline with LangChain

LangChain enhances

 our chatbot by facilitating more sophisticated AI applications and conversational memory management:

```python
from langchain import PromptTemplate, LLMChain
from langchain.llms import HuggingFacePipeline

prompt = PromptTemplate(input_variables=["instruction"], template="{instruction}")
llm = HuggingFacePipeline(pipeline=generate_text)
llm_chain = LLMChain(llm=llm, prompt=prompt)
```

### Building a Conversational Memory

To make interactions seamless and context-aware, we incorporate a memory system that tracks the conversation history:

```python
from langchain.chains.conversation.memory import ConversationBufferWindowMemory
from langchain.chains import ConversationChain

memory = ConversationBufferWindowMemory(memory_key="history", k=5, return_only_outputs=True)
chat = ConversationChain(llm=llm, memory=memory, verbose=True)

chat.prompt.template = """The following is a friendly conversation between a human and an AI. The AI is conversational but concise in its responses without rambling. If the AI does not know the answer to a question, it truthfully says it does not know.

Current conversation:
{history}
Human: {input}
AI:"""

res = chat.predict(input='hi how are you?')
print(res)
```

### Wrapping Up and Next Steps

Congratulations! You have built a sophisticated MPT-30B chatbot integrated with LangChain. This setup not only allows the chatbot to generate contextually appropriate responses but also adapts to the conversation's flow, making it an engaging and intelligent conversational partner.

Feel free to explore further by adjusting the conversation templates, experimenting with different settings for the model, and integrating additional data sources for more robust interactions.
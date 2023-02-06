---
layout: post
title:  "Loading Large LLMs"
date:   2023-02-06 00:00:00 -0500
categories: jekyll update
regenerate: true
---

I'm building a project I'm calling "deck chat" where you can upload a deck and some notes and get back a website hosting the deck, and a built in chatbot trained on the content. The front end is all React and the backend is FastAPI. It's a pretty straight forward app other than the chatbot part.

Something I messed around with a bit was sharding models to make them smaller for loading. For instance `EleutherAI/gpt-j-6B` is about 20G. It won't fit on my card at all:

```python
torch.cuda.OutOfMemoryError: CUDA out of memory. Tried to allocate 20.00 MiB (GPU 0; 15.70 GiB total capacity; 8.22 GiB already allocated; 48.56 MiB free; 8.62 GiB reserved in total by PyTorch) If reserved memory is >> allocated memory try setting max_split_size_mb to avoid fragmentation.  See documentation for Memory Management and PYTORCH_CUDA_ALLOC_CONF
```

Even in cases where the model was small enough to fit on card, I did not find it helpful to mess with the [CUDA memory management](https://pytorch.org/docs/stable/notes/cuda.html#memory-management). If it didn't "just load", tweaking some settings didn't help.

We do have another approach though. If you have a model that's too big to load, you can load it into CPU memory (assuming it'll fit there), and then dump it to disk in shards. Then when you use it for inference, it'll load each shard in turn, do the computaitons, merge them together, and then return the result. Yes, it is as slow as it sounds. However, the alternative is not being able to do it at all...

Here's an example piece of code:

```python
from accelerate import init_empty_weights
from transformers import AutoConfig, AutoTokenizer, AutoModelForCausalLM
import torch

model_name = "EleutherAI/gpt-j-6B"
new_model_name = 'gpt-j-6B-sharded'
model = AutoModelForCausalLM.from_pretrained(model_name, revision='float16', torch_dtype=torch.float16, low_cpu_mem_usage=True)
model.save_pretrained(new_model_name, max_shard_size="4GB")
```

Once it's sharded, you can load it like this:

```python
model = AutoModelForCausalLM.from_pretrained(
    new_model_name,
    device_map="auto",
    offload_folder=f"/tmp/offload-accelerate/{new_model_name}",
)
```

It'll load each part of the model as needed. It takes a lot longer that a non-sharded model.

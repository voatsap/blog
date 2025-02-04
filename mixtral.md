---
title: "Welcome Mixtral - a SOTA Mixture of Experts on Hugging Face"
thumbnail: /blog/assets/mixtral/thumbnail.jpg
authors:
- user: lewtun
- user: philschmid
- user: osanseviero
- user: pcuenq
- user: olivierdehaene
- user: lvwerra
- user: ybelkada
---

# Welcome Mixtral - a SOTA Mixture of Experts on Hugging Face

Mixtral 8x7b is an exciting large language model released by Mistral today, which sets a new state-of-the-art for open-access models and outperforms GPT-3.5 across many benchmarks. We’re excited to support the launch with a comprehensive integration of Mixtral in the Hugging Face ecosystem 🔥!

Among the features and integrations being released today, we have:

- [Models on the Hub](https://huggingface.co/models?search=mistralai/Mixtral), with their model cards and licenses (Apache 2.0)
- [🤗 Transformers integration](https://github.com/huggingface/transformers/releases/tag/v4.36.0)
- Integration with Inference Endpoints
- Integration with [Text Generation Inference](https://github.com/huggingface/text-generation-inference) for fast and efficient production-ready inference
- An example of fine-tuning Mixtral on a single GPU with 🤗 TRL.

## Table of Contents

- [What is Mixtral 8x7b](#what-is-mixtral-8x7b)
  - [About the name](#about-the-name)
  - [Prompt format](#prompt-format)
  - [What we don't know](#what-we-dont-know)
- [Demo](#demo)
- [Inference](#inference)
  - [Using 🤗 Transformers](#using-🤗-transformers)
  - [Using Text Generation Inference](#using-text-generation-inference)
- [Fine-tuning with 🤗 TRL](#fine-tuning-with-🤗-trl)
- [Disclaimers and ongoing work](#disclaimers-and-ongoing-work)
- [Additional Resources](#additional-resources)
- [Conclusion](#conclusion)

## What is Mixtral 8x7b?

Mixtral has a similar architecture to Mistral 7B, but comes with a twist: it’s actually 8 “expert” models in one, thanks to a technique called Mixture of Experts (MoE). For transformers models, the way this works is by replacing some Feed-Forward layers with a sparse MoE layer. A MoE layer contains a router network to select which experts process which tokens most efficiently. In the case of Mixtral, two experts are selected for each timestep, which allows the model to decode at the speed of a 12B parameter-dense model, despite containing 4x the number of effective parameters! 

For more details on MoEs, see our accompanying blog post: [hf.co/blog/moe](https://huggingface.co/blog/moe)

**Mixtral release TL;DR;**

- Release of base and Instruct versions
- Supports a context length of 32k tokens.
- Outperforms Llama 2 70B and matches or beats GPT3.5 on most benchmarks
- Speaks English, French, German, Spanish, and Italian.
- Good at coding, with 40.2% on HumanEval
- Commercially permissive with an Apache 2.0 license

So how good are the Mixtral models? Here’s an overview of the base model and its performance compared to other open models on the [LLM Leaderboard](https://huggingface.co/spaces/HuggingFaceH4/open_llm_leaderboard) (higher scores are better):

| Model                                                                             | License         | Commercial use? | Pretraining size [tokens] | Leaderboard  score ⬇️ |
| --------------------------------------------------------------------------------- | --------------- | --------------- | ------------------------- | -------------------- |
| [mistralai/Mixtral-8x7B-v0.1](https://huggingface.co/mistralai/Mixtral-8x7B-v0.1) | Apache 2.0      | ✅               | unknown                   | 68.42                |
| [meta-llama/Llama-2-70b-hf](https://huggingface.co/meta-llama/Llama-2-70b-hf)     | Llama 2 license | ✅               | 2,000B                    | 67.87                |
| [tiiuae/falcon-40b](https://huggingface.co/tiiuae/falcon-40b)                     | Apache 2.0      | ✅               | 1,000B                    | 61.5                 |
| [mistralai/Mistral-7B-v0.1](https://huggingface.co/mistralai/Mistral-7B-v0.1)     | Apache 2.0      | ✅               | unknown                   | 60.97                |
| [meta-llama/Llama-2-7b-hf](https://huggingface.co/meta-llama/Llama-2-7b-hf)       | Llama 2 license | ✅               | 2,000B                    | 54.32                |

For instruct and chat models, evaluating on benchmarks like MT-Bench or AlpacaEval is better. Below, we show how [Mixtral Instruct](https://huggingface.co/mistralai/Mixtral-8x7B-Instruct-v0.1) performs up against the top closed and open access models (higher scores are better):

| Model                                                                                               | Availability    | Context window (tokens) | MT-Bench score ⬇️ |
| --------------------------------------------------------------------------------------------------- | --------------- | ----------------------- | ---------------- |
| [GPT-4 Turbo](https://openai.com/blog/new-models-and-developer-products-announced-at-devday)        | Proprietary     | 128k                    | 9.32             |
| [GPT-3.5-turbo-0613](https://platform.openai.com/docs/models/gpt-3-5)                               | Proprietary     | 16k                     | 8.32             |
| [mistralai/Mixtral-8x7B-Instruct-v0.1](https://huggingface.co/mistralai/Mixtral-8x7B-Instruct-v0.1) | Apache 2.0      | 32k                     | 8.30             |
| [Claude 2.1](https://www.anthropic.com/index/claude-2-1)                                            | Proprietary     | 200k                    | 8.18             |
| [openchat/openchat_3.5](https://huggingface.co/openchat/openchat_3.5)                               | Apache 2.0      | 8k                      | 7.81             |
| [HuggingFaceH4/zephyr-7b-beta](https://huggingface.co/HuggingFaceH4/zephyr-7b-beta)                 | MIT             | 8k                      | 7.34             |
| [meta-llama/Llama-2-70b-chat-hf](https://huggingface.co/meta-llama/Llama-2-70b-chat-hf)             | Llama 2 license | 4k                      | 6.86             |

Impressively, Mixtral Instruct outperforms all other open-access models on MT-Bench and is the first one to achieve comparable performance with GPT-3.5!

### About the name

The Mixtral MoE is called **Mixtral-8x7B**, but it doesn't have 56B parameters. Shortly after the release, we found that some people were misled into thinking that the model behaves similarly to an ensemble of 8 models with 7B parameters each, but that's not how MoE models work. Only some layers of the model (the feed-forward blocks) are replicated; the rest of the parameters are the same as in a 7B model. The total number of parameters is not 56B, but about 45B. A better name [could have been `Mixtral-45-8e`](https://twitter.com/osanseviero/status/1734248798749159874) to better convey the architecture. For more details about how MoE works, please refer to [our "Mixture of Experts Explained" post](https://huggingface.co/blog/moe).

### Prompt format

The [base model](https://huggingface.co/mistralai/Mixtral-8x7B-v0.1) has no prompt format. Like other base models, it can be used to continue an input sequence with a plausible continuation or for zero-shot/few-shot inference. It’s also a great foundation for fine-tuning your own use case. The [Instruct model](https://huggingface.co/mistralai/Mixtral-8x7B-Instruct-v0.1) has a very simple conversation structure.

```bash
<s> [INST] User Instruction 1 [/INST] Model answer 1</s> [INST] User instruction 2[/INST]
```

This format has to be exactly reproduced for effective use. We’ll show later how easy it is to reproduce the instruct prompt with the chat template available in `transformers`. 

### What we don't know

Like the previous Mistral 7B release, there are several open questions about this new series of models. In particular, we have no information about the size of the dataset used for pretraining, its composition, or how it was preprocessed.

Similarly, for the Mixtral instruct model, no details have been shared about the fine-tuning datasets or the hyperparameters associated with SFT and DPO.

## Demo

You can chat with the Mixtral Instruct model on Hugging Face Chat! Check it out here: [https://huggingface.co/chat/?model=mistralai/Mixtral-8x7B-Instruct-v0.1](https://huggingface.co/chat/?model=mistralai/Mixtral-8x7B-Instruct-v0.1).

## Inference

We provide two main ways to run inference with Mixtral models:

- Via the `pipeline()` function of 🤗 Transformers.
- With Text Generation Inference, which supports advanced features like continuous batching, tensor parallelism, and more, for blazing fast results.

For each method, it is possible to run the model in half-precision (float16) or with quantized weights. Since the Mixtral model is roughly equivalent in size to a 45B parameter dense model, we can estimate the minimum amount of VRAM needed as follows:

| Precision | Required VRAM |
| --------- | ------------- |
| float16   | >90 GB        |
| 8-bit     | >45 GB        |
| 4-bit     | >23 GB        |

### Using 🤗 Transformers

With transformers [release 4.36](https://github.com/huggingface/transformers/releases/tag/v4.36.0), you can use Mixtral and leverage all the tools within the Hugging Face ecosystem, such as:

- training and inference scripts and examples
- safe file format (`safetensors`)
- integrations with tools such as bitsandbytes (4-bit quantization), PEFT (parameter efficient fine-tuning), and Flash Attention 2
- utilities and helpers to run generation with the model
- mechanisms to export the models to deploy

Make sure to use the latest `transformers` release:

```bash
pip install -U "transformers==4.36.0" --upgrade
```

In the following code snippet, we show how to run inference with 🤗 Transformers and 4-bit quantization. Due to the large size of the model, you’ll need a card with at least 30 GB of RAM to run it. This includes cards such as A100 (80 or 40GB versions), or A6000 (48 GB).

```python
from transformers import AutoTokenizer
import transformers
import torch

model = "mistralai/Mixtral-8x7B-Instruct-v0.1"

tokenizer = AutoTokenizer.from_pretrained(model)
pipeline = transformers.pipeline(
    "text-generation",
    model=model,
    model_kwargs={"torch_dtype": torch.float16, "load_in_4bit": True},
)

messages = [{"role": "user", "content": "Explain what a Mixture of Experts is in less than 100 words."}]
prompt = pipeline.tokenizer.apply_chat_template(messages, tokenize=False, add_generation_prompt=True)
outputs = pipeline(prompt, max_new_tokens=256, do_sample=True, temperature=0.7, top_k=50, top_p=0.95)
print(outputs[0]["generated_text"])
```

> \<s>[INST] Explain what a Mixture of Experts is in less than 100 words. [/INST] A
Mixture of Experts is an ensemble learning method that combines multiple models,
or "experts," to make more accurate predictions. Each expert specializes in a
different subset of the data, and a gating network determines the appropriate
expert to use for a given input. This approach allows the model to adapt to
complex, non-linear relationships in the data and improve overall performance.
> 

### Using Text Generation Inference

**[Text Generation Inference](https://github.com/huggingface/text-generation-inference)** is a production-ready inference container developed by Hugging Face to enable easy deployment of large language models. It has features such as continuous batching, token streaming, tensor parallelism for fast inference on multiple GPUs, and production-ready logging and tracing.

You can deploy Mixtral on Hugging Face's [Inference Endpoints](https://ui.endpoints.huggingface.co/new?repository=mistralai%2FMixtral-8x7B-Instruct-v0.1&vendor=aws&region=us-east-1&accelerator=gpu&instance_size=2xlarge&task=text-generation&no_suggested_compute=true&tgi=true&tgi_max_batch_total_tokens=1024000&tgi_max_total_tokens=32000), which uses Text Generation Inference as the backend. To deploy a Mixtral model, go to the [model page](https://huggingface.co/mistralai/Mixtral-8x7B-Instruct-v0.1) and click on the [Deploy -> Inference Endpoints](https://ui.endpoints.huggingface.co/new?repository=meta-llama/Llama-2-7b-hf) widget.

*Note: You might need to request a quota upgrade via email to **[api-enterprise@huggingface.co](mailto:api-enterprise@huggingface.co)** to access A100s*

You can learn more on how to **[Deploy LLMs with Hugging Face Inference Endpoints in our blog](https://huggingface.co/blog/inference-endpoints-llm)**. The **[blog](https://huggingface.co/blog/inference-endpoints-llm)** includes information about supported hyperparameters and how to stream your response using Python and Javascript.

You can also run Text Generation Inference locally on 2x A100s (80GB) with Docker as follows:

```bash
docker run --gpus all --shm-size 1g -p 3000:80 -v /data:/data ghcr.io/huggingface/text-generation-inference:1.3.0 \
	--model-id mistralai/Mixtral-8x7B-Instruct-v0.1 \
	--num-shard 2 \
	--max-batch-total-tokens 1024000 \
	--max-total-tokens 32000
```

## Fine-tuning with 🤗 TRL

Training LLMs can be technically and computationally challenging. In this section, we look at the tools available in the Hugging Face ecosystem to efficiently train Mixtral on a single A100 GPU.

An example command to fine-tune Mixtral on OpenAssistant’s [chat dataset](https://huggingface.co/datasets/OpenAssistant/oasst_top1_2023-08-25) can be found below. To conserve memory, we make use of 4-bit quantization and [QLoRA](https://arxiv.org/abs/2305.14314) to target all the linear layers in the attention blocks. Note that unlike dense transformers, one should not target the MLP layers as they are sparse and don’t interact well with PEFT.

First, install the nightly version of 🤗 TRL and clone the repo to access the [training script](https://github.com/huggingface/trl/blob/main/examples/scripts/sft.py):

```bash
pip install -U transformers
pip install git+https://github.com/huggingface/trl
git clone https://github.com/huggingface/trl
cd trl
```

Then you can run the script:

```bash
accelerate launch --config_file examples/accelerate_configs/multi_gpu.yaml --num_processes=1 \
	examples/scripts/sft.py \
	--model_name mistralai/Mixtral-8x7B-v0.1 \
	--dataset_name OpenAssistant/oasst_top1_2023-08-25 \
	--batch_size 2 \
	--gradient_accumulation_steps 1 \
	--learning_rate 2e-4 \
	--save_steps 20_000 \
	--use_peft \
	--peft_lora_r 16 --peft_lora_alpha 32 \
	--target_modules q_proj k_proj v_proj o_proj \
	--load_in_4bit
```

This takes about 9 hours to train on a single A100, but can be easily parallelised by tweaking `--num_processes` to the number of GPUs you have available.

## Disclaimers and ongoing work

- **Quantization**: Quantization of MoEs is an active area of research. Although TheBloke has done initial experiments to achieve 4-bit and 8-bit quantization, the model quality degrades significantly. It will be exciting to see the development in the coming days and weeks in this area. Additionally, recent work such as [QMoE](https://arxiv.org/abs/2310.16795), which achieves sub-1-bit quantization for MoEs, could be applied here.
- **High VRAM usage**: MoEs run inference very quickly but still need a large amount of VRAM (and hence an expensive GPU). This makes it challenging to use it in local setups. MoEs are great for setups with many devices and large VRAM. Mixtral requires 90GB of VRAM in half-precision 🤯

## Additional Resources

- [Mixture of Experts Explained](https://huggingface.co/blog/moe)
- [Mixtral of experts](https://mistral.ai/news/mixtral-of-experts/)
- [Models on the Hub](https://huggingface.co/models?other=mixtral)
- [Open LLM Leaderboard](https://huggingface.co/spaces/HuggingFaceH4/open_llm_leaderboard)
- [Chat demo on Hugging Chat](https://huggingface.co/chat/?model=mistralai/Mixtral-8x7B-Instruct-v0.1)

## Conclusion

We're very excited about Mixtral being released! In the coming days, be ready to learn more about ways to fine-tune and deploy Mixtral. 

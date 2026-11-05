# Literature findings

Recorded 5 November 2026, before any experiment in this repository. This file is what the study is allowed to treat as already known. Figures below are from the cited papers and model cards. Where a source was a secondary write-up and was not re-checked on the live leaderboard, it is omitted.

## The question this study is allowed to ask

A joint text-and-image embedding space is already a product category. Cutting the language trunk of that space, and baking a system prompt into the indexed vector, are not. Video in this study is a transfer test of a text-photo model, not a claim that a video embedder has been trained.

## Tail pruning on text decoders

*LLMs Are Overparameterized Text Encoders* drops the last \(p\%\) of layers of a raw decoder, then runs 1,000 steps of LoRA supervised contrastive training on the public E5 mixture. Attention stays causal. Pooling is weighted mean. The benchmark is MTEB-15, the 15-task subset used by LLM2Vec, not the full MTEB. There is no published score for the pruned model before that fine-tune. The truncation keeps the prefix and drops the tail, including the original final layer. That is a different cut from methods that drop late layers but keep the final layer.

On Llama-3-8B-Instruct the finetuned full model is about 65.0 on MTEB-15. Fifty percent pruning scores 63.10. Eighty percent pruning scores 59.69, which the introduction rounds to the line “64.9 to 59.8.” Mistral-7B stays within about 0.1 through 30% removed. Qwen2-7B gains 0.32 at 10% removed and then drops at 30%. Phi-3 is nearly flat through 20% and falls once the model is under about 2B. The gentle part of the curve is a 7–8B result. A 1B or 1.7B trunk sits in the band where that paper’s smaller model breaks.

Source: [LLMs Are Overparameterized Text Encoders](https://arxiv.org/abs/2410.14578), also [RepL4NLP 2025](https://aclanthology.org/2025.repl4nlp-1.13/).

## Where a frozen decoder should be read

The last layer of a raw pretrained decoder is often a weak semantic readout. That does not pick one depth for every family.

*Token Prepending* (arXiv 2412.11556) says the final layer is consistently suboptimal for STS. On LLaMA2-7B, PromptEOL scores 70.03 Spearman and prepending raises that to 77.19. The readout they publish for PromptEOL is layer 27. Prepending stops after layer 8. Those are different depths. The URL that was labeled Token Prepending in the original proposal, arXiv 2505.12831, is *Contrastive Prompting*, which also reads an intermediate layer and cites Token Prepending.

The page that was labeled DeeLM, arXiv 2311.05296, is *BeLLM*. On SentEval STS it finds a turning point at the penultimate layer and makes only that last attention layer bidirectional. It does not describe a middle layer, and it does not make the rest of the stack bidirectional.

*Layer by Layer* (arXiv 2502.02013) runs frozen models, including decoder feature taps, on 32 MTEB tasks and finds an intermediate layer ahead of the final layer on nearly every task, often by several points and often near mid-depth for autoregressive models. *Does Representation Matter?* (arXiv 2412.09563) reports the same pattern on its own 32-task slice, including an LLM2Vec 8B last-layer average of 64.7 against a best-layer average of 66.8. These two papers should stay separate. Their numbers are not one result.

*KV-Embedding* (January 2026, arXiv 2601.01046) is training-free. Useful layers were about 12–21 of 36 for Qwen3-4B and about 13–19 of 32 for Mistral-7B. Llama-3.1-8B’s low-intrinsic-dimension layers included layers 26–31 of 32, so a “drop the last 30%” rule fights that model. Flipping a frozen causal model to bidirectional attention without training scored below a plain last-token baseline. Absolute training-free scores stayed well below models trained as embedders.

PromptEOL’s template is the last-token state of `This sentence: “[text]” means in one word:“`. The original paper evaluates a full forward pass. It also shows decoder sentence spaces growing more anisotropic as OPT scales, and it treats contrastive training as the mitigation. A cosine ranking of raw last-token states can track isotropy. For a training-free layer probe, whitening belongs on each layer’s pooled vectors, fit without the test split.

Sources: [Token Prepending](https://arxiv.org/abs/2412.11556), [Contrastive Prompting](https://arxiv.org/abs/2505.12831), [BeLLM](https://arxiv.org/abs/2311.05296), [Layer by Layer](https://arxiv.org/abs/2502.02013), [Does Representation Matter?](https://arxiv.org/abs/2412.09563), [KV-Embedding](https://arxiv.org/abs/2601.01046), [PromptEOL](https://arxiv.org/abs/2307.16645).

## TILE is a tabular probe

TILE probes every layer and ships a truncated checkpoint. The README keeps Llama-3.2-1B at 10 of 16 layers, Llama-3.1-8B at 14 of 32, and Qwen3-8B-Base at 19 of 36. The selector is a Ridge on pooled cell states over 27 tables, with no LLM fine-tune. Several published cuts are one of the top three layers rather than the unique best. Depths fall between about 40% and 63% of the network. They are not a retrieval cutoff, and they are not a 60–70% rule. This study does not import a TILE layer index.

Sources: [TILE repository](https://github.com/soda-inria/tile), [TILE paper](https://arxiv.org/abs/2610.17065).

## Models trained as embedders use the full stack

Qwen3-Embedding reads the last-layer hidden state at the end-of-sequence token under causal attention. The authors reported that causal attention beat a bidirectional variant in early runs. Official June 2025 means are multilingual 64.33 / 69.45 / 70.58 and English v2 70.70 / 74.60 / 75.22 for the 0.6B, 4B, and 8B. The license is Apache 2.0. Compression is Matryoshka dimensions. The 0.6B is natively 1024, usable from 32 upward. Query instructions are worth about 1–5%. Documents stay unprefixed so the index is built once. There is no Qwen3-Embedding-1.7B. Qwen3-1.7B is a post-trained causal chat model, 28 layers, hidden size 2048, with a thinking mode. It is the wrong frozen trunk for this study: it has no vision encoder.

Llama-Embed-Nemotron-8B starts from Llama-3.1-8B, replaces the causal mask with bidirectional attention, fine-tunes all weights, and mean-pools the final layer. On multilingual MTEB it was Borda rank 1 at mean 69.46 on 21 October 2025, while Qwen3-Embedding-8B had the higher mean (70.58). EmbeddingGemma-300M mean-pools a bidirectional encoder and then projects. Its English v2 mean is 69.67 and its multilingual mean is 61.15 at 768 dimensions, under the Gemma license. Its truncation is dimensions, plus a term against cone collapse.

A May 2026 study of 25 public embedders, including Qwen3-Embedding, finds decoder embedders peak late. Inference-time block skipping that they consider protects the first three and the last three layers. Dropping the deepest blocks is the harmful baseline. FastE, on models that already read out from the end of the network, compresses prefix tokens in deeper layers and still sends the readout through the full depth.

Two statements stay separate. On a raw decoder, some layer other than the last is often a better frozen feature, and tail layers can be deleted if the remaining stack is then contrastively trained. On a model whose loss already trained an embedding, the final layer is the product.

LLM2Vec does not truncate. It removes the causal mask, runs masked next-token prediction, then unsupervised SimCSE, and mean-pools. Dropping the mask without that masked-prediction step drops Llama-3-8B from 43.98 to 30.56 on full MTEB. Mistral is the exception and improves from the mask flip alone. Unsupervised Mistral-7B then averages 56.80. A further supervised stage reaches about 65 on full MTEB for Llama-3-8B. That 65 and the pruning paper’s 64.9 are different benchmarks. The 1,000-step counts in LLM2Vec and in E5-Mistral are LoRA on a full stack. E5-Mistral’s short step count is a batch of 2,048 on about 1.8 million examples, with the instruction on the query only. A frozen projection head on a truncated decoder is not those results. It is the negative control in this study.

Sources: [Qwen3-Embedding](https://arxiv.org/abs/2506.05176), [Qwen3-Embedding issue on causal attention](https://github.com/QwenLM/Qwen3-Embedding/issues/9), [Qwen3-1.7B card](https://huggingface.co/Qwen/Qwen3-1.7B), [Llama-Embed-Nemotron-8B](https://arxiv.org/abs/2511.07025), [EmbeddingGemma](https://arxiv.org/abs/2509.20354), [Layer-wise Representation Dynamics](https://arxiv.org/abs/2605.12714), [FastE](https://arxiv.org/abs/2609.08407), [LLM2Vec](https://arxiv.org/abs/2404.05961), [E5-Mistral](https://arxiv.org/abs/2401.00368).

## Joint text and media spaces already exist

Llama-Nemotron-Embed-VL-1B-v2 is a fine-tune of the full Llama 3.2 1B (16 layers, hidden size 2048, language backbone about 1.23B) plus SigLIP2-400M in an Eagle 2 tiling setup. Pooling is a mean over the language model’s output tokens, to 2048 dimensions. Queries on the card are text. Documents can be text, a page image, or image plus OCR. The card does not describe a truncated trunk. The Build card’s older three-set Recall@5 average is 80.9%. The Hugging Face card’s later visual-page averages are about 71–73% depending on text, image, or image-plus-OCR. Text-chunk Recall@5 stays level with the text-only 1B sibling (67.42 versus 67.19). Those two cards use different sets. They are not one number.

Qwen3-VL-Embedding (2B and 8B) is a single vector from the last end-of-text state, causal, with Matryoshka dimensions, and it accepts text, image, video, or a mix. The paper’s MMTEB mean for the 8B is 67.9, against 70.6 for text-only Qwen3-Embedding-8B. GME, VLM2Vec, and jina-embeddings-v4 also put text and images in one model. jina-embeddings-v4 offers both a dense vector and late interaction, under the Qwen research license. ColModernVBERT is a small late-interaction document model under MIT.

Nemotron ColEmbed V2 trains the same recipe as a pooled vector and as late interaction. Late interaction leads by about 9–12% relative nDCG@10 on ViDoRe v3. The pooled index is a few gigabytes per million pages in their 1B comparison. The late-interaction cousin is thousands of gigabytes per million, with the caveat that the pair is not a pure storage ablation. A pooled shortlist plus a reranker closes most of that gap in their 1B setup. This study uses one vector because the claim is a comparable search vector, not a page-retrieval leaderboard. Document charts, OCR, and ViDoRe are out of scope here. The media are photos and video frames.

MM-Matryoshka early-exits the language encoder of ColPali-style multi-vector retrievers. An untrained middle layer of public ColPali keeps 53.8% of that model’s own final-layer ViDoRe v1 score and 37.4% of v2. After layer-wise score distillation the same budget keeps 98.7% and 91.0%. Removing the distillation loss drops middle-layer nDCG@5 by about 30 points. No paper found in the 5 November 2026 search drops language layers from a mean-pooled vision-language embedder and recovers image-text alignment. Text-only prune numbers do not license that cut. This gap is H1 and H2.

Sources: [Nemotron VL Build card](https://build.nvidia.com/nvidia/llama-nemotron-embed-vl-1b-v2/modelcard), [Nemotron VL weights](https://huggingface.co/nvidia/llama-nemotron-embed-vl-1b-v2), [Qwen3-VL-Embedding](https://arxiv.org/abs/2601.04720), [Nemotron ColEmbed V2](https://arxiv.org/abs/2602.03992), [MM-Matryoshka](https://arxiv.org/abs/2606.07654).

## Index-time prompts

E5 and Qwen3-Embedding instruct the query and leave the document bare. Qwen3-Embedding’s own evaluation puts the query-instruction effect at about 1–5%, and recommends English instructions. That design exists so one index can serve many query wordings. It does not measure a system prompt written into the indexed vector. A modality label, a task sentence, and a mismatched sentence on the indexed side are the open factor. They are H3. The query prefix stays fixed so the contrast is the index prompt.

## What was considered and set aside

Jev, as of 2026, is a categorical decision readout: a fixed option set in, a distribution over those options out. Open clones describe a head on a frozen next-token trunk as a weak plumbing check, and train LoRA with the head. *LLM-as-Jev* argues a small Qwen already behaves like this decision model with a training-free logit readout. GritLM shows one unfrozen text model can be trained jointly for generation and embedding. None of that is a vision-language embedding result. A second decision head is out of scope.

Matryoshka nested dimensions are a real way to serve several widths from one vector. *2D Matryoshka Sentence Embeddings* (arXiv 2402.14776) trains shallow layers to match the last layer. It is evidence that early exit can be trained, not evidence that an untouched middle layer is already the embedding. Dimension nesting is not this study’s claim.

Serving notes, recorded so they are not rediscovered and then built by accident. Text Embeddings Inference documents `cls`, `mean`, `last-token`, and `splade`, and lists Qwen3. Mean pooling does not remove a causal mask. vLLM’s embed task covers causal Qwen3 and has separate classes for bidirectional embedders. A custom cut trunk is not a serving target of this study. Int8 quality for such a trunk was not found in the sources read on 5 November 2026, and no number is invented here.

## Consequences for the protocol

- The backbone is a small open vision-language model that returns every hidden state. Qwen3-1.7B is excluded.
- The layer is measured on this model. TILE’s 10-of-16 and Token Prepending’s layer 27 are not hypotheses.
- Whitening is part of the training-free probe. Contrastive LoRA is the adaptation if the probe passes. A frozen head is the negative control.
- A released vision-language embedder is a ceiling reported beside the result. Beating it is not a success criterion.
- Video is frame-mean transfer with no video loss. The experiment order and the branches that stay out of this paper are in [../protocol/phases.md](../protocol/phases.md).

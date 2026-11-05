# VLMRAG

A feasibility study for a paper. The question is whether a vision-language model with a shortened language trunk can embed text and photos as comparable search vectors, whether a system prompt at indexing changes retrieval, and whether that space transfers to video by mean-pooling frames.

Nothing in this repository has been trained or scored. There is no experiment code yet.

## Read this first

- [Literature findings](docs/findings/literature-review.md) — what is already published, which citations in the original proposal point at the wrong papers, and which claim is still open. Recorded 5 November 2026.
- [Study protocol](docs/protocol/study-protocol.md) — draft pre-registration. Hypotheses, the stop rule, and the three items still open.
- [Phases and branches](docs/protocol/phases.md) — what runs in which order, and which follow-ups are not this paper.
- [Work register](docs/work/README.md) — stories US-001 through US-008.

## What is already settled

Joint text-and-image embedders exist. Cutting their language trunk is not a published pooled-vector result, and index-time prompts are not what E5 and Qwen3-Embedding measured. Video in this study is a zero-shot frame-pool test, not a trained video model.

## What is not in scope

Beating a released embedder, multi-vector late interaction, document-page retrieval, quantization, a serving stack, and a second decision head.

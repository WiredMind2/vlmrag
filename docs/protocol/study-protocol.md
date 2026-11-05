# Study protocol

**Status: draft pre-registration. No split has been scored. No experiment code is in the repository.**

This note may be edited until the first test-split evaluation. After that run, hypotheses, margins, splits, prompt strings, frame count, and the stop rule stay as written. Exploratory cuts go in an appendix and are labeled as such.

The literature this protocol is not allowed to re-claim is in [../findings/literature-review.md](../findings/literature-review.md). The order of experiments and the branches that stay out of the paper are in [phases.md](phases.md).

## Question

Can a vision-language model with its language trunk cut short of the last layer embed text and photos as vectors of one dimension, in one cosine index, and do system prompts applied at indexing move retrieval? Does that photo-text space transfer to video when a fixed number of frames are mean-pooled and no video loss is trained?

“Comparable” means one dimension, cosine similarity, and one index. A text query retrieves its paired photo. A text query retrieves its paired text. Video transfer beats chance. A separate index per modality fails the claim even if each index is accurate.

## Locked decisions

These were set on 5 November 2026.

- The end product is a paper. Serving, quantization, and a leaderboard run are out of scope.
- The study is staged. The training-free probe and the prompt factor come first. Training starts only if H1 passes.
- Video is zero-shot transfer. No text-video pairs and no video loss.
- The negative control for adaptation is a projection head on a frozen trunk, not the method.
- Qwen3-1.7B is not the backbone. It has no vision encoder, and it is a post-trained chat model.
- TILE layer indexes are not imported.
- Document-page RAG, OCR, charts, and late-interaction indexes are out of scope. Media are photos and video frames.
- Photos are the COCO Karpathy split. Flickr30k is the fallback only if COCO cannot be used. Layer selection uses 1,000 image-text pairs drawn from the official dev split before any score. The official test split is held out and is not used to pick a layer, a pooling, or a prompt.
- The text control is the public SciFact test split, embedded with the same model as the photos.
- Video uses 8 frames, sampled uniformly, the same count for every MSR-VTT clip.
- Layer selection uses the empty index prefix. A prompt cannot choose the layer.
- If the photo prompt effect is null, video is scored with the empty index prefix only. If it is not null, video is scored under all four prefixes. That branch adds no video loss.
- Backbone attempt order, once the one-batch check runs: Qwen2.5-VL-3B, then Qwen3-VL-2B, then Llama-Nemotron-Embed-VL-1B-v2. Keep the first whose forward pass returns every language-trunk hidden state. If none do, the study stops.

## Hypotheses

- **H1, readout.** At least one layer short of the final layer keeps held-out text-to-photo Recall@10 at or above 90% of the final layer’s Recall@10. Whitening is fit on the dev split only, per layer. The layer is chosen on dev. The 90% comparison is on the held-out split.
- **H2, adaptation.** If H1 passes, contrastive LoRA on the cut trunk, vision encoder frozen, with the same data, steps, and seed budget as a full-depth run, reaches within 5 Recall@10 points of that full-depth run on the held-out text-to-photo split. A released small vision-language embedder is reported as an external ceiling. Matching or beating that ceiling is not required.
- **H3, index prompts.** The query prompt is fixed. Four document prompts are all reported: none, modality label, task instruction, mismatched instruction. The contrast is a paired bootstrap of nDCG@10 against the empty index prompt. No condition is dropped after the test numbers are seen.
- **H4, video transfer.** The mean of 8 uniformly sampled frames from each video, embedded by the photo-text model with no video training, beats chance Recall@10 on the MSR-VTT test split.

## Stop rule

If H1 fails, H2 shrinks to one negative-control training run at the best weak layer, and training stops. The paper reports the failed cut. If H3 is null, the paper says index-time prompts did not move retrieval under this design. H4 is still scored on the checkpoint the study actually keeps (the final layer if no earlier layer passed, otherwise the chosen layer).

## What will be measured

Primary photo endpoint: Recall@10. Primary prompt endpoint: nDCG@10 difference versus the empty index prompt. Secondary: Recall@1 and Recall@5. The text-to-text slice uses the same vectors, so a cut that only preserves photos is visible. No full MTEB. No ViDoRe.

## Procedure

Phase 0 has not run. Do not use a dev score to choose a layer until the loaded backbone is written into this file.

1. One batch through the backbone order above. Record the checkpoint and whether it is a raw vision-language model or an already-trained embedder. Stop if none return every hidden state.
2. On the 1,000-pair dev draw, under the empty index prefix, pool each layer two ways: mean over trunk tokens, and last token. Fit PCA whitening per layer on those dev vectors. Rank by dev text-to-photo Recall@10. Freeze the readout and the layer before the test split.
3. Score H1 on the held-out photo split. Score SciFact with the same vectors. Report the full layer curve.
4. At the chosen layer and at the final layer, run all four index prompts. The query prefix stays the pre-registered string. One extra cell removes the query prefix and is not used to pick a prompt or a layer.
5. If H1 failed, train one projection head on the frozen trunk at the best weak layer, then stop training. If H1 passed, run cut-trunk LoRA, full-depth LoRA, and the frozen-head control on the same image-text data, steps, and seed budget, vision encoder frozen. Score a released small vision-language embedder once as a ceiling.
6. Embed MSR-VTT with 8 frames, mean-pool them, and score H4 on the checkpoint this study kept. Use the empty index prefix when H3 is null, and all four prefixes when it is not. Do not train on MSR-VTT.

## Proposed prompt strings

Proposed only. They become frozen at the first test run. They are English, one sentence, and identical across modalities except the modality label.

Query, held fixed:

`Represent this search query for retrieving a matching item:`

Indexed item:

| Condition | Prefix |
| --- | --- |
| None | No prefix. |
| Modality label | `text`, `photo`, or `video`, matching the item. |
| Task instruction | `Represent this item so a search query can find it.` |
| Mismatched | `Write a detailed caption of this item:` |

For a photo or a frame, the prefix is text beside the image in the model’s own chat template. The template is whatever the chosen backbone documents. It is not redesigned per condition.

## Still open

The dataset rules above are set. These items are not, and they are written here before the run that needs them.

- **Which backbone loaded.** The attempt order is fixed. The one-batch check has not been run, so no checkpoint is named yet. When it is, this file also records the license and whether the checkpoint is a raw vision-language model or an already-trained embedder. Those two answer different questions.
- **Training mixture, step count, and seeds.** One seed for the H1 gate. Three seeds only for the training comparison in the main table. The step count and the public image-text mixture are written here before that training run, and they match across the cut trunk and the full-depth control.
- **Chance level for H4.** The MSR-VTT test-pool size determines chance Recall@10. The paper states the pool size and the chance value next to the score. The pool is not subsampled after seeing H4.

## Limitations the paper will state

One consumer GPU. Small public splits. Frame-mean video, not a temporal model. No memory figure until a run measures it. A null truncation or a null prompt effect is a finding.

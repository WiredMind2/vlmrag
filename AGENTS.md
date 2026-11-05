# VLMRAG

Cross-modal feasibility study. The question is whether a vision-language model with a shortened language trunk can embed text and photos as comparable search vectors, whether index-time system prompts move retrieval, and whether that photo-text space transfers to video by mean-pooling frames. The end product is a paper. No experiment has been run.

## Work register

Stories live in `docs/work/items/`. Read `docs/work/README.md` before adding work. Scan `docs/work/archive/` before minting an id. Do not delete `docs/agile/` or `docs/work/archive/`.

## Study documents

- Findings to date: `docs/findings/literature-review.md`
- Protocol (draft, nothing scored): `docs/protocol/study-protocol.md`
- Phase tree: `docs/protocol/phases.md`

Do not add training or evaluation code until the user asks to implement. Later phases run only on the branch the earlier result opened. The loaded backbone is still unnamed; US-007 is the one-batch check that names it.

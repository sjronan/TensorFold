# sm_103 EXL3 grouped default

The load-ahead loop reduced GLM EXL3 short-prompt throughput on GB300. Keeping
the existing plain grouped loop recovered the loss with identical reply IDs.
The sm_120+ selection and the consumer checkpoint-FP4 admission rule remain
unchanged. No arithmetic or tolerance changes were made.

`screen.json` pins the public checkpoint bytes, prompts, settings and raw
round aggregate rates. Base/candidate/candidate/base used fresh model
processes, each with one excluded warm round and five retained rounds per
cell. All four prompts produced 128 greedy tokens at context 8192, BF16 KV,
with no prompt cache. C1 sent the prompts sequentially; C4 sent all four
simultaneously. All output ID arrays matched across source heads, draft
modes and solo/concurrent runs. Native token fingerprints use decimal IDs
joined by commas, then SHA256.

For C4 with MTP+copies, base loads returned 203.92 [203.81,204.02] and
204.74 [204.48,205.01] aggregate tok/s. Candidate loads returned 189.75
[189.67,189.84] and 189.10 [188.89,189.31]. These are t95 intervals over five
complete four-prompt rounds in each load; they describe within-load repeat
variation. Active samples stayed at 2070 MHz with no throttle flags. The
512-token release benchmark, C16, cold long prefill and tasks are separate
measurements; these short screening intervals do not replace them.

To isolate the cause, a diagnostic server used the candidate host sources
and 148 unchanged candidate fatbins, replacing only exl3_experts.fatbin with
the same-toolchain base image. It returned 204.53 aggregate tok/s in the same
C4 drafted cell and preserved every checked ID. The JSON contains that
cell's interval and the other diagnostic cells. This composed build is a
diagnostic receipt; canonical source-build qualification is recorded
separately by the operator.

The source change retains the existing plain grouped implementation on
sm_103. The instruction's availability alone does not establish that the
lookahead policy is a faster default on that chip. Broader GB300 qualification
remains specific to the exact build, checkpoint and precision recipe.

The matching server recipe is:

```sh
tensorfold serve Mia-AiLab/GLM-5.3-Flash-EXL3-4bpw-TensorFold \
  --context 8192 --parallel 4 --prompt-cache-gib 0 --no-thinking --name bench
```

Send the JSON's four prompts as user messages with its request parameters,
first sequentially and then with four clients, for both draft modes. Retain
one excluded warm round and five measured rounds per cell. Aggregate rate
is 512 completion tokens divided by the round's client wall time. Original
requests used the same research-only LiteLLM 1.104.1 route for every arm, so
that wall time includes routing/network overhead. Check returned ID arrays
against reference_ids; cache counts must be 0. The exact raw per-round rate
arrays and their interval calculations are in the JSON.

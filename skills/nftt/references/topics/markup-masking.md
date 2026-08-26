# Markup and masking — templates, markers, think modes

Training supervises the **assistant turn only**. Which tokens that
means — and whether think data reaches training at all — is decided by
the model's **chat template**, not by any hardcoded marker set.

**Maintaining this file:** verbatim-synced into the consuming-project
skill; no development-only references; must stand alone.

## Declaring a non-ChatML base

Two keys exist that the init scaffold does not print: `model.markers`
(see below) and `model.chat_template` (see below). `nftt config`
documents both — a base outside the Qwen/ChatML family starts with
checking whether they apply to you. The default `model.lora_targets`
(null = the standard attention/MLP projection set: q/k/v/o/gate/up/down
proj) resolved fine on a Ministral base in practice; the resolved set
is recorded in train provenance, so the default is a reasonable first
attempt for common architectures.

**Seeing the base template you are extending:** when the base's
tokenizer is locally cached (or a local-dir base), `nftt status --json`
carries the templates — `chat_template.base` is the embedded template
(the one to copy and extend), `chat_template.resolved` is what training
will use, and `override_declared` tells them apart. Text-mode status
prints the embedded size and points at `--json`. This is the sanctioned
surface — no hand-fetching `tokenizer_config.json` from a guessed repo.
When the tokenizer is not cached yet, status skips silently (it never
downloads for this); any train or export run warms the cache.

## Masking: template diff (default) or declared markers

- **Default (template diff):** the full conversation and the prompt-only
  prefix are rendered through the same template; the supervised span is
  everything after the prefix. This requires the template to be
  **prefix-stable** (the prompt-only render is a token-identical prefix
  of the full render) — true for the common families.
- **Fallback (`model.markers`):** for templates the diff cannot serve,
  declare explicit markers — a flat object; declaring it replaces the
  diff entirely:

```jsonc
"model": { "markers": {
  "assistant_start": "<|assistant|>",  // required pair
  "assistant_end": "</s>",
  "think_start": "[THINK]", "think_end": "[/THINK]"  // optional pair
} }
```

Declared markers are not only a fallback: even under a stable diff they
can be the better choice — they pin the supervision span explicitly and
match a serve format you know (`[INST]`…`[/INST]`-style bases), instead
of trusting template evolution between versions.

Markers are **token sequences** — single-token (the usual case) or
multi-token (a Llama-style `<|start_header_id|>assistant<|end_header_id|>`
boundary resolves as the sequence of its tokens and matches exactly, in
order). Prefer special-token boundaries: a marker built from ordinary
tokens can coincide with user content and silently misplace the
supervised span, and no probe can catch content-dependent occurrences.
A marker that does not resolve to a vocabulary token sequence aborts at
load; a resolved sequence that does not delimit the rendered assistant
span aborts at the probe — both name the marker and the model.

Think mode `strip` resolves think markers too — they default to
`<think>`/`</think>` when not declared, and they MUST exist in the
base's vocabulary (a mismatch aborts at load time with both resolution
paths named).

**Where failures surface:**

- `nftt status` runs a best-effort probe when the base's tokenizer is
  already local: instability prints a **WARNING** naming the fallback
  (status never fails on markup), and the `--json` summary carries
  `markup_probe` as `skipped` / `stable` / `unstable`.
- `nftt train` hard-fails at load time — before any training work — on
  instability, on declared markers missing from the vocabulary, or on
  strip-mode think markers the vocabulary lacks. No raw tracebacks.
- A row that breaks the prefix invariant aborts training by name.

**Multimodal bases:** some bases return a processor wrapper instead of
a tokenizer from the loader; the trainer unwraps it transparently —
anything else aborts at load time, named.

## Think modes and the attach-key contract

`model.think` decides what happens to think data at training time:

- `strip` — think content (template-emitted or in-field) removed from
  targets with real token ids; training aborts if any think token
  survives.
- `keep` — think text in content passes through untouched. **Caveat:**
  with `keep` + a declared `task.think_field`, the field itself is
  silently ignored (nothing reads it in keep mode) — the mode is honest
  about content, not about fields.
- `render-from-field` — the field's payload is attached to the
  assistant message and the chat template renders it (reasoning-model
  repair lanes).

**The attach-key rule (structural, every family):** stock chat
templates render think from inline content or typed chunks — *no* base
renders an arbitrary side field, because side think fields are a row-
contract concept unknown to transformers templates. And the trainer
attaches the payload under the literal `"thinking"` message key,
**regardless of the declared `task.think_field` name**. So
`render-from-field` requires a template that renders the `thinking`
message key — a custom template guarding the declared field name
(`message.get('reasoning')`) renders nothing: the guard is always
falsy, and the run trains answer-only.

This is enforced at train load: the think-render probe renders one
built row through the resolved template with and without the payload
and aborts (`think-field-not-rendered`) when the template ignores the
`thinking` key — both loss shapes (stock template ignoring it; custom
template keyed on the wrong name) are caught before any GPU spend, and
the error states the contract plus both resolution paths.

## Custom training templates

A declared `model.chat_template` shapes **training only** — it cannot
ride the export to serving (see `exports.md` for identity recording and
the divergence warning). Verify your template matches serve shape
yourself: render the same conversation through your template and
through the served model, then diff the byte shape; byte-identical
prompts mean train and serve agree even though recorded identity
hashes differ.

# Bringing your own data

nFTT never models where data comes from. Everything a project trains on
enters as jsonl files you deliver, in a shape declared once in `task`.
This reference is split by topic — one fetch answers one question:

| topic | file |
|---|---|
| Row shapes, think fields, extras, last-exchange view | `topics/rows.md` |
| Pipeline stages, prompts, protocols, tables, generate design | `topics/pipeline.md` |
| Templates, markers, think modes, the attach-key contract | `topics/markup-masking.md` |
| Serve options, eval log anatomy, record schema, ground-truth mode | `topics/eval-logs.md` |
| GGUF/Modelfile, base identity, the export load cap | `topics/exports.md` |
| Validators, decontamination, system_parity, gates | `topics/decontam-gates.md` |

Worked examples use a toy task ("uppercase a word via shell");
substitute your domain.

**Maintaining these files:** the consuming-project skill bundles this
stub and every topic file verbatim as its end-user reference — sync is
a plain copy per file, no edits. Do NOT add development-only
references (design-decision records, internal change logs, source-file
names): end users of the skill have no way to read them. Each file must
stand alone; the installed package's `docs/`, `nftt docs`, and
`nftt <sub> --help` are the only things they may point to.

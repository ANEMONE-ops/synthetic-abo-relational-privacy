# Synthetic ABO Relational Privacy Corpus

**[Download the four dataset splits (ZIP)](synthetic-abo-relational-privacy-v1.0.0.zip).** Extract the archive and run the example below from the extracted folder. Paths in the tables refer to files inside the ZIP.
Chinese natural-language requests for compiling structured queries over synthetic donor–recipient ABO records. The corpus accompanies *Medical Collaboration—All-Branch Disclosure Control for Relational Privacy* by Zhe Yang, Jiazhe Wu, Yu Wang, and Chen Yan.

## Dataset splits

| File | Requests | Question families | Intended use |
|---|---:|---:|---|
| `data/train.jsonl` | 24,000 | 50 | Training pool |
| `data/dev.jsonl` | 2,400 | 20 | Development and checkpoint selection |
| `data/test.jsonl` | 6,000 | 100 | Main test with held-out question families |
| `data/test_ood.jsonl` | 2,400 | 100 | Test under held-out request strategies |
| **Total** | **34,800** | | |

The paper uses a fixed 6,000-row subset of the training pool. `reproduction/training-6000-ids.json` specifies the exact IDs and order. This deposit provides the four request splits.

## How the data were built

The generator expands controlled Chinese question templates with fixed context and request-strategy rules. The template catalog was developed with AI assistance; the generator builds each row and its gold label deterministically, without calling a language model during generation. The source files contain synthetic requests and contexts, with no patient records or source values or frequencies from UCI or other third-party datasets. Labels are derived from the generator's query rules.

Training, development and main test use distinct base question families. The main and held-out-strategy tests share 1,200 underlying question templates across 100 families, which isolates the effect of the request strategy. The held-out set includes tool-result injection, serialized role injection, forged policy attachments, nested forwarding, misleading examples and grading pressure. Report these two test sets separately. The medical framing supplies a setting for logical disclosure, and the labels describe record queries.

## Format and labels

Each line in the four split files is one UTF-8 JSON object.

| Field | Meaning |
|---|---|
| `id` | Stable row ID. |
| `split`, `cohort` | Data role and evaluation cohort. The held-out-strategy file uses `split=test` and `cohort=heldout_attack_strategies`. |
| `family_id`, `question_template_id` | Base question family and template identifiers for grouping and split checks. |
| `question`, `request` | Underlying question and full user request with context and any embedded instructions. Use `request` as model input. |
| `history` | Optional simulated prior messages, stored as an array (empty when absent). The paper's compiler uses the current request. |
| `gold_dsl` | Reference JSON object with a single `query` value; reserve for supervision or scoring. |
| `attack_strategy`, `attack_placement`, `context_id` | Strategy, location of the wrapper and synthetic context ID. |
| `negative_type`, `difficulty`, `query_structure`, `generation_rule` | Generator metadata for interpreting supported and unsupported requests and auditing the label. |

The 12 query labels are `donor` and `recipient` (retrieve one field); `pair` (retrieve the ordered pair); `same` and `different`; `donor_O`, `recipient_O`, `donor_AB`, `recipient_AB` (one-field membership checks); `both_O` and `either_O` (conjunction and inclusive disjunction); and `unsupported`. Unsupported requests include active clinical advice or mixed unsupported and record-query requests. A cancelled clinical question can leave a supported record query.

The ABO values, synthetic medical language and role claims are experimental inputs. They do not provide clinical advice, compatibility labels or permission to access patient information.

## Load the corpus

```python
import json
from pathlib import Path


def load_jsonl(path):
    with Path(path).open(encoding="utf-8") as f:
        return [json.loads(line) for line in f]


pool = {row["id"]: row for row in load_jsonl("data/train.jsonl")}
ids = json.loads(Path("reproduction/training-6000-ids.json").read_text(encoding="utf-8"))
paper_train = [pool[row_id] for row_id in ids]
main_test = load_jsonl("data/test.jsonl")
heldout_test = load_jsonl("data/test_ood.jsonl")
```

See `SHA256SUMS` for the distributed files and `reproduction/source-manifest-v3.json` for construction parameters, split audits and original checksums. The manifest also describes multi-turn scripts outside this deposit. `reproduction/training-6000-manifest.json` records the original training subset checksum. To match it, serialize each selected row with `ensure_ascii=False, sort_keys=True, separators=(",", ":")`, followed by a single LF, and encode as UTF-8.

## License and citation

The corpus and accompanying dataset documentation are available under [Creative Commons Attribution 4.0 International](LICENSE). Cite the dataset version used in your work. GitHub provides citation metadata from [CITATION.cff](CITATION.cff).

> Yang, Z., Wu, J., Wang, Y., & Yan, C. (2026). *Synthetic ABO Relational Privacy Corpus* (Version 1.0.0) [Data set]. GitHub. https://github.com/ANEMONE-ops/synthetic-abo-relational-privacy/releases/tag/v1.0.0

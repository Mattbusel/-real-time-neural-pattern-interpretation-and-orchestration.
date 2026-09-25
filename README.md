# Neural Intelligence System (Raven + Seraph)

A small Python prototype that chains OpenAI chat prompts into two "intelligences", one that interprets patterns and one that reviews the ethics of acting on them, and saves every result to a JSON memory store.

> **Status: exploratory Python prototype, one file, runnable with an OpenAI API key.** Despite the "neural pattern" framing, nothing here reads or decodes brain signals. The input is a text string (for example `"110010011001 - Synaptic burst encoding"`), and the "interpretation" is whatever GPT-4 writes back. Treat it as an experiment in prompt orchestration with an ethics-review step.

## Why this exists

This is the first code layer of a larger speculative framework in which an interpreting mind ("Raven") is always paired with an ethical reviewer ("Seraph"). The prototype tests the plumbing for that idea: every interpretation is automatically passed through an ethics review, and everything is written to disk so it can be searched later. The same Raven and Seraph names appear in [Raven-Intelligence-Model-v1-](https://github.com/Mattbusel/Raven-Intelligence-Model-v1-) and in the ANGELCORE PDF in [Mycelium-Based-AI-Integration](https://github.com/Mattbusel/Mycelium-Based-AI-Integration).

## What the code does

All of it lives in [`Neural Intelligence System.py`](./Neural%20Intelligence%20System.py).

| Class | What it actually does |
| --- | --- |
| `LLMAdapter` | Wraps `openai.ChatCompletion.create` (default model `gpt-4`), keeps a running conversation history, reads `OPENAI_API_KEY` from the environment |
| `MemorySystem` | Writes each result as a JSON file in `./memory_store/`, keeps the last 50 in memory, and supports `retrieve(id)`, `search(memory_type, keywords)` and an "active context" dict |
| `RavenIntelligence` | Prompts the model to interpret a pattern string (`interpret_pattern`), analyze a dict describing a system (`analyze_system_dynamics`), or compare two patterns (`compare_patterns`) |
| `SeraphIntelligence` | Prompts the model for an ethical evaluation (`evaluate_ethics`) or a value-alignment check (`analyze_value_alignment`), then regex-parses lines like `Concern: ...`, `Benefit: ...` and `Recommendation: ...` out of the reply |
| `OrchestrationEngine` | Runs Raven, then Seraph on the proposed action (`process_pattern_with_ethical_review`), or runs a system analysis and an ethics check per identified pattern, then labels the result "Proceed" or "Caution" (`analyze_system_with_safety_checks`) |

## Quick start

Requires Python 3.8+ and an OpenAI API key. The code uses the pre-1.0 OpenAI SDK interface (`openai.ChatCompletion`), so pin the SDK below 1.0:

```bash
git clone "https://github.com/Mattbusel/-real-time-neural-pattern-interpretation-and-orchestration..git" neural-intelligence
cd neural-intelligence

python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install "openai<1.0"

export OPENAI_API_KEY="sk-..."     # Windows PowerShell: $env:OPENAI_API_KEY="sk-..."
python "Neural Intelligence System.py"
```

The demo interprets one sample pattern, runs an ethics review on it, runs the orchestrated pipeline, then analyzes a four-node sample graph and prints how many patterns were judged safe. Every step makes GPT-4 calls, so it costs a few cents per run. Results land in `./memory_store/*.json`.

Note that the repository name starts with a dash and ends with a dot, so quote it on the command line, and clone into a folder with a normal name as shown above.

## Using it from your own code

The file name contains spaces, so it cannot be imported directly. Copy it to an importable name first:

```bash
cp "Neural Intelligence System.py" neural_intelligence.py
```

```python
from neural_intelligence import (
    LLMAdapter, MemorySystem, RavenIntelligence,
    SeraphIntelligence, OrchestrationEngine,
)

llm = LLMAdapter(model="gpt-4")          # reads OPENAI_API_KEY
memory = MemorySystem(storage_path="./memory_store")
raven = RavenIntelligence(llm, memory)
seraph = SeraphIntelligence(llm, memory)
orchestrator = OrchestrationEngine(raven, seraph, memory)

result = orchestrator.process_pattern_with_ethical_review(
    "110010011001 - Synaptic burst encoding - Phase alignment: Positive"
)
print(result["interpretation_result"]["interpretation"])
print(result["ethics_result"]["evaluation"])

# Search what has been stored so far
hits = memory.search(memory_type="pattern_interpretation", keywords=["synaptic"])
```

## How it works

```
pattern string
     |
     v
RavenIntelligence.interpret_pattern ---> GPT-4 (Raven system prompt)
     |
     v
action = "Trigger memory recall based on: <interpretation>"
     |
     v
SeraphIntelligence.evaluate_ethics ----> GPT-4 (Seraph system prompt)
     |                                    regex: Concern / Benefit / Recommendation
     v
MemorySystem.store ---------------------> ./memory_store/<type>_<timestamp>.json
```

## Limitations

- **No signal processing.** Inputs are free text; there is no EEG or neural data path.
- **The `confidence` value is a hard-coded placeholder** (`0.85`).
- **Ethics parsing is best-effort.** Concerns, benefits and recommendations are only extracted if the model happens to write lines starting with `Concern:`, `Benefit:` or `Recommendation:`. The "Proceed" or "Caution" label simply compares how many of each were found.
- **Shared conversation history.** Raven and Seraph share one `LLMAdapter`, so its history grows with every call and each prompt carries all earlier ones.
- **Old SDK.** Requires `openai<1.0`. The `base_url` option sets an attribute the old SDK ignores, so custom endpoints do not work without a code change.
- **Memory IDs are per-second timestamps**, so two stores of the same type in the same second overwrite each other.
- There are no tests or dependency files in the repository.

## Where the idea goes next

The longer-term vision, all unbuilt: bio-inspired and mycelial computing, DNA-based memory, symbolic reasoning modelled on human thought, and ethics built into an architecture from the start rather than bolted on.

## Contact

Questions or collaboration: mattbusel@gmail.com

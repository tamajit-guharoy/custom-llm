# LLM Training & Fine-tuning — Session Learnings

> A summary of concepts explored around building lightweight, specialized language models.

---

## 1. Model Distillation vs Fine-tuning

### Fine-tuning
Take an existing pretrained model and continue training it on your own domain-specific data. The model architecture and size **stay the same** — you're adjusting its weights to specialize it for your task.

- **Input:** Base model + your labeled examples
- **Output:** Same-sized model, now specialized
- **Goal:** Improve accuracy on a narrow domain
- **Analogy:** A general doctor doing a residency to become a cardiologist

### Distillation
Train a **smaller "student" model** to mimic the behavior of a larger "teacher" model. The student learns from the teacher's outputs rather than ground-truth labels alone.

- **Input:** Large teacher model + data
- **Output:** A **smaller, faster model** that approximates the teacher
- **Goal:** Compress capability into a lighter model
- **Analogy:** A senior expert writing a concise textbook so junior students can learn efficiently

### Comparison Table

| | Fine-tuning | Distillation |
|---|---|---|
| Changes model size? | No | Yes (smaller) |
| Needs a teacher model? | No | Yes |
| Needs labeled data? | Yes | Not always |
| Goal | Specialization | Compression |
| Result | Better at your task | Smaller & faster |

### Combined Approach (Most Practical)
Use a large model (e.g. DeepSeek-R1 70B via API) as a teacher to generate high-quality training data, then fine-tune a small model (e.g. DeepSeek-R1 1.5B) on that data. This is what DeepSeek itself did internally.

---

## 2. System Prompt vs Fine-tuning

### System Prompt Can Do
- Define tone, persona, response format
- Give instructions and rules
- Provide few-shot examples
- Set boundaries on what to answer

For simple, well-defined tasks a system prompt is often **enough and the right choice**.

### Where Fine-tuning Wins

| Reason | Explanation |
|---|---|
| Consistency | Behavior baked into weights — never "forgotten" mid-conversation |
| Token efficiency | No repeated prompt cost on every API call |
| Edge cases | 1000 examples teaches nuances that instructions can't capture |
| Hard-to-describe tasks | Showing examples works better than writing rules |
| On-device use | No API layer for system prompts — behavior must be in the weights |
| Speed & cost | Smaller fine-tuned models can outperform larger prompted models |

### Decision Framework
```
Can a system prompt solve it?
        │
        ├── Yes, reliably → Use system prompt (faster, cheaper)
        │
        └── No, because...
              ├── Too many edge cases        → Fine-tune
              ├── Need token efficiency      → Fine-tune
              ├── Running on-device          → Fine-tune
              ├── Behavior too subtle        → Fine-tune
              └── Need fresh/updated facts   → RAG
```

---

## 3. Fine-tuning for Behavior vs Facts

### Fine-tuning for Behavior ✅ (Reliable)
Behavior = *how* the model responds. Fine-tuning is very reliable here.
- Response format (e.g. always output JSON)
- Tone and persona
- Task structure (e.g. classify, extract, summarize)
- Reasoning style (e.g. step-by-step)

### Fine-tuning for Facts ⚠️ (Unreliable)
- Model may "learn" facts but hallucinate them at inference time
- Doesn't generalize well to combinations of facts
- Doesn't know when it doesn't know something

### The Right Tool for Each Goal

| Goal | Right Approach |
|---|---|
| New behavior / task format | ✅ Fine-tuning |
| New reasoning style | ✅ Fine-tuning |
| Domain-specific facts | ✅ RAG |
| Private / frequently updated data | ✅ RAG |
| Best of both | ✅ Fine-tuning + RAG combined |

### What is RAG?
Retrieval-Augmented Generation — instead of baking facts into weights, the model **looks up relevant information at query time** from a database or document store, then generates an answer grounded in that retrieved context.

---

## 4. Teaching a Model a New Programming Language

### The Core Insight
Programming language knowledge is **not facts** — it's **compressed patterns**. Java syntax is highly regular and compositional. The model learns structural patterns from repetition, not isolated facts.

| | Isolated Facts | Java Syntax |
|---|---|---|
| Example | "Einstein born 1879" | `for (int i=0; i<n; i++)` |
| Appears in data | Once or a few times | Millions of times |
| Compositional? | No | Yes — patterns combine |
| Fine-tuneable? | Unreliable | Works well |

### The Two-Stage Solution

#### Stage 1 — Continued Pretraining
Feed the model massive raw code with **no explanations** — just raw `.java` files.

```
Input:  Millions of Java files from GitHub / open-source repos
Scale:  Hundreds of millions to billions of tokens
Goal:   Model infers Java syntax, idioms, and patterns statistically
Format: Raw code — no instructions, no Q&A pairs
```

The model learns like a child learning language through immersion — it infers grammar rules from patterns without being taught them explicitly.

#### Stage 2 — Instruction Fine-tuning
Now teach it to follow coding instructions for your specific task.

```json
{
  "instruction": "Write a Java class for a bank account with deposit and withdraw methods",
  "output": "public class BankAccount { private double balance; ... }"
}
```

Here you can include both code tasks AND concept explanations.

### How the Model Learns Without Explanations
Seeing patterns like this millions of times:
```java
public class Dog extends Animal {
    private String name;
    public Dog(String name) { this.name = name; }
}
```
...the model statistically infers:
- `class` always has a name after it
- `extends` expresses a relationship between two classes
- `private/public` appear before type declarations
- Constructor name matches class name

No one explains "this is called inheritance" — it learns the pattern.

### Analogy
```
Continued Pretraining   =   Immersion in a foreign country
                            (hear the language constantly,
                             infer rules from patterns)

Instruction Fine-tuning =   Taking a conversation class
                            (learn to use the language
                             for specific purposes)
```

### Where RAG Fits
RAG alone won't help if the model doesn't know Java — it can't understand retrieved docs. But RAG is very useful **on top of** a Java-capable model for:
- Custom internal APIs and classes
- Specific library versions and method signatures
- Frequently updated documentation
- Proprietary frameworks

### Practical Shortcut
Instead of continued pretraining (expensive), **start with a model already trained on code**:
- **DeepSeek-Coder 1.3B** — tiny, MIT licensed, already knows Java
- **CodeGemma 2B** — Google, Apache 2.0
- **StarCoder2 3B** — BigCode, permissive license

Then only do instruction fine-tuning for your specific task.

### Full Recommended Pipeline

```
1. Continued Pretraining (if base model has no Java knowledge)
   └── Train on large Java corpus (e.g. The Stack dataset on Hugging Face)

2. Instruction Fine-tuning
   └── QLoRA with Unsloth on 500–2000 task-specific examples

3. RAG (optional, at runtime)
   └── Attach custom API docs / internal library references
   └── Tools: LlamaIndex, ChromaDB

4. Quantize & Deploy on-device
   └── Convert to GGUF Q4 format
   └── Run via llama.cpp or MLC LLM
```

---

## 5. Fine-tuning Tools & Resources

| Tool | Purpose |
|---|---|
| [Unsloth](https://github.com/unslothai/unsloth) | Fastest QLoRA fine-tuning on consumer hardware |
| [Axolotl](https://github.com/axolotl-ai-cloud/axolotl) | YAML-driven, beginner-friendly fine-tuning |
| [Hugging Face PEFT + TRL](https://huggingface.co/docs/peft) | Most flexible fine-tuning framework |
| [The Stack (HuggingFace)](https://huggingface.co/datasets/bigcode/the-stack) | Billions of tokens of curated code by language |
| [llama.cpp](https://github.com/ggerganov/llama.cpp) | Run quantized models on CPU |
| [MLC LLM](https://github.com/mlc-ai/mlc-llm) | On-device inference for Android/iOS |
| [Ollama](https://ollama.com) | Easiest local model deployment |

### QLoRA Resource Requirements
| Model Size | RAM Needed | GPU Needed |
|---|---|---|
| 1–2B (quantized) | 2–4 GB | None (CPU ok) |
| 3–4B (quantized) | 4–6 GB | Optional |
| 7B (quantized) | 6–8 GB | 8 GB VRAM |

> **Rule of thumb:** 500 clean, high-quality examples outperform 5,000 noisy ones.

---

*Document generated from a learning session on LLM training, fine-tuning, and on-device deployment.*

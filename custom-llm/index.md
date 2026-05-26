# Fine-Tuning & Distillation Tutorial

A progressive, hands-on tutorial for building a task-specialized lightweight LLM.

**Teacher model:** DeepSeek (via API)
**Target:** model small enough to run on laptop / mobile
**Hardware:** single GPU or free-tier Colab

---

## Chapters

### 1. Setup & Environment ✅
- Install dependencies, download base models, configure DeepSeek API access
- Choose the student base model: **SmolLM 135M**
- **Code:** `chapters/01-setup/setup.ipynb`

### 2. Task 1 — Text Classification (Fine-Tuning Basics) ✅
- Train on a small labeled dataset (500 synthetic examples, 4 classes)
- Transform a general LM into a specialized classifier
- Cover: tokenization, training loop, LoRA, evaluation
- **Code:** `chapters/02-classification/classification.ipynb`

### 3. Task 2 — Structured Extraction ✅
- Train the model to extract fields from free text into consistent JSON
- Cover: prompt formatting, output parsing, handling malformed outputs
- **Code:** `chapters/03-extraction/extraction.ipynb`

### 4. Task 3 — Instruction-Style Response Formatting ✅
- Train the model to answer in a specific format (tone, style, structure)
- Cover: instruction tuning, system prompts, behavioral alignment
- **Code:** `chapters/04-instruction/instruction.ipynb`

### 5. Generating Training Data with DeepSeek (Teacher) ✅
- Use DeepSeek API to generate high-quality labeled examples for all three tasks
- Cover: prompt engineering for the teacher, output validation, cost management
- **Code:** `chapters/05-teacher-data/generate-data.ipynb`

### 6. Distillation — Teacher to Student ✅
- Train the small model using teacher outputs (hard labels + soft labels/logits)
- Compare: raw student vs. fine-tuned student vs. teacher performance
- Cover: knowledge distillation loss, temperature scaling, when to use soft vs. hard labels
- **Code:** `chapters/06-distillation/distillation.ipynb`

### 7. Quantization & Export ✅
- Compress the final model: GGUF, ONNX, or bitsandbytes quantization
- Run inference on CPU (laptop / mobile simulator)
- Cover: INT8/INT4 quantization, size-vs-accuracy tradeoffs, on-device inference
- **Code:** `chapters/07-quantization/quantization.ipynb`

### 8. End-to-End Pipeline ✅
- Assemble the full pipeline: teacher → data generation → fine-tuning → distillation → quantization → local inference
- A single notebook that runs the entire flow for a custom task
- Final deliverable: a working quantized model specialized for one task
- **Code:** `chapters/08-pipeline/pipeline.ipynb`

---

## Running Order

```
1 → 2 → 3 → 4 → 5 → 6 → 7 → 8
```

Each chapter builds on the previous one. Chapter 5 (DeepSeek data generation) can be done standalone, but is placed after the task chapters so you understand what good training data looks like before automating its creation.

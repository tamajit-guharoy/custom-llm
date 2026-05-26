# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Goals

This project is a tutorial on LLM **fine-tuning** and **distillation** using the simplest possible examples to explain each concept. The tutorial progresses step-by-step from basic ideas to the end goal: producing a lightweight, task-specialized LLM that can run on constrained devices (mobile phones, laptops).

### End deliverables
- A working tutorial (notebook or scripts) that walks through fine-tuning a small base model on a narrow task
- A distillation step that compresses the fine-tuned model into an even smaller student model
- The resulting model should be small enough to run locally on a laptop or mobile device
- Every concept is demonstrated with minimal, self-contained code — no unnecessary abstractions

### Principles
- **Simplest possible example** for each concept — if an example can be made simpler, it should be
- **Progressive layering** — each section builds on the previous one, introducing only one new idea at a time
- **Reproducible** — all examples must run on modest hardware (a single GPU or free-tier colab)
- **Production-aware** — note real-world considerations where they differ from the simplified example, but don't let them bloat the core tutorial

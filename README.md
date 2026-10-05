# Smart Algorithmi — AI Tutoring Agent

> **Source code is not published.** This project is intellectual property of the
> Universidade Politécnica de Tomar. This page describes the architecture and
> results. A demo video is available on request.

Final degree project (Computer Engineering, grade 18/20), developed as a pair with
[André Benquerer](https://github.com/Benquerer).. The goal was a modular AI tutoring system that helps students learn
introductory programming on the **Algorithmi** platform, using a Socratic approach:
guiding the student with questions instead of handing out solutions. All models run
locally.

## System overview

The system has three parts, connected through an abstraction layer (Provider pattern)
that lets different answer-generation mechanisms be swapped without changing the rest
of the code.

| Component | Role | Main technologies |
|---|---|---|
| **Smart API** | Entry point; exposes the system to client apps, supports streaming | Python, FastAPI, Pydantic, httpx |
| **Smart Tutor** | The tutoring agent | Python, FastAPI, Ollama, RAG |
| **Training Module** | Fine-tuning pipeline for specialised models | PyTorch, Hugging Face Transformers, PEFT, TRL |

## Smart Tutor (the agent)

A single agent built on a locally-run LLM, wrapped in a deterministic pipeline that
controls what goes in, what the model sees and what comes out:

student message → **InputGuard** (validation and request classification) →
**Retriever** (RAG over the language's knowledge base) → **PromptBuilder** (pedagogical
rules + retrieved knowledge) → **ModelConnector** (local LLM, streaming)

Each stage has a single responsibility and can be tested or replaced independently.

## Training Module (fine-tuning)

A pipeline that turns platform exercises into specialised models:

- Dataset built from real exercises, with controlled injection of synthetic bugs to
  teach the model to detect and explain errors, and negative examples to recognise
  correct code
- Train/evaluation split by program, so examples from the same exercise never land in
  both sets (avoids data leakage)
- QLoRA fine-tuning (4-bit NF4 + LoRA adapters) with TRL's SFTTrainer, NEFTune
  regularisation and early stopping
- Adapter merge, evaluation, and conversion to GGUF (llama.cpp) for local execution
  with Ollama
- Flask dashboard to inspect and compare training runs

Some of the base models compared: Qwen2.5 Coder 7B, Qwen2.5 Instruct 7B, Mistral 7B, Codestral 22B
and Llama 3.2 1B.

## What we learned

The experiments showed that a single agent with local execution, RAG and a framing
logic layer fulfils the essential functions of a tutor. They also showed its limits:
tasks that require verifying or judging code benefit only partially from prompt
engineering, because LLMs infer from patterns rather than verify objectively.

This pointed to the next step: a multi-agent architecture (planned with LangGraph) with
objective verification by external tools.

## Tech stack

Python · FastAPI · Pydantic · Ollama · RAG · QLoRA · PyTorch · Hugging Face
(Transformers, PEFT, TRL) · llama.cpp / GGUF · Flask · LangGraph (planned)

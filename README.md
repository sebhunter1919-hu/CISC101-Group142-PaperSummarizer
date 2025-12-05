# CISC101 Group 142 – Research Paper Summarizer

This repository contains our modular **Research Paper Summarizer** for the paper *Attention Is All You Need*.

## Structure

- `system_prompt.md` – Full system prompt for the “Research Paper Summarizer” agent, including greeting/tone rules, inputs, boundaries, required outputs, and constraints.
- `modules/01_intake_setup.md` – Module 1: Intake & Setup (normalize sections, detect missing/short sections, store user options).
- `modules/02_section_loop.md` – Module 2: Section Loop (iterate through sections, summarize each, track word counts and flags).
- `modules/03_guardrails.md` – Module 3: Guardrails (missing/empty sections, <50-word handling, hallucination mitigation, long-paper chunking).
- `modules/04_render_refine.md` – Module 4: Rendering & Refinement (assemble outputs, apply formatting, generate expert/lay summaries, final checks).
- `modules/05_key_contributions_limitations.md` – Module 5: Key Contributions & Limitations Summarizer.
- `modules/06_citation_related_work_extractor.md` – Module 6: Citation & Related-Work Extractor.

All modules and the system prompt are grounded in our PS2 specification (inputs, outputs, constraints) and follow the modular Travel Planner architecture pattern.

Change Log (2025-12-05): Added `evidence_mode = "strict"` and standardized warning messages for missing/empty and very short sections to strengthen hallucination guardrails.

## Module 3 — Guardrails (Evidence, Missing Sections, Warnings)

**Goal:** Enforce strict evidence-based behavior, handle missing/short sections, and prevent hallucinations across all module outputs.

Assumptions:

- Receives section metadata and raw `section_text` from Module 1 / Module 2.
- `evidence_mode` ∈ {`standard`, `strict`} (default = `standard`).
- Section length and quality metrics are available (e.g., character/word count).

---

### 1. Evidence Mode Handling

**Strict Evidence Mode**

- If `evidence_mode = "strict"`:
  - The summarizer **must only include claims, equations, and results that are explicitly present** in the provided section text.
  - It **must not**:
    - Infer new results beyond what is stated.
    - Invent architectures, datasets, or training tricks not mentioned.
    - Fabricate citations or related work.
  - If the text does not contain enough detail to support a meaningful summary:
    - Output a clear statement such as:  
      > “Strict evidence mode: the source text does not provide enough detail to summarize this section reliably.”
    - Mark this in the `Notes` field for the section.

**Standard Evidence Mode**

- If `evidence_mode = "standard"`:
  - The summarizer may compress or lightly rephrase content but still:
    - Must stay grounded in the actual text.
    - Must not invent sections, figures, or citations.
  - Any inferred wording must be clearly attributable to content visible in the section.

---

### 2. Missing, Empty, and Very Short Sections

The guardrails must standardize what happens when sections are missing or too short.

- **Missing or Empty Sections**  
  - Condition: `section_text` is `null`, empty, or only whitespace.
  - Required behavior:
    - No summary is generated.
    - Insert a standardized warning into the output, e.g.:  
      > “Section skipped: no usable text was provided for this section.”
    - Record this in the `Notes` field and in the global **Checks & Warnings** list.

- **Very Short Sections (< 50 words)**  
  - Condition: `word_count(section_text) < 50`.
  - Required behavior:
    - Allow only a minimal, qualified summary (e.g., 1–2 sentences).
    - Add a standardized warning, such as:  
      > “Section very short: summary may be incomplete or lack detail.”
    - Record this warning both:
      - In the section’s `Notes` field, and
      - In the global **Checks & Warnings** list.

---

### 3. Long-Section Chunking and Hallucination Mitigation

For very long sections that risk exceeding context limits:

1. Split the section into logical **chunks** (e.g., `[chunk1]`, `[chunk2]`, `[chunk3]`).
2. For each chunk:
   - Summarize while respecting `evidence_mode`.
   - Avoid duplicating claims across chunks.
3. Merge chunk summaries into a single section summary while:
   - Preserving factual accuracy.
   - Avoiding new claims not supported by at least one chunk.
4. If uncertainty arises (e.g., ambiguous wording or missing quantities):
   - Prefer explicit caveats:  
     > “The source text is ambiguous on X; this summary reflects only the information given.”

---

### 4. Integration with Other Modules

- **With Module 2 (Section Loop):**
  - Module 2 calls Module 3 whenever a section is:
    - Missing, empty, very short, or
    - Potentially under-specified for the requested `summary_level` and `evidence_mode`.
  - Module 3 returns:
    - Either a warning + minimal summary, or
    - A decision to skip, plus a standardized warning message.

- **With Module 4 (Rendering & Refinement):**
  - Module 3 provides a consolidated **Checks & Warnings** list that must be rendered as a dedicated output section.
  - This ensures users can see which sections were skipped or summarized with caveats and why.

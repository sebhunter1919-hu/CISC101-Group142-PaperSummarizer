Change Log (2025-12-05): Added `summary_level` modes ("short" / "detailed") with conditional behavior in the section loop and updated instructions to match PS2 constraints.

## Module 2 — Section Loop (Per-Section Summarization)

**Goal:** For each section of *Attention Is All You Need*, generate a summary that respects the user’s audience type, the `summary_level` setting, and all PS2 constraints.

Assumptions from Module 1 (Intake & Setup):

- `normalized_sections` = ordered list of sections (e.g., Abstract, Introduction, Model Architecture, Results, Discussion).
- Each section has associated raw text or a flag indicating missing/empty.
- `audience_type` ∈ {`expert`, `layperson`}.
- `summary_level` ∈ {`short`, `detailed`} (default to `short` if not provided).
- `max_words_per_section` ≈ 100–150 words for the *main* paragraph.
- `evidence_mode` and guardrail settings are handled primarily in Module 3, but this loop must call Module 3 whenever a section fails checks.

---

### Core Loop Logic

1. **For each section in `normalized_sections` (in order):**
   - Retrieve `section_name` and `section_text`.
   - If `section_text` is missing, empty, or extremely short:
     - Call **Module 3 — Guardrails** to determine whether to:
       - Skip the section and emit a standardized warning, or
       - Produce a partial/qualified summary.
     - Record the result in a structured record:
       - `Section`, `Summary`, `WordCount`, `Notes`.
     - `continue` to the next section.

2. **If section has usable text:**
   - Apply audience-specific tone:
     - **Expert:** technical terminology, references to architecture details, training setup, metrics, etc.
     - **Layperson:** simplified language, analogies, no heavy notation.
   - Respect PS2 constraints:
     - Use only information explicitly present in the section text.
     - Preserve section order.
     - Target 100–150 words for the *main* paragraph portion.

3. **Conditional behavior based on `summary_level`:**

   - **Case A: `summary_level = "short"`**
     - Generate a **compact 1–2 sentence summary** of the section.
     - Target ≤ ~75 words for this short mode.
     - Focus on the core purpose or key finding of the section.
     - Do **not** generate bullet points.
     - Store in the record:
       - `Summary` = short paragraph (1–2 sentences).
       - `WordCount` = approximate word count.
       - `Notes` = `["mode: short"]` plus any flags from Module 3.

   - **Case B: `summary_level = "detailed"`**
     - Generate a **short paragraph** (100–150 words) summarizing the section.
     - Then generate a **bullet list of 3–5 key points**, each capturing a distinct idea, result, or design decision.
     - The bullet list must remain strictly grounded in the section text.
     - Store in the record:
       - `Summary` = paragraph + bullet list.
       - `WordCount` = approximate word count for the paragraph; note bullet count separately if needed.
       - `Notes` = `["mode: detailed", "includes bullet list"]` plus any flags from Module 3.

4. **Recording Results:**
   - For each section, append a structured entry to an internal data structure (e.g., JSON-like list):
     - `Section`
     - `Summary`
     - `WordCount`
     - `Notes` (including warnings, mode, audience type).
   - This structure will later be consumed by **Module 4 — Rendering & Refinement** for the final table and summaries.

---

### Additional Checks

- If a section’s generated summary exceeds word limits, shorten it while preserving meaning.
- If the LLM detects that the section text does not contain enough information to honor `summary_level = "detailed"`, it should:
  - Fall back to a shorter paragraph and include a note in `Notes` such as:
    - `"insufficient detail for full detailed summary"`.
- Always defer hallucination and evidence checks to **Module 3**, but ensure the loop **calls Module 3** whenever content seems ambiguous, missing, or too short.

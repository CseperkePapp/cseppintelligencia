# The Five-Stage LLM Usage Pipeline
**Created:** November 22, 2025
**Last Updated:** November 22, 2025

A systematic approach to structuring AI interactions for reliability, maintainability, and optimal results.

## Table of Contents
1. [Overview](#overview)
2. [The Five Stages](#the-five-stages)
3. [Why This Matters](#why-this-matters)
4. [Implementation Examples](#implementation-examples)
5. [Anti-Patterns to Avoid](#anti-patterns-to-avoid)
6. [Model-Specific Considerations](#model-specific-considerations)

---

## Overview

**The Problem:** Most LLM failures happen because we throw messy input directly at the model, then wonder why results are inconsistent.

**The Solution:** A structured five-stage pipeline that separates concerns, validates at each step, and keeps the LLM focused on what it does best: reasoning.

```
Intent Capture → Normalization → Context Assembly → Reasoning Call → Post-Processing
   (messy)      (deterministic)     (retrieval)        (LLM core)      (validation)
```

**Key Principle:** Use deterministic code for everything you can. Reserve the LLM for the irreducibly fuzzy parts.

---

## The Five Stages

### 1. Intent Capture
**What it is:** Receiving the raw, messy input from user or system.

**Examples:**
- "map this brand kit to my theme system"
- "fix the bugs in the checkout flow"
- "make this email sound more professional"
- User uploads a PDF and says "summarize this"

**What NOT to do:** Pass this directly to the LLM.

**What to do:** Accept it as-is. This is the rawest form of user intent—preserve it for context.

**Why it matters:** Understanding the original request helps with error handling and clarifying questions. Don't lose this signal.

---

### 2. Normalization
**What it is:** Converting messy input into a structured task definition.

**This is deterministic code, NOT the LLM.**

**Structured task components:**
- **Type:** classification, transformation, generation, analysis, extraction, etc.
- **Target format:** JSON schema, markdown template, specific file type, API response shape
- **Constraints:** character limits, required fields, forbidden content, style requirements
- **Dependencies:** what data/context is needed to complete this task

**Example transformation:**

```
INPUT (messy):
"map this brand kit to my theme system"

OUTPUT (normalized):
{
  "task_type": "schema_mapping",
  "source": "brand_kit_colors.json",
  "target_format": "theme_system_v2_schema",
  "constraints": {
    "preserve_hex_values": true,
    "map_semantic_names": true,
    "validate_contrast_ratios": true
  },
  "required_context": [
    "theme_system_spec",
    "previous_mappings",
    "semantic_color_naming_guide"
  ]
}
```

**Why it matters:**
- Deterministic parsing is 100% reliable and instant
- Structured tasks enable better context retrieval
- Clear constraints prevent scope creep in LLM responses
- Makes the system testable and debuggable

**Implementation approaches:**
- Pattern matching (regex, parsers)
- Intent classification (small/fast ML model or rule-based)
- Schema validation
- File type detection
- Metadata extraction

---

### 3. Context Assembly
**What it is:** Retrieving only the relevant information needed for this specific task.

**NOT the LLM. This is:**
- Database queries
- Vector similarity search (RAG)
- File system lookups
- API calls
- Decision history retrieval
- Example selection

**What to retrieve:**
- **Specifications:** Schema definitions, API docs, style guides
- **Examples:** Previous successful outputs, template instances
- **Constraints:** Business rules, validation requirements
- **History:** Past decisions, user preferences, related work

**Example:**

For the brand kit mapping task:
```python
context = {
  "theme_spec": load_file("specs/theme_system_v2.json"),
  "examples": retrieve_similar_mappings(
    query="brand kit color mapping",
    limit=3
  ),
  "constraints": load_validation_rules("color_contrast"),
  "previous_decision": get_last_mapping_decision(user_id)
}
```

**Why it matters:**
- LLMs perform better with relevant, focused context
- Reduces hallucination (grounded in real data)
- Keeps prompts manageable (token efficiency)
- Enables retrieval optimization independent of LLM choice

**Best practices:**
- Retrieve lazily (only what's needed)
- Rank by relevance (don't dump everything)
- Include metadata (source, date, confidence)
- Cache aggressively (same context = same retrieval)

---

### 4. Reasoning Call (The Kernel)
**What it is:** THE LLM STEP. This is where the actual AI reasoning happens.

**Now and ONLY now do you build the full prompt.**

**Prompt structure:**
```
[System context]
You are a design system expert mapping brand colors to theme tokens.

[Task specification]
Given this brand kit and target schema, create a mapping that...

[Retrieved context]
Theme specification: {...}
Example mappings: {...}
Validation rules: {...}

[Input data]
Brand kit: {...}

[Output format]
Return a JSON object with...

[Constraints]
- All hex values must be preserved exactly
- Semantic names must follow the naming guide
- Contrast ratios must meet WCAG AA standards
```

**Reasoning patterns:**

**Single-step:**
```
Prompt → LLM → Answer
```
Use for: Simple transformations, classification, straightforward generation

**Multi-step:**
```
Prompt 1 → Plan
Prompt 2 (with plan) → Solve
Prompt 3 (with solution) → Summarize/Format
```
Use for: Complex tasks, code generation, analysis requiring verification

**Why it matters:**
- Clean separation: deterministic prep, fuzzy reasoning, deterministic validation
- Testable: swap LLMs, adjust prompts, measure performance
- Cost-effective: only invoke LLM for irreducibly fuzzy parts
- Debuggable: clear input/output boundary

**Model selection considerations:**
- **GPT-5.1**: Variable-complexity tasks (adaptive reasoning optimizes cost)
- **Claude Sonnet 4.5**: Multi-step tool execution, coding tasks
- **Gemini 3 Pro**: Multimodal understanding, complex reasoning
- Match the model to the reasoning pattern

---

### 5. Post-Processing & Validation
**What it is:** Parse, validate, and ensure the LLM output meets requirements.

**This is deterministic code again.**

**Validation steps:**

**1. Parse the output**
```python
try:
    result = json.loads(llm_response)
except JSONDecodeError:
    # Attempt repair or fail gracefully
```

**2. Check invariants**
```python
assertions = [
    all_hex_values_preserved(result, input_brand_kit),
    meets_contrast_requirements(result),
    follows_naming_convention(result),
    all_required_tokens_mapped(result, theme_spec)
]
```

**3. Run domain validators**
```python
validators = [
    design_system_validator(result),
    color_contrast_validator(result),
    theme_schema_validator(result)
]
```

**4. Handle failures**

**Option A: Auto-repair with another LLM call**
```python
if validation_failed:
    repair_prompt = build_repair_prompt(
        original_output=result,
        validation_errors=errors,
        constraints=constraints
    )
    result = llm.call(repair_prompt)
    # Re-validate
```

**Option B: Surface error to user**
```python
if validation_failed_after_repair:
    return {
        "status": "failed",
        "partial_result": result,
        "errors": errors,
        "suggested_fixes": generate_fix_suggestions(errors),
        "retry_strategy": "manual_intervention_required"
    }
```

**Why it matters:**
- Catch errors before they reach production/user
- LLM outputs are probabilistic—validation makes them deterministic
- Auto-repair can fix minor formatting issues without user friction
- Failed validations teach you where your prompts need improvement

**Best practices:**
- Log all validation failures for prompt improvement
- Set retry limits (don't loop infinitely)
- Distinguish between recoverable and fatal errors
- Provide actionable error messages

---

## Why This Matters

### Reliability
- Deterministic steps don't fail randomly
- Validation catches LLM mistakes before they propagate
- Clear error handling prevents silent failures

### Debuggability
- Each stage has clear inputs and outputs
- Can test stages independently
- Easy to identify where failures occur
- Reproducible (deterministic stages + prompt versioning)

### Cost Efficiency
- Only invoke LLM for reasoning (not parsing, validation, formatting)
- Context assembly ensures minimal token usage
- Single-step for simple tasks, multi-step only when needed
- Caching at normalization and context layers

### Maintainability
- Swap LLMs without rewriting entire system (kernel is isolated)
- Improve prompts without touching parsing/validation
- Deterministic code is easier to test and refactor
- Clear contracts between stages

### Scalability
- Context assembly can be optimized independently (better RAG, caching)
- Normalization layer handles increasing input variety
- Validation layer scales with business rules
- Parallel processing (batch similar tasks)

---

## Implementation Examples

### Example 1: Email Tone Adjuster

**1. Intent Capture**
```
"make this email sound more professional"
+ email_body
```

**2. Normalization**
```python
task = {
    "type": "text_transformation",
    "transformation": "tone_adjustment",
    "target_tone": "professional",
    "preserve": ["facts", "names", "dates"],
    "constraints": {
        "max_length_change": "20%",
        "preserve_structure": True
    }
}
```

**3. Context Assembly**
```python
context = {
    "style_guide": load_writing_style_guide(),
    "examples": get_tone_examples(
        from_tone="casual",
        to_tone="professional",
        limit=2
    ),
    "user_preferences": get_user_writing_preferences()
}
```

**4. Reasoning Call**
```
System: You are a professional communication expert.
Task: Adjust this email's tone to be more professional while preserving facts.
Examples: [2 before/after pairs]
Input: [email_body]
Constraints: Keep within 20% of original length, maintain paragraph structure.
```

**5. Post-Processing**
```python
# Validate length constraint
assert len(output) <= len(input) * 1.2

# Check fact preservation
assert all_names_preserved(input, output)
assert all_dates_preserved(input, output)

# Grammar check
grammar_score = grammar_checker(output)
if grammar_score < 0.95:
    # Auto-repair with grammar-focused prompt
```

---

### Example 2: Code Documentation Generator

**1. Intent Capture**
```
"document this function"
+ code_snippet
```

**2. Normalization**
```python
task = {
    "type": "code_documentation",
    "language": detect_language(code_snippet),
    "doc_format": "docstring",  # JSDoc, Python docstring, etc.
    "required_sections": ["description", "params", "returns", "examples"],
    "code_location": extract_location(code_snippet)
}
```

**3. Context Assembly**
```python
context = {
    "doc_style_guide": load_doc_standards(task.language),
    "similar_functions": find_similar_documented_functions(code_snippet),
    "type_hints": extract_type_information(code_snippet),
    "usage_examples": find_usage_in_codebase(task.code_location)
}
```

**4. Reasoning Call**
```
System: You are an expert technical writer for [language] codebases.
Task: Generate complete docstring following [style guide].
Code: [snippet]
Type info: [hints]
Usage examples found: [examples]
Required sections: description, params, returns, examples
```

**5. Post-Processing**
```python
# Parse docstring
parsed = docstring_parser(output)

# Validate completeness
assert has_section(parsed, "description")
assert all_params_documented(parsed, code_params)
assert return_type_documented(parsed)

# Check examples compile
if "examples" in parsed:
    for example in parsed.examples:
        assert syntax_valid(example, task.language)
```

---

### Example 3: Data Extraction from PDF

**1. Intent Capture**
```
User uploads PDF: "extract invoice data"
```

**2. Normalization**
```python
task = {
    "type": "structured_extraction",
    "source_format": "pdf",
    "target_schema": "invoice_v2",
    "extraction_fields": [
        "invoice_number", "date", "vendor", "line_items", "total"
    ],
    "validation_rules": load_invoice_validation_rules()
}
```

**3. Context Assembly**
```python
# Pre-process PDF (deterministic)
text = pdf_to_text(pdf_file)
tables = extract_tables(pdf_file)
metadata = pdf_metadata(pdf_file)

context = {
    "invoice_schema": load_schema("invoice_v2"),
    "example_extractions": get_similar_invoices(vendor_name, limit=2),
    "validation_rules": task.validation_rules,
    "extracted_text": text,
    "extracted_tables": tables
}
```

**4. Reasoning Call**
```
System: You are a data extraction specialist.
Task: Extract invoice data matching this exact schema.
Schema: [invoice_v2_schema]
Examples: [2 successful extractions]
Text: [extracted_text]
Tables: [extracted_tables]
Output: JSON matching schema exactly.
```

**5. Post-Processing**
```python
# Parse JSON
invoice_data = json.loads(output)

# Schema validation
validate_against_schema(invoice_data, "invoice_v2")

# Business rule validation
assert valid_invoice_number(invoice_data.invoice_number)
assert valid_date(invoice_data.date)
assert line_items_sum_to_total(invoice_data)

# If validation fails
if errors:
    # Attempt repair with focused prompt
    repair_prompt = f"Fix these validation errors: {errors}"
    invoice_data = llm.call(repair_prompt, previous_output)

    # Re-validate
    if still_invalid:
        return {
            "status": "needs_review",
            "data": invoice_data,
            "errors": errors,
            "confidence": "low"
        }
```

---

## Anti-Patterns to Avoid

### ❌ Direct Pass-Through
**Bad:**
```python
user_input = "make this professional"
result = llm.call(user_input)
return result
```

**Why it fails:**
- No structure
- No validation
- No context
- Inconsistent results
- Impossible to debug

**Good:**
```python
task = normalize(user_input)
context = assemble_context(task)
prompt = build_prompt(task, context)
result = llm.call(prompt)
validated = validate_and_repair(result, task.constraints)
return validated
```

---

### ❌ LLM for Parsing
**Bad:**
```python
# Using LLM to parse deterministic formats
json_string = llm.call("convert this CSV to JSON: " + csv_data)
```

**Why it fails:**
- Expensive (tokens for trivial task)
- Unreliable (LLM might format incorrectly)
- Slow (LLM inference vs. instant parsing)

**Good:**
```python
# Use deterministic code
import csv, json
data = json.dumps(list(csv.DictReader(csv_data)))
```

---

### ❌ No Validation
**Bad:**
```python
result = llm.call(prompt)
database.save(result)  # YOLO
```

**Why it fails:**
- Silent corruption
- Cascading failures
- No feedback loop for improvement

**Good:**
```python
result = llm.call(prompt)
validated = validate(result)
if validated.errors:
    result = repair_or_fail(result, validated.errors)
database.save(result)
log_validation_metrics(validated)
```

---

### ❌ Context Dumping
**Bad:**
```python
# Throw entire knowledge base at LLM
prompt = f"{all_documentation}\n\nNow answer: {question}"
```

**Why it fails:**
- Token waste
- Context dilution (relevant info buried)
- Slow and expensive
- Reduced accuracy (more noise)

**Good:**
```python
# Retrieve only relevant context
relevant_docs = semantic_search(question, top_k=3)
prompt = f"{relevant_docs}\n\nQuestion: {question}"
```

---

### ❌ Unbounded Retries
**Bad:**
```python
while not valid(result):
    result = llm.call(prompt)  # Infinite loop possible
```

**Why it fails:**
- Cost explosion
- No convergence guarantee
- Masks underlying prompt problems

**Good:**
```python
max_retries = 2
for attempt in range(max_retries):
    result = llm.call(prompt)
    if valid(result):
        break
    prompt = add_repair_instructions(prompt, validation_errors)
else:
    return error_response(result, "max_retries_exceeded")
```

---

## Model-Specific Considerations

### GPT-5.1
- **Adaptive reasoning:** Best for variable-complexity tasks; model self-optimizes thinking time
- **Steerability:** Excellent at following structured output formats—be explicit
- **Verbosity:** Can be verbose; specify desired output length in normalization
- **Best for:** Tasks with unpredictable complexity, conversational interfaces

**Pipeline adjustments:**
- Normalization: Include verbosity constraints
- Reasoning: Use single-step for simple, multi-step for complex (let adaptive reasoning optimize)
- Validation: Check for over-explanation; truncate if needed

---

### Claude Sonnet 4.5
- **Tool use:** Excels at parallel tool execution
- **Long-horizon:** Can maintain context across extended sessions
- **Code-focused:** Best coding model; use for generation/analysis tasks

**Pipeline adjustments:**
- Context assembly: Provide tool definitions in structured format
- Reasoning: Explicitly request parallel tool calls when applicable
- Validation: Strong at self-correction; repair prompts work well

---

### Gemini 3 Pro
- **Multimodal:** Best for tasks involving images, video, audio + text
- **Direct prompts:** Prefers concise, direct instructions
- **Temperature:** Keep at 1.0 for reasoning tasks

**Pipeline adjustments:**
- Normalization: Simplify structure (avoids over-analysis)
- Context assembly: Can handle multimodal inputs natively
- Reasoning: Use direct, logical prompts (avoid persuasive language)
- Validation: Strong reasoning; fewer repair cycles needed

---

## Measuring Pipeline Effectiveness

### Stage-Specific Metrics

**Normalization success rate:**
- % of inputs successfully parsed
- % requiring fallback handling

**Context retrieval quality:**
- Relevance scores (human-eval sample)
- Context utilization (did LLM use retrieved info?)

**Reasoning call performance:**
- First-pass success rate (% passing validation without repair)
- Token efficiency (tokens used / baseline)
- Latency by complexity

**Validation metrics:**
- Error types frequency
- Repair success rate
- Human override rate

**End-to-end:**
- Task completion rate
- User acceptance rate
- Cost per successful task
- Mean time to completion

---

## Evolution & Improvement

**How to improve each stage:**

**Normalization:**
- Log parsing failures → add new patterns
- User feedback → refine task classification
- Edge cases → expand constraint definitions

**Context Assembly:**
- Track context utilization → improve retrieval
- A/B test retrieval strategies
- Monitor token usage → optimize context size

**Reasoning:**
- Failed validations → improve prompts
- Compare models on same tasks
- Version prompts, track performance over time

**Validation:**
- New error patterns → add validators
- False positives → refine rules
- User corrections → update constraints

---

## Summary

**The Five Stages:**
1. **Intent Capture** - Accept messy input
2. **Normalization** - Structure deterministically
3. **Context Assembly** - Retrieve relevant info
4. **Reasoning Call** - LLM does what only LLMs can do
5. **Post-Processing** - Validate and repair

**Core Principles:**
- Use deterministic code wherever possible
- Reserve LLM for irreducibly fuzzy reasoning
- Validate everything
- Make each stage independently testable
- Log, measure, improve

**Result:**
- Reliable, debuggable, maintainable AI systems
- Cost-efficient (LLM only where needed)
- Scalable (optimize stages independently)
- Improves over time (metrics drive iteration)

---

## Related Resources

- [Prompt Engineering Resources](./PROMPT_ENGINEERING_RESOURCES.md) - Model-specific best practices
- [Solo Enterprise Workflow Lab](./aiProject-solo-enterprize-workflow-lab.md) - Automation patterns
- [AI Incident Logs](../INCIDENT LOGS/README.md) - Real-world failure cases

---

*This pipeline architecture applies to all modern LLMs. Adjust model-specific details, but keep the fundamental structure intact.*

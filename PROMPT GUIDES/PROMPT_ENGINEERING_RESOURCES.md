# Prompt Engineering Resources
**Created:** September 18, 2025
**Last Updated:** November 22, 2025

[Prompt Engineering Basics](https://www.promptingguide.ai/introduction/basics)

# Reference: Similar Experiments & Current Techniques

## Framework-Building Approaches (Similar to This Repo)
**Projects developing systematic AI interaction patterns**

- [Microsoft's PromptBase](https://github.com/microsoft/promptbase) - Research-grade framework development (Medprompt, dynamic few-shot)
- [NirDiamant's Prompt Engineering](https://github.com/NirDiamant/Prompt_Engineering) - 22 hands-on implementations, framework-focused
- [DAIR.AI Guide](https://github.com/dair-ai/Prompt-Engineering-Guide) - Most comprehensive, actively maintained, 3M+ users

## Current Techniques (2025)
**Selected based on practical utility and recent developments**

### Defensive & Production Patterns
- **Prompt scaffolding** - Wrapping user inputs in structured templates for security
- **Context engineering** - Beyond prompts: RAG, JSON inputs, structured workflows
- **Model-specific optimization** - Different models need different formatting approaches
- **Version control & testing** - Systematic optimization through controlled testing cycles

### Advanced Reasoning
- **Chain-of-thought (CoT)** - Break complex reasoning into sequential steps; "Let's think step by step"
- **Zero-shot CoT** - Effective on arithmetic and commonsense questions without examples
- **Few-shot CoT** - Supply handcrafted exemplars (3-8 examples) for complex domains
- **Dynamic few-shot** - Select different examples based on input rather than fixed examples
- **Prompt chaining** - Sequential prompts for complex multi-step tasks; professional use involves orchestrations of multiple specialized prompts

### Practical Applications
- **Temperature tuning** - 0 for factual, higher for creative tasks (Note: Gemini 3 should stay at default 1.0)
- **Iterative refinement** - Systematic testing and improvement cycles
- **Cost optimization** - Prompt engineering as cost management tool

## Model-Specific Best Practices (November 2025)

### GPT-5.1 (OpenAI)
**Key Strengths:** Adaptive reasoning, highly steerable, excellent instruction-following

**Best Practices:**
- **Steerability**: GPT-5.1 is highly steerable - be explicit about desired behaviors, personality, and communication frequency
- **Output control**: Can be verbose; explicitly specify desired output detail level
- **Adaptive reasoning**: Model decides when to think before responding; works well for variable-complexity tasks
- **Migration note**: GPT-5.1 with none reasoning effort fits most low-latency use cases from GPT-4.1; better-calibrated reasoning token consumption than GPT-5
- **Instruction conflicts**: Check for conflicting instructions if behavior issues arise; model excels at following clear, non-contradictory guidance

**Resources:**
- [GPT-5.1 Prompting Guide](https://cookbook.openai.com/examples/gpt-5/gpt-5-1_prompting_guide) (OpenAI Cookbook)
- [ChatGPT-5 Prompting Best Practices](https://www.jeffsu.org/chatgpt-5-prompting-best-practices/)

### Claude Sonnet 4.5 (Anthropic)
**Key Strengths:** Best coding model, long-horizon reasoning, parallel tool execution, extended autonomous work

**Best Practices:**
- **Clear & explicit**: Claude 4.x models are trained for precise instruction following; be specific about desired outputs
- **Provide context**: Explain the "why" behind instructions to help Claude understand your needs
- **Parallel tool execution**: Instruct model to make all independent tool calls in parallel; Sonnet 4.5 excels at firing multiple operations simultaneously
- **Thinking capabilities**: Use thinking for reflection after tool use or complex multi-step reasoning
- **Long-horizon tasks**: Excellent state tracking across extended sessions; focus on incremental progress
- **Examples matter**: Models pay close attention to details in examples; ensure they align with desired behaviors
- **Migration note**: Users wanting "above and beyond" behavior from older Claude models may need to explicitly request these behaviors in Claude 4.x

**Resources:**
- [Claude 4 Prompt Engineering Best Practices](https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/claude-4-best-practices) (Official Anthropic)
- [Claude Sonnet 4.5 System Card](https://assets.anthropic.com/m/12f214efcc2f457a/original/Claude-Sonnet-4-5-System-Card.pdf)

### Gemini 3 Pro (Google)
**Key Strengths:** Multimodal understanding, advanced thinking, terminal operations, state-of-the-art reasoning

**Best Practices:**
- **Be direct**: Favor directness over persuasion, logic over verbosity; state goals clearly and concisely
- **Structured prompts**: Use consistent delimiters (XML-style tags like `<context>`, `<task>` or Markdown headings)
- **Control verbosity**: Provides direct, efficient answers by default; explicitly request conversational/detailed responses if needed
- **Advanced thinking**: Leverage thinking capabilities for complex tasks; prompt to plan or self-critique before final response
- **Temperature warning**: Keep at default 1.0; changing temperature (especially below 1.0) may cause looping or degraded performance in complex math/reasoning
- **Migration note**: May over-analyze verbose or overly complex prompt engineering from older models; simplify prompts
- **Knowledge cutoff**: January 2025

**Resources:**
- [Prompt Design Strategies](https://ai.google.dev/gemini-api/docs/prompting-strategies) (Official Google AI)
- [Gemini 3 Developer Guide](https://ai.google.dev/gemini-api/docs/gemini-3) (Official Google AI)
- [Gemini 3 Prompting Best Practices](https://www.philschmid.de/gemini-3-prompt-practices)

## Official Documentation (Updated November 2025)
- [OpenAI Best Practices](https://help.openai.com/en/articles/6654000-best-practices-for-prompt-engineering-with-the-openai-api)
- [OpenAI Cookbook - GPT-5.1](https://cookbook.openai.com/examples/gpt-5/gpt-5-1_prompting_guide)
- [Anthropic Prompt Engineering](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview)
- [Google AI - Gemini Prompt Strategies](https://ai.google.dev/gemini-api/docs/prompting-strategies)
- [IBM's 2025 Guide](https://www.ibm.com/think/prompt-engineering)

## Research Papers & Academic Sources
*Updated periodically with relevant findings*

---
*Focus: Building AI interaction frameworks and practical implementation*
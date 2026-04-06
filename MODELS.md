# AI Models Reference Guide

**For Backend II Students - Vibecode with Safety Masterclass**

This guide provides an overview of the major AI language models available in 2026 for AI-assisted software development. Understanding these models will help you make informed decisions about which tools to use for your projects while maintaining GDPR compliance and data safety.

---

## Overview

Large Language Models (LLMs) are statistical text prediction engines trained on massive datasets. When choosing a model for development work, consider:

- **Task requirements**: Does your task need deep reasoning, code generation, or quick responses?
- **Cost**: API pricing varies significantly between models
- **Privacy**: Where does your data go, and is it used for training?
- **Context window**: How much code/documentation can the model process at once?
- **Compliance**: Does the model meet GDPR and data residency requirements?

---

## Model Comparison Table

| Model Family                 | Provider   | Context Window  | Privacy Model                | Cost Tier | Best For                                  |
|------------------------------|------------|-----------------|------------------------------|-----------|-------------------------------------------|
| **GPT*    *                  | OpenAI     | 270K+ tokens    | May train (opt-out)          | $$$       | Professional work, complex reasoning      |
| **Claude (Sonnet/Opus)**     | Anthropic  | 200K tokens     | No training on user data     | $$-$$$    | Complex code, safety-focused, reasoning   |
| **Gemini**                   | Google     | 1M tokens       | Google ecosystem integration | $$        | Multimodal, massive context               |
| **Qwen**                     | Alibaba    | 32K-128K tokens | Open source, self-hostable   | Free-$$   | Multilingual, Asian languages             |
| **Mistral**                  | Mistral AI | 128K tokens     | EU-hosted, GDPR-compliant    | $$        | European compliance, open weights         |
| **Gemma**                    | Google     | 8K tokens       | Self-hostable, fully open    | Free      | Edge deployment, learning                 |

---

## Detailed Model Descriptions

### OpenAI GPT Series

#### GPT (2026)

**Provider**: OpenAI  
**Context Window**: 270K+ tokens (~675 pages)  
**Parameters**: Proprietary (estimated 500B+)

**Capabilities & Strengths**:
- Most capable model for professional work requiring extended reasoning
- Excellent at complex, multi-step problems
- Strong code generation across all major languages
- Advanced reasoning with extended "thinking time"
- Multimodal support (text, images, audio)

**Use Cases**:
- Complex backend architecture design
- Multi-file code refactoring
- Security-critical code review
- Advanced debugging and optimization
- Technical documentation generation

**Pricing/Availability**:
- Available via OpenAI API
- ChatGPT Plus/Pro subscriptions
- Enterprise deployments with dedicated capacity

**Privacy Considerations**:
⚠️ **Data may be used for model training** unless you opt-out via API settings or use enterprise tier. **Do not use free ChatGPT for work with client data or PII.**

**Comparison Notes**:
- More expensive than alternatives but highest capability
- Better for complex tasks than simpler models
- Faster than GPT-4o predecessor
- Less transparent than open-source alternatives

### Anthropic Claude Series

#### Claude Sonnet

**Provider**: Anthropic  
**Context Window**: 200K tokens (~500 pages)  
**Parameters**: Proprietary (estimated 200-300B)

**Capabilities & Strengths**:
- **Excellent for long-document reasoning** - can analyze entire codebases
- Strong safety and ethical guardrails
- Very good at following complex instructions
- Refuses unsafe or unethical requests reliably
- Excellent at explaining its reasoning

**Technical Specs**:
- API access with prompt caching
- Fast inference speeds
- Structured output support

**Use Cases**:
- Reviewing large pull requests (entire feature branches)
- Analyzing multiple related files simultaneously
- Complex refactoring with safety considerations
- GDPR-compliant code generation
- Educational explanations and learning

**Pricing/Availability**:
- Claude API (pay-per-use)
- Claude Pro subscription ($20/month)
- Enterprise plans with dedicated capacity

**Privacy Considerations**:
✅ **Does NOT train on user data** when using Claude API or Pro tier. Excellent choice for working with client code.

**Comparison Notes**:
- Better than GPT models for long context understanding
- More conservative/safer outputs than GPT
- Preferred by many developers for code review
- Strong performance on European data protection requirements

---

#### Claude Opus

**Provider**: Anthropic  
**Context Window**: 200K tokens  
**Parameters**: Proprietary (estimated 400B+)

**Capabilities & Strengths**:
- Most capable Claude model
- Best-in-class reasoning and analysis
- Excellent for complex, nuanced tasks
- Superior at understanding context and intent

**Use Cases**:
- Mission-critical code review
- Complex system architecture design
- Security vulnerability analysis
- Detailed technical documentation
- Complex data structure design

**Pricing/Availability**:
- Higher cost than Sonnet but more capable
- Available via API and Claude Pro

---

### Google Gemini Series

#### Gemini Pro

**Provider**: Google  
**Context Window**: **1M tokens** (~2,500 pages) - largest available  
**Parameters**: Proprietary (estimated 1T+)

**Capabilities & Strengths**:
- **Massive context window** - can process entire small-to-medium codebases in one request
- Native multimodal (text, images, audio, video)
- Strong code generation
- Integrated with Google ecosystem (Docs, Drive, etc.)

**Technical Specs**:
- Native function calling
- Streaming output
- Grounding with Google Search

**Use Cases**:
- Analyzing entire application codebases
- Cross-repository refactoring
- Documentation generation from large projects
- Multimodal applications (image analysis + code)
- Integration with Google Workspace

**Pricing/Availability**:
- Google AI Studio (free tier with limits)
- Vertex AI (enterprise)
- Integrated into Google products

**Privacy Considerations**:
⚠️ **Data processing integrated with Google ecosystem**. Review data handling agreements for enterprise use. Free tier may use data for improvements.

**Comparison Notes**:
- Unmatched context window size
- Good value for massive document processing
- Less popular in developer community than Claude/GPT
- Strong multimodal capabilities

---

### Google Gemma Series (Open Source)

#### Gemma

**Provider**: Google (Open Source)  
**Context Window**: 8K tokens  
**Parameters**: 2B - 33B (multiple sizes)

**Capabilities & Strengths**:
- **Fully open source** - can self-host and customize
- Multiple size options for different deployment scenarios
- Strong performance for size
- Multimodal variants available (image + text)
- No data leaves your infrastructure when self-hosted

**Technical Specs**:
- Apache 2.0 or Gemma license
- Runs on commodity hardware (smaller models)
- Optimized for edge deployment
- TPU, GPU, and CPU support

**Use Cases**:
- **Learning and education** - understand model internals
- Edge deployment (mobile, embedded)
- Air-gapped environments
- GDPR-strict applications (medical, financial)
- Custom fine-tuning for domain-specific tasks
- On-premises deployment

**Pricing/Availability**:
- **Free** to download and use
- Self-hosting costs (compute infrastructure)
- No API fees

**Privacy Considerations**:
✅ **Full data privacy** - you control all data when self-hosted. Ideal for GDPR compliance and sensitive applications.

**Comparison Notes**:
- Much smaller context window than commercial models
- Lower capabilities than GPT/Claude but sufficient for many tasks
- Best option for learning about LLMs
- Perfect for Portuguese students due to EU alignment
- Requires technical setup (unlike APIs)

---

### Alibaba Qwen Series

#### Qwen

**Provider**: Alibaba Cloud  
**Context Window**: 32K - 128K tokens (varies by model)  
**Parameters**: 0.8B - 397B (8 sizes available)

**Capabilities & Strengths**:
- **Excellent multilingual support** - particularly strong in Chinese, Portuguese, and other non-English languages
- Open source with commercial-friendly license
- Multiple size options for different use cases
- Strong code generation (Qwen-Coder variants)
- Good performance at smaller sizes

**Technical Specs**:
- Apache 2.0 license (most models)
- Multimodal variants available
- Both base and instruction-tuned versions
- Quantized versions for lower memory usage

**Use Cases**:
- Multilingual applications
- Portuguese language code comments and documentation
- Asia-Pacific deployments
- Self-hosted European deployments
- Learning and research
- Custom fine-tuning for specific domains

**Pricing/Availability**:
- **Free** to download and use (open source)
- Also available via Alibaba Cloud API
- Self-hosting costs depend on model size

**Privacy Considerations**:
✅ **Full control when self-hosted**. API version may process data in China (review data residency requirements for GDPR).

**Comparison Notes**:
- Best multilingual performance in class
- Strong Portuguese language support
- More transparent than closed-source alternatives
- Growing popularity in European and Asian markets
- Smaller community than OpenAI/Anthropic models

---

### Mistral AI Series

#### Mistral

**Provider**: Mistral AI (France)  
**Context Window**: 128K tokens  
**Parameters**: Proprietary (estimated 200-300B)

**Capabilities & Strengths**:
- **EU-based infrastructure** - excellent for GDPR compliance
- Strong reasoning capabilities
- Good multilingual support (European languages)
- Competitive performance with OpenAI/Anthropic
- Open weights available for some models

**Technical Specs**:
- Hosted in European data centers
- Function calling support
- JSON mode for structured outputs
- Fast inference

**Use Cases**:
- **European clients with strict data residency requirements**
- GDPR-compliant development workflows
- Multilingual European applications
- Government and regulated industry projects
- Privacy-sensitive applications

**Pricing/Availability**:
- Mistral AI API
- Azure AI (Microsoft partnership)
- Self-hosted via open weights (some models)

**Privacy Considerations**:
✅ **EU data processing** - GDPR-compliant by design. Excellent choice for Portuguese and European projects.

**Comparison Notes**:
- Best choice for EU regulatory compliance
- Competitive pricing with OpenAI
- Smaller ecosystem than US providers
- Strong support for European languages
- Growing in popularity in European developer community

---

## Choosing the Right Model: Decision Matrix

### For Students Learning to Code

**Recommended**: 
1. **Claude Sonnet** - Great for learning, good explanations, free tier available
2. **Gemma** (self-hosted) - Learn how models work, fully transparent
3. **Qwen** (smaller models) - Good Portuguese support, free to use
4. **Mistral** - EU-based, good for learning GDPR-compliant development

---

### For Working with Production User Data

**Recommended**:
1. **Self-hosted Gemma or Qwen** - Data never leaves your infrastructure
2. **Mistral** - EU-hosted, GDPR-compliant
3. **Claude API** (with proper contract) - Guaranteed no training

**Avoid**:
- ❌ Free ChatGPT (data may be used for training)
- ❌ OpenAI API without enterprise agreement and opt-out
- ❌ Any model where data residency is unclear

**Why**: Legal liability, GDPR Article 28 requirements, client trust.

---

### For Complex Multi-File Refactoring

**Recommended**:
1. **Gemini Pro** - Largest context window (1M tokens)
2. **Claude Sonnet** - Excellent at long-context reasoning (200K tokens)
3. **GPT** - Strong reasoning capabilities

**Why**: Large context windows allow processing entire codebases.

---

### For Budget-Constrained Projects

**Recommended**:
1. **Gemma** (self-hosted) - Free (compute costs only)
2. **Qwen** (self-hosted) - Free, excellent smaller models
3. **GPT** - Cheapest commercial API option
4. **Mistral** - Low-cost, EU-hosted

---

## Privacy and GDPR Considerations

### Safe for Production Client Data

✅ **Recommended**:
- Claude API/Pro (contractual no-training guarantee)
- Mistral AI (EU-hosted, GDPR-compliant infrastructure)
- Self-hosted Gemma, Qwen (full data control)

### Use with Caution

⚠️ **Review Agreements**:
- OpenAI API (enable opt-out, use enterprise tier)
- Google Gemini (review data processing agreements)

### Avoid for Production Data

❌ **Do Not Use**:
- Free ChatGPT (consumer tier)
- Any model without clear data handling policies
- Models with unclear data residency
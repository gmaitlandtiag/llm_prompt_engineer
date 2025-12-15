# Practical Prompt Engineering

The goal of this reference document is to prime/structurize prompt techniques to maximize the value add we can glean from LLMs. The LLMs we use in databricks/vscode/copilot are very powerful if queries are structured effectively. Below are prompt techniques and examples we can use and reference as data professionals to increase our engineering velocity. Additionally, some of these techniques can be combined so feel free to jump to the specific section/technique you need.

## Quick Reference Table

| Technique | Best For | Complexity |
|-----------|----------|------------|
| Zero-shot | Simple, clear tasks | Low |
| Few-shot | Pattern matching, classification | Medium |
| Chain-of-thought | Complex reasoning, debugging | Medium |
| Role prompting | Domain-specific tasks | Low |
| Structured output | API integrations, pipelines | Medium |
| Negative prompting | Avoiding known failure modes | Low |
| Self-consistency | Critical outputs, verification | High |

## Emotional/Importance Cues

- "This is important to my career"
- "Take a deep breath and work through this step by step"
- "My job depends on getting this right"
 
**What it is:** Research shows that adding stakes or importance to prompts can improve output quality. They should supplement, not replace, clear technical instructions.


## Common Pitfalls

1. **Over-engineering:** Longer prompts aren't always better. Start simple, add complexity only when needed.

2. **Vague instructions:** Telling the model, "Make it better" fails. Whereas, "Improve readability by using descriptive variable names and adding docstrings" succeeds.

3. **Ignoring format:** If you need JSON, ask for JSON explicitly. Don't assume the model will guess your needs.

4. **No examples for complex tasks:** Few-shot prompting significantly improves consistency for non-trivial tasks.

5. **Forgetting context limits:** Long prompts with lots of examples eat into your context window. Be strategic. (More on context windows, temperature, system prompts, user prompts later*)

6. **Not validating outputs:** Always validate structured outputs programmatically. Models can and do produce malformed JSON.

---

> Below are the Full Prompt Engineering Techniques:

 Prompt engineering is the practice of crafting inputs to get the best possible outputs from large language models (LLMs) [GIGO - Garbage In, Garbage Out]. For data/system engineers/analysts/stewards working with LLMs in production pipelines—whether for data extraction, classification, code generation, or analysis—the difference between a mediocre prompt and a well-crafted one can mean outputs you can trust vs. constant manual intervention/refactoring/reprompting.


---

## Section 1: Core Techniques

### Zero-Shot Prompting

**What it is:** Asking the model to perform a task without providing any examples.

**When to use:** Simple, well-defined tasks where the model's training already covers the domain.

```
❌ Bad: "Fix this data"

✅ Good: "You are a data quality specialist. Identify and list all data 
quality issues in the following CSV rows. For each issue, specify: 
the row number, the column, the problem, and a suggested fix."
```

### Few-Shot Prompting

**What it is:** Providing 1-5 examples of the desired input/output pattern before your actual request.

**When to use:** Pattern recognition tasks, classification, format conversion, or when you need consistent output structure.

**Example:**

```
Convert disaster report descriptions to structured categories.

Example 1:
Input: "Flooding reported in downtown area, water levels at 3 feet"
Output: {"disaster_type": "flood", "severity": "moderate", "location": "urban"}

Example 2:
Input: "Category 4 hurricane making landfall, evacuation ordered"
Output: {"disaster_type": "hurricane", "severity": "severe", "location": "coastal"}

Now the model can effectively classify:
Input: "Minor earthquake felt, no structural damage reported"
Output: 
```

> **💡 Pro tip:** Research shows that the format and label distribution of your examples matters more than whether individual example labels are perfectly correct. Consistency is key.

### Chain-of-Thought (CoT) Prompting

**What it is:** Guiding the model to show its reasoning process step by step before arriving at an answer.

**When to use:** Complex reasoning tasks, multi-step calculations, debugging, or any task where you need to verify the logic.

**Zero-Shot CoT (simplest):**
```
Analyze this SQL query for performance issues. 
Think through this step by step before giving your recommendations.
```

**Few-Shot CoT (more powerful):**
```
Question: Why might this Spark job be running slowly?
Code: df.groupBy("date").agg(collect_list("items"))

Reasoning: First, I'll examine the operations. The groupBy on "date" 
will cause a shuffle. Then collect_list aggregates all items per date 
into memory. If any single date has millions of items, this will cause 
memory pressure. Additionally, collect_list doesn't scale well on 
skewed data.

Answer: The job likely suffers from data skew on the "date" column 
combined with collect_list causing memory issues. Consider using 
collect_set if duplicates aren't needed, or implement salting for 
skewed keys.

Now analyze this query using the same step-by-step approach:
[Your query here]
```

> **📊 Research insight:** Chain-of-thought prompting is most effective on larger models (100B+ parameters). With modern models like GPT-5, Claude, and Gemini, CoT can improve accuracy significantly on reasoning tasks.

---

## Section 2: Prompt Patterns That Work

### Role/Persona Assignment

**What it is:** Instructing the LLM to adopt a specific identity, expertise level, or perspective.

**How to use:**
```
You are a senior data engineer with 30 years of experience in 
distributed systems and data pipeline optimization. You specialize 
in Databricks and PySpark.

[The rest of your query here]
```

**Why it works:** Role prompting activates domain-specific knowledge patterns from training and shapes the tone, vocabulary, and depth of responses.

> **⚠️ Modern note:** Simple role prompts ("act as a doctor") are less impactful on newer models like GPT-5 and Claude 4+ compared to older models. For best results, use detailed role descriptions with specific expertise areas.

### Structured Output Requests

**What it is:** Explicitly specifying the format you need the response in.

**JSON Output Pattern:**
```
Extract entity information from the following text. 
Return ONLY valid JSON in this exact format:
{
  "entities": [
    {"name": "string", "type": "person|organization|location", "confidence": 0.0-1.0}
  ],
  "relationships": [
    {"from": "entity_name", "to": "entity_name", "type": "string"}
  ]
}

Do not include any text outside the JSON object.

Text: [Your text here]
```

> **💡 Pro tips for structured output:**
> - Include a sample structure in your prompt
> - Explicitly state "Return ONLY valid JSON" or similar
> - Specify "Do not include any text outside the JSON/code block"

### Explicit Constraints

**What it is:** Setting clear boundaries on length, format, scope, or content.

**Examples:**
```
Summarize in exactly 3 bullet points, each under 20 words.

List the top 5 causes, ranked by likelihood. Do not include more than 5.

Respond using only information from the provided context. If the 
answer isn't in the context, say "Information not found in provided context."
```

### Delimiter Usage

**What it is:** Using clear markers to separate different parts of your prompt.

**Patterns:**
```
### Instructions ###
[Your instructions here]

### Context ###
[Background information]

### Data ###
[Your actual data]

### Output Requirements ###
[Format specifications]
```

**Why it works:** Delimiters help the model understand the structure of your prompt and distinguish between instructions, context, and data—especially important when your data might contain text that could be confused with instructions.

---

## Section 3: Advanced Techniques

### Iterative Refinement

**What it is:** Building on the model's previous response to improve quality.

**Pattern:**
```
[Initial prompt and response]

Good start. Now improve this by:
1. Adding error handling for null values
2. Using more descriptive variable names
3. Adding inline comments explaining the business logic
```

### Negative Prompting

**What it is:** Explicitly stating what NOT to do.

**Example:**
```
Generate a PySpark function to calculate rolling averages.

Do NOT:
- Use pandas or convert to pandas DataFrame
- Use collect() or toPandas() 
- Hardcode window sizes
- Assume column names
```

> **📊 Important:** Research suggests that positive instructions ("do this") tend to work better than negative instructions ("don't do this"). Use negative prompting sparingly, for known failure modes.

### Self-Consistency / Verification

**What it is:** Having the model check its own work or generate multiple approaches.

**Pattern:**
```
[Initial response]

Now review your solution above. Check for:
1. Edge cases that aren't handled
2. Potential performance issues with large datasets
3. Any assumptions that should be validated

List any issues found and provide an improved version.
```

---

## Section 4: Data Engineering Applications

### Code Generation (PySpark/SQL)

**Template:**
```
You are an expert 10 year experienced PySpark engineer working with Databricks and 
Delta Lake.

Task: [Describe what you need]

Requirements:
- Use native PySpark operations (no pandas conversion)
- Handle null values appropriately  
- Include proper error handling
- Optimize for large-scale data (assume billions of rows)
- Use Delta Lake best practices where applicable

Input schema:
[Provide your schema]

Expected output:
[Describe expected result]
```

### Schema Inference and Documentation

**Template:**
```
Analyze this sample data and generate:
1. An inferred PySpark schema with appropriate data types
2. A data dictionary with column descriptions
3. Suggested data quality checks for each column

Sample data:
[Your sample rows]

Return the schema as a PySpark StructType definition, the dictionary 
as a markdown table, and the quality checks as pytest assertions.
```

### Error Message Interpretation

**Template:**
```
I'm getting this error in my Databricks job:

[Paste full error message and stack trace]

Job context:
- Runtime: Databricks Runtime 13.3 LTS
- Cluster: [specs]
- Data volume: approximately [X] rows

Step through the error:
1. What is the root cause?
2. What are the possible fixes, ranked by likelihood?
3. How can I prevent this in the future?
```

### Data Validation Prompt

**Template:**
```
Generate data quality validation rules for this schema:

Schema: [Your schema]

Business context: [What this data represents]

For each column, provide:
1. Null check (if applicable)
2. Type validation
3. Range/format constraints  
4. Referential integrity checks (if applicable)
5. Business logic validations

Return as Great Expectations or Pydantic validation code.
```

### FEMA-Specific Examples

**Disaster Classification:**
```
You are a 20 year disaster response data scientist. Classify the following 
incident reports into FEMA disaster categories.

Categories: Hurricane, Flood, Tornado, Earthquake, Fire, Winter Storm, Other

For each report, extract:
- disaster_type: The primary category
- severity: low/medium/high/critical  
- affected_area: Geographic description
- immediate_needs: List of urgent requirements
- confidence: Your confidence in this classification (0.0-1.0)

Report: "[Paste incident report]"

Return as JSON.
```

**Damage Assessment Extraction:**
```
Extract structured damage assessment data from this field report.

Required fields:
- property_type: residential/commercial/infrastructure/other
- damage_level: destroyed/major/minor/affected/none
- estimated_cost: numeric or "unknown"
- occupiable: true/false/unknown
- utilities_status: power/water/gas status

Field report:
"[Paste report text]"

Return as JSON. If any field cannot be determined from the text, 
use null and explain why in a separate "extraction_notes" field.
```

---

### Version Control for Prompts

Treat prompts like code:

```python
# prompts/damage_classification_v2.py

DAMAGE_CLASSIFICATION_PROMPT = """
Version: 2.1
Last updated: 2024-11-15
Author: [Your name]
Changes: Added confidence scoring, fixed edge case with partial damage

[Prompt text here]
"""
```

### A/B Testing Prompts

For production systems:
1. Create prompt variants
2. Run both on the same sample data
3. Measure: accuracy, latency, token usage, failure rate
4. Document which variant wins and why

---

### A/B Testing LLMS

LLMs have different attributes: token limits, active parameters, context window length, response latency, pricing, and reasoning capabilities. To get the best response, you often need to evaluate outputs across multiple models. For example, you can input the same prompt into GPT-5, Sonnet 4, and or Gemini 3. Then select the best response for your specific use case. Model tunes change by the day (sometimes by the hour) so getting the most of LLMs requires A/B testing as another skill.

---

---

## Contributing

This is a living document. PRs welcome for:
- New techniques with evidence they work
- Additional domain-specific examples
- Corrections or clarifications
- Agent prompt techniques/chains

## Resources

- [Anthropic Prompting Guide](https://docs.anthropic.com)
- [OpenAI Best Practices](https://platform.openai.com/docs)
- [Prompt Engineering Guide](https://promptingguide.ai)
- [LangChain Documentation](https://docs.langchain.com)

---

*Last updated: December 15 2025*
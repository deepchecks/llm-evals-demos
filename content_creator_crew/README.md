# Multi-Agent Demo: Content Creator Crew

Evaluating and debugging a Multi-Agent application, step by step.

## The Use Case

This demo evaluates a CrewAI-powered multi-agent content creation workflow. The workflow is designed to create a blog post based on a given topic, target audience, and optional context. The workflow is evaluated using [Deepchecks](https://app.llm.deepchecks.com) to identify performance issues and root causes.

## The Workflow

The system consists of three sequential agents that collaborate to create blog posts. Each agent has specific tools and capabilities to perform its role:

- **Writer** - Drafts the initial blog post based on topic, audience, and context
- **Reviewer** - Critiques the draft and identifies improvements
- **Editor** - Applies fixes and enhancements to produce the final post

**Tool configuration:**
All agents have access to internet search via SerperDevTool.
The Editor agent also gets three custom LLM-powered tools:

- **Hook Improver** - Enhances blog post openings
- **Tone Adjuster** - Matches tone to target audience
- **Content Rewriter** - Fixes specific issues in the draft

## The Problem

The workflow produces generic content even though the Editor has tools to improve it. Why aren't the tools being used? We use Deepchecks tracing to find out.

## Evaluation Workflow

### Step 1: Log Evaluation Data

Set up the crew, run test cases, and log traces to Deepchecks. The SDK traces every step of the CrewAI workflow with a few lines of code:

```python
from deepchecks_llm_client.otel import CrewaiIntegration
from deepchecks_llm_client.data_types import EnvType

tracer_provider = CrewaiIntegration().register_dc_exporter(
    host="https://app.llm.deepchecks.com",
    api_key="YOUR_API_KEY",
    app_name="Content Creator Crew",
    version_name="Baseline - Claude 3.5 Sonnet",
    env_type=EnvType.EVAL
)
```

This captures inputs/outputs, tool calls, latency, token usage, and intermediate reasoning for every agent step.

> Full details: [Logging the Data](https://llmdocs.deepchecks.com/docs/uploading-the-data-content-creator)

### Step 2: Analyze the Base Version

The baseline (Claude 3.5 Sonnet) scores **45% overall**. Tool interactions score only **23%** — something is clearly broken there.

Deepchecks breaks down performance by interaction type (Root, Agent, LLM, Tool) and measures properties like Tool Completeness, Tool Coverage, Plan Efficiency, Reasoning Integrity, and Instruction Following.

> Full details: [Analyze Performance](https://llmdocs.deepchecks.com/docs/analyze-performance-content-creator)

### Step 3: Root Cause Analysis

Use the Score Breakdown, Property Failure Analysis, and Insights to understand the issue and fix it.

Looking at traces reveals a **silent failure**:
1. Writer writes initial post
2. Reviewer identifies weakness (e.g. "The hook is weak")
3. Editor attempts to call tools but **fails due to missing inputs**
4. Editor returns the draft unchanged

From the outside everything looks fine — no errors, grammatically correct output. But the Editor's tools are silently failing. The root cause: tool definitions lack type hints and docstrings, so the LLM can't figure out what parameters to pass.

> Full details: [Root Cause Analysis](https://llmdocs.deepchecks.com/docs/root-cause-analysis-rca-content-creator)

### Step 4: Compare Versions

Test the new configuration and verify the issue has been resolved. Try multiple models to balance result quality with cost.

**The fix:** No changes to code logic, prompts, or model. Just add type hints and docstrings to the tool definitions:

```python
@tool("Hook Improver")
def hook_fix_tool(post: str, issue: str) -> str:
    """Improves the hook/opening of a blog post.

    Args:
        post: The full blog post text
        issue: What's wrong with the hook (e.g., 'too clickbaity')
    """
```

**Results:**
| Metric | Baseline | Enhanced |
|---|---|---|
| Overall Session Score | 45% | 65% |
| Tool Interaction Score | 23% | 99% |
| Tool Completeness | 1.89 | 4.8 |

The demo also compares across models (Claude 3.5 Sonnet, Claude 3.7 Sonnet, Nova Micro) to weigh quality vs. cost.

> Full details: [Compare Versions](https://llmdocs.deepchecks.com/docs/compare-versions-content-creator)

## Key Takeaway

> "Reliability in agentic workflows often comes down to clear interfaces."

The schema fix worked across all models. Trace visibility caught failures that traditional monitoring missed entirely.

## Getting Started

### Open the Demo Notebook

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1pjjzKn9wRTFOiQLf6Sf3ehOM-Xx8v7Kk)

Or use the notebook in this directory: [Content_Creator_Sagemaker_0_4.ipynb](Content_Creator_Sagemaker_0_4.ipynb)

### Prerequisites

```bash
pip install "deepchecks-llm-client[otel]" crewai crewai-tools litellm
```

### Configuration

1. Get your Deepchecks API key from the user menu in the [Deepchecks app](https://app.llm.deepchecks.com)
2. Set up your LLM provider credentials (e.g. AWS Bedrock)
3. Set up a [Serper API key](https://serper.dev/) for the search tool

## Full Documentation

For the complete walkthrough, visit the [Deepchecks documentation](https://llmdocs.deepchecks.com/docs/multi-agent-demo-content-creator-crew).

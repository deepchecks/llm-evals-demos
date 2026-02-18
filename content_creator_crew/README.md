# Multi-Agent Demo: Content Creator Crew

Evaluating and debugging a Multi-Agent application, step by step.

## The Use Case

This demo evaluates a CrewAI-powered multi-agent content creation workflow. The workflow is designed to create a blog post based on a given topic, target audience, and optional context. The workflow is evaluated using [Deepchecks](https://app.llm.deepchecks.com) to identify performance issues and root causes.

## The Workflow

The crew has three agents that run sequentially:

- **Writer** - Drafts the initial blog post based on topic, audience, and context. Has access to internet search (SerperDevTool).
- **Reviewer** - Critiques the draft and identifies improvements. Has access to internet search (SerperDevTool).
- **Editor** - Applies fixes and enhancements to produce the final post. Has access to internet search (SerperDevTool) and three custom LLM-powered tools:
 - **Hook Improver** - Enhances blog post openings
 - **Tone Adjuster** - Matches tone to target audience
 - **Content Rewriter** - Fixes specific issues in the draft

## The Problem

We run the crew and the output looks fine on the surface - no errors, grammatically correct posts, yet the blog posts seem generic and boring. Are the agents actually using their tools? Is the content as good as it could be? We use Deepchecks tracing to find out.

## What the Demo Covers

1. [**Log evaluation data**](https://llmdocs.deepchecks.com/docs/uploading-the-data-content-creator) - Set up tracing and run the crew on a set of test inputs
2. [**Analyze performance**](https://llmdocs.deepchecks.com/docs/analyze-performance-content-creator) - Break down scores by interaction type (Root, Agent, LLM, Tool) and identify weak spots
3. [**Root cause analysis**](https://llmdocs.deepchecks.com/docs/root-cause-analysis-rca-content-creator) - Find out why the Editor's tools are silently failing
4. [**Compare versions**](https://llmdocs.deepchecks.com/docs/compare-versions-content-creator) - Fix the issue, re-run, and compare across models (Claude 3.5 Sonnet, Claude 3.7 Sonnet, Nova Micro)

## Getting Started

### Open the Demo Notebook

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/deepchecks/llm-evals-demos/blob/main/content_creator_crew/Content_Creator_Sagemaker_0_4.ipynb)

Or use the notebook in this directory: [Content_Creator_Sagemaker_0_4.ipynb](Content_Creator_Sagemaker_0_4.ipynb)

### Prerequisites

```bash
pip install -U "deepchecks-llm-client[otel]" boto3 botocore
pip install crewai==0.203.0 crewai-tools==0.76.0
pip install litellm==1.79.1
```

### Configuration

1. **Deepchecks** - Set your host URL and API key ([how to get your API key](https://llmdocs.deepchecks.com/docs/setup-python-sdk-installation-api-key-retrieval))
2. **AWS Bedrock** - Configure credentials, or set the SageMaker Partner App ARN for SageMaker deployments
3. **Serper** - Set up a [Serper API key](https://serper.dev/) for the search tool

## Full Documentation

For the complete walkthrough, visit the [Deepchecks documentation](https://llmdocs.deepchecks.com/docs/multi-agent-demo-content-creator-crew).

# build-ai-agent-business-consultant

- Created at: 2025-03-13
- Created by: `🐢 Arun Godwin Patel @ Code Creations`

## Table of contents

- [Setup](#setup)
  - [System](#system)
  - [Installation](#installation)
- [Walkthrough](#walkthrough)
  - [Tech stack](#tech-stack)
  - [Build Instructions](#build-instructions)
    - [1. AI Agent](#1-ai-agent)
    - [2. Tool - FindSimilarTools](#2-tool---findsimilartools)
    - [3. Tool - EstimateAnnualRecurringRevenue](#3-tool---estimateannualrecurringrevenue)
    - [4. Tool - WhitespaceAnalysis](#4-tool---whitespaceanalysis)
    - [5. Tool - ResearchMarketSize](#5-tool---researchmarketsize)
    - [6. Tool - IdentifyTargetAudience](#6-tool---identifytargetaudience)
    - [7. Structured Output Parser](#7-structured-output-parser)
    - [8. LLM - GenerateEmailReport](#8-llm---generateemailreport)

## Setup

### System

This guide was tested on the following setup:

- Windows 10
- Docker Desktop version 4.31.1 (153621)

### Installation

1. Install Docker Desktop from the official website: [Docker Desktop](https://docs.docker.com/desktop/)
2. Visit the official n8n documentation for installation instructions: [n8n Installation](https://docs.n8n.io/hosting/installation/docker/)
3. Start the n8n container. From the terminal, run:

```bash
docker volume create n8n_data

docker run -it --rm --name n8n -p 5678:5678 -v n8n_data:/home/node/.n8n docker.n8n.io/n8nio/n8n
```

This command creates a volume to store persistent data, downloads the required n8n image, and starts your container, exposed on port 5678. To save your work between container restarts, it also mounts a docker volume, n8n_data, to persist your data locally. Once running, you can access n8n by opening: http://localhost:5678

n8n be default stores data in a `sqlite` database, but you can configure this n8n instance to work with a `PostgreSQL` database by following the instructions on the documentation site https://docs.n8n.io/hosting/installation/docker/#using-with-postgresql

## Walkthrough

This tutorial will be using `n8n` as an orchestration layer to build a `Tools Agent` leveraging LLM APIs to generate a report for a business idea. There will not be any code snippets required to complete this build, but I have included all of the prompts for the AI nodes and tools that you'll need to setup the workflow.

### Tech stack

**Orchestration**

- Automation: `n8n`

**AI**

- AI Agent: `n8n`
- LLM: `Claude`

**Communication**

- Email report: `Gmail API` & `Google Cloud Console`

### Build Instructions

To build this AI Agent for yourself, first you need to start the locally running n8n container on your machine. Alternatively, you can use the n8n cloud service to build this workflow, but this will require you to create a paid account.

Once you have access to the n8n workflow editor, follow the steps in the YouTube video to build the AI Agent.

[![Build AI Agent](https://img.youtube.com/vi/VIDEO-ID/0.jpg)](https://www.youtube.com/watch?v=VIDEO-ID)

Use the snippets below to build the relevant nodes in the n8n workflow.

#### 1. AI Agent

The AI Agent that we're building is a `Tools Agent`. Tools agents are AI agents that utilise a set of tools that is has available to complete the task at hand. In this case, the AI agent will be evaluating an early stage business idea and providing analysis and insights into if and how the business should proceed with executing this business. The AI agent will decide if/when to utilise the tools and will combine their usage to complete the task.

Here is the prompt for the AI agent:

```
## ROLE
You are a business consultant that is evaluating an early stage business idea and providing analysis and insights into if and how I should proceed with executing this business.

## TASK
You will be provided with some text describing a business idea. This idea may not be fully formed and could be vague. You must reason and think creatively to provide useful advice. When given the business idea, you must complete the following:
1. Research on the internet to find a maximum of 5 businesses that sell tools, products or services similar to this idea.
2. If there are similar businesses, use openly available data to estimate the value of these tools that these businesses are selling. The valuation should be in terms of Annual Recurring Revenue in GBP.
3. If there are similar businesses, perform a whitespace analysis to identify unique selling points that I should focus on.
4. Based on the whitespace analysis, recommend the best target audience. Be as specific as possible.
4. Extrapolate the relevant industry that this business idea and target market should focus on and perform research to estimate the market size in terms of Total Addressable Market (GBP and number of people).
5. Combine all of your workings into a summarised report, no more than 3000 words.

## RULES
- Return your response by strictly following the JSON schema provided. Do not include any additional information in the response:
{
    "success": boolean,
    "idea": string,
    "research": {
        "similar_tools": [
        {
            "name_of_tool": string,
            "business_name":  string,
            "website_url": string,
            "estimated_arr": string,
        }
        ],
        "whitespace_analysis": {
        "unique_selling_points": list[string]
        },
        "recommended_industry": {
        "industry_name": string,
        "market_size_tam": string
        }
    }
}

## DATA
- Business idea: {{ $json.chatInput }}
```

#### 2. Tool - FindSimilarTools

This tool is used to find similar tools to the business idea. Here is the prompt for the tool:

```
## ROLE
You are a business consultant that is evaluating an early stage business idea and providing analysis and insights into if and how I should proceed with executing this business.

## TASK
Find a maximum of 5 businesses that sell tools, products or services similar to the business idea provided. For each business, provide the following information:
1. Name of the tool, product or service.
2. Name of the business that sells the tool, product or service.
3. Website URL of the business.

## DATA
- Business idea: {{ $json.text }}
```

#### 3. Tool - EstimateAnnualRecurringRevenue

This tool is used to research the estimated Annual Recurring Revenue of a tool that a business sells. Here is the prompt for the tool:

```
## ROLE
You are a business consultant that is evaluating an early stage business idea and providing analysis and insights into if and how I should proceed with executing this business.

## TASK
If there are similar tools to the business idea that has been proposed, use openly available data to estimate the value of these tools that these businesses are selling. The valuation should be in terms of Annual Recurring Revenue in GBP.

## DATA
- Business idea: {{ $json.text }}
```

#### 4. Tool - WhitespaceAnalysis

This tool conducts a whitespace analysis for a business idea to find opportunities for unique selling points. Here is the prompt for the tool:

```
## ROLE
You are a business consultant that is evaluating an early stage business idea and providing analysis and insights into if and how I should proceed with executing this business.

## TASK
Based on the business idea and similar tools that businesses sell, perform a whitespace analysis to identify unique selling points that I should focus on.

## RULES
- Return your response in a list of strings in valid JSON format without any proceeding or following string characters

## DATA
- Business idea: {{ $json.text }}
```

#### 5. Tool - ResearchMarketSize

This tool is used to estimate the market size for a business idea in terms of total addressable market. Here is the prompt for the tool:

```
## ROLE
You are a business consultant that is evaluating an early stage business idea and providing analysis and insights into if and how I should proceed with executing this business.

## TASK
Extrapolate the relevant industry that this business idea should focus on and perform research to estimate the market size in terms of Total Addressable Market (GBP and number of people).

## DATA
- Business idea: {{ $json.text }}
```

#### 6. Tool - IdentifyTargetAudience

This tool is used to estimate the market size for a business idea in terms of total addressable market. Here is the prompt for the tool:

```
## ROLE
You are a business consultant that is evaluating an early stage business idea and providing analysis and insights into if and how I should proceed with executing this business.

## TASK
Recommend the best target audience. Be as specific as possible.

## DATA
- Business idea: {{ $json.text }}
```

#### 7. Structured Output Parser

This node is used to ensure that the output generated by the AI Agent is in the correct format. Here is the JSON schema:

```json
{
  "$id": "https://example.com/business_idea.schema.json",
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "Business Idea Research Schema",
  "type": "object",
  "properties": {
    "success": {
      "type": "boolean"
    },
    "idea": {
      "type": "string"
    },
    "research": {
      "type": "object",
      "properties": {
        "similar_tools": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "name_of_tool": {
                "type": "string"
              },
              "business_name": {
                "type": "string"
              },
              "website_url": {
                "type": "string"
              },
              "estimated_arr": {
                "type": "string"
              }
            },
            "required": [
              "name_of_tool",
              "business_name",
              "website_url",
              "estimated_arr"
            ]
          }
        },
        "whitespace_analysis": {
          "type": "object",
          "properties": {
            "unique_selling_points": {
              "type": "array",
              "items": {
                "type": "string"
              }
            }
          },
          "required": ["unique_selling_points"]
        },
        "recommended_industry": {
          "type": "object",
          "properties": {
            "industry_name": {
              "type": "string"
            },
            "market_size_tam": {
              "type": "string"
            }
          },
          "required": ["industry_name", "market_size_tam"]
        }
      },
      "required": [
        "similar_tools",
        "whitespace_analysis",
        "recommended_industry"
      ]
    }
  },
  "required": ["success", "idea", "research"]
}
```

#### 8. LLM - GenerateEmailReport

This node is used to generate an email report that summarises all of the findings from the AI Agent. Here is the prompt for the node:

```
## ROLE
You are a business consultant that is evaluating an early stage business idea and providing analysis and insights into if and how I should proceed with executing this business.

## TASK
Summarise all of your findings using the data provided in less than 3000 words.

## DATA
- Business idea: {{ $('When chat message received').item.json.chatInput }}
- Similar tools: {{ $json.output.research.similar_tools.map((obj, idx) => ` ${idx + 1}. Tool: ${obj.name_of_tool}, Business: ${obj.business_name}, Website URL: ${obj.website_url}`) }}
- Whitespace analysis: {{ $json.output.research.whitespace_analysis.unique_selling_points }}
- Recommended industry: {{ $json.output.research.recommended_industry.industry_name }}, market size = {{ $json.output.research.recommended_industry.market_size_tam }}
```

Outside of these LLM based nodes, you can also add a `Simple Memory Node` to store the business idea that is provided by the user. This will allow you to reference the business idea in the subsequent messages.

Finally, we need a `Gmail` node to send the email report to the user. You can follow the instructions on the n8n documentation to setup the Gmail node: [Gmail Node](https://docs.n8n.io/integrations/builtin/credentials/google/oauth-single-service/?utm_source=n8n_app&utm_medium=credential_settings&utm_campaign=create_new_credentials_modal#create-a-google-cloud-console-project)

This completes the setup of our AI Business Consultant Agent running on n8n locally! You can test it by issuing a chat message in the n8n workflow editor and waiting for the email report to be generated.

Here are some example busines ideas that you can use to test the AI Agent:

```
- Business idea: An app that analyses my wardrobe and suggests outfits based on the weather forecast and my mood.
- Business idea: An web extension that analyses content on any webpage and makes it more accessible for people with dyslexia.
- Business idea: A device that translates spoken language in real-time whilst wearing it in ear.
```

## Happy building! 🚀

```bash
🐢 Arun Godwin Patel @ Code Creations
```

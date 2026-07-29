# SEO Blog Engine — Automated Content Pipeline

An end-to-end AI-powered SEO content automation system built in n8n using a no-code approach. Takes a topic as input and produces a fully researched, human-refined, SEO-optimized blog post published directly to WordPress as a draft — in under 5 minutes.

---

## What It Does

1. **Keyword & Niche Research** — searches Google via SerpAPI for real search data on the input topic
2. **Niche Analysis** — Google Gemini analyzes search data, identifies keyword opportunities, competition levels, and selects the best target keyword
3. **SERP Analysis** — fetches real competitor data for the chosen keyword from Google's top results
4. **Content Brief** — Groq Llama 4 Scout builds a detailed SEO content brief including keyword blueprint, competitive gap analysis, H1-H3 skeleton, and internal linking opportunities
5. **Blog Writing** — Google Gemini writes a complete long-form blog post using a strict human-style protocol — no AI clichés, varied sentence structure, active voice, exhaustive depth per section
6. **SEO Audit** — Groq Qwen analyzes the blog post and generates meta title, meta description, slug, focus keyword, tags, category, image alt texts, and schema type in structured format
7. **Human Editing Layer** — Groq Llama 3.1 acts as a senior copyeditor — strips hallucinations, removes robotic patterns, refines flow, and rewrites the draft into polished final prose while preserving all headers
8. **Markdown to HTML Conversion** — converts the refined Markdown post to clean HTML for WordPress rendering
9. **WordPress Publishing** — saves the complete post as a draft via WordPress REST API

---

## Architecture — Multi-LLM Design

This pipeline uses a deliberate multi-LLM architecture assigning each task to the model best suited for it:

| Node | Model | Role |
|---|---|---|
| AI - Niche Analysis | Google Gemini | Strategic keyword research and niche analysis |
| AI - Content Brief | Groq Llama 4 Scout | Competitive gap analysis and content skeleton |
| AI - Write Blog Post | Google Gemini | Long-form human-style blog writing |
| AI - SEO Optimization | Groq Qwen 3 32B | Structured SEO field extraction and audit |
| AI - Editor Checklist | Groq Llama 3.1 8B | Human copyediting and hallucination removal |

---

## Workflow Structure

```
Manual Trigger
→ Set Topic (input your niche/topic)
→ Niche Research (SerpAPI — Google search data)
→ Wait 1 (30 seconds)
→ AI - Niche Analysis (Gemini — keyword selection)
→ SerpAPI - SERP Analysis (competitor data)
→ Wait 2 (30 seconds)
→ AI - Content Brief (Groq Llama 4 — SEO brief)
→ Prepare Data (merge topic + niche + SERP data)
→ Wait 3 (30 seconds)
→ AI - Write Blog Post (Gemini — full article)
→ Wait 4 (15 seconds)
→ AI - SEO Optimization (Groq Qwen — meta data)
→ Wait 5 (5 seconds)
→ AI - Editor Checklist (Groq Llama 3.1 — copyedit)
→ Format Data (JavaScript — package for WordPress)
→ Markdown (convert to HTML)
→ HTTP Request (publish draft to WordPress REST API)
```

---

## Key Design Decisions

**Why multiple LLMs?**
Distributing tasks across Google Gemini and Groq's different models serves two purposes. First, each model has specific strengths — Gemini excels at strategic analysis and long-form writing, while Groq's models are faster for structured output generation and editorial tasks. Second, spreading calls across providers avoids hitting any single provider's free tier rate limits mid-execution.

**Why Wait nodes between AI calls?**
Gemini's free tier allows 15 requests per minute. Without Wait nodes the workflow fires all AI calls in rapid succession and triggers rate limit errors. Wait nodes of 5-30 seconds between stages keep requests within limits for uninterrupted execution.

**Why a separate Editor node?**
The AI - Editor Checklist node is not actually a checklist — it's a full human copyediting pass. It takes the raw blog post and rewrites it to remove AI patterns, hallucinations, and robotic flow issues. This two-stage writing process (write then refine) produces significantly more natural output than a single writing pass.

**Why Markdown to HTML conversion?**
WordPress renders HTML natively. Sending raw Markdown produces unformatted output. The Markdown node converts the refined post to clean HTML so headers, bold text, bullet points, and structure all render correctly in the WordPress editor.

**Why save as draft not publish directly?**
AI-generated content requires human review before publishing. The draft workflow allows editorial review, image addition, internal link verification, and final quality check before going live.

---

## Tech Stack

- **Automation:** n8n (cloud or self-hosted)
- **AI Models:** Google Gemini, Groq Llama 4 Scout, Groq Qwen 3 32B, Groq Llama 3.1 8B
- **Search Data:** SerpAPI (Google search and SERP results)
- **Publishing:** WordPress REST API via HTTP Request
- **Format Conversion:** n8n Markdown node
- **Language:** JavaScript (n8n Code nodes)

---

## How to Use

### Prerequisites
- n8n instance (cloud or self-hosted)
- Google Gemini API key — free at aistudio.google.com
- Groq API key — free at console.groq.com
- SerpAPI key — free tier at serpapi.com
- WordPress site with REST API enabled and Application Password generated

### Setup Steps

**1. Import the workflow**
- Download `seo-blog-engine.json`
- In n8n go to Workflows → Import from file
- Select the JSON file

**2. Add credentials**
- Google Gemini API → Settings → Credentials → New → Google Gemini(PaLM) API
- Groq → Settings → Credentials → New → Groq
- WordPress Basic Auth → Settings → Credentials → New → Basic Auth (username + application password)
- SerpAPI → edit both HTTP Request nodes (Niche Research and SerpAPI - SERP Analysis) → update the Query Auth credential with your SerpAPI key named `api_key`

**3. Set your topic**
- Open the Set Topic node
- Change the value field to your target topic
- Example: `best free AI tools for content creation`

**4. Execute**
- Click Execute Workflow
- Wait 3-5 minutes for all stages to complete
- Check your WordPress drafts for the published post

---

## Output

Each run produces a WordPress draft containing:
- Full long-form SEO-optimized blog post in HTML format
- Properly formatted headers, bullet points, and bold text
- Editor notes appended at the bottom including focus keyword

---

## Prompt Engineering Approach

Each AI node uses a carefully designed system prompt with:

- **Role definition** — each model is given a specific expert identity
- **Phased execution** — prompts are structured in phases (audit → execute → output)
- **Strict formatting rules** — banned phrases, output format requirements, and structural guardrails
- **Anti-hallucination constraints** — explicit instructions to use only provided data
- **Human style enforcement** — varied sentence structure, active voice, banned AI clichés

The Editor node prompt explicitly bans these phrases across all output:
```
"In today's digital landscape" / "Delve deeper" / "Furthermore" / 
"Moreover" / "In conclusion" / "Tapestry" / "Beacon" / "Testament"
```

---

## Rate Limiting Strategy

| Provider | Free Tier Limit | Nodes Using It |
|---|---|---|
| Google Gemini | 15 requests/minute | Niche Analysis, Write Blog Post |
| Groq | 30 requests/minute | Content Brief, SEO Optimization, Editor |
| SerpAPI | 100 searches/month | Niche Research, SERP Analysis |

Total API calls per run: 7 (2 SerpAPI + 5 AI calls)
SerpAPI usage per month at daily runs: ~210 searches (requires paid plan beyond 100)

---

## Files

```
/seo-blog-engine
  seo-blog-engine.json    — importable n8n workflow
  README.md               — this file
```

---

## Author

Muhammad Zaeem — github.com/zaeem270

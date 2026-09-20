# Product Signal Architecture

## Overview

Product Signal is implemented as an n8n-based automation pipeline that transforms raw news feeds into a ranked daily intelligence brief.

The current MVP focuses on two news sources:

- OpenAI News RSS
- TechCrunch RSS

The pipeline processes articles published within the last 24 hours, removes duplicates, analyzes each article with an LLM, calculates a signal score, selects the top signals, and generates a daily brief.

---

## End-to-End Pipeline

```text
OpenAI News RSS ──────┐
                      │
TechCrunch RSS ───────┤
                      ↓
                Normalize Fields
                      ↓
                24h Time Filter
                      ↓
                    Merge
                      ↓
              Remove Duplicates
                      ↓
               News Analyst
                      ↓
             Structured Analysis
                      ↓
              Signal Calculation
                      ↓
              Signal Classification
                      ↓
               Sort by Score
                      ↓
                  Top 5
                      ↓
                AI Daily Editor
                      ↓
             Telegram Formatter
                      ↓
                  Telegram
```

---

## 1. Schedule Trigger

The workflow starts through an n8n Schedule Trigger.

The current workflow is configured to run once per day at approximately **19:35**.

The trigger starts the complete news intelligence pipeline.

---

## 2. News Collection

The current MVP collects articles from two RSS sources.

### OpenAI News

OpenAI's official news feed is used as a direct source for OpenAI-related announcements and developments.

### TechCrunch

TechCrunch provides broader technology, startup, business, and AI coverage.

The architecture is intentionally source-agnostic, allowing additional RSS or other structured sources to be added later.

---

## 3. Normalization

Different RSS sources expose information using different field structures.

The normalization layer converts incoming records into a common schema.

The core fields are:

```text
title
url
source
published_at
content
```

This allows downstream nodes to process articles consistently regardless of their source.

---

## 4. Time Filtering

The workflow keeps articles published within the most recent **24 hours**.

Older articles are removed before the ranking pipeline.

This ensures that the daily brief focuses on current information rather than repeatedly processing historical stories.

---

## 5. Source Merge

The normalized articles from different sources are merged into a single stream.

Conceptually:

```text
OpenAI RSS
     \
      → Normalized Article Stream
     /
TechCrunch RSS
```

This allows the downstream AI and ranking layers to evaluate articles across sources using the same logic.

---

## 6. Duplicate Removal

Duplicate articles are removed using the article URL as the deduplication key.

This prevents the same article from consuming analysis capacity or appearing multiple times in the final brief.

The current implementation uses URL-level deduplication.

Future versions may introduce semantic duplicate detection to identify different URLs covering the same underlying event.

---

## 7. AI News Analysis

Each article is passed to the **News Analyst**.

The News Analyst evaluates the article across four scoring dimensions:

- Relevance
- Impact
- Novelty
- Actionability

It also produces:

- Category
- Why it matters
- Product takeaway

The result is structured JSON.

Detailed prompt documentation is available in:

`prompts/news-analyst.md`

---

## 8. Structured Output

The News Analyst returns structured data rather than free-form text.

The current schema is:

```json
{
  "category": "AI",
  "relevance": 3,
  "impact": 3,
  "novelty": 2,
  "actionability": 3,
  "why_it_matters": "This matters because it may affect how AI products are built and adopted.",
  "product_takeaway": "Product teams should evaluate the implications for their AI workflows."
}
```

Structured output allows the workflow to perform deterministic calculations and ranking after the LLM step.

---

## 9. Signal Calculation

The workflow calculates a normalized score from 0 to 10 using:

```text
(relevance + impact + novelty + actionability) / 16 × 10
```

The score is stored as:

```text
signal_score
```

The workflow then assigns a qualitative signal level:

```text
>= 8       → Must Know
>= 6       → Worth Knowing
< 6        → Noise
```

The complete scoring model is documented in:

`docs/scoring-model.md`

---

## 10. Ranking

After scoring, articles are sorted by `signal_score` in descending order.

The current MVP selects the top 5 signals.

This creates a deliberate compression layer:

```text
Many Articles
      ↓
Analyzed Articles
      ↓
Ranked Signals
      ↓
Top 5
```

The objective is to reduce information overload rather than maximize the number of articles delivered to the user.

---

## 11. AI Daily Editor

The selected top 5 signals are passed to the AI Editor.

The AI Editor creates a concise daily intelligence brief.

For each selected story, the output contains:

- Headline
- Source
- What happened
- Why it matters
- Product takeaway
- Signal score

The editor also creates a:

### Today's Pattern

This section identifies the most important common theme or pattern across the selected stories.

Detailed prompt documentation is available in:

`prompts/daily-editor.md`

---

## 12. Telegram Formatting

The generated daily brief is passed through a formatting layer before delivery.

The formatter:

- Adds the Product Signal header
- Adds the current date
- Converts Markdown headings into a Telegram-friendly format
- Adds visual labels for key sections
- Formats the "Today's Pattern" section

The current output is designed for Telegram delivery.

---

## 13. Delivery

The final formatted brief is sent to Telegram.

Telegram is currently the primary delivery channel for the MVP.

Future versions may support additional channels such as:

- Email
- Slack
- Microsoft Teams
- Web dashboard
- Mobile notifications

---

## Architecture Principles

### 1. Separate Collection from Intelligence

News collection and news interpretation are separate stages.

This allows sources to change without redesigning the analysis layer.

### 2. Separate Analysis from Editorial Synthesis

The News Analyst evaluates individual articles.

The AI Editor synthesizes the highest-value signals.

This separation makes each AI responsibility easier to evaluate and iterate.

### 3. Structured AI Output

LLM output is converted into structured data before deterministic ranking logic is applied.

This reduces dependency on free-form model output.

### 4. Deterministic Ranking

The current ranking formula is explicit and inspectable.

The system does not rely entirely on the LLM to decide the final ordering.

### 5. Modular Delivery

Telegram is treated as a delivery layer rather than being deeply coupled to the intelligence pipeline.

This makes future channels easier to add.

---

## Current Technology Stack

| Layer | Technology |
|---|---|
| Workflow orchestration | n8n |
| News ingestion | RSS |
| AI analysis | OpenAI |
| Model | GPT-5.4-mini |
| Structured output | JSON |
| Delivery | Telegram |
| Local runtime | Docker |
| Version control | Git |
| Repository | GitHub |

---

## Current MVP Architecture

The current architecture intentionally prioritizes simplicity and transparency.

```text
Sources
  ↓
Normalize
  ↓
Filter
  ↓
Deduplicate
  ↓
LLM Analysis
  ↓
Deterministic Scoring
  ↓
Ranking
  ↓
Top 5
  ↓
LLM Editorial Synthesis
  ↓
Telegram
```

This provides a foundation for later experiments with personalization, semantic deduplication, trend detection, feedback loops, and more advanced ranking systems.
# Signal Scoring Model

## Purpose

Product Signal uses a lightweight scoring model to prioritize news based on four dimensions:

- Relevance
- Impact
- Novelty
- Actionability

The goal is not to determine whether a news article is objectively important.

The goal is to estimate how valuable the article is as a professional signal for the target audience.

---

## Scoring Dimensions

Each dimension is scored by the News Analyst from **0 to 4**.

### 1. Relevance

Measures how relevant the article is to a Product, Business, or Technology professional.

| Score | Meaning |
|---|---|
| 0 | Not relevant |
| 1 | Weak relevance |
| 2 | Moderately relevant |
| 3 | Highly relevant |
| 4 | Directly relevant |

---

### 2. Impact

Measures the potential business, product, technology, or market impact of the information.

| Score | Meaning |
|---|---|
| 0 | Negligible |
| 1 | Low potential impact |
| 2 | Moderate potential impact |
| 3 | High potential impact |
| 4 | Very high potential impact |

---

### 3. Novelty

Measures how new or significant the information is.

| Score | Meaning |
|---|---|
| 0 | Not novel |
| 1 | Limited novelty |
| 2 | Moderately novel |
| 3 | Highly novel |
| 4 | Highly significant/new |

---

### 4. Actionability

Measures whether a professional could learn from, react to, or act on the information.

| Score | Meaning |
|---|---|
| 0 | Not actionable |
| 1 | Low actionability |
| 2 | Moderately actionable |
| 3 | Actionable |
| 4 | Highly actionable |

---

## Signal Score

The four dimensions are normalized into a score from **0 to 10**.

```text
Signal Score =
(relevance + impact + novelty + actionability) / 16 × 10
```

The maximum possible raw score is:

```text
4 + 4 + 4 + 4 = 16
```

Therefore:

```text
16 / 16 × 10 = 10
```

The minimum possible score is:

```text
0 / 16 × 10 = 0
```

This creates a normalized signal score between **0 and 10**.

---

## Signal Levels

The current MVP uses three qualitative signal levels.

| Signal Score | Signal Level |
|---:|---|
| `>= 8` | Must Know |
| `>= 6 and < 8` | Worth Knowing |
| `< 6` | Noise |

### Must Know

High-priority signals that score at least 8/10.

These are candidates for the most prominent position in the daily brief.

### Worth Knowing

Signals scoring between 6 and 8.

These may be useful to the target audience but are less critical than Must Know signals.

### Noise

Signals scoring below 6.

These are deprioritized and excluded from the final Top 5 selection.

---

## Ranking Logic

After scoring:

1. Each article receives a `signal_score`.
2. Each article receives a `signal_level`.
3. Articles are sorted by `signal_score` in descending order.
4. The top 5 signals are selected.
5. The selected signals are passed to the AI Editor.
6. The AI Editor creates the daily intelligence brief.

Conceptually:

```text
News Articles
      ↓
AI Analysis
      ↓
4-Dimension Scoring
      ↓
Signal Score (0–10)
      ↓
Signal Level
      ↓
Sort Descending
      ↓
Top 5
      ↓
AI Daily Editor
```

---

## Why Four Dimensions?

The scoring model intentionally separates four different concepts.

A story can be:

- highly relevant but low impact
- highly impactful but not very actionable
- novel but irrelevant
- relevant and actionable without being particularly novel

Using multiple dimensions prevents the ranking from depending on a single concept such as popularity or recency.

The current model is intentionally simple and interpretable so that it can be evaluated and iterated as the product evolves.

---

## Current Limitations

The current scoring model is a heuristic MVP.

It does not yet include:

- User-specific personalization
- Historical engagement signals
- Source reliability weighting
- Topic-level trend detection
- Time-decay scoring
- Cross-source corroboration
- Learned ranking
- Human evaluation feedback
- Calibration against actual user behavior

These are potential areas for future experimentation.

---

## Future Direction

A future version could combine the current interpretable score with behavioral and contextual signals.

For example:

```text
Final Signal Score
        =
Content Signal
        +
User Relevance
        +
Historical Engagement
        +
Trend Momentum
        +
Source Confidence
```

Any future scoring model should be evaluated against actual user outcomes rather than optimizing the numerical score itself.
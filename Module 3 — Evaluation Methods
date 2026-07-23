# Module 3 — Evaluation Methods

> **Module Goal:** Understand the actual mechanisms used to score LLM outputs. By the end of this module, you should be able to explain programmatic, deterministic, human, and LLM-as-a-judge evaluation, compare their tradeoffs, and choose the right method for a given task.

---

## 📍 Where This Fits

Module 2 established *what* to evaluate — the workflow, failure points, risk categories, and the reference-based vs. reference-free distinction. This module answers the next question: *how* do you actually produce a score?

```mermaid
flowchart LR
    M2[Module 2: Application Evaluation] --> M3["Module 3: Evaluation Methods<br/>(you are here)"]
    M3 --> M4[Module 4: Online vs Offline Evaluation]
```

Every method described here can be used for both reference-based and reference-free evaluation — the method is *how* you score, while reference-based/free is *what you're comparing against* (or not comparing against).

---

## 1. Programmatic Evaluation

### Intuition

Imagine you want to check whether a model's output is valid JSON, contains a phone number in the right format, or stays under a word limit. You don't need a human or another model to judge this — you need code. Write a function, run it against the output, get a pass/fail or a numeric score, instantly and at zero marginal cost.

**Programmatic evaluation** is evaluation by code: any check that can be automated using logic, pattern matching, or computed metrics.

### Definition

**Programmatic Evaluation** is the use of automated code-based checks — regular expressions, parsers, computed statistical metrics, or scripted business logic — to score LLM outputs without human or model-based judgment.

### Why It Exists

Programmatic checks are fast, cheap, perfectly consistent (the same input always produces the same score), and can run at massive scale — thousands or millions of evaluations per run, in seconds. Wherever a property can be checked mechanically, doing so is far more efficient than involving a human or another model.

### Examples of Programmatic Checks

| Check Type | Example |
|---|---|
| **Format validation** | Is the output valid JSON / valid markdown / a valid email format? |
| **Length constraints** | Is the summary under 100 words? |
| **Keyword / pattern presence** | Does the response include a required disclaimer? Does it avoid a banned word list? |
| **Structural checks** | Does a generated SQL query parse without syntax errors? |
| **Computed similarity metrics** | BLEU, ROUGE, or embedding cosine similarity against a reference |
| **Code correctness** | Does generated code pass a set of unit tests? |

### Real-World Analogy

Programmatic evaluation is like a **factory's automated inspection scanner** — a machine that checks every item on the line for weight, dimensions, or barcode validity in a fraction of a second, without needing a human inspector to look at each one.

### Practical Example

An application generates a JSON object representing a calendar event from a user's natural-language request ("lunch with Sam next Tuesday at noon"). A programmatic eval can check, without any human involvement:

- Is the output valid JSON? *(format check)*
- Does it contain all required fields — `title`, `date`, `time`? *(structural check)*
- Does the parsed date fall on a Tuesday? *(logic check against the input)*

### Industry Use Case

Programmatic checks are the backbone of **continuous integration for LLM applications** — they're cheap enough to run on every single change to a prompt or pipeline, providing instant feedback before anything more expensive (like human review) is needed.

### Common Mistakes

- Over-relying on programmatic checks for properties that are inherently subjective (like "is this response helpful?"), where code simply can't capture the nuance.
- Using surface-level metrics like word overlap (e.g., naive BLEU/ROUGE scores) as a stand-in for actual quality or correctness, when they often correlate weakly with human judgment on open-ended text.
- Treating a passing programmatic check as proof of overall quality, when it only verifies the narrow property it was written to check.

### Interview Questions

- What kinds of properties are well-suited to programmatic evaluation, and which aren't?
- Why might a metric like ROUGE be a poor proxy for summary quality?
- How would you programmatically evaluate whether a model's output is safe to render in a UI?

### Key Takeaways

- Programmatic evaluation uses code — not humans or models — to score outputs, making it fast, cheap, and perfectly consistent.
- It excels at structural, format, and objectively computable checks.
- It's a poor fit for subjective quality judgments, which need human or LLM-as-a-judge evaluation instead.

---

## 2. Deterministic Evaluation

### Intuition

Deterministic evaluation is closely related to programmatic evaluation, but it's worth separating out as its own concept because it emphasizes a specific property: **given the same input, you always get the same score, with no ambiguity or variance in the judgment itself.**

Think of it as the difference between a **ruler** and an **art critic**. A ruler gives you the exact same measurement every time, regardless of who's holding it or what mood they're in. An art critic might give a slightly different opinion each time they look at the same painting.

### Definition

**Deterministic Evaluation** refers to scoring methods that produce identical, reproducible results for identical inputs — typically rule-based exact-match or logic-based checks, as opposed to methods involving human or model judgment, which can vary between runs or raters.

### Why It Exists

Determinism matters because it enables **trustworthy comparison over time**. If your scoring method itself is noisy, you can't tell whether a score changed because the system actually got better or worse, or just because the judge happened to be inconsistent that day. Deterministic evaluation removes that ambiguity entirely for the properties it covers.

### Deterministic vs. Non-Deterministic Scoring

| | Deterministic | Non-Deterministic |
|---|---|---|
| **Same input → same score?** | Always | Not guaranteed |
| **Examples** | Exact string match, regex match, unit test pass/fail | Human ratings, LLM-as-a-judge scores |
| **Best for** | Objective, well-defined correctness | Subjective quality, nuanced judgment |
| **Tradeoff** | Can't capture nuance or open-ended quality | Captures nuance, but introduces variance |

### Real-World Analogy

A **spell-checker** is deterministic — the same misspelled word will be flagged the same way every single time. A **book review**, even from the same reviewer, might shift slightly depending on mood, context, or what else they've recently read. Deterministic evaluation aims to be the spell-checker: rigid, but perfectly reliable within its scope.

### Practical Example

- **Deterministic:** "Does the model's answer to `2 + 2` exactly equal `4`?" — always the same result.
- **Not deterministic (by contrast):** "Rate the helpfulness of this response on a scale of 1–5" — asked to a human twice, or to an LLM judge twice, may yield slightly different scores each time.

### Industry Use Case

Deterministic checks are the first line of defense in most eval pipelines — they run first, catch clear-cut failures cheaply, and free up more expensive human or LLM-judge evaluation for the cases that genuinely require nuanced judgment.

### Common Mistakes

- Assuming all programmatic evaluation is automatically deterministic — some computed metrics (like embedding-based similarity, depending on model versioning) can introduce subtle non-determinism.
- Forcing deterministic scoring onto inherently subjective tasks, producing a score that's reproducible but meaningless.
- Neglecting to combine deterministic checks with judgment-based methods — relying on determinism alone leaves quality gaps that only human or LLM judgment can catch.

### Interview Questions

- What does it mean for an evaluation method to be "deterministic," and why does that matter?
- Give an example of a task where deterministic evaluation is sufficient, and one where it isn't.
- Can programmatic evaluation ever be non-deterministic? Explain.

### Key Takeaways

- Deterministic evaluation always produces the same score for the same input — critical for reliable, comparable measurement over time.
- It's ideal for objective correctness checks, but cannot capture subjective quality.
- Most robust eval pipelines combine deterministic checks with judgment-based methods rather than relying on either alone.

---

## 3. Human Evaluation

### Intuition

Some qualities simply can't be captured by code: Is this response genuinely *helpful*? Does this joke actually land? Does this tone feel appropriately empathetic for a sensitive topic? For these, there's no substitute for a human looking at the output and forming a judgment — the same way a human ultimately decides whether a movie is good, not an algorithm.

### Definition

**Human Evaluation** is the process of having human raters review LLM outputs and score or judge them according to a defined rubric or their own expert judgment.

### Why It Exists

Humans remain the gold standard for nuanced, subjective, and context-sensitive judgment — especially for qualities like tone, empathy, cultural appropriateness, and genuinely novel failure modes that no automated system was designed to catch. Human evaluation exists because, ultimately, LLM applications are built for humans, and human judgment is the most direct proxy for whether real users will be satisfied.

### How It Typically Works

```mermaid
flowchart LR
    O["Model / Application Output"] --> R["Human Rater(s)"]
    Rub["Rubric / Guidelines"] --> R
    R --> S["Score or Verdict"]
    S --> Agg["Aggregate Across Multiple Raters<br/>(for reliability)"]
```

Human evaluation typically uses one or more of these formats:

- **Absolute scoring** — rate a single output on a scale (e.g., 1–5 for helpfulness).
- **Pairwise comparison** — shown two outputs (e.g., from two model versions), pick which is better.
- **Binary pass/fail against a rubric** — does this output meet a defined bar or not?

### Real-World Analogy

Human evaluation is like a **restaurant critic's review**. No automated sensor can tell you whether a dish is genuinely delicious, well-balanced, or memorable — that judgment requires an actual palate and human context. Some things can only be assessed by the audience they're built for.

### Practical Example

A company redesigns its customer support bot's tone to feel "warmer." No automated metric can directly measure "warmth." Instead, they have a panel of human raters:

- Read pairs of responses (old tone vs. new tone) to the same customer question
- Pick which response feels warmer and more genuinely helpful
- Provide brief notes explaining their choice

This produces a preference signal that guides the redesign — something no regex or word-count check could ever provide.

### Industry Use Case

Human evaluation (often called **human preference evaluation** when done via pairwise comparison) is central to how AI labs fine-tune models for helpfulness and safety — human raters' judgments are frequently used as the basis for techniques like reinforcement learning from human feedback (RLHF). It's also standard practice for product teams validating high-stakes or highly subjective features before launch.

### Common Mistakes

- Using a single rater with no cross-checking, introducing individual bias into the results.
- Providing vague rubrics ("rate the quality"), leading to inconsistent judgments across raters.
- Treating human evaluation as infinitely scalable — it's slow and expensive, so it needs to be used selectively (see the comparison section below).

### Interview Questions

- When is human evaluation necessary, even when automated methods are available?
- What are the main drawbacks of relying entirely on human evaluation?
- How would you design a rubric to reduce inconsistency between human raters?

### Key Takeaways

- Human evaluation is essential for subjective, nuanced qualities that automated methods can't capture.
- It typically takes the form of absolute scoring, pairwise comparison, or rubric-based pass/fail.
- It's slow and expensive relative to automated methods, making it best reserved for high-value or high-ambiguity evaluation needs.

---

## 4. LLM-as-a-Judge

### Intuition

Human evaluation is powerful but slow and expensive. Programmatic evaluation is fast and cheap but blind to nuance. What if you could get something in between — a judge that can assess nuanced, subjective qualities like a human, but that runs automatically, at scale, in seconds?

That's the idea behind **LLM-as-a-judge**: use another (often more capable, or specially prompted) language model to evaluate the output of the system under test.

### Definition

**LLM-as-a-Judge** is an evaluation method where a language model is prompted to assess and score the output of another model (or the same model) according to a defined rubric, standing in for a human evaluator.

### Why It Exists

LLM-as-a-judge exists to bridge the gap between the speed/cost of programmatic evaluation and the nuance of human evaluation. It allows teams to run large-scale, judgment-based evaluation continuously — on every prompt change, every model update, every day in production — which would be prohibitively slow and expensive with human raters alone.

### How It Works

```mermaid
flowchart LR
    O["Output to Evaluate"] --> JP["Judge Prompt<br/>(rubric + instructions)"]
    JP --> J["Judge Model"]
    J --> V["Verdict / Score<br/>+ optional reasoning"]
```

A judge prompt typically includes:

- The original input/task
- The output being evaluated
- A clear rubric or set of criteria
- Instructions on the output format (e.g., a score from 1–5, or a pass/fail verdict, often with a brief justification)

### Real-World Analogy

LLM-as-a-judge is like having a **teaching assistant grade exams using a detailed rubric provided by the professor**. The TA isn't the ultimate authority the way the professor is, and their grading might occasionally miss what a professor would catch — but with a clear enough rubric, they can grade hundreds of exams consistently and quickly, at a fraction of the professor's time cost.

### Practical Example

To evaluate whether a customer support response is "empathetic and accurate," a judge prompt might instruct another model:

> "You will be shown a customer's message and a support agent's response. Rate the response from 1–5 on empathy, and separately assess whether it is factually consistent with the provided policy document. Explain your reasoning briefly before giving your score."

This can be run against thousands of examples automatically — something no human team could do at that scale, on that timeline.

### Comparison: LLM-as-a-Judge vs. Human Evaluation

| | LLM-as-a-Judge | Human Evaluation |
|---|---|---|
| **Speed** | Fast — thousands of evaluations in minutes | Slow — limited by human throughput |
| **Cost** | Low relative to human labor | High — requires paid human time |
| **Consistency** | Can be more consistent across large volumes, but can also inherit systematic biases | Can vary between raters, but captures genuinely novel judgment |
| **Nuance ceiling** | Bounded by the judge model's own capability and biases | Generally higher for deeply subjective or novel qualities |
| **Best used for** | Continuous, large-scale evaluation | High-stakes, ambiguous, or foundational calibration work |

### Industry Use Case

LLM-as-a-judge has become one of the most widely adopted evaluation methods in the industry precisely because it allows teams to run "human-like" judgment continuously in CI pipelines and production monitoring — something that would be financially and logistically impossible with human raters at the same frequency and scale.

### Common Mistakes

- Treating LLM-as-a-judge scores as infallible ground truth — judge models can have their own blind spots, biases, and inconsistencies.
- Using a vague or underspecified judge prompt, leading to unreliable verdicts (the same "garbage in, garbage out" problem as any evaluation).
- Never validating the judge model itself against human judgment — a healthy practice is to periodically check that judge verdicts actually correlate with what human raters would say.

### Interview Questions

- How would you validate that an LLM-as-a-judge is producing trustworthy scores?
- What are the risks of using LLM-as-a-judge without any human oversight?
- Why has LLM-as-a-judge become so popular in production evaluation pipelines?

### Key Takeaways

- LLM-as-a-judge uses a language model to assess another model's output against a rubric, bridging speed and nuance.
- It enables continuous, large-scale, judgment-based evaluation that would be infeasible with human raters alone.
- Its reliability depends on rubric quality and periodic validation against human judgment — it's a powerful tool, not an infallible oracle.

---

## 5. Comparison of All Methods

### Side-by-Side Comparison

| Method | Speed | Cost | Consistency | Captures Nuance | Best Fit |
|---|---|---|---|---|---|
| **Programmatic** | Very fast | Very low | Perfectly consistent | No | Structural/format checks, computed metrics |
| **Deterministic** | Very fast | Very low | Perfectly consistent | No | Objective correctness with a known answer |
| **Human** | Slow | High | Variable (rater-dependent) | Highest | Subjective, high-stakes, or novel judgment |
| **LLM-as-a-Judge** | Fast | Low–Moderate | Moderate–High (with good rubric) | Moderate–High | Scalable, nuanced evaluation across large volumes |

### Visualizing the Tradeoff Space

```mermaid
flowchart LR
    subgraph SpeedCost["Speed & Cost Efficiency"]
        direction LR
        Prog["Programmatic /<br/>Deterministic"] --> Judge["LLM-as-a-Judge"] --> Human["Human Evaluation"]
    end
```

Moving left to right: speed decreases, cost increases, and — generally — the ability to capture subjective nuance increases.

---

## 6. When to Use Each Method

### Intuition

The real skill in LLM evaluation isn't picking a single "best" method — it's knowing how to **combine** methods based on what property you're measuring, and how much scale and rigor the situation demands.

### Decision Framework

```mermaid
flowchart TD
    Q1{"Is there an objective,<br/>checkable correctness criterion?"}
    Q1 -->|Yes| Q2{"Can it be expressed<br/>as code/logic?"}
    Q2 -->|Yes| Prog["Use Programmatic /<br/>Deterministic Evaluation"]
    Q2 -->|No| Judge1["Consider LLM-as-a-Judge<br/>with a strict rubric"]

    Q1 -->|No, it's subjective/open-ended| Q3{"Is this high-stakes,<br/>foundational, or novel?"}
    Q3 -->|Yes| Human["Use Human Evaluation<br/>(at least to calibrate)"]
    Q3 -->|No, needs continuous/large-scale coverage| Judge2["Use LLM-as-a-Judge,<br/>validated periodically against humans"]
```

### Practical Guidance

| Situation | Recommended Method |
|---|---|
| Checking output format, structure, or length | Programmatic / Deterministic |
| Verifying factual correctness against a known answer | Deterministic (reference-based) |
| Evaluating code correctness | Deterministic (test-based) |
| Assessing tone, empathy, or persuasiveness | Human, or LLM-as-a-judge validated against human ratings |
| Running evaluation continuously on every model/prompt change | LLM-as-a-judge, backed by periodic programmatic checks |
| Making a foundational decision (e.g., which model to adopt company-wide) | Human evaluation, supplemented by LLM-as-a-judge and programmatic metrics |
| High-volume production monitoring | Programmatic checks + LLM-as-a-judge, with human spot-checks |

### Real-World Analogy

Think of these four methods like a **hospital's diagnostic toolkit**. You don't run an MRI (expensive, slow, human-intensive — like human evaluation) for every single patient. You start with quick, cheap vital signs checks (programmatic/deterministic), escalate to more sophisticated imaging or specialist judgment (LLM-as-a-judge) when needed, and reserve the most expensive, highest-fidelity diagnostic tools (human evaluation) for the cases that truly demand it.

### Common Mistakes

- Using only one method across an entire evaluation strategy, rather than combining methods based on what's being measured.
- Skipping human evaluation entirely because LLM-as-a-judge is "good enough," without ever validating that assumption.
- Over-investing in expensive human evaluation for properties that could be captured reliably and cheaply with programmatic checks.

### Interview Questions

- How would you design an evaluation strategy that combines multiple methods for a single LLM application?
- If you had to choose only one evaluation method for a resource-constrained team, how would you decide?
- How would you validate that your LLM-as-a-judge scores are trustworthy over time?

### Key Takeaways

- No single evaluation method is sufficient on its own — mature eval strategies combine programmatic, deterministic, human, and LLM-as-a-judge methods.
- The right method depends on whether correctness is objective or subjective, and how much scale/speed the situation demands.
- LLM-as-a-judge should be periodically validated against human judgment to ensure it remains trustworthy.

---

## 📌 Module 3 Summary

```mermaid
mindmap
  root((Evaluation Methods))
    Programmatic
      Code-based checks
      Fast, cheap, scalable
    Deterministic
      Same input, same score
      Objective correctness
    Human
      Highest nuance
      Slow and expensive
    LLM-as-a-Judge
      Model judges model
      Scalable nuance
    Choosing a Method
      Match method to property being measured
      Combine methods for full coverage
```

You now understand the full toolkit for *scoring* LLM outputs — from cheap, deterministic checks to nuanced human and LLM-based judgment — and how to combine them intelligently based on the task.

Module 4 shifts focus to the **lifecycle** of evaluation: the difference between offline and online evaluation, how evaluation pipelines fit into the development process, and how continuous monitoring and self-improving loops keep systems reliable after launch.

---


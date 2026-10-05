# SYSTEM ONE DECISION MODELS

## Section 1 — System One Fundamentals

1. OVERVIEW

System One is a model paradigm designed for decision-making workloads inside software systems. Its primary purpose is not open-ended text generation, but converting input context into structured, typed decisions that downstream software can directly consume.

A conventional generative LLM is fundamentally optimized around autoregressive token generation:

Input text
   ↓
Transformer
   ↓
Predict next token
   ↓
Predict next token
   ↓
Generated text

A System One model instead follows the conceptual pattern:

Input state + typed decision question
                ↓
        System One model
                ↓
       Typed decision

The distinction is important because many software tasks are naturally decision problems: classification, routing, thresholding, ranking, approval decisions, escalation, and binary checks.

2. SYSTEM ONE IS A MODEL PARADIGM, NOT ONE ARCHITECTURE

"System One" should not be interpreted as the name of one fixed neural-network architecture.

It describes a decision-oriented model paradigm and interface. Different System One models can use different underlying architectures.

Conceptually:

                 SYSTEM ONE
                     │
        ┌────────────┼────────────┐
        ↓            ↓            ↓
       Jev          Laya         Kev
        │            │            │
   Proprietary    Encoder      Decoder-based
   architecture   architecture LLM architecture

Therefore, the neural architecture of Jev, Laya, and Kev needs to be examined separately even though they belong to the same broader System One category.

3. THE THREE CORE DECISION PRIMITIVES

The main System One output primitives are:

• Choice
• Score
• Noul

They correspond to three common forms of software decision-making.

Choice asks:
"Which option?"

Score asks:
"How much / how severe / where on an ordered scale?"

Noul asks:
"Is this true?"

4. CHOICE

Choice is a categorical decision.

A set of valid alternatives is defined, and the model selects one of them.

Example:

Question:
Which department should handle this request?

Allowed options:
- Billing
- Technical Support
- Account
- Sales
- Other

The model may identify:

Account

A key feature is that the decision can be represented with probabilities over the available choices.

Example:

Billing             0.01
Technical Support   0.08
Account             0.89
Sales               0.01
Other               0.01

The selected class is Account because it has the highest probability.

The probability distribution is valuable because an application can distinguish between a highly certain classification and an ambiguous one.

For example:

Account = 0.96
→ strong classification

Account = 0.42
→ uncertain classification

This makes Choice useful for classification, routing, categorization, and other multi-class decisions.

The documented Choice primitive supports up to 255 options.

5. SCORE

Score is intended for ordered outcomes.

Unlike Choice, where the options may simply represent different categories, Score assumes that the possible outcomes have an ordering.

Example:

Question:
How severe is the issue?

Level 0 = Minor
Level 1 = Moderate
Level 2 = Severe

Suppose the model produces:

P(0) = 0.05
P(1) = 0.70
P(2) = 0.25

A representative numerical score can be obtained from the distribution:

Score =
0 × 0.05
+ 1 × 0.70
+ 2 × 0.25
= 1.20

This preserves more information than simply returning "Moderate."

The result indicates that the probability mass is concentrated around level 1, with some probability of level 2.

Score is therefore appropriate for quantities such as:

• Severity
• Urgency
• Relevance
• Quality
• Risk
• Priority
• Other ordered attributes

The documented Score primitive supports 2 to 10 ordered levels.

6. NOUL

Noul is the binary decision primitive.

It represents the probability of a YES/NO outcome.

Example:

Question:
Is this request restricted?

Possible output:

Noul = 0.97

This can be interpreted as approximately 97% probability of YES.

Other examples:

0.95 → strong evidence for YES
0.75 → moderate evidence for YES
0.50 → highly uncertain
0.20 → evidence toward NO
0.05 → strong evidence for NO

Noul is especially useful when a software system needs to apply a threshold.

For example:

P(YES) ≥ 0.95
→ take an automatic action

0.60 ≤ P(YES) < 0.95
→ apply additional verification

P(YES) < 0.60
→ treat as NO or send to a fallback path

The exact thresholds are application-specific.

7. COMPARISON OF THE THREE TYPES

Choice:
• Categorical
• Select one option from a defined set
• Example: "Which department?"
• Output: selected option + probability distribution

Score:
• Ordered
• Determine a position on an ordered scale
• Example: "How severe?"
• Output: score + probability distribution

Noul:
• Binary
• Determine probability of YES
• Example: "Is this restricted?"
• Output: probability between 0 and 1

A useful mental model is:

Choice → Which one?
Score  → How much?
Noul   → Is it true?

8. MULTIPLE QUESTIONS ON THE SAME STATE

A System One request can contain multiple decision questions evaluated against the same underlying state.

Conceptually:

                    INPUT STATE
                         │
          ┌──────────────┼──────────────┐
          ↓              ↓              ↓
       Choice           Noul           Score
          ↓              ↓              ↓
       Category       Binary test     Ordered result

For example, the same user request could be evaluated for:

• Category
• Whether it satisfies a condition
• Severity
• Priority

This is different from treating every decision as a separate text-generation problem.

9. PARALLEL DECISION EVALUATION

Multiple independent decision questions can be evaluated against the same input state. This allows a system to obtain several related decisions without treating each one as a separate sequential text-generation task.

For example, the same input can be evaluated with:

Question 1:
Is the request restricted?
→ Noul

Question 2:
Is the request deterministic?
→ Noul

Question 3:
What category does the request belong to?
→ Choice

Question 4:
How urgent is the request?
→ Score

Conceptually:

                    SAME INPUT STATE
                           │
          ┌────────────────┼────────────────┐
          ↓                ↓                ↓
        Noul             Choice            Score
          ↓                ↓                ↓
     Binary test        Category         Urgency

The ability to evaluate multiple questions in parallel is useful for decision-oriented software because several independent properties of the same input can be obtained together. The resulting decisions can then be combined by application logic to determine the next processing path.

This is different from a conventional sequential generation workflow in which one model interaction generates text, that text is interpreted, and another model interaction is made for another decision.

10. TYPED DECISIONS

One of the central ideas in System One is the concept of typed outputs.

Consider a conventional generative LLM.

A prompt might say:

"Classify this request as A, B, or C and return JSON."

The model is still fundamentally generating tokens. The desired structure is imposed through instructions and output formatting.

System One instead makes the expected decision type explicit.

Conceptually:

Question type = Choice
Allowed values = A, B, C

                ↓

Model

                ↓

A / B / C

Similarly:

Question type = Noul
→ probability of YES

Question type = Score
→ ordered result

This makes the output more naturally suited to software consumption.

The application can reason about the result as a decision object rather than parsing free-form generated prose.

10. PROBABILITIES AND CALIBRATION

A major benefit of decision-oriented outputs is the ability to work with probabilities.

Consider a binary classifier that produces:

P(YES) = 0.90

A calibrated model should ideally have a meaningful relationship between this probability and observed frequency.

For example, among many predictions assigned approximately 90% probability, the positive outcome should occur roughly 90% of the time, assuming the evaluation population is representative.

This property is called calibration.

Calibration is particularly important in software because the application can make decisions based on thresholds.

For example:

             Probability
                  │
0.0 ──────────────┼──────────────── 1.0
                  │
       low        │       high
                  │
      fallback    │   automatic action

Instead of treating every model prediction as equally reliable, an application can establish policies such as:

High probability
→ automatic decision

Intermediate probability
→ verification or secondary model

Low probability
→ alternative path

Calibration does not mean that the model is always correct. A probability is still a model estimate. The appropriate thresholds must be determined through evaluation on the target application's data.

11. WHY THIS DIFFERS FROM ORDINARY LLM CLASSIFICATION

A normal LLM can certainly perform classification.

For example:

Input:
"Is this request restricted?"

Prompt:
"Answer with RESTRICTED or RELEVANT."

Output:
"RESTRICTED"

However, the underlying mechanism remains text generation.

A System One decision model is designed around the decision itself.

Conceptually:

Traditional LLM:

Question
 ↓
Generate tokens
 ↓
"RESTRICTED"

System One:

Typed binary question
 ↓
Decision model
 ↓
P(RESTRICTED) = 0.97

The difference is therefore not that conventional LLMs cannot classify. They can.

The distinction is that System One makes structured decision-making, typed outputs, and usable probabilities central to the model interface.

12. SYSTEM ONE AS A SOFTWARE DECISION LAYER

System One models are particularly suited to situations where model output controls what software should do next.

A general pattern is:

                Application
                    │
                    ↓
             Decision Model
                    │
             Structured result
                    │
                    ↓
              Application logic
                    │
       ┌────────────┼────────────┐
       ↓            ↓            ↓
    Path A        Path B       Path C

The model makes the decision; application code determines the resulting action.

This separation is important.

For example, the model may determine:

Choice = "restricted"
Probability = 0.97

The application then decides what "restricted" means operationally, such as blocking, escalating, logging, or sending the request to another component.

The model itself does not have to execute those downstream actions.

13. SYSTEM ONE VS GENERATIVE AI

The distinction can be summarized as:

Generative AI:

Input
 ↓
Language model
 ↓
Generated language

System One:

State + typed question
 ↓
Decision model
 ↓
Structured decision + probability

This does not make System One a replacement for generative AI.

Instead, it is useful for a different class of workloads where the desired result is a decision rather than an explanation or generated document.

14. SYSTEM ONE AND THE MODEL FAMILY

The System One paradigm can contain models with different architectures.

For example:

System One
├── Jev
├── Laya
└── Kev

These models can differ substantially in:

• Base architecture
• Parameter count
• Training method
• Inference requirements
• Open/closed availability
• Latency
• Hardware requirements
• Fine-tuning approach

What connects them is the broader decision-oriented System One design rather than one identical Transformer architecture.

15. KEY TAKEAWAY

System One models are designed to turn context into structured, probabilistic decisions.

The three central primitives are:

CHOICE
Select one option from a predefined set.

SCORE
Determine an ordered value or position.

NOUL
Return the probability of a binary YES/NO outcome.

The fundamental conceptual difference is:

Traditional generative LLM:
"Generate text that represents the answer."

System One:
"Produce a typed decision that software can directly use."

This decision-oriented design makes System One particularly relevant to classification, routing, thresholding, ranking, escalation, filtering, and other software workflows where the output is an action-driving decision rather than free-form language.

---

## Section 2 — Laya: Architecture and Working

Laya — Architecture and Working
================================

1. Overview
-----------
Laya is a non-autoregressive decision model designed for System One-style typed decisions. Unlike a conventional generative LLM, Laya does not generate a textual answer token by token. It takes a state/context and typed questions and produces structured decisions with probability distributions.

Primary decision types:
- Choice — select one option from candidates.
- Score — estimate an ordered/ordinal score.
- Noul — produce the probability of a binary condition being true.

The current Laya architecture uses a ModernBERT-large encoder as its backbone, followed by a specialized decision head.


2. High-Level Architecture
--------------------------
Input
  |
  +-- State
  +-- Question
  +-- Options
  |
  v
Tokenization
  |
  v
ModernBERT Encoder
  |
  v
Contextual representations
  |
  v
2-Layer Decision Transformer
  |
  +---------------------------+
  |                           |
  v                           v
Option-marker states       Pooled decision
  |                       representation
  v                           |
Option Scorer MLP              v
  |                       Act/Escalate Head
  v                           |
Logits                         v
  |                       Action logits
  v                           |
Temperature + Softmax          v
  |                       Softmax
  v                           |
Probabilities                  v
  |                       Act probability
  +-------------+-------------+
                |
        +-------+-------+
        |       |       |
      Choice  Score    Noul


3. Input Structure
------------------
Laya operates on a state + typed question representation. The state can contain natural language or structured information such as text, emails, tickets, or JSON.

Conceptual Choice input:

[CLS]
choice question: What type of issue is this?
[SEP]
[MASK] Billing
[MASK] Technical
[MASK] Account
[MASK] Other
[SEP]
User is unable to access their account after changing their password.
[SEP]

Each possible option gets its own [MASK] marker.

Conceptually:
Question + Question Type + Options + State

The exact serialization/tokenization can vary between implementations.


4. ModernBERT Encoder
---------------------
The first major component is the ModernBERT-large encoder.

ModernBERT is an encoder-only Transformer with bidirectional attention. The representation of an option marker is influenced by the question, option, other options, and state/context.

For example, the representation of [MASK] Technical is not simply an embedding of "Technical"; it is a contextual representation of the question, option, and state.


5. Decision Transformer
-----------------------
After ModernBERT, Laya uses a specialized two-layer decision Transformer to further process the encoder representations for decision-making.

ModernBERT:
  general language/context representation
        |
        v
Decision Transformer:
  decision-oriented representation

The decision head also incorporates question-type information such as Choice, Score, or Noul.


6. Option Marker Representations
--------------------------------
After the decision Transformer, Laya extracts the hidden representation corresponding to each option's [MASK] marker.

For example:

[MASK] Billing    -> h1
[MASK] Technical  -> h2
[MASK] Account    -> h3
[MASK] Other      -> h4

Each hi is a high-dimensional vector that is passed to the option-marker scorer.


7. Option Scorer
----------------
The option scorer is a small trainable neural network (MLP):

Hidden representation
        |
     LayerNorm
        |
      Linear
        |
       GELU
        |
      Linear
        |
   Scalar Logit

Mathematically:

hi -> LayerNorm -> Linear -> GELU -> Linear -> zi

where hi is the contextual representation of option i and zi is its raw logit.

Example:
Billing      -> 1.2
Technical    -> 4.5
Account      -> 0.8
Other        -> 0.4

These are raw logits, not probabilities.


8. Dynamic Options
------------------
The options are supplied as part of the input. Laya does not require a fixed output layer containing a predefined label set.

The same scorer is reused for every option:

Billing marker    -> shared scorer -> z1
Technical marker  -> shared scorer -> z2
Account marker    -> shared scorer -> z3
Other marker      -> shared scorer -> z4

Therefore, the option set can change without changing the scorer's output layer.


9. From Logits to Probabilities
-------------------------------
After scoring:

z1, z2, ..., zn

Laya applies temperature-scaled softmax to obtain the probability distribution.

Technical note: Laya's implementation/documentation describes the use of temperature scaling and softmax for calibrated probabilities. The following equation is a mathematical representation of that operation included to explain the underlying process; it is not presented as an explicit equation in the Laya documentation.

                 exp(zi / T)
pi = --------------------------------
       sum_j exp(zj / T)

T is a fitted temperature used for calibration.

Example:

Raw logits:
Billing      1.2
Technical    4.5
Account      0.8
Other        0.4

Possible calibrated probabilities:
Billing      0.03
Technical    0.88
Account      0.06
Other        0.03


10. Choice Decision
-------------------
For Choice, the model selects the option with the highest probability:

Choice = argmax_i(pi)

Example:
Billing      0.05
Technical    0.82
Account      0.08
Other        0.05

Output:
Answer = Technical

The full probability distribution can also be retained.


11. Score Decision
------------------
Score represents an ordered scale.

Example:
1 = Low
2 = Medium
3 = High
4 = Critical

After scoring and softmax:

Low       -> 0.05
Medium    -> 0.15
High      -> 0.65
Critical  -> 0.15

Laya's documentation describes the Score output as the expected level on an ordinal rubric, together with the probability distribution.

Technical note: The following equation is a mathematical representation of this expected-value calculation, included to explain the documented ordinal-score behavior; it is not presented as an explicit equation in the Laya documentation.

E[S] = sum_i (pi * si)

E[S] =
(0.05)(1) + (0.15)(2) + (0.65)(3) + (0.15)(4)
= 2.90

Thus the result can be represented as an expected ordinal score of about 2.9/4 while retaining the full distribution.

The ordering matters:
Low < Medium < High < Critical


12. Noul Decision
-----------------
Noul is a binary decision.

Example:
Question: Is this request urgent?
Options: YES / NO

Scorer output:
YES -> 2.8
NO  -> 0.7

After softmax:
YES -> 0.89
NO  -> 0.11

Therefore:
P(True) = 0.89
P(False) = 0.11

The same option-marker scoring mechanism can therefore handle a binary decision.


13. Choice vs Score vs Noul
---------------------------
Choice:
  Multiple arbitrary options
  Output: selected option
  Interpretation: select highest probability

Score:
  Ordered levels
  Output: expected/ordinal score
  Interpretation: use the probability distribution and ordering

Noul:
  Binary
  Output: probability of True
  Interpretation: P(YES/True)

Common pipeline:
Input
 -> ModernBERT
 -> Decision Transformer
 -> Option-marker representations
 -> Shared scorer
 -> Logits
 -> Temperature-scaled Softmax
 -> Probability distribution
 -> Typed interpretation


14. Multiple Questions
----------------------
Laya can evaluate multiple typed questions against the same state in a single forward pass.

Example state:
Customer cannot access account after changing password.

Q1: What type of issue is this?
    Choice: Billing / Technical / Account / Other

Q2: Is this urgent?
    Noul: Yes / No

Q3: How severe is the issue?
    Score: Low / Medium / High / Critical

Conceptually:

                 Same State
                     |
        +------------+------------+
        v            v            v
       Q1           Q2           Q3
     Choice         Noul         Score
        |            |            |
        +------------+------------+
                     |
              Single forward pass
                     |
                     v
               Multiple outputs


15. Act/Escalate Head
---------------------
In addition to the option-scoring branch, Laya contains a separate Act/Escalate head.

Its purpose is to produce an action-related probability in addition to the main typed decision.

Conceptually:

Decision representation
        +
Additional features
        |
        v
Act/Escalate MLP
        |
        v
  +----------------+
  | Action logits  |
  | ACT / ESCALATE |
  +-------+--------+
          |
          v
       Softmax
          |
          v
   act_probability

For example, conceptually:

ACT       -> 0.85
ESCALATE  -> 0.15

corresponds to:
act_probability = 0.85

This is different from Choice/Score/Noul confidence.

For example:
Decision:
    Intent = Refund
    P(Refund) = 0.95

Action:
    P(ACT) = 0.70

The model can therefore be confident about what the request is while being less confident about whether the action should be taken automatically.

Important limitation:
The current published Laya checkpoint has a known issue with this branch: evaluations have reported that act_probability can become effectively saturated near 1.0 across very different inputs. Therefore, the current checkpoint's act_probability should not be treated as a reliable production-quality Act/Escalate signal without further validation or modification.

The exact semantics of the additional features supplied to this head are not fully documented publicly and should not be assumed.


16. Complete End-to-End Flow
----------------------------
INPUT
  |
  v
State + Typed Question + Options
  |
  v
Tokenization
  |
  v
ModernBERT Encoder
  |
  v
Contextual embeddings
  |
  v
2-Layer Decision Transformer
  |
  +--------------------+
  |                    |
  v                    v
Option markers      Pooled decision
  |                  representation
  v                    |
Scorer MLP              v
  |                Act/Escalate
  v                    |
Logits                  v
  |                Action probability
  v
Temperature
  |
  v
Softmax
  |
  v
Probabilities
  |
  +----------+----------+
  |          |          |
  v          v          v
Choice     Score       Noul
  |          |          |
  v          v          v
Option    Expected    P(True)
           Score


17. Training
------------
Laya's backbone and decision head are trained for decision-making rather than text generation.

The current Laya model uses RLCD (Reinforcement Learning for Calibrated Decisions), which focuses on producing useful probability distributions rather than only maximizing the correct class. Proper scoring rules and calibration techniques are used.

The current architecture uses a fully fine-tuned ModernBERT-large backbone, while the decision head is trained for these typed decisions.


18. Key Architectural Characteristics
-------------------------------------
- Encoder-based: uses an encoder architecture rather than an autoregressive decoder-only LLM.
- Non-autoregressive: does not generate a response token by token.
- Typed decisions: explicitly supports Choice, Score, and Noul.
- Dynamic option scoring: options are represented in the input and scored dynamically.
- Probability-oriented: produces probability distributions rather than only hard predictions.
- Shared scorer: the same option-scoring network can be applied to every option marker.
- Parallel decisions: multiple questions can be evaluated against the same state in one forward pass.
- Specialized decision head: converts contextual language representations into structured decisions.


19. Core Concept
---------------
The key idea is:

Language Understanding
        |
        v
Decision Representation
        |
        v
Option Scoring
        |
        v
Probability Distribution
        |
        v
Typed Decision

More concretely:

ModernBERT
     |
     v
Understand state/question/options
     |
     v
Decision Transformer
     |
     v
Decision-specific representations
     |
     v
Scorer MLP
     |
     v
Raw logit for every option
     |
     v
Softmax + calibration
     |
     v
Probability distribution
     |
     +-----------+-----------+
     |           |           |
     v           v           v
   Choice      Score        Noul
   argmax      expected     P(True)

In short, Laya is not primarily trying to generate an answer. It transforms contextual language representations into calibrated, typed decisions.

---

## Section 3 — Kev: Architecture, Working, Training, and Inference

KEV DECISION MODEL
Architecture, Working, Training, and Inference
Source-Verified Technical Report
Updated: October 2026

1. OVERVIEW

Kev is a family of decision models designed to answer typed decision questions over a shared state. Instead of generating a natural-language answer token by token, Kev takes State + Typed Questions + Options and returns a probability distribution over the supplied options.

Supported decision types:
- Choice: select among named options.
- Score: probability distribution over ordered levels plus an expected level.
- Noul: binary yes/no probability.

Kev is a causal-language-model-based decision system. The language-model backbone is run in prefill-oriented operation, and a pointer head converts hidden representations into probabilities over the supplied options.

2. VERSION DISTINCTION

Kev is a family, not one fixed configuration.

Current family:
- Kev-0.8B: Qwen3.5-0.8B-Base + rank-16 LoRA + pointer head.
- Kev-4B: Qwen3.5-4B-Base + rank-16 LoRA + pointer head.
- Kev-9B: Qwen3.5-9B-Base + rank-16 LoRA + pointer head.
- Kev-27B v2: Qwen3.8-27B + pointer head; every backbone weight is fine-tuned.

Superseded generations:
- Kev-0.5B: Qwen2.5-0.5B + LoRA + pointer head.
- Earlier Qwen3-based Kev checkpoints.

Therefore, statements such as “the backbone is frozen” or “the backbone is fully fine-tuned” must be tied to a specific checkpoint.

3. HIGH-LEVEL ARCHITECTURE

State + Question + Options
        |
        v
Qwen causal LM backbone
        |
        v
Contextual hidden states
        |
        +---------------------------+
        |                           |
        v                           v
<decide> hidden state          </opt> hidden states
        |                           |
        v                           v
Query projection               Key projections
        |                           |
        +-------------+-------------+
                      |
                      v
             Query-Key compatibility
                      |
                      v
                 Raw logits
                      |
                      v
              Temperature scaling
                      |
                      v
                   Softmax
                      |
                      v
             Probability distribution
                      |
              +-------+-------+
              |       |       |
              v       v       v
            Choice  Score    Noul

4. INPUT REPRESENTATION

Kev uses special structural tokens to mark the state, questions, options, and decision point. A simplified representation is:

<state> ...state...
<q> instructions
    <opt> option 1 </opt>
    <opt> option 2 </opt>
    ...
    <decide>

For multiple questions, each question has its own option list and decision marker.

Attention-only Qwen3 versions can pack questions into one sequence with a block-causal mask. Current Qwen3.5/Qwen3.8 versions contain full-attention and Gated DeltaNet layers; because DeltaNet layers are recurrent and do not obey attention masks in the same way, current Kev runs each question as its own row: shared state followed by that question. The state can be computed once and its cache reused. Questions are therefore isolated.

5. QWEN BACKBONE

Kev uses a causal language model as its representation backbone.

Kev-9B:
- Qwen3.5-9B-Base
- 32 text layers
- 24 Gated DeltaNet layers
- 8 full-attention layers
- hidden size 4096
- frozen backbone
- rank-16 LoRA adapter

Kev-4B:
- Qwen3.5-4B-Base
- rank-16 LoRA
- pointer head
- frozen base while LoRA/head are trained

Kev-27B v2:
- Qwen3.8-27B
- 64 layers
- 48 Gated DeltaNet layers
- 16 full-attention layers
- hidden size 5120
- every backbone weight fine-tuned

Thus “Kev = Qwen + LoRA” is correct for the current 0.8B/4B/9B family, but not for Kev-27B v2.

6. WHY A CAUSAL LM IS USED

A normal causal LM predicts the next token. Kev instead uses the causal LM mainly as a contextual representation engine.

The model processes the state, question instructions, and options and produces hidden representations. The <decide> representation summarizes the decision context, while each option’s closing </opt> representation summarizes that option in context.

The pointer head performs the final decision readout.

7. POINTER HEAD

For one question:

h_decide = hidden state at <decide>

h_opt_i = hidden state at the closing </opt> token for option i

Two learned projections create a comparison space:

q = W_Q h_decide
k_i = W_K h_opt_i

For the original Kev-0.5B implementation, the Qwen hidden size was 896 and both projections were 896 -> 256. Current models retain the same conceptual design with dimensions appropriate to their backbone.

8. QUERY AND KEY INTUITION

The <decide> representation becomes the Query: “What kind of answer am I looking for?”

Each option representation becomes a Key: “What does this option represent in this context?”

The pointer head compares the Query with every option Key and produces one compatibility score per option.

This is why the head can operate on options supplied dynamically at inference time.

9. SCALED DOT-PRODUCT

The pointer operation is conceptually:

score_i = (q · k_i) / sqrt(d)

where q is the projected <decide> representation, k_i is the projected </opt> representation, and d is the pointer-head dimension.

For the original Kev-0.5B, d = 256.

The scaling keeps dot-product magnitudes in a suitable range before softmax.

10. SOFTMAX AND CALIBRATION

The scores are converted into probabilities:

P(i) = exp(score_i / T) / sum_j exp(score_j / T)

where T is the checkpoint’s fitted temperature.

Example:
Billing      0.70
Technical    0.20
Account      0.10

The model can therefore provide both the most likely option and a probability distribution.

Temperature calibration is important because Kev is intended to provide probabilities that can be thresholded, not just rankings.

11. WHY “POINTER”

A pointer head effectively points from the decision representation toward one of the supplied option representations.

It does not require a fixed classifier such as Linear(hidden_size, 37), where each output neuron permanently represents one class.

If 3 options are supplied, 3 option scores are produced. If 20 options are supplied, 20 scores are produced.

The output space is therefore determined by the input options.

12. CHOICE

For Choice, the pointer head calculates one score for every named option, softmax converts the scores into probabilities, and the selected option is:

argmax P(option)

Current model cards specify up to 255 named options.

13. NOUL

Noul is the binary yes/no decision type. The key output is the probability of YES:

P(YES)

For example, P(YES) = 0.83.

This is useful when an application needs a thresholdable decision probability rather than generated text.

14. SCORE

Score uses an ordered set of levels, for example:

1 = Very Low
2 = Low
3 = Medium
4 = High
5 = Very High

The model returns a probability distribution over the levels. An expected level can be represented as:

E[level] = sum_i i * P(i)

This is an explanatory mathematical formulation of the documented expected-level behavior, not a claim that the exact equation is written verbatim in every model card.

Current Kev model cards support up to 255 ordered score levels.

15. MULTIPLE QUESTIONS

Kev can answer multiple typed questions over the same state.

Example:
State = customer complaint

Q1: Which product? Credit Card / Loan / Savings
Q2: Is it urgent? Yes / No
Q3: Severity? 1 / 2 / 3 / 4 / 5

Questions are intended to be independent: Q1 cannot use Q2’s answer, and Q2 cannot use Q3’s answer.

Conceptually:

Shared State
   +--> Q1 -> Distribution 1
   +--> Q2 -> Distribution 2
   +--> Q3 -> Distribution 3

Current Qwen3.5/Qwen3.8 models implement this using separate question rows with a shared state representation/cache.

16. PREFILL-ONLY OPERATION

Normal generative LLM:
prompt -> next-token probability -> generate token -> repeat

Kev:
state + question + options -> Qwen prefill -> hidden states -> pointer head -> probabilities

No natural-language answer needs to be generated token by token.

17. TRAINING

Kev is trained as a supervised decision model using cross-entropy/log loss against the correct option.

Training example:
State + Question + Options -> target option

The objective increases probability for the correct option and decreases probability for alternatives.

The learned system therefore combines contextual representations with a trained pointer readout.

18. LORA TRAINING

For current Kev-0.8B, 4B, and 9B, the Qwen base is frozen and a rank-16 LoRA adapter is trained together with the pointer head.

Conceptually:

Frozen Qwen backbone
        +
Trainable LoRA adapters
        +
Trainable pointer head

The original Kev-0.5B used the same broad approach.

Kev-27B v2 is different: every Qwen3.8-27B backbone weight is fine-tuned, together with the pointer head.

19. ORIGINAL KEV-0.5B CONFIGURATION

The original prototype is useful for understanding the concrete implementation:

Backbone:
Qwen/Qwen2.5-0.5B, 494M parameters, frozen

LoRA:
rank 16
alpha 32
dropout 0.05
targets: q_proj, k_proj, v_proj, o_proj, gate_proj, up_proj, down_proj

Pointer head:
Query projection: 896 -> 256
Key projection: 896 -> 256
scaled dot product
softmax over options

Trainable parameters: approximately 9.3M (about 8.8M LoRA + 0.46M head).

This is the original prototype and is NOT the universal configuration of current Kev.

20. CURRENT KEV-9B EXAMPLE

Kev-9B uses Qwen3.5-9B-Base.

Architecture:
32 text layers
24 Gated DeltaNet
8 full attention
hidden size 4096

Training:
frozen backbone
rank-16 LoRA
pointer head

The pointer head maps the 4096-dimensional decision/option representations into a 256-dimensional comparison space, computes compatibility, and converts the scores to calibrated probabilities.

21. CURRENT KEV-27B EXAMPLE

Kev-27B v2 uses Qwen3.8-27B.

Architecture:
64 layers
48 Gated DeltaNet
16 full attention
hidden size 5120

Unlike the 0.8B/4B/9B family, every backbone weight is fine-tuned.

Pointer head:
5120 -> 256 Query projection
5120 -> 256 Key projection

Then:
Query-Key score -> temperature scaling -> softmax -> probabilities.

22. CALIBRATION

Calibration means probabilities should have statistical meaning.

If many cases receive P(correct) approximately 0.80, a well-calibrated model should be correct roughly 80% of the time on comparable data.

Calibration does not mean P=0.80 guarantees correctness.

Kev checkpoints ship with a fitted temperature. Temperature scaling changes probability sharpness while preserving the basic ranking.

23. DECISION MODEL VS GENERATIVE LLM

Generative LLM:
Input -> Transformer -> next-token probabilities -> generated text

Kev:
State + typed question + options -> Qwen representation -> pointer head -> option probabilities

Example generated-LLM output:
“The issue appears to be related to billing.”

Kev output:
Billing 0.82
Technical 0.12
Account 0.06

The structured output is directly useful for routing, classification, triage, thresholding, human review, and analytics.

24. DYNAMIC OPTIONS

Traditional classifier:
Linear(hidden, N)
where N is fixed.

Kev:
supplied options -> contextual option representations -> pointer comparisons -> probability distribution

The decision space is therefore dynamic and determined by the options supplied at inference time.

25. WHAT POINTER DOES NOT MEAN

The pointer head is not simple string matching or copying.

For:
<opt> Billing </opt>

the </opt> hidden state is contextual. It is influenced by the state, question, option, and surrounding context.

The pointer mechanism compares contextual representations, so the decision is learned semantic compatibility rather than merely matching the option text.

26. END-TO-END EXAMPLE

State:
“Customer reports that a payment was charged twice.”

Question:
“What category best describes the issue?”

Options:
Billing
Technical
Account

Processing:
1. Tokenize the state/question/options.
2. Run the Qwen backbone.
3. Extract h_decide.
4. Extract each closing </opt> representation.
5. Project h_decide to Query.
6. Project each option representation to a Key.
7. Calculate Query-Key compatibility.
8. Apply temperature.
9. Apply softmax.
10. Return the distribution.

Example:
Billing      0.91
Technical    0.05
Account      0.04

Choice = Billing.

No explanatory sentence is required.

27. COMPLETE DATA FLOW

State / Document
      |
      v
Typed Question + Named Options
      |
      v
Qwen Causal LM
      |
      v
Contextual Hidden States
      |
      +-------------------+
      |                   |
      v                   v
<decide> state       </opt> states
      |                   |
      v                   v
W_Q projection       W_K projections
      |                   |
      v                   v
Query                Keys
      |                   |
      +---------+---------+
                v
       Scaled Dot Products
                |
                v
            Raw Logits
                |
                v
      Temperature Scaling
                |
                v
             Softmax
                |
                v
     Probability Distribution
                |
        +-------+-------+
        |       |       |
      Choice  Score    Noul

28. SOURCE-VERIFIED VS EXPLANATORY

Directly supported by published Kev documentation/model cards:
- Kev is a decision model rather than a text-generation system.
- It uses a causal LM backbone in prefill-oriented operation.
- It uses a pointer head over supplied options.
- The pointer head scores each option’s closing </opt> representation against the question’s <decide>/final representation.
- Two learned projections are used for the pointer comparison.
- Softmax converts option scores into probabilities.
- Kev supports Choice, Score, and Noul.
- Multiple questions can share state while remaining independent.
- Current Qwen3.5/Qwen3.8 versions run questions as separate rows and reuse the state cache.
- Current 0.8B/4B/9B models use Qwen3.5 Base + rank-16 LoRA + pointer head.
- Kev-27B v2 uses Qwen3.8-27B with full backbone fine-tuning.
- Checkpoints use fitted temperatures.

Explanatory mathematical formulations:
score_i = q · k_i / sqrt(d)
P(i) = softmax(score_i / T)
E[level] = sum_i i P(i)

These equations explain the documented mechanics; they should not be presented as verbatim quotations from the documentation.

29. KEY TAKEAWAYS

1. Kev is a decision model built on a causal language model.
2. Qwen acts mainly as the contextual representation engine.
3. The pointer head maps those representations to probabilities over supplied options.
4. <decide> becomes the Query representation.
5. Each closing </opt> representation becomes a Key.
6. Query-Key compatibility gives one score per option.
7. Softmax gives the probability distribution.
8. Choice selects the highest-probability option.
9. Noul gives a yes/no probability.
10. Score gives probabilities over ordered levels and an expected level.
11. Multiple questions can share the same state while remaining isolated.
12. Current Qwen3.5/Qwen3.8 Kev uses hybrid attention/DeltaNet backbones, so execution differs from the original attention-only implementation.
13. Current 0.8B/4B/9B uses LoRA + pointer-head training on frozen Qwen3.5 Base models.
14. Kev-27B v2 fully fine-tunes its Qwen3.8-27B backbone.
15. Kev-0.5B is a superseded prototype and should not be used as the universal description of current Kev.

30. PRIMARY SOURCES

1. Kev official repository:
https://github.com/jaredpalmer/kev

2. Kev README — architecture and execution:
https://github.com/jaredpalmer/kev/blob/main/README.md

3. Kev-0.5B model card:
https://github.com/jaredpalmer/kev/blob/main/docs/model-cards/kev-0.5b.md

4. Kev-0.8B model card:
https://github.com/jaredpalmer/kev/blob/main/docs/model-cards/kev-0.8b.md

5. Kev-4B model card:
https://github.com/jaredpalmer/kev/blob/main/docs/model-cards/kev-4b.md

6. Kev-9B model card:
https://github.com/jaredpalmer/kev/blob/main/docs/model-cards/kev-9b.md

7. Kev-27B v2 model card:
https://github.com/jaredpalmer/kev/blob/main/docs/model-cards/kev-27b.md

NOTE:
This report is intentionally version-aware because backbone architecture and training strategy differ between Kev checkpoints.

---

## Section 4 — Jev: Architecture, Working, and Training

JEV — SYSTEM ONE MODEL
Architecture, Working, and Training
Concise Source-Verified Report
Updated: October 2026

1. OVERVIEW

Jev is TypeSafe AI's flagship model and the first System One model.

System One models are designed for fast, structured decisions that software can consume directly. Jev evaluates a state against typed questions and returns structured values and probabilities rather than generated text.

2. CORE CONCEPT

    STATE + TYPED QUESTIONS
              |
              v
             JEV
              |
              v
    STRUCTURED DECISIONS + PROBABILITIES

The application defines what it wants Jev to judge. Jev supplies the probabilistic judgment, while normal application code controls the resulting business logic.

3. INPUT: STATE

The state is the information Jev evaluates.

Supported state forms include:
- Text strings
- JSON objects containing text
- Arrays of text

Jev currently does not support images, audio, or video as state inputs.

4. TYPED QUESTIONS

Jev provides three core question primitives:
- Choice — select one option from a predefined set.
- Score — evaluate the state against an ordered scale.
- Noul — determine the probability that a yes/no statement is true.

These question types can be mixed in the same request.

5. PARALLEL QUESTION EVALUATION

Multiple questions can be evaluated against the same state in a single request.

                 STATE
                   |
        +----------+----------+
        |          |          |
       Q1         Q2         Q3
        |          |          |
        v          v          v
      Result     Result     Result

The questions are evaluated independently and in parallel. One question's answer is not passed as context to another.

6. OUTPUT

Jev returns typed results rather than generated prose.

Depending on the primitive:
- Choice: selected option, per-option probabilities, and confidence.
- Score: score, probability distribution over levels, and confidence.
- Noul: probability that the statement is true.

The output can therefore be consumed directly by application code without parsing natural-language model responses.

7. DECISION FLOW

    Application state
          |
          v
    Typed questions
          |
          v
    Jev decision model
          |
          v
    Probability distributions
          |
          v
    Application logic

Jev provides the decision signals; deterministic application code decides what action to take.

8. CALIBRATED PROBABILITIES

Calibration is a central System One objective.

The probabilities are intended to reflect uncertainty across groups of predictions. For example, predictions assigned approximately 0.8 probability should be correct at approximately that rate over a suitable population if the model is well calibrated.

Calibration does not guarantee that an individual prediction is correct.

This makes the probabilities useful for confidence thresholds, routing, escalation, and other decision policies.

Jev is associated with TypeSafe's:

    RLCD — Reinforcement Learning for Calibrated Decisions

The objective differs from conventional text-generation optimization. System One training focuses on producing decisions whose probabilities correspond meaningfully to observed outcomes.

Conceptually:

    State + Question
          |
          v
       Decision
          |
          v
      Probability
          |
          v
    Compare with outcome
          |
          v
    Improve calibration
          |
          v
         RLCD

9. INTERNAL NEURAL ARCHITECTURE

The detailed neural architecture of Jev is not publicly disclosed.

RLCD (Reinforcement Learning for Calibrated Decisions) is TypeSafe's training approach for System One. It focuses on producing decisions whose probabilities are meaningfully aligned with observed outcomes and calibration.

Unlike open models such as Laya and Kev, TypeSafe does not currently publish enough implementation detail to establish:
- Exact backbone architecture
- Transformer layer configuration
- Number of parameters
- Attention mechanism
- Exact decision-head architecture
- Exact option-scoring mechanism
- Model weights

Therefore, it is incorrect to claim that Jev internally uses Qwen, ModernBERT, a specific pointer head, or another architecture without an authoritative source.

The verified architectural description is:

    State + Typed Questions
              |
              v
       Proprietary Jev model
              |
              v
       Typed probabilities

Conventional LLM:

    Prompt -> next-token prediction -> generated response

Jev:

    State + Questions -> Decision -> Typed probabilities

The application does not need to ask an LLM to write a response and then parse it back into a structured decision.

A key System One principle is that the model supplies the judgment while application code owns the policy.

Example:

    Jev:
        refund probability = 0.91

    Application:
        if probability > threshold:
            continue workflow
        else:
            request review

The threshold and business action are controlled by the application.

Jev:
- Proprietary System One model
- Internal architecture unpublished
- RLCD training
- Structured probabilistic decisions

Laya:
- Open-weight decision model
- ModernBERT/mmBERT-based
- Public implementation and decision head

Kev:
- Open decision-model family
- Qwen-based checkpoints
- Public pointer-head implementation for published models

They share the System One decision philosophy, but their neural architectures should not be assumed to be the same.

10. VERIFIED VS UNKNOWN

Verified:
- Jev is TypeSafe's flagship and first System One model.
- It evaluates typed questions against a state.
- It returns structured decisions and probabilities instead of generated text.
- Choice, Score, and Noul are the core primitives.
- Multiple questions can be evaluated independently and in parallel against the same state.
- Calibration is a core design objective.
- RLCD is TypeSafe's calibrated-decision training approach.
- Jev currently accepts text-oriented state rather than image/audio/video input.

Not publicly established:
- Exact neural backbone
- Exact parameter count
- Exact layer architecture
- Exact attention mechanism
- Exact decision head
- Exact pointer/scoring implementation
- Model weights

11. KEY TAKEAWAYS

- Jev is the first and flagship System One model from TypeSafe.
- Its purpose is fast, structured decision-making rather than text generation.
- The application supplies the state and typed questions.
- Jev returns typed decisions and probabilities.
- Multiple independent questions can be evaluated together against the same state.
- Calibration is a core design objective.
- RLCD is TypeSafe's training approach for calibrated decisions.
- Jev's internal neural architecture and weights are proprietary/unpublished.
- Therefore, its architecture should be described at the functional level unless TypeSafe releases further implementation details.
- Laya and Kev should not be assumed to reproduce Jev's internal architecture.

12. PRIMARY SOURCES

TypeSafe System One documentation:
https://github.com/chujianyun/typesafe-wiki/blob/main/references/docs/Concepts/System%20One.md

TypeSafe introduction documentation:
https://github.com/chujianyun/typesafe-wiki/blob/main/references/docs/Getting%20Started/Introduction.md

Jev documentation mirror:
https://github.com/lezgoverci/jev-docs

Technical reference with source tracking:
https://github.com/docxology/daf-jev/blob/main/docs/models.md

NOTE:
Claims about Jev's proprietary internal architecture are intentionally excluded unless supported by published material.

---

## Section 5 — Jev vs Laya vs Kev

SYSTEM ONE MODEL COMPARISON
Jev vs Laya vs Kev
Concise Technical Report
Updated: October 2026

1. HIGH-LEVEL COMPARISON

Jev, Laya, and Kev implement the same System One decision philosophy, but use different architectural and deployment approaches.

| Aspect | Jev | Laya | Kev |
|---|---|---|---|
| Developer | TypeSafe AI | ConvAI Innovations | Jared Palmer |
| Type | Proprietary decision model | Open-weight decision model | Open decision-model family |
| Weights | Closed | Open | Open |
| Deployment | Managed/API | Self-hostable | Self-hostable |
| Core architecture | Proprietary / undisclosed | Encoder-based | Causal decoder LM |
| Backbone | Not publicly disclosed | ModernBERT / mmBERT | Qwen3.5 / Qwen3.8 |
| Decision mechanism | Proprietary | Decision heads | Pointer head |
| Training philosophy | RLCD | Decision-model training | Supervised decision training + calibration |
| Architecture transparency | Low | High | High |

Jev is the only one whose internal neural architecture is not publicly exposed in comparable detail.

2. ARCHITECTURAL PHILOSOPHY

The three models implement the same high-level concept in different ways:

                    SYSTEM ONE
                        |
           +------------+------------+
           |            |            |
           v            v            v
          JEV          LAYA         KEV
           |            |            |
      Proprietary    Encoder      Decoder LM
           |            |            |
           v            v            v
       Decision      Decision      Pointer
        system         head          head
           |            |            |
           +------------+------------+
                        |
                        v
              Probabilistic decisions

The important difference is how each model converts its internal representations into decisions.

3. JEV

Jev is TypeSafe AI's flagship and first System One model.

Its internal neural architecture is proprietary, so its exact backbone and decision head cannot be reliably specified from public documentation.

The verified functional architecture is:

    State + Typed Questions
              |
              v
             Jev
              |
              v
    Typed decisions + probabilities

Key characteristics:
- Managed model/API
- Proprietary weights
- Structured probabilistic output
- Independent questions over shared state
- Calibration as a core objective
- RLCD training approach

4. LAYA

Laya uses an encoder-model approach.

Its published architecture is broadly:

    Input
      |
      v
    ModernBERT / mmBERT
      |
      v
    Decision representation
      |
      v
    Decision heads
      |
      v
    Option scores
      |
      v
    Probabilities

Published Laya checkpoints include ModernBERT-large and mmBERT-based multilingual variants.

Laya is open-weight and designed for self-hosted inference.

5. KEV

Kev uses a causal decoder language model as its representation backbone.

Conceptually:

    Input
      |
      v
    Qwen
      |
      v
    <decide> representation
          +
    </opt> representations
      |
      v
    Query / Key projections
      |
      v
    Pointer head
      |
      v
    Probabilities

Current Kev models use Qwen3.5/Qwen3.8 and span a wide range of model sizes.

The important architectural idea is that the decoder LM is used primarily to produce contextual representations; normal autoregressive text generation is not required for the final decision.

6. BIGGEST ARCHITECTURAL DIFFERENCE

Jev:

    Proprietary architecture
            |
            v
       Decision model

Laya:

    Encoder
       |
       v
    Decision head

Kev:

    Causal decoder LM
            |
            v
       Pointer head

Therefore, Jev, Laya, and Kev should not be treated as different checkpoints of the same neural architecture. They share the System One decision philosophy but use different implementations.

7. OPEN VS CLOSED

Jev:

    TypeSafe infrastructure
           |
           v
          Jev
           |
          API

The model weights are not publicly available.

Laya:

    Open weights
        |
        v
    User infrastructure
        |
        v
      Inference

Kev:

    Open model components
          |
          v
    User infrastructure
          |
          v
        Inference

Laya and Kev therefore provide substantially more control over deployment and model customization than Jev.

8. CUSTOMIZATION

Jev:
- Proprietary model
- Access primarily through the provided System One interface/API
- Model weights are not available for direct modification

Laya:
- Open weights and implementation
- Can be self-hosted and adapted

Kev:
- Open model family and implementation
- Supports custom training/fine-tuning workflows
- Provides multiple backbone/model sizes

Therefore, Laya and Kev offer greater direct control over model customization than Jev.

9. MODEL SIZE PHILOSOPHY

Laya focuses on relatively compact encoder models, including approximately:
- 322M parameters for the multilingual mmBERT-based model
- 421M parameters for the ModernBERT-large model

Kev provides a broader size range, including approximately:
- 0.8B
- 4B
- 9B
- 27B

This gives Kev a wider capacity/latency tradeoff.

Jev's exact parameter count is not publicly disclosed.

10. INFERENCE STRATEGY

Laya uses an encoder-style architecture designed directly for decision inference.

Kev uses the prefill phase of a causal LM and does not perform normal token-by-token generation for the decision output:

    Qwen prefill
         |
         v
    Hidden states
         |
         v
    Pointer head
         |
         v
    Decision

Jev exposes the same fundamental decision-oriented interface, but its proprietary internal inference implementation is not publicly specified in sufficient detail for a deeper architectural comparison.

11. QUESTION PARALLELISM

All three follow the System One principle that multiple questions can be evaluated against the same state.

Conceptually:

                    STATE
                      |
           +----------+----------+
           v          v          v
          Q1         Q2         Q3
           |          |          |
           v          v          v
          P1         P2         P3

The questions are independent decisions over the same state.

Kev explicitly isolates questions and, for its current Qwen3.5/Qwen3.8 models, processes each question as its own row while reusing the state computation/cache.

The System One design similarly treats questions as independent evaluations against a shared state.

12. PRIMARY SOURCES

System One documentation:
https://github.com/chujianyun/typesafe-wiki/blob/main/references/docs/Concepts/System%20One.md

Kev repository:
https://github.com/jaredpalmer/kev

Laya repository:
https://github.com/he-jev/laya

Laya model:
https://huggingface.co/convaiinnovations/laya

NOTE:
Jev's internal neural architecture is intentionally not specified beyond what is publicly documented. No Qwen, ModernBERT, pointer-head, parameter-count, or layer-level architecture is attributed to Jev without authoritative evidence.

---

## Section 6 — Benchmark Comparison

SYSTEM ONE DECISION MODELS
BENCHMARK COMPARISON REPORT

6. BENCHMARK COMPARISON

This section compares Jev, Laya, Laya Typed Decisions, and Kev across standard classification, decision-making, out-of-distribution, and reasoning-oriented benchmarks.

Important checkpoint distinction:
Laya and Laya Typed Decisions are separate checkpoints. The results below use Laya for the general/base model and Laya Typed Decisions only where the specialized checkpoint was actually evaluated.

6.1 BENCHMARK DATASETS

| Benchmark | What it tests | Classes / decision space |
|---|---|---:|
| AG News | News articles classified by topic | 4 |
| DAIR Emotion | Emotion classification from short text | 6 |
| Banking77 | Banking customer queries classified by intent | 77 |
| BoolQ | Answering yes/no questions using a passage | 2 |
| MNLI | Natural-language inference between premise and hypothesis | 3 |
| SST-5 | Sentiment intensity | 5 |
| Yelp Polarity | Positive/negative review classification | 2 |
| Typed Decisions | Structured decisions across multiple workflows | Varies |
| Transfer-v4 | Generalization to new decision tasks/sources | Varies |
| Hard-v1 | More difficult reasoning and decision tasks | Varies |
| Documents-v1 | Decisions involving real-world documents | Varies |
| MMLU-Pro | Academic and professional knowledge/reasoning | 10 |

Low vs high-cardinality decisions:

- Low-cardinality: 2–6 possible classes
- High-cardinality: dozens of possible classes
- Banking77: particularly challenging because there are 77 possible intents

6.2 EVALUATION METRICS

Accuracy

Measures how many predictions are exactly correct.

Accuracy = Correct Predictions / Total Predictions

Higher is better.

Brier Score

Brier measures how well the model's predicted probabilities match the actual outcome.

For a multiclass problem:

Brier = (1/N) × Σ_n Σ_k (p_nk - y_nk)^2

where:
- p_nk = predicted probability for class k
- y_nk = actual outcome, represented as 1 for the correct class and 0 for others

Lower is better.

Expected Calibration Error (ECE)

ECE measures whether the model's confidence matches its actual accuracy.

For example, if a model says it is 90% confident but is correct only 70% of the time for predictions at that confidence level, it is poorly calibrated.

Lower ECE is better.

Soft Accuracy

Soft accuracy evaluates how closely the predicted probability distribution agrees with a soft target distribution.

Unlike ordinary accuracy, it does not only ask whether the model selected the correct option. It also considers how probability was distributed across the options.

The exact calculation can depend on the benchmark implementation, so it is not a universally standardized metric like accuracy.

Higher is better.

Confident Errors

Measures predictions where the model is wrong but highly confident.

Example:
Correct answer = B
Model prediction = A with 99% confidence

Lower is better.

Coverage at <=5% Error

Measures how much of the dataset can be automatically handled while maintaining an error rate of at most 5%.

For example, 80% coverage at <=5% error means the model can confidently handle 80% of decisions while keeping the error rate within 5% on those decisions.

Higher is better.

6.3 BENCHMARK RESULTS

A. TYPED DECISIONS

| Model | Accuracy | Soft Accuracy | Brier |
|---|---:|---:|---:|
| Laya Typed Decisions | 76.6% | 47.1% | 0.062 |
| Jev | 72.7% | 58.0% | 0.148 |
| Laya | 36.2% | — | — |

Analysis:

The important distinction is between the two Laya checkpoints:

Laya: 36.2%
Laya Typed Decisions: 76.6%

This demonstrates the effect of specialization.

The Typed Decisions model was specifically fine-tuned for structured decision workflows, while the base Laya model is a general decision model.

Jev achieves 72.7% accuracy, slightly below Laya Typed Decisions, but has a stronger soft-accuracy result and better probability-oriented behavior on this benchmark.

B. AG NEWS

| Model | Accuracy |
|---|---:|
| Laya | 95.0% |
| Kev-0.5B* | 94.0% |
| Jev | 91.0% |

*Historical Kev-0.5B result; this is not the same as the current Kev variants discussed later.

Analysis:

Laya performs extremely well on this conventional 4-class classification task.

C. DAIR EMOTION

| Model | Accuracy |
|---|---:|
| Laya | 59.5% |
| Jev | 48.0% |

Analysis:

Laya performs better than Jev on this six-class emotion classification benchmark.

D. BANKING77

| Model | Accuracy |
|---|---:|
| Jev | 87.0% |
| Laya | 42.5% |

Analysis:

Banking77 contains 77 possible intents, making it a much higher-cardinality classification problem.

Jev achieves 87.0%, while Laya achieves 42.5%.

This highlights a major difference between the models: Laya is strong on conventional low-cardinality classification, while Jev handles large decision spaces considerably better.

There is also an evaluation caveat: the Laya result uses the full 77-label setup, while some other evaluation configurations use fewer labels, so comparisons should be made carefully.

E. TRANSFER-v4 — OUT-OF-DISTRIBUTION GENERALIZATION

Transfer-v4 evaluates how well models generalize to new decision tasks and sources.

Development Set:

| Model | Accuracy |
|---|---:|
| Kev-0.8B | 64.8% |
| Kev-4B | 81.7% |
| Kev-9B | 82.0% |
| Kev-27B | 85.1% |
| Jev | 85.7% |

Locked Test Set:

| Model | Accuracy | Brier |
|---|---:|---:|
| Kev-0.8B | 69.7% | 0.397 |
| Kev-4B | 83.8% | 0.224 |
| Kev-9B | 85.2% | 0.199 |
| Kev-27B | 88.9% | 0.154 |

Important comparison note:

The Jev 85.7% result is from the Transfer-v4 development comparison.

The Kev 88.9% result is from the locked test set.

Therefore, these two numbers should not be treated as a direct head-to-head comparison.

6.4 KEV SCALING RESULTS

| Benchmark | Kev-0.8B | Kev-4B | Kev-9B | Kev-27B | Jev |
|---|---:|---:|---:|---:|---:|
| Held-out breadth | 23.3% | 38.0% | 41.0% | 52.3% | 54.0% |
| Transfer-v4 Dev | 64.8% | 81.7% | 82.0% | 85.1% | 85.7% |
| Transfer-v4 Test | 69.7% | 83.8% | 85.2% | 88.9% | — |
| Hard Skills | 66.5% | 80.3% | 83.4% | 91.8% | — |
| Developer Tooling | 63.7% | 75.6% | 79.1% | 79.0% | — |
| Real Documents | 85.1% | 90.3% | 90.0% | 90.8% | — |
| MMLU-Pro | 23.0% | 56.5% | 59.0% | 67.5% | 84.0% |

Main observation:

Kev shows a strong scaling trend.

Increasing model size from 0.8B to 4B to 9B to 27B generally produces significant improvements in difficult decision-making and reasoning tasks.

For example:

Transfer-v4 Development:
64.8% -> 85.1%

Hard Skills:
66.5% -> 91.8%

for the Kev 0.8B -> 27B progression.

6.5 OVERALL COMPARISON

| Model | Main Strength | Main Limitation |
|---|---|---|
| Laya | Strong conventional classification | Struggles with high-cardinality decisions |
| Laya Typed Decisions | Excellent on specialized typed decisions | Specialized rather than general-purpose |
| Jev | High-cardinality decisions, calibration and general decision-making | Managed/API model; less control over deployment |
| Kev | Strong scaling, OOD/generalization, reasoning and documents | Larger variants require substantial compute |

Laya

Best suited for:
General, low-cardinality classification.

Examples:
- AG News: 95.0%
- DAIR Emotion: 59.5%

However, performance drops significantly on Banking77:
- Banking77: 42.5%

where the number of possible intents increases to 77.

Laya Typed Decisions

Laya Typed Decisions is a specialized checkpoint rather than simply another name for the base Laya model.

On Typed Decisions:
- Laya: 36.2%
- Laya Typed Decisions: 76.6%

This large improvement demonstrates the benefit of domain/task-specific fine-tuning.

Jev

Jev demonstrates strong performance when the decision space becomes more complex.

Examples:
- Typed Decisions: 72.7%
- Banking77: 87.0%
- Transfer-v4 Dev: 85.7%
- MMLU-Pro: 84.0%

Jev also provides probability-oriented outputs, making metrics such as Brier score and calibration particularly relevant.

Kev

Kev's biggest advantage is scaling.

Increasing model size from 0.8B to 27B produces substantial improvements on difficult tasks.

The 27B model reaches:
- Transfer-v4 Test: 88.9%
- Hard Skills: 91.8%
- Real Documents: 90.8%
- MMLU-Pro: 67.5%

However, this comes with a significant increase in compute and memory requirements.

6.6 FINAL TAKEAWAY

The benchmark results suggest four different strengths:

Laya
General, low-cardinality classification.

Laya Typed Decisions
Specialized structured decision workflows.

Jev
Complex/high-cardinality decisions and probability-aware decision-making.

Kev
Scalable decision-making, difficult reasoning and strong out-of-distribution performance.

High-level view:

Laya -> lightweight classification

Laya Typed Decisions -> specialized decision classification

Jev -> strong general decision model

Kev -> scalable decision/reasoning models

6.7 SOURCES

- Laya — official Hugging Face model:
  https://huggingface.co/convaiinnovations/laya

- Laya Typed Decisions — official Hugging Face model:
  https://huggingface.co/convaiinnovations/laya-typed-decisions

- Kev — official GitHub repository:
  https://github.com/jaredpalmer/kev

- Kev model cards:
  https://github.com/jaredpalmer/kev/tree/main/docs/model-cards

- Kev releases:
  https://github.com/jaredpalmer/kev/releases

- TypeSafe / Jev documentation:
  https://github.com/chujianyun/typesafe-wiki

CHECKPOINT NAMING RULE

Whenever there is no published result for Laya Typed Decisions, the comparison contains only Laya rather than adding an empty Typed Decisions column.

---

## Section 7 — Model Variants and GPU Requirements

SYSTEM ONE DECISION MODELS
MODEL VARIANTS AND GPU REQUIREMENTS

7. MODEL VARIANTS AND GPU REQUIREMENTS

7.1 JEV

Jev is primarily used as a managed/API-based decision model rather than a model that is downloaded and deployed locally.

Therefore:

- No model weights need to be stored locally.
- No local GPU is required for inference.
- GPU memory requirements are handled by the service provider.
- Deployment requirements are mainly API access, network connectivity, latency, and usage cost.

| Model | Local Weights | Local GPU |
|---|---|---|
| Jev | No | Not required |

7.2 LAYA

Laya is based on ModernBERT-large and has approximately 421M parameters.

For deployment and memory calculations, the different Laya task variants can be treated as the same model-size class when they use the same underlying architecture.

The main distinction is in fine-tuning/specialization, not in the basic GPU-memory requirement.

| Variant | Base Architecture | Approx. Parameters | Main Purpose |
|---|---|---:|---|
| Laya | ModernBERT-large | ~421M | General decision classification |
| Laya Multilingual | mmBERT-base | ~322M | Multilingual decisions |
| Laya Typed Decisions | ModernBERT-large | ~421M | Specialized typed decisions |

For GPU sizing, Laya Typed Decisions does not need to be treated as a separate memory category from Laya.

The important difference is that Typed Decisions has been fine-tuned specifically for structured decision workflows.

7.3 LAYA — FP32 MEMORY REQUIREMENT

Laya has approximately 421 million parameters.

In FP32:

421M × 4 bytes ≈ 1.68 GB

So the model weights themselves require approximately 1.7 GB.

However, GPU memory usage is not limited to model weights.

Additional memory is required for:

- Activations
- Intermediate tensors
- CUDA/PyTorch runtime
- Temporary buffers
- Input/output tensors
- Batch processing

Realistic single-inference estimate:

| Component | Approx. GPU Memory |
|---|---:|
| Model weights | ~1.7 GB |
| Activations/intermediate tensors | ~0.2–0.8 GB |
| Runtime/framework overhead | ~0.5–1.0 GB |
| Temporary buffers | ~0.2–0.5 GB |
| Estimated total | ~2.6–4.0 GB |

Recommended GPU:

A 4 GB GPU may work for small single-request inference, but it leaves little headroom.

For practical deployment:

8 GB GPU is recommended.

The exact requirement depends on sequence length, batch size, inference framework, and concurrency.

7.4 LAYA MULTILINGUAL — FP32

The multilingual version is approximately 322M parameters.

FP32 weight memory:

322M × 4 bytes ≈ 1.29 GB

Estimated deployment memory:

| Component | Approx. Memory |
|---|---:|
| Model weights | ~1.3 GB |
| Activations/intermediate tensors | ~0.2–0.7 GB |
| Runtime/framework | ~0.5–1.0 GB |
| Temporary buffers | ~0.2–0.5 GB |
| Estimated total | ~2.0–3.5 GB |

A 4 GB GPU may be sufficient for light inference, while 8 GB provides more practical headroom.

7.5 KEV MODEL VARIANTS

Kev provides several model sizes.

| Model | Approx. Model Size | Base Model |
|---|---:|---|
| Kev-0.8B | ~0.8B | Qwen3.5-0.8B |
| Kev-4B | ~4B | Qwen3.5-4B |
| Kev-9B | ~9B | Qwen3.5-9B |
| Kev-27B | ~27B | Qwen3.8-27B |

The smaller Kev models use a base model with LoRA and a pointer head, while the 27B version uses full-weight fine-tuning.

7.6 KEV — THEORETICAL FP32 AND FP16 WEIGHT MEMORY

A simple estimate is:

FP32:
Memory ≈ Parameters × 4 bytes

FP16:
Memory ≈ Parameters × 2 bytes

| Model | FP32 Weights | FP16 Weights |
|---|---:|---:|
| Kev-0.8B | ~3.2 GB | ~1.6 GB |
| Kev-4B | ~16 GB | ~8 GB |
| Kev-9B | ~36 GB | ~18 GB |
| Kev-27B | ~102 GB* | ~51 GB* |

*Theoretical estimate based on parameter count. Actual loaded memory can be higher because of runtime buffers, activations, CUDA overhead, KV/state memory, etc.

For the Kev-27B deployment, BF16 is used in the official serving configuration. BF16 has the same 2 bytes/parameter weight-memory class as FP16.

7.7 REALISTIC GPU REQUIREMENTS FOR KEV

Theoretical weight memory alone is not enough for GPU sizing.

A practical deployment also needs memory for activations, runtime components, temporary buffers and other inference state.

Kev-0.8B

FP16:
- Weights: ~1.6 GB
- Other memory: ~1.0–2.5 GB
- Estimated total: ~2.6–4.1 GB
- Recommended: 8 GB GPU

FP32:
- Weights: ~3.2 GB
- Other memory: ~1.0–2.5 GB
- Estimated total: ~4.2–5.7 GB
- Recommended: 8 GB GPU

Kev-4B

FP16:
- Weights: ~8 GB
- Other memory: ~2–5 GB
- Estimated total: ~10–13 GB
- Recommended: 16–24 GB GPU

FP32:
- Weights: ~16 GB
- Other memory: ~2–5 GB
- Estimated total: ~18–21 GB
- Recommended: 24 GB+ GPU

Kev-9B

FP16:
- Weights: ~18 GB
- Other memory: ~3–6 GB
- Estimated total: ~21–24 GB
- Recommended: 24–32 GB GPU

FP32:
- Weights: ~36 GB
- Other memory: ~3–6 GB
- Estimated total: ~39–42 GB
- Recommended: 48 GB GPU

Kev-27B

The 27B model is substantially different from the smaller variants.

Theoretical BF16 weight memory is approximately:

27B × 2 bytes ≈ 54 GB

The actual loaded backbone is around 25.6B parameters, giving approximately 51 GB of raw BF16 weights.

With serving/runtime overhead, approximately 66 GB resident memory can be required.

Therefore:

80 GB-class GPU is recommended for practical single-GPU deployment.

Longer contexts, higher concurrency, larger batches, or additional inference state can increase the requirement further.

7.8 OVERALL GPU REQUIREMENT

| Model | Precision | Approx. Weight Memory | Practical Estimated Memory | Recommended GPU |
|---|---|---:|---:|---:|
| Jev | API | — | No local GPU | No GPU required |
| Laya | FP32 | ~1.7 GB | ~2.6–4.0 GB | 8 GB |
| Laya Multilingual | FP32 | ~1.3 GB | ~2.0–3.5 GB | 4–8 GB |
| Kev-0.8B | FP16 | ~1.6 GB | ~2.6–4.1 GB | 8 GB |
| Kev-0.8B | FP32 | ~3.2 GB | ~4.2–5.7 GB | 8 GB |
| Kev-4B | FP16 | ~8 GB | ~10–13 GB | 16–24 GB |
| Kev-4B | FP32 | ~16 GB | ~18–21 GB | 24 GB+ |
| Kev-9B | FP16 | ~18 GB | ~21–24 GB | 24–32 GB |
| Kev-9B | FP32 | ~36 GB | ~39–42 GB | 48 GB |
| Kev-27B | BF16 | ~51–55 GB | ~66 GB measured/estimated | 80 GB |

Note: The memory figures for Laya and the smaller Kev models are practical estimates rather than fixed official hardware requirements. Actual usage depends on sequence length, batch size, concurrency, inference framework, quantization, and runtime configuration.

7.9 DEPLOYMENT COMPARISON

| Model | Deployment | GPU Requirement | Main Trade-off |
|---|---|---|---|
| Jev | Managed API | None locally | Lowest infrastructure burden |
| Laya | Self-hosted | Low | Lightweight and easy to deploy |
| Laya Multilingual | Self-hosted | Low | Supports multilingual use cases |
| Kev-0.8B | Self-hosted | Low | Lowest Kev compute cost |
| Kev-4B | Self-hosted | Medium | Better capability with moderate GPU requirement |
| Kev-9B | Self-hosted | High | Stronger performance but higher infrastructure cost |
| Kev-27B | Self-hosted | Very high | Highest capability, but requires ~80 GB-class GPU |

7.10 FINAL TAKEAWAY

From a deployment perspective:

Jev
→ Easiest infrastructure because it is accessed through an API and does not require local GPU resources.

Laya
→ Lightweight self-hosted model. Approximately 1.7 GB of FP32 weights and roughly 8 GB GPU recommended for comfortable deployment.

Kev
→ Provides a wide range of model sizes, from 0.8B to 27B, allowing a trade-off between capability and infrastructure cost.

The major jump occurs with Kev-27B, where practical deployment moves into the 80 GB GPU class.

Simplified hierarchy:

Jev
→ No local GPU

Laya
→ Low GPU requirement

Kev-0.8B / 4B
→ Low-to-medium GPU requirement

Kev-9B
→ High GPU requirement

Kev-27B
→ Very high GPU requirement / 80 GB-class GPU

Note: Laya Typed Decisions is not separately listed in the GPU table because it uses the same underlying Laya model class and therefore has essentially the same deployment/memory characteristics. Its distinction is primarily its task-specific fine-tuning, not its hardware requirement.

---

## Section 8 — Do We Need to Fine-Tune the Model?

SYSTEM ONE DECISION MODELS
SECTION 8 — DO WE NEED TO FINE-TUNE THE MODEL?

8. DO WE NEED TO FINE-TUNE THE MODEL?

Not necessarily. Fine-tuning should depend on how well the base model performs on the target decision tasks.

8.1 RECOMMENDED APPROACH

1. Start with the base model

- Evaluate Laya, Jev, or the selected Kev variant directly on representative target data.
- Measure accuracy, calibration, and confident errors.

2. Fine-tune only if required

Fine-tuning is useful when:

- The model consistently makes errors on domain-specific decisions.
- The required output format is difficult to achieve with prompting alone.
- Sufficient labeled examples are available.
- Higher accuracy is required than the base model provides.

3. Use parameter-efficient fine-tuning where possible

- LoRA/PEFT can adapt larger models without updating all model parameters.
- This reduces training memory and compute requirements.

8.2 DECISION FLOW

Base model
    ↓
Evaluate on target data
    ↓
Is performance sufficient?
    ↓
 ┌───────────────┐
 │               │
 YES             NO
 │               │
Use base      Fine-tune
model             ↓
             Re-evaluate

8.3 FINAL RECOMMENDATION

Do not fine-tune initially.

First benchmark the base model on representative target data. If there is a clear performance gap, then fine-tune using domain-specific labeled examples.

This approach avoids unnecessary training cost and establishes a clear baseline for measuring the benefit of fine-tuning.

---

## Final Conclusion

SYSTEM ONE DECISION MODELS
FINAL CONCLUSION

The evaluation shows that no single model is best for every requirement.

- Laya is a lightweight option for general, low-cardinality classification.
- Jev performs strongly on complex and high-cardinality decision-making, especially where probability and calibration matter.
- Kev provides the strongest scaling path, with larger models offering better performance at significantly higher compute cost.

For deployment, Jev requires no local GPU, Laya has relatively low resource requirements, while Kev provides a range of capability–compute trade-offs.

RECOMMENDED STRATEGY

Start with a suitable base model
        ↓
Evaluate it on representative target data
        ↓
Is the required performance achieved?
        ↓
   YES       NO
    ↓         ↓
Use base    Fine-tune
model          ↓
           Re-evaluate

Start with a suitable base model, evaluate it on representative target data, and fine-tune only if the required performance is not achieved.

This avoids unnecessary training and infrastructure costs while providing a clear baseline for measuring any improvement from fine-tuning.

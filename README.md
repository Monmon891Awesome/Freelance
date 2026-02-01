# BART Summarization Assignment: A Complete Guide

> **Teaching Philosophy**: This guide uses the Feynman Technique - if you can't explain it simply, you don't understand it well enough. Let's learn this together, step by step!

---

## Table of Contents
1. [The Big Picture: What Are We Building?](#the-big-picture-what-are-we-building)
2. [Understanding BART: The Friendly Robot Reader](#understanding-bart-the-friendly-robot-reader)
3. [The Encoder-Decoder Architecture: A Tale of Two Brains](#the-encoder-decoder-architecture-a-tale-of-two-brains)
4. [Attention: Teaching Machines to Focus](#attention-teaching-machines-to-focus)
5. [Self-Attention vs Cross-Attention: The Key Difference](#self-attention-vs-cross-attention-the-key-difference)
6. [Walking Through the Code](#walking-through-the-code)
7. [ROUGE Scores: How Do We Know If It's Good?](#rouge-scores-how-do-we-know-if-its-good)
8. [Running the Notebook](#running-the-notebook)

---

## The Big Picture: What Are We Building?

Imagine you're a student who needs to read a 10-page news article but only has 5 minutes. What do you do? You **skim** the article, **identify** the key points, and **write** a short summary in your own words.

That's exactly what we're teaching a computer to do!

```
📰 Long News Article (1000+ words)
        ↓
    [BART Model]
        ↓
📝 Short Summary (50-100 words)
```

**Our Goal**: Fine-tune a BART model to read news articles and generate human-like summaries.

---

## Understanding BART: The Friendly Robot Reader

### What is BART?

**BART** stands for **B**idirectional and **A**uto-**R**egressive **T**ransformers. Don't let the fancy name scare you! Let's break it down:

Think of BART as a student who learned to summarize by doing a clever exercise:

1. **Training Phase** (How BART Learned):
   - Take a sentence: "The cat sat on the mat"
   - Mess it up: "The [MASK] sat [MASK] the mat" or shuffle words around
   - Ask BART to fix it back to the original
   - By doing this millions of times, BART learned how language works!

2. **Our Task** (Fine-tuning):
   - BART already knows language well
   - We just need to teach it specifically how to summarize news
   - It's like hiring someone who's great at writing, then training them for journalism

### Why BART for Summarization?

```
Imagine three types of readers:

📖 Reader A (Encoder-only, like BERT):
   - Great at UNDERSTANDING text
   - Can't write new text
   - Like someone who reads books but never writes

✍️ Reader B (Decoder-only, like GPT):
   - Great at WRITING text
   - Reads left-to-right only
   - Like someone typing without looking back

📖✍️ Reader C (BART - Encoder-Decoder):
   - Can UNDERSTAND the full article first
   - Then WRITE a summary
   - Like a professional journalist!
```

BART is Reader C - the best of both worlds!

---

## The Encoder-Decoder Architecture: A Tale of Two Brains

Think of BART as having two specialized brains working together:

### The Encoder: "The Reader Brain" 🧠📖

```
Input Article: "Scientists at NASA discovered water on Mars yesterday.
               The discovery was made using the Perseverance rover.
               This finding could change our understanding of the planet."
                                    ↓
                            [ENCODER BRAIN]
                                    ↓
              Creates a "mental map" of the entire article
              Understands: WHO (NASA scientists), WHAT (water discovery),
                          WHERE (Mars), HOW (Perseverance rover)
```

**Key Feature**: The encoder reads the ENTIRE article at once, both forward and backward (bidirectional). It's like reading a page and understanding how all the sentences connect to each other.

### The Decoder: "The Writer Brain" 🧠✍️

```
              Mental map from Encoder
                        ↓
                [DECODER BRAIN]
                        ↓
    Generates summary ONE WORD AT A TIME:

    "NASA" → "NASA scientists" → "NASA scientists discovered" →
    "NASA scientists discovered water" → "NASA scientists discovered water on Mars"
```

**Key Feature**: The decoder writes left-to-right, one word at a time, like how you'd write a sentence. Each new word depends on what it already wrote.

### How They Work Together

```
┌─────────────────────────────────────────────────────────────┐
│                        ENCODER                               │
│  ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐           │
│  │NASA │ │found│ │water│ │ on  │ │Mars │ │ ... │           │
│  └──┬──┘ └──┬──┘ └──┬──┘ └──┬──┘ └──┬──┘ └──┬──┘           │
│     │       │       │       │       │       │               │
│     └───────┴───────┴───┬───┴───────┴───────┘               │
│                         │                                    │
│              [Rich Understanding of Article]                 │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          │ ← This connection is CROSS-ATTENTION!
                          │
┌─────────────────────────┴───────────────────────────────────┐
│                        DECODER                               │
│                                                              │
│  "NASA" → "scientists" → "found" → "water" → "on" → "Mars"  │
│                                                              │
│  (Generates summary word by word, constantly checking        │
│   back with the encoder: "What should I write next?")       │
└─────────────────────────────────────────────────────────────┘
```

---

## Attention: Teaching Machines to Focus

### The Problem Without Attention

Imagine trying to summarize a book by reading it once and then closing it forever. Hard, right? You'd forget important details!

Early AI models had this problem - they tried to compress an entire article into a single fixed-size "memory" before generating a summary. Important information got lost.

### The Solution: Attention Mechanism

**Attention** lets the model "look back" at the original article while writing each word of the summary.

```
Writing the summary word "Mars":

The model "looks" at the article:
   "Scientists at NASA discovered water on [MARS] yesterday..."
                                            ↑
                                     HIGH ATTENTION (0.8)

   "The discovery was made using the Perseverance rover..."
                                            ↑
                                     LOW ATTENTION (0.1)

The model focuses on "Mars" because that's what it's trying to write about!
```

### A Simple Analogy: The Highlighter Student

Think of attention like a student with a highlighter:

```
📄 Original Article (with attention weights):

[0.9] "NASA scientists discovered water on Mars."  ← Very important!
[0.3] "The announcement was made Tuesday."         ← Somewhat relevant
[0.1] "Weather in Houston was sunny that day."     ← Not relevant
[0.7] "This could indicate past life on Mars."     ← Important!

The model "highlights" what matters for the summary.
```

---

## Self-Attention vs Cross-Attention: The Key Difference

This is the heart of understanding BART! Let's use a classroom analogy.

### Self-Attention: Students Discussing Among Themselves

**Where it happens**: Inside the Encoder, and inside the Decoder (separately)

```
Imagine 5 students (words) sitting in a circle, discussing:

Article: "The cat sat on the mat"

Each word asks: "How do I relate to everyone else?"

"The" looks at: cat(relevant), sat(somewhat), on(less), the(less), mat(less)
"cat" looks at: The(relevant), sat(very relevant - cat does the sitting!), ...
"sat" looks at: cat(who sat?), mat(where?)

They build understanding WITHIN their own group.
```

**In the Encoder**: Words in the article understand each other
- "He" learns it refers to "John" mentioned earlier
- "The discovery" connects to "finding water"

**In the Decoder**: Words in the summary understand each other
- Prevents repetition ("I already said 'NASA', don't say it again")
- Maintains grammar ("After 'scientists', I need a verb")

### Cross-Attention: Students Asking the Teacher

**Where it happens**: In the Decoder, reaching back to the Encoder

```
Imagine the Decoder is a student writing a summary,
and the Encoder is a teacher who read the whole article:

Student (Decoder): "I'm about to write something about a planet..."
                   *raises hand*
                   "Teacher, which planet was in the article?"

Teacher (Encoder): *points to notes*
                   "Look here - 'Mars' had high importance!"

Student writes: "Mars"

Student: "Now I need to say what happened there..."
         *raises hand again*
         "Teacher, what was discovered?"

Teacher: *points to notes*
         "'Water' and 'discovered' are key here!"

Student writes: "water was discovered"
```

### Why This Difference Matters for Summarization

| Aspect | Self-Attention | Cross-Attention |
|--------|---------------|-----------------|
| **Question** | "How do words in MY sequence relate?" | "What from the ARTICLE should I use now?" |
| **Purpose** | Build understanding within a text | Bridge article → summary |
| **Analogy** | Students discussing among themselves | Students asking the teacher |
| **For Summarization** | Understand article / Keep summary coherent | Decide WHAT to include in summary |

### The Magic Formula

```
Cross-Attention(Query, Key, Value):

Query (Q) = "What am I looking for?" → Comes from DECODER
Key (K)   = "What's available?"      → Comes from ENCODER
Value (V) = "What's the content?"    → Comes from ENCODER

Score = How well Query matches each Key
Output = Weighted sum of Values based on Scores

Simple example:
- Decoder is writing and needs info about "location"
- Query: "I need location information"
- Keys: ["NASA"=org, "water"=substance, "Mars"=location, "rover"=vehicle]
- Scores: [0.1, 0.1, 0.9, 0.1]  ← "Mars" matches "location" query!
- Output: Mostly pulls information from "Mars"
```

---

## Walking Through the Code

Let's trace through the notebook step by step:

### Step 1: Loading the Data

```python
# We get news articles and their human-written summaries
ds_train_tf = tfds.load('cnn_dailymail:3.4.0', split='train[:1%]')

# Think of it like:
# Article: "A 3000-word news story about NASA..."
# Summary: "NASA found water on Mars using Perseverance rover."
```

**Why CNN/DailyMail?** It's a huge dataset of news articles paired with bullet-point summaries written by journalists. Perfect training data!

### Step 2: Tokenization

```python
tokenizer = BartTokenizer.from_pretrained("facebook/bart-base")

# Converts text to numbers the model understands:
# "NASA found water" → [3458, 2034, 514]

# Like translating English to "Computer-ese"
```

**Why tokenize?** Computers don't understand words - they understand numbers. The tokenizer is like a dictionary that converts back and forth.

### Step 3: Fine-tuning

```python
trainer = Seq2SeqTrainer(
    model=model,
    train_dataset=tokenized_train,
    ...
)
trainer.train()

# The model sees thousands of (article, summary) pairs
# It learns: "Oh, when I see these patterns, I should write this kind of summary"
```

**What's happening inside?**
1. Model reads an article
2. Model generates a summary
3. We compare it to the human summary
4. Model adjusts its weights to do better next time
5. Repeat thousands of times!

### Step 4: Generating Summaries

```python
summary_ids = model.generate(
    inputs['input_ids'],
    num_beams=4,  # Try 4 different paths, pick the best
    ...
)

# Like writing a summary, but considering multiple word choices
# and picking the most coherent path
```

### Step 5: Evaluation with ROUGE

```python
scorer = rouge_scorer.RougeScorer(['rouge1', 'rouge2', 'rougeL'])
scores = scorer.score(reference_summary, generated_summary)

# Measures: How much does our summary overlap with the human's?
```

---

## ROUGE Scores: How Do We Know If It's Good?

ROUGE = **R**ecall-**O**riented **U**nderstudy for **G**isting **E**valuation

Think of it as measuring how much your summary "matches" the human-written one.

### ROUGE-1: Word Matching

```
Human Summary:    "NASA scientists discovered water on Mars"
Machine Summary:  "Scientists found water on Mars"

Matching words: scientists, water, on, Mars (4 matches)
Human summary words: 6
ROUGE-1 Recall = 4/6 = 0.67 (67%)

"You captured 67% of the important words!"
```

### ROUGE-2: Phrase Matching

```
Human:   "NASA scientists" "scientists discovered" "discovered water" ...
Machine: "Scientists found" "found water" "water on" ...

Matching pairs: "water on", "on Mars" (2 matches out of 5)
ROUGE-2 ≈ 0.40 (40%)

"You captured 40% of the important phrases!"
```

### ROUGE-L: Longest Common Sequence

```
Human:   "NASA scientists discovered water on Mars"
Machine: "Scientists found water on Mars"

Longest matching sequence: "water on Mars" (3 words in order)
ROUGE-L captures how well the ORDER is preserved
```

### What's a Good Score?

| Score | Interpretation |
|-------|---------------|
| ROUGE-1 > 0.40 | Decent word coverage |
| ROUGE-2 > 0.20 | Good phrase matching |
| ROUGE-L > 0.35 | Good overall structure |

State-of-the-art models achieve ~0.45 ROUGE-1 on CNN/DailyMail.

---

## Running the Notebook

### Requirements

- Google Colab (recommended for free GPU)
- Or local machine with Python 3.8+

### Quick Start

1. **Open in Colab**: Upload `bart_summarization_assignment.ipynb` to Google Colab

2. **Enable GPU**: Runtime → Change runtime type → GPU

3. **Run the first cell** (installations):
   ```python
   !pip install transformers datasets tensorflow tensorflow_datasets rouge-score
   ```

4. **Restart Runtime**: Runtime → Restart runtime

5. **Run all cells**: Runtime → Run all

### Expected Runtime

| Step | Time (with GPU) |
|------|-----------------|
| Data loading | 2-3 minutes |
| Tokenization | 1-2 minutes |
| Fine-tuning (3 epochs) | 30-60 minutes |
| Evaluation | 5-10 minutes |

### Troubleshooting

**Out of Memory?**
- Reduce `per_device_train_batch_size` from 4 to 2
- Reduce training samples from 2000 to 1000

**Slow Training?**
- Make sure GPU is enabled
- Check with: `!nvidia-smi`

---

## Key Takeaways

1. **BART** = Encoder (reads & understands) + Decoder (writes summary)

2. **Self-Attention** = Words understanding each other WITHIN a sequence
   - Encoder: Words in article understand relationships
   - Decoder: Summary stays coherent and grammatical

3. **Cross-Attention** = Decoder asking Encoder "What should I write next?"
   - This is how the summary stays faithful to the article
   - Each generated word "looks back" at the relevant parts

4. **Fine-tuning** = Teaching a pre-trained model our specific task
   - BART already knows language
   - We just teach it to summarize news

5. **ROUGE** = Measuring how well machine summaries match human ones

---

## Further Reading

- [BART Paper](https://arxiv.org/abs/1910.13461) - The original research paper
- [Attention Is All You Need](https://arxiv.org/abs/1706.03762) - The Transformer paper
- [Hugging Face Course](https://huggingface.co/course) - Free NLP course
- [The Illustrated Transformer](https://jalammar.github.io/illustrated-transformer/) - Visual explanations

---

*Created for the Generative AI assignment on Encoder-Decoder Attention in BART Summarisation*

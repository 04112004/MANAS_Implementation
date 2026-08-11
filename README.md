# MANAS

**A local, continuously growing memory / knowledge system written in Rust.**

MANAS (Mini-MANAS) is an experimental retrieval engine that learns facts, stores them as local "neurons," and retrieves the most relevant one in response to a query — without depending on a pretrained LLM for recall. It is designed to explore how far a lightweight, explainable, embedding + heuristic retrieval pipeline can go as a knowledge base scales from a few hundred to 1000+ facts.

---

## ✨ Key Features

- **Local-first** — all knowledge is stored in a plain-text brain file (`brain.manas.txt`), no external database or API required.
- **Simple ingestion pipeline** — teach individual facts, or ingest entire `.txt` files (including text extracted from PDFs).
- **Custom embedding** — a lightweight hash-based `embed()` function converts text into normalized vectors without external ML dependencies.
- **Composite ranking algorithm** — combines cosine similarity, keyword overlap, importance, and a custom **definition bonus** to surface the most useful fact, not just the most similar one.
- **Scales to 1000+ facts** — stress-tested against a growing knowledge base spanning Machine Learning, Deep Learning, Computer Vision (RT-DETR), Java Programming, and Generative AI.

---

## 🧠 How It Works

```
Knowledge
   ↓
MANAS learns/stores it
   ↓
Knowledge becomes facts/neurons
   ↓
Facts are stored locally
   ↓
User asks a question
   ↓
MANAS searches its stored knowledge
   ↓
Most relevant fact is returned
```

Each stored **neuron** (fact) carries:

| Field        | Description                                  |
|--------------|-----------------------------------------------|
| `fact`       | The raw text of the fact                      |
| `weights`    | Embedding vector                              |
| `importance` | Manually or heuristically assigned weight     |
| `protection` | Flag to prevent overwriting critical facts    |
| `source`     | Origin file/document                          |
| `activations`| Retrieval frequency / usage tracking          |

---

## 🔎 Retrieval & Ranking

Retrieval combines four signals into a single ranking score:

```rust
let definition_bonus = definitional_bonus(query, &n.fact);

let score =
    similarity
    + (keyword_overlap * 0.50)
    + (n.importance * 0.10)
    + definition_bonus;
```

| Signal              | Purpose                                                        |
|---------------------|------------------------------------------------------------------|
| `similarity`         | Cosine similarity between query and fact embeddings             |
| `keyword_overlap`     | Rewards facts sharing key terms with the query                  |
| `importance`          | Boosts facts explicitly marked as more significant              |
| `definition_bonus`    | Rewards facts that read like genuine definitions (e.g. contain "is", "refers to", "means") over headings that merely *look* similar |

### Why the definition bonus exists

Early testing showed that section **headings** (e.g. *"Machine Learning: Concepts and Foundations"*) could outrank actual **definitions** (e.g. *"Machine learning is the branch of artificial intelligence..."*) purely on similarity score. The `definitional_bonus()` function was introduced to detect genuinely definitional language rather than superficial cues like a colon — which headings also contain — ensuring MANAS returns the real answer, not just the closest-looking heading.

---

## 📂 Project Structure

```
manas-mini/
├── main.rs           # Source code / core logic
├── manas              # Compiled executable (Linux/Colab build)
├── brain.manas.txt     # Persistent stored knowledge (generated at runtime)
└── *.txt               # Ingested knowledge sources (incl. PDF-extracted text)
```

> **Note:** `main.rs` (logic) and `brain.manas.txt` (knowledge) are independent. Changing the knowledge base — adding, deleting, or re-ingesting facts — does **not** require recompilation. Recompilation is only needed when `main.rs` itself changes.

---

## 🚀 Getting Started

### Prerequisites

- Rust toolchain (`rustc`, `cargo`)
- Python 3 with `pypdf` (optional, for PDF ingestion)

### Installation

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
```

### Build

```bash
rustc -O main.rs -o manas
```

### Run

```bash
./manas
```

---

## 🛠️ Usage

### Teach a fact

```bash
./manas teach "A cat is a small domesticated animal with fur and whiskers"
```

### Ask a question

```bash
./manas ask "What is a cat?"
```

### Ingest a document

```bash
./manas ingest ./knowledge/rtdetr.txt
```

### Inspect stored knowledge

```bash
./manas inspect
```

### Reset the knowledge base

```bash
rm -f brain.manas.txt
```

---

## 📊 Example Output

```
$ ./manas ask "What is Machine Learning?"

Top 3 Answers:

1. Machine learning is the branch of artificial intelligence that gives
   computer systems the ability to improve their performance on a task by learning...
   score: 1.91 (similarity: 0.60), importance: 0.10, source: 01_Machine_Learning.txt

2. What sets deep learning apart from classical machine learning is that a deep
   network can learn its own useful features directly from raw data...
   score: 1.67 (similarity: 0.36), importance: 0.10, source: 02_Deep_Learning.txt

3. Even with these challenges, machine learning is now deployed across nearly
   every industry.
   score: 1.59 (similarity: 0.45), importance: 0.10, source: 01_Machine_Learning.txt
```

---

## 🧪 Testing at Scale

The knowledge base was progressively scaled to validate ranking robustness:

| Stage   | Fact Count |
|---------|------------|
| Initial | ~397       |
| Stage 2 | 481        |
| Stage 3 | 752        |
| Final   | 1000+      |

Evaluation covered four query categories:

- **Definition questions** — *"What is Machine Learning?"*, *"What is RT-DETR?"*
- **Technical questions** — *"How does RT-DETR use Hungarian matching?"*
- **Comparison questions** — *"What is the difference between CNN and Transformer?"*
- **Weak-match / unrelated questions** — queries with no strong match in the brain

---

## 🗺️ Roadmap

- [ ] Improve embedding quality beyond hash-based bucketing
- [ ] Add fact deduplication on ingestion
- [ ] Support incremental re-ranking as importance scores evolve
- [ ] Expose a simple HTTP/CLI API for external integrations

---

## 📄 License

This project is currently unlicensed. Add a `LICENSE` file to specify usage terms before public distribution.

---

## 🙋 Author

**Harini P**

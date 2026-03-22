# Knowledge Graph Construction, Alignment, Reasoning & RAG

**Course:** Web Datamining and Semantics  
**Authors:** Léa Sagroun and François Saint-Jean — DIA 5  
**Domain:** AI Research (LLMs, Machine Learning, Deep Learning)  
**Data source:** distill.pub + Wikipedia  

---

## Project Overview

This project implements a complete Knowledge Graph pipeline over the AI Research domain:

1. **Lab 1** — Web crawling + NER + relation extraction
2. **Lab 2** — RDF construction + Wikidata alignment + SPARQL expansion
3. **Lab 3** — SWRL reasoning + Knowledge Graph Embedding (TransE, ComplEx)
4. **Lab 4** — RAG chatbot (NL → SPARQL + self-repair via local LLM)

**Final KB:** 65,103 triples · 50,469 entities · 31 relations

---

## Repository Structure

```
web-datamining-project/
├── data/
│   ├── extracted_knowledge.csv     # NER output (entities)
│   ├── extracted_relations.csv     # co-occurrence relations
│   ├── crawler_output.jsonl        # raw crawled text
│   └── family.owl                  # ontology for SWRL reasoning
├── notebooks/
│   ├── Lab1_web_datamining.ipynb   # Crawling + NER
│   ├── Lab_4_web_datamining.ipynb  # KB construction + alignment
│   ├── Lab5_web_datamining.ipynb   # SWRL + KGE
│   └── Lab_6_web_datamining.ipynb  # RAG pipeline
│   └── output/
│       ├── private_kb.nt           # Initial RDF graph
│       ├── private_kb.ttl          # Initial RDF graph (Turtle)
│       ├── expanded_kb.nt          # Expanded KB (65k triples)
│       ├── ontology.ttl            # OWL ontology
│       ├── alignment_full.ttl      # Entity + predicate alignments
│       ├── stats_report.txt        # KB statistics
│       ├── mapping_table.csv       # Entity linking results
│       └── kge/
│           ├── train.txt           # 80% split (50,138 triples)
│           ├── valid.txt           # 10% split (2,185 triples)
│           ├── test.txt            # 10% split (2,217 triples)
│           ├── entities.dict       # Entity URI → ID mapping
│           └── relations.dict      # Relation URI → ID mapping
├── README.md
├── requirements.txt
└── .gitignore
```

---

## Hardware Requirements

| Component | Spec |
|-----------|------|
| Machine | Acer Aspire A515-57 |
| OS | Windows 11 |
| CPU | Intel Core i7 ~2300 MHz |
| RAM | 16 GB |
| GPU | None (CPU only) |

**Training times (CPU):**
- TransE 100 epochs: ~9 minutes
- ComplEx 100 epochs: ~22 minutes

---

## Installation

### 1. Clone the repository

```bash
git clone https://github.com/leasagroun/web-datamining-project.git
cd web-datamining-project
```

### 2. Create and activate the conda environment

```bash
conda create -n tp_wine python=3.11
conda activate tp_wine
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Install Ollama (for RAG lab only)

Download from https://ollama.com/download and install.

Then pull the model:
```bash
ollama pull gemma:2b
```

---

## How to Run Each Module

> All code is organized in Jupyter notebooks. Open them with Jupyter Lab or Jupyter Notebook.

### Lab 1 — Web Crawling + NER
Open and run `notebooks/Lab1_web_datamining.ipynb`

**Output:** `data/extracted_knowledge.csv`, `data/extracted_relations.csv`

---

### Lab 2 — KB Construction, Alignment & Expansion
Open and run `notebooks/Lab_4_web_datamining.ipynb`

Runs in order: RDF construction → entity linking → predicate alignment → SPARQL expansion → KGE splits → statistics report.

**Output:** `output/expanded_kb.nt`, `output/kge/train.txt`, `valid.txt`, `test.txt`

---

### Lab 3 — SWRL Reasoning + KGE
Open and run `notebooks/Lab5_web_datamining.ipynb`

**Part 1:** SWRL rule on `data/family.owl` — infers Peter (70) and Marie (69) as `oldPerson`

**Part 2:** Trains TransE and ComplEx via PyKEEN, runs size sensitivity analysis (20k / 50k / full), produces t-SNE plot and nearest neighbor analysis.

---

### Lab 4 — RAG Pipeline
Open and run `notebooks/Lab_6_web_datamining.ipynb`

Start Ollama first:
```bash
ollama serve
```
Verify at http://localhost:11434 — should display "Ollama is running"

Then run all cells in the notebook. Type your questions in the input cell at the bottom.

Special commands:
- `eval` — runs 5 evaluation questions (baseline vs RAG)
- `quit` — exits the loop

---

## RAG Demo Screenshot

```
============================================================
  RAG Chatbot — AI Research Knowledge Graph
  Model: gemma:2b | Graph: output/private_kb.ttl
============================================================

Question: Who is related to Dario Amodei?

--- Baseline (No RAG) ---
I am unable to provide personal information...

--- SPARQL-generation RAG ---
  [SPARQL Query Generated]
  SELECT ?related WHERE {
    local:DarioAmodei onto:relatedTo ?related .
  }

  [Results — 1 row(s)]
  related
  --------------------------------
  local:TomBrown
```

---

## Key Results

### Entity Linking
- 281 / 377 entities matched to Wikidata (74.5%)
- Confidence threshold: 0.75 (Jaccard similarity)

### KB Statistics
| Metric | Initial KB | Expanded KB |
|--------|-----------|-------------|
| Triples | 2,246 | 65,103 |
| Entities | 434 | 50,469 |
| Relations | 8 | 31 |

### KGE Results
| Model | MRR | Hits@10 | Time |
|-------|-----|---------|------|
| TransE | 0.0136 | 0.0334 | 533s |
| ComplEx | 0.0009 | 0.0009 | 1343s |

### RAG Evaluation
| Question | RAG Result | Correct? |
|----------|-----------|---------|
| Who is related to Dario Amodei? | local:TomBrown | YES |
| Persons affiliated with Google Research? | local:AdamPearce | YES |
| Organizations in the graph? | 493 entities | PARTIAL |
| Who is Ilya Sutskever related to? | SPARQL error | NO |
| Persons related to OpenAI? | SPARQL error | NO |

---

## Requirements

See `requirements.txt` for the full list. Key dependencies:

```
rdflib>=6.0
requests>=2.32
networkx>=3.0
pykeen==1.11.1
torch>=2.0
scikit-learn>=1.2
matplotlib>=3.7
owlready2>=0.46
```

---

## Ollama Setup

1. Install: https://ollama.com/download
2. Pull model: `ollama pull gemma:2b`
3. Ollama starts automatically on Windows after installation
4. Verify: open http://localhost:11434 in your browser

If port 11434 is already in use, Ollama is already running — no need to start it manually.

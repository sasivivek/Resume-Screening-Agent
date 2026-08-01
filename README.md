# Resume Screening Agent

An intelligent, NLP-powered AI agent that parses resumes, scores them against a Job Description using multi-signal semantic analysis, and outputs a ranked shortlist with per-candidate reasoning.

**Built for the ROOMAN AI 24-Hour Agent Challenge** (Resume Screening Agent - Intermediate)

> "My agent takes a job description + a folder of resumes and produces a ranked, scored shortlist with per-candidate reasoning."

## Features

- **Multi-Signal Scoring**: Evaluates candidates across 4 weighted signals (semantic similarity, skill matching, experience, education)
- **Semantic NLP Analysis**: Uses `sentence-transformers` (`all-MiniLM-L6-v2`) for deep contextual matching beyond keyword search
- **LLM-Enhanced Extraction** (Optional): Google Gemini for structured resume/JD parsing and reasoning generation
- **Robust Fallback**: Works entirely locally without any API key using regex extraction + embedding scoring
- **Multi-Format Parsing**: Handles PDF, DOCX, and plain text resumes
- **Batch Processing**: Processes 10+ resumes in a single run
- **Dual Output**: Exports results to both JSON and CSV formats



## Quick Start

### 1. Clone the repository
```bash
git clone <repo_url>
cd resume_screening_agent
```

### 2. Install dependencies
```bash
pip install -r requirements.txt
```

### 3. (Optional) Set up Gemini API key
Get a free key at https://aistudio.google.com/apikey

```bash
# Option A: Environment variable
export GOOGLE_API_KEY="your_api_key_here"

# Option B: Create a .env file
cp .env.example .env
# Edit .env and add your key
```

> **Note**: The agent works perfectly without an API key. It uses local embedding models for scoring and regex for extraction. The API key adds LLM-powered structured extraction and reasoning.

### 4. Run the agent
```bash
# With command-line arguments
python main.py --jd sample_jd/jd.txt --resumes sample_resumes/

# Disable LLM even if API key is available
python main.py --jd sample_jd/jd.txt --resumes sample_resumes/ --no-llm

# Interactive mode (will prompt for paths)
python main.py
```

### CLI Options
| Flag | Description | Default |
|------|------------|---------|
| `--jd` | Path to job description file | _(interactive prompt)_ |
| `--resumes` | Path to directory of resumes | _(interactive prompt)_ |
| `--output` | Output directory for results | `output` |
| `--top` | Show detailed view for top N candidates | `5` |
| `--no-llm` | Disable LLM even if API key is set | `False` |

## Sample Output

Running with the provided sample data (Python Backend Developer JD + 12 resumes):

```
RANKING RESULTS:
+--------+-------------------+---------+-----------+
|   Rank | Name              |   Score | Matched   |
+========+===================+=========+===========+
|      1 | ALICE JOHNSON     |    69.3 | 8 skills  |
|      2 | CAROL WILLIAMS    |    61.5 | 6 skills  |
|      3 | DAVID LEE         |    60.4 | 6 skills  |
|      4 | FRANK GARCIA      |    59.5 | 5 skills  |
|      5 | DR. LIAM TAYLOR   |    59.1 | 4 skills  |
|      6 | KATE BROWN        |    58.8 | 4 skills  |
|      7 | BOB SMITH         |    56.2 | 4 skills  |
|      8 | GRACE KUMAR       |    54.5 | 3 skills  |
|      9 | EMILY CHEN        |    54.4 | 3 skills  |
|     10 | JAMES WILSON      |    53.9 | 7 skills  |
|     11 | HENRY ZHANG       |    49.2 | 2 skills  |
|     12 | ISABELLA MARTINEZ |    41.0 | 1 skills  |
+--------+-------------------+---------+-----------+

--- Candidate Detail: ALICE JOHNSON ---
Rank: 1
Composite Score: 69.3
Matched Skills: django, git, github, postgresql, python, django rest framework, pytest, unittest
Missing Skills: rest apis
Reasoning: Ranked #1 with a composite score of 69.3/100. Matched 8/9 required skills.
           Key gaps: missing rest apis. Meets experience requirements.
```

Results are saved to `output/ranked_results.json` and `output/ranked_results.csv`.

## Scoring Method

The agent uses a **4-signal weighted composite score** (detailed in [scoring_method.md](scoring_method.md)):

| Signal | Weight | Method |
|--------|--------|--------|
| Semantic Similarity | 40% | Cosine similarity between resume and JD embeddings (`all-MiniLM-L6-v2`) |
| Skill Match | 30% | Exact + semantic fuzzy matching of required skills |
| Experience | 15% | Years comparison (proportional if below, 100% if meets/exceeds) |
| Education | 15% | Degree hierarchy comparison (PhD > Master's > Bachelor's) |

**Composite Score** = (Semantic x 0.40) + (Skills x 0.30) + (Experience x 0.15) + (Education x 0.15)

## Project Structure

```
resume_screening_agent/
├── main.py                  # CLI entry point (the agent)
├── config.py                # Data models, weights, configuration
├── resume_parser.py         # Resume parsing (LLM + regex fallback)
├── jd_parser.py             # JD parsing (LLM + regex fallback)
├── scorer.py                # 4-signal scoring engine
├── ranker.py                # Ranking + reasoning generation
├── utils.py                 # File I/O, formatting, display
├── requirements.txt         # Pinned dependencies
├── .env.example             # API key template
├── scoring_method.md        # Detailed scoring methodology
├── sample_jd/
│   └── jd.txt               # Sample JD (Python Backend Developer)
├── sample_resumes/          # 12 sample resumes (varied quality)
│   ├── resume_01_alice_johnson.txt    # Strong match
│   ├── resume_02_bob_smith.txt        # Strong match
│   ├── resume_03_carol_williams.txt   # Strong match
│   ├── resume_04_david_lee.txt        # Medium match
│   ├── resume_05_emily_chen.txt       # Medium match
│   ├── resume_06_frank_garcia.txt     # Medium match
│   ├── resume_07_grace_kumar.txt      # Weak match (Java dev)
│   ├── resume_08_henry_zhang.txt      # Weak match (Frontend)
│   ├── resume_09_isabella_martinez.txt # Weak match (Data analyst)
│   ├── resume_10_james_wilson.txt     # Edge case (Fresh grad)
│   ├── resume_11_kate_brown.txt       # Edge case (Career changer)
│   └── resume_12_liam_taylor.txt      # Edge case (Overqualified)
├── output/                  # Generated output
│   ├── ranked_results.json
│   └── ranked_results.csv
└── tests/
    └── test_agent.py        # 19 unit tests
```

## Design Choices & Tradeoffs

### Why `sentence-transformers` for semantic similarity?
Using a lightweight local embedding model (`all-MiniLM-L6-v2`, 22M params) provides fast, offline, and cost-free dense vector comparison. It captures contextual meaning that keyword matching misses (e.g., "built microservices" is semantically relevant to a "backend developer" JD even without exact keyword overlap).

### Why multi-signal scoring instead of a single LLM prompt?
A single LLM prompt asking "rate this candidate 0-100" is opaque, non-reproducible, and prone to hallucination. Our 4-signal deterministic approach is:
- **Explainable**: Each score dimension is transparent and auditable
- **Reproducible**: Same input always produces the same output
- **Debuggable**: Easy to diagnose why a candidate scored high or low

### Why Gemini with regex fallback?
Gemini excels at extracting nuanced skills and generating reasoning. However, a regex fallback ensures the agent **never fails** due to API rate limits, network issues, or missing API keys. This makes reviewer setup foolproof.

### What would improve with more time?
- **Fine-tuned extraction models**: Specialized NER models for resume parsing (e.g., SpaCy with custom entities)
- **Streamlit/Gradio UI**: Visual interface with drag-and-drop resume upload
- **Better skill taxonomy**: A curated skill ontology with synonyms (e.g., "React.js" = "ReactJS" = "React")
- **PDF OCR**: Support for image-based/scanned PDFs via Tesseract
- **Configurable weights**: Let users adjust scoring weights via CLI

## Running Tests

```bash
cd resume_screening_agent
python -m pytest tests/test_agent.py -v
```

All 19 tests cover: data model validation, scoring functions, file discovery, JSON/CSV output, and edge cases.

## Limitations

- PDF parsing quality depends on document structure (image-heavy PDFs need OCR, not included)
- Regex-based skill extraction may miss unconventional skill formats; LLM mode is more robust
- Semantic similarity scores tend to cluster in a narrow range for text documents (mitigated by the multi-signal approach)
- The `exit code 1` on Windows PowerShell is a false alarm caused by HuggingFace stderr warnings, not an actual failure

## License

MIT License

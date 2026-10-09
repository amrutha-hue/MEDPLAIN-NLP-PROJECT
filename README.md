# 🩺 MedPlain: Medical Text Simplifier (NLP)

> Turns clinical wording like *"Discontinue aspirin immediately; contraindicated in patients with hypertension."*
> into plain language like *"Stop taking aspirin right away. It is not safe to use if you have high blood pressure."*

![Python](https://img.shields.io/badge/python-3.10%2B-blue)
![NLP](https://img.shields.io/badge/NLP-rule--based-8b5cf6)
![UI](https://img.shields.io/badge/UI-Gradio-orange)
![License](https://img.shields.io/badge/license-MIT-green)

**Author:** Amrutha S, built while learning Python, NumPy, SQL and NLP in an AI-integrated data science course.

> ⚕️ **Disclaimer:** MedPlain is an educational NLP project. It does **not** give medical advice and must not be used to make health decisions. Always follow your doctor or pharmacist.

---

## 📑 Table of contents
1. [Why this project?](#-why-this-project)
2. [Features](#-features)
3. [Demo](#-demo)
4. [How it works](#-how-it-works)
5. [Project structure](#-project-structure)
6. [Dataset](#-dataset)
7. [Installation & usage](#-installation--usage)
8. [Results](#-results-and-an-honest-note)
9. [Code walkthrough](#-code-walkthrough)
10. [Extending the rules](#-extending-the-rules)
11. [Limitations](#-known-limitations)
12. [Roadmap](#-roadmap)
13. [Tech stack](#-tech-stack)
14. [License](#-license)

---

## 💡 Why this project?
Doctors' notes and prescriptions are full of terms like *PRN*, *BID*, *dyspnea* and *contraindicated*. Many patients can't follow them, which makes it harder to take medicines correctly. MedPlain explores one small part of that problem: **automatically rewriting medical instructions in simple words** and showing which words were difficult.

## ✨ Features
- **Text simplification** using abbreviation rules, sentence patterns and a medical-word dictionary.
- **Dosing-frequency decoding:** `q.d.`, `BID`, `t.i.d.`, `qhs`, `q 8 h` become "once a day", "twice a day", "three times a day", "at bedtime", "every 8 hours".
- **Topic detection** across 7 topics: dosing, stop medicine, follow-up, monitoring, warning signs, diagnosis, diet and care.
- **Readability statistics:** words per sentence, average word length, count of medical terms.
- **Dataset analysis:** category counts chart and the most common medical terms across 10,000 rows.
- **Interactive dark-theme web app (Gradio)** with highlighted hard words, topic chips, a word-meaning list, a sentence breakdown and live statistics.

## 🖼️ Demo
Add a screenshot of your app to `assets/` and link it here:

```markdown
![MedPlain app](assets/screenshot.png)
```

Example input and output:

| Complex (input) | Simple (output) |
|---|---|
| `Initiate cephalexin 500 mg oral BID.` | `Start taking cephalexin 500 mg by mouth twice a day.` |
| `Patient presents with hypertension. Monitor BP; consult physician if visual disturbances worsens.` | `You have high blood pressure. Keep an eye on your blood pressure. Talk to your doctor if changes in your vision gets worse.` |
| `Initiate cetirizine 5 mg oral qhs PRN for rhinorrhea.` | `Start taking cetirizine 5 mg by mouth at bedtime only when needed for a runny nose.` |

## ⚙️ How it works
MedPlain is **rule-based NLP**. It uses no neural network and no training step. Every output can be traced to a rule you can read.

```
 Raw clinical text
        │
        ▼
 1. Remove dots in short forms        q.d. → qd, p.o. → PO
 2. Expand "what to monitor"          Monitor BP → Monitor your blood pressure
 3. Apply sentence patterns           "Discontinue X immediately" → "Stop taking X right away"
 4. Replace medical words             dyspnea → shortness of breath
    (longest phrases first)
 5. Decode dosing frequency           BID → twice a day
 6. Clean up spacing / capital letters
        │
        ▼
 Plain-language text
```

Why "longest phrases first" in step 4? So that `chronic obstructive pulmonary disease` is replaced as a whole before the single word `chronic` can be changed on its own.

### Topic detection
Each topic has a small keyword list (for example `stop_medicine` looks for `discontinue`). A text gets every topic whose keywords appear in it.

### Two engines in this repo
| Engine | Where | Best for |
|---|---|---|
| **Dataset pipeline** (sentence patterns + abbreviations + word list) | `medplain/` package | Reproducing the notebook results on `medplain_10k.csv` |
| **Web-app engine** (larger general dictionary + regex topic detection) | `app.py` | Free-form text typed into the UI |

They were developed in different parts of the notebook. Merging them into one engine is the first item on the [roadmap](#-roadmap).

## 📁 Project structure
```
medplain/
├── app.py                      # Gradio web app (dark UI)
├── medplain/                   # Reusable Python package
│   ├── __init__.py
│   ├── rules.py                # All rule tables: abbreviations, patterns, word list, topic keywords
│   └── simplify.py             # simplify(), get_topics(), get_sentences(), text_stats() ...
├── scripts/
│   └── evaluate.py             # Re-runs the accuracy evaluation on the CSV
├── tests/
│   └── test_simplify.py        # Unit tests (pytest)
├── notebooks/
│   └── MedPlain_NLP_Project.ipynb   # Original Google Colab notebook with outputs
├── data/
│   └── README.md               # Dataset description (CSV not committed)
├── assets/                     # Screenshots for the README
├── requirements.txt
├── .gitignore
├── LICENSE
└── README.md
```

## 📊 Dataset
`medplain_10k.csv` has **10,000 rows and 4 columns**: `id`, `complex`, `plain`, `category`. There are no missing values.

| Topic tag | Rows containing it |
|---|---|
| dosing | 6,156 |
| warning | 3,276 |
| stop_medicine | 3,021 |
| follow_up | 2,960 |
| diagnosis | 2,938 |
| monitoring | 2,808 |
| diet | 2,687 |

A row can carry several tags (for example `dosing+stop_medicine`), so the counts add up to more than 10,000. See [`data/README.md`](data/README.md).

## 🚀 Installation & usage

### Option A: Google Colab (easiest)
1. Open `notebooks/MedPlain_NLP_Project.ipynb` in Colab.
2. Run the cells in order. When asked, upload `medplain_10k.csv`.
3. The last cell launches the web app and prints a temporary public link.

### Option B: Run on your computer
```bash
# 1. Clone
git clone https://github.com/<your-username>/medplain.git
cd medplain

# 2. (Recommended) virtual environment
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate

# 3. Install
pip install -r requirements.txt

# 4. Launch the web app, then open the local URL it prints
python app.py

# 5. Run the tests
pytest -q

# 6. Re-run the evaluation (needs data/medplain_10k.csv)
python scripts/evaluate.py data/medplain_10k.csv
```

### Use it as a library
```python
from medplain import simplify, get_topics, text_stats

text = "Initiate cephalexin 500 mg oral BID."
print(simplify(text))      # Start taking cephalexin 500 mg by mouth twice a day.
print(get_topics(text))    # dosing
print(text_stats(text))
```

## 📈 Results and an honest note
Measured on all 10,000 rows of the dataset:

| Metric | Result |
|---|---|
| Exact match between `simplify()` and the dataset's `plain` column | **100%** |
| Topic detection (exact set of tags) | **100%** |
| Sample readability: average word length | 6.18 → **4.13** |
| Sample readability: medical terms | 7 → **0** |

**How to read these numbers:** the rules were written by studying this same dataset, and the sentences in it follow repeating templates. So 100% shows that the rules fully cover this dataset. It does **not** mean MedPlain would score 100% on real doctors' notes. Testing on unseen, real-world text is the most important next step.

## 🔍 Code walkthrough
| Function | What it does |
|---|---|
| `simplify(text)` | Runs the 6-step pipeline above |
| `get_tokens(text)` | Splits text into lowercase word tokens |
| `get_sentences(text)` | Splits on `. ! ?` |
| `get_topics(text)` | Keyword matching to find topics |
| `count_hard_words(text)` | Counts words from the medical word list |
| `text_stats(text)` | Words per sentence, average word length, medical-term count |

In `app.py`: `analyze_text()` builds the whole results page (original text with highlighted hard words, simple version, topic chips, word meanings, sentence breakdown, statistics, disclaimer).

## 🧩 Extending the rules
Open `medplain/rules.py` and add an entry:

```python
words['bradycardia'] = 'a slow heartbeat'        # new medical word
sentence_rules.append((r'Hold (.+?) until', r'Do not take \1 until'))   # new sentence pattern
```
Then add a test in `tests/test_simplify.py` and run `pytest`.

## ⚠️ Known limitations
- Rules are tuned to one templated dataset and may miss wording they have never seen.
- Some outputs are grammatically awkward, for example "if changes in your vision gets worse".
- The web app's topic detection is broad: words like "return" or "check" can trigger a topic in unrelated sentences.
- No handling of negation or drug-name checking, and no medical validation by a clinician.
- English only.

## 🗺️ Roadmap
- [ ] Merge the dataset pipeline and the web-app engine into one
- [ ] Evaluate on real, unseen medical text with a proper train/test split
- [ ] Add readability scores (Flesch Reading Ease) and BLEU/SARI metrics
- [ ] Train a small seq2seq or transformer simplifier and compare it with the rules
- [ ] Handle plurals and grammar agreement ("gets" vs "get")
- [ ] Deploy on Hugging Face Spaces
- [ ] Malayalam translation of the simple output

## 🛠️ Tech stack
Python · `re` (regular expressions) · pandas · matplotlib · Gradio · pytest · Google Colab

## 📄 License
Released under the [MIT License](LICENSE).

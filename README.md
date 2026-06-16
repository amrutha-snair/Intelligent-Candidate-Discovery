# Redrob AI Candidate Ranking System

This repository contains my submission for the **Intelligent Candidate Discovery & Ranking Challenge** hosted by Redrob.

The system processes a pool of 100,000 candidates and ranks them against the "Senior AI Engineer — Founding Team" Job Description, outputting the top 100 best-fit candidates.

## Features

* **High Performance**: Processes 100,000 candidates in ~60 seconds on CPU (well under the 5-minute budget). Memory efficient (streams JSONL line by line).
* **Multi-Pillar Scoring Engine**: Evaluates candidates across 6 dimensions: Role Fitness, Skill Depth, Career Trajectory, Experience Band, Behavioral Signals, and Logistics.
* **Smart Honeypot Detection**: Successfully detects and filters out exactly 100 honeypot profiles by looking for subtle impossibilities (e.g. expert skills with 0 months experience, career duration mismatches).
* **Nuanced Text Analysis**: Does not blindly match skills. It analyzes career descriptions for terms indicating real production deployment ("deployed", "at scale", "live") and penalizes keyword stuffing (candidates with >50% irrelevant skills).
* **Fact-Grounded Reasoning**: Automatically generates a 1-2 sentence, specific reasoning string for every candidate in the top 100, referencing their actual title, experience, skills, and engagement metrics.

## Setup and Usage

### Prerequisites
* Python 3.9+
* No external libraries required (uses only standard library)

### Running the Pipeline

You can run the pipeline with a single command. Point `--candidates` to the JSONL data file:

```bash
python rank.py --candidates ./candidates.jsonl --out ./submission.csv
```

**Note:** The script has auto-discovery built in. If you provide a path that doesn't exist (e.g. just `./candidates.jsonl`), it will search the project directory and subdirectories to find the dataset automatically.

### Validating the Output

Use the provided validator script to ensure the generated CSV meets the hackathon specification:

```bash
python validate_submission.py submission.csv
```

## Architecture

The system is built purely in Python using the standard library to guarantee reproduction within the sandboxed environment constraints (No GPUs, no external network calls, ≤16GB RAM, ≤5 mins).

### Core Modules

1. **`rank.py`**: The main entry point. Streams candidates from the JSONL, orchestrates filtering and scoring, ranks the results, triggers reasoning generation, and writes the final CSV.
2. **`scorer.py`**: The 6-pillar scoring engine. Returns a normalized composite score for each candidate.
3. **`filters.py`**: A fast, coarse Stage 1 filter to drop obviously irrelevant candidates (e.g., wrong experience band, zero technical signal) to save compute.
4. **`honeypot.py`**: The detector for the synthetic "trap" candidates. Checks for conflicting data points (like 15 years of claimed experience but only 3 years in career history).
5. **`reasoning.py`**: Generates human-readable justifications tailored to the candidate's exact profile and the specific JD requirements.
6. **`config.py`**: Centralized configuration holding all scoring weights, skill dictionaries (must-have vs irrelevant), title tiers, and thresholds.

## Scoring Methodology

The composite score is a weighted sum of 6 pillars:

1. **Role Fitness (25%)**: Prioritizes tier A titles (e.g., "AI Engineer", "ML Engineer") and analyzes career descriptions for relevant keywords.
2. **Skill Depth (20%)**: Rewards must-have and strong skills while heavily penalizing profiles flooded with irrelevant keywords.
3. **Career Trajectory (15%)**: Favors product-company experience and evidence of shipping code. Penalizes consulting-only backgrounds and frequent "title chasers" (<18 month stints).
4. **Behavioral Signals (20%)**: A heavy modifier requested by the JD. Highly active candidates (recent login, high response rate, open to work) are boosted; dormant profiles are penalized.
5. **Experience Band (10%)**: Scores proximity to the target 5-9 year sweet spot using a gradient falloff.
6. **Logistics (10%)**: Evaluates location fit (Pune/Noida preferred), notice period, and salary expectations.

## Output

The final deliverable is `submission.csv`, containing the 100 best candidates in descending order of composite score, including `candidate_id`, `rank`, `score`, and `reasoning`. Tie-breaks are handled deterministically (candidate_id ascending).

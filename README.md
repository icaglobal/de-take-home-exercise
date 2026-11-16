# Data Engineer Take-Home: PubMed Daily Updates Pipeline

## Overview

You'll build a small, working data pipeline over real PubMed data: ingest a set of daily export files, parse and model them in Postgres, check data quality, and produce analytics tables that would power a dashboard.

We care more about your decisions than about how much code you write. A clean, small pipeline with well-reasoned choices beats a large one.

**You're not expected to finish everything.** The task is deliberately larger than 2 hours. Choose what to build first and go as far as you can. In `FINDINGS.md`, note what you didn't get to and how you'd approach it. We'll talk through those parts on the call.

## Format

- **Take-home:** we suggest spending about 2 hours. Start whenever you like and spend more or less time if you want; just tell us roughly how long you spent in `DECISIONS.md`.
- **Follow-up call (60 minutes):** you walk us through your pipeline, we hand you a new file to process, and you extend the pipeline live. We'll also discuss how you'd run this in production.

## AI tools

Use AI assistants (Claude Code, Codex, Copilot, etc.) if you want to. Just tell us how you used them in `AI_USAGE.md`: which tools, what for, and your prompts or a usage log.

On the follow-up call we'll start from your AI usage and then dig deeper into the code itself, so be ready to explain and change every line you submit. Part of the live change is done with limited or no AI assistance.

No LLM API calls inside the pipeline itself.

## What we provide

- **This repository:** a sample setup if you'd like one: a `docker-compose.yml` that runs Postgres, and an empty Dagster project. Use any orchestrator and any stack you prefer (a plain Python script, Airflow, Prefect, dbt, local emulators, anything else). The only requirement is that your results end up in Postgres. Say what you chose and why in `DECISIONS.md`.
- **Data:** the update files are in this repository under `data/`.

## Getting started (optional)

If you'd like to use the sample setup, Postgres runs with:

```sh
docker compose up -d          # Postgres on localhost:5432, db/user/password: pubmed
```

And the Dagster starter (requires [uv](https://docs.astral.sh/uv/)):

```sh
cd pipeline
uv sync
uv run dagster dev            # Dagster UI on http://localhost:3000
```

Delete or replace any of this if you prefer; just keep the run instructions current.

## The data

The files are gzipped XML exports of PubMed citation records from the U.S. National Library of Medicine (NLM). Each file is one daily update and contains new and revised citation records.

- `data/initial/`: the files that make up your starting load.
- `data/daily/`: the subsequent daily update files.

Files are named in sequence (e.g. `pubmed26n1234.xml.gz`), and the sequence is the order NLM intends them to be applied in. A record can appear again in a later file with revised content.

Documentation for the XML elements: https://dtd.nlm.nih.gov/ncbi/pubmed/doc/out/250101/index.html

Data courtesy of the U.S. National Library of Medicine.

**Deleted citations are out of scope.** Some files contain a `DeleteCitation` element; you may ignore it.

## The task

### 1. Ingest and parse

Load the `data/initial/` files, then apply the `data/daily/` files. For each citation, extract at least:

- PMID
- Article title
- Journal (title, plus an identifier for the journal)
- Publication date
- DOI, where present
- Authors, in order
- MeSH descriptor terms, including whether each is a major topic
- Revision date (`DateRevised`)

You may extract more if it helps you.

### 2. Model

Design a Postgres schema for this data. How you handle one-to-many data (authors, MeSH terms, identifiers) is up to you.

### 3. Make it safe to run

- Applying the files must result in the correct current state of each record.
- Re-running the pipeline, in whole or for a single file, must not create duplicates or incorrect state.

### 4. Data quality

Add checks that run as part of the pipeline. For each check, decide what happens when it fails: stop the run, quarantine the affected records, or warn and continue. Record your reasoning.

Look at the data, not only at the schema. We're interested in what you actually find.

### 5. Analytics tables

Build tables or views (plain SQL or dbt, your choice) that would power a dashboard answering:

1. Number of articles per journal per month, by **publication date**.
2. Top MeSH terms by publication year.
3. For each update file applied: how many records were new and how many were revisions.
4. A data-quality summary per pipeline run: records passed, quarantined and failed, by check.

You don't need to build a dashboard.

## What to submit

By the date we agreed, send us your work as a zip file (leave out the `data/` folder; we have it), or give us access to a private GitHub repository if you prefer (please don't fork this one, since forks of a public repo are public). Include:

- Your pipeline code and SQL/dbt models.
- Instructions to run it from a fresh copy in a few commands.
- `DECISIONS.md`: the choices you made and why. Include the grain of each analytics table and how you defined "publication date."
- `FINDINGS.md`: what you found in the data, including any problems, how your pipeline handles them, and what you'd do with more time.
- `AI_USAGE.md`: prompts or a usage log.

Short and precise is better than long. Bullet points are fine.

## Out of scope

- Deleted citations
- Building a dashboard UI
- Search indexing
- Cloud deployment, CI/CD, or production hardening (we'll discuss these on the call)

## Questions

If something is ambiguous, make a reasonable decision and record it in `DECISIONS.md`.

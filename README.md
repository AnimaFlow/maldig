# Maldig

[MyAnimeList.net] manga and anime trend data.

## Overview

This project collects manga and anime data from [MyAnimeList.net] for the
purpose of tracking current and historical popularity trends. The data is
provided in CSV format.

You can download the latest files here:

| File                    | Description        |
| ----------------------- | ------------------ |
| [`anime.csv`]           | Anime title data   |
| [`anime-ranking.csv`]   | Anime ranking data |
| [`manga.csv`]           | Manga title data   |
| [`manga-ranking.csv`]   | Manga ranking data |

These files are automatically updated every week.

The schema of these records is expressed in the following schema files:

| File             | Description                |
| ---------------- | -------------------------- |
| [`anime.json`]   | Anime title schema         |
| [`manga.json`]   | Manga title schema         |
| [`ranking.json`] | Anime/Manga ranking schema |

## Usage

You can read and work with the data files from this project using Python's standard `csv` module.
The files contain a header row, and each row contains the fields in order. For fields with nested structures (such as genres or authors), the cell value is serialized as a JSON string.

If you would prefer to load and convert these records into dictionaries, a reference implementation is provided below.

### Loading Records as Dictionaries

The reference implementation below reads the CSV file and uses the `csv.DictReader` class to load rows, while parsing cells that contain JSON-serialized lists or dicts.

```py
import csv
import json
from pathlib import Path

def parse_cell(val):
    try:
        return json.loads(val)
    except (json.JSONDecodeError, TypeError):
        return val

with open(Path('data') / 'manga.csv', newline='', encoding='utf-8') as file:
    reader = csv.DictReader(file)
    records = []
    for row in reader:
        record = {k: parse_cell(v) for k, v in row.items()}
        records.append(record)
```

The `records` variable should now contain the records as dictionaries.

## Continuous Integration

This project is set up with automated pipelines to fetch MyAnimeList trends weekly and publish the updated CSV files back to the repository.

### GitHub Actions & Blacksmith CI
Workflows are located in the `.github/workflows/` directory:
- [GitHub Actions Workflow](.github/workflows/update.yml): Triggered on pushes/PRs to the main branches, weekly schedules, or manually (`workflow_dispatch`).
- [Blacksmith CI Workflow](.github/workflows/update-blacksmith.yml): Same as the GitHub Actions workflow but utilizes Blacksmith CI's high-performance runners (`runs-on: blacksmith-2vcpu-ubuntu-2204`).

**Required Secrets / Settings:**
- `MAL_CLIENT_ID`: Your MyAnimeList client ID for API access.
- **Workflow Permissions**: Ensure the repository has Read and Write permissions for workflows (Settings -> Actions -> General -> Workflow permissions).

### GitLab CI
The pipeline is defined in [.gitlab-ci.yml](.gitlab-ci.yml):
- Executes static analysis, updates the datasets, and publishes changes back to GitLab.

**Required Variables:**
- `MAL_CLIENT_ID`: Your MyAnimeList client ID.
- `SSH_PRIVATE_KEY`: Private SSH key with write permissions to the repository (configured as a deploy key).


<!-- links -->

[`anime.json`]: schema/anime.json
[`anime.csv`]: data/anime.csv
[`anime-ranking.csv`]: data/anime-ranking.csv
[`manga.json`]: schema/manga.json
[`manga.csv`]: data/manga.csv
[`manga-ranking.csv`]: data/manga-ranking.csv
[`ranking.json`]: schema/ranking.json

[MyAnimeList.net]: https://myanimelist.net

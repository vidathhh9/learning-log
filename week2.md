# Week 2: Python, virtual environments and pandas

## What I built
A CSV cleaner (`week2-csv-cleaner` on GitHub) that loads a messy list of URLs, cleans it, adds simple features, and saves the result.

## Python skills
- Variables, lists, dictionaries, functions and loops
- Running scripts with `python3`
- Virtual environments (`python3 -m venv .venv`) so each project has its own packages
- Installing packages with `pip`, and recording them with `pip freeze > requirements.txt`

## pandas skills
- Loading a CSV into a DataFrame with `read_csv`
- Inspecting data: `head`, `info`, `describe`, `value_counts`
- Cleaning: `drop_duplicates`, `dropna`, `fillna`, plus tidying text with `str.strip().str.lower()`
- Creating features: `url_length`, `uses_https`, `dot_count`
- Summarising with `groupby` and saving with `to_csv`

## Result
The script cleaned 10 rows down to 8 by removing a duplicate and a row with no URL. It labelled the row with a missing label as "unknown". In this small dataset, phishing URLs averaged about 32.5 characters and safe ones about 21.7.

## Project hygiene
- `requirements.txt` lists exact package versions so the project can be rebuilt anywhere.
- `.gitignore` keeps the `.venv` folder out of GitHub, because it can be rebuilt and is specific to my Mac.
- I tested it by deleting `.venv`, recreating it, and running `pip install -r requirements.txt` followed by the script. That proved the project rebuilds from scratch.

## What I learned
- A model can't read a website the way I do, so I turn each URL into numbers called features.
- Commit the recipe (`requirements.txt`), not the cooked meal (`.venv`).
- Real data is messy, and cleaning it is a big part of the work.

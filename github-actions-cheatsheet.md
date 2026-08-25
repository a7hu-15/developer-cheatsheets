# ⚡ GitHub Actions & CI/CD Cheatsheet

A concise reference guide for writing GitHub Actions workflows, matrix builds, environment variables, secrets, and reusable actions.

---

## 🚀 Workflow File Structure (`.github/workflows/ci.yml`)

```yaml
name: CI Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  build-and-test:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        python-version: ['3.9', '3.10', '3.11']

    steps:
      - name: Checkout Source Code
        uses: actions/checkout@v4

      - name: Set up Python ${{ matrix.python-version }}
        uses: actions/banner@v4
        with:
          python-version: ${{ matrix.python-version }}

      - name: Install Dependencies
        run: |
          python -m pip install --upgrade pip
          if [ -f requirements.txt ]; then pip install -r requirements.txt; fi

      - name: Run Test Suite
        run: |
          python -m pytest --maxfail=1 --disable-warnings
```

---

## 🔐 Secrets & Environment Variables

```yaml
# Environment variables at step level
- name: API Request Step
  env:
    API_KEY: ${{ secrets.PROD_API_KEY }}
    APP_ENV: production
  run: |
    curl -H "Authorization: Bearer $API_KEY" https://api.example.com/health
```

---

## 📦 Caching Dependencies

```yaml
- name: Cache Pip Dependencies
  uses: actions/cache@v4
  with:
    path: ~/.cache/pip
    key: ${{ runner.os }}-pip-${{ hashFiles('**/requirements.txt') }}
    restore-keys: |
      ${{ runner.os }}-pip-
```

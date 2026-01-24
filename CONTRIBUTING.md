# Contributing to PubMed Review Automation

Thank you for your interest in contributing! 🎉

## Quick Start

1. **Fork and Clone**
   ```bash
   git clone https://github.com/YOUR_USERNAME/pubmed_review.git
   cd pubmed_review
   ```

2. **Install Dependencies**
   ```bash
   pip install -r requirements.txt
   ```

3. **Run Tests**
   ```bash
   pytest tests/ -v
   ```

## Development Workflow

### 1. Create a Branch
```bash
git checkout -b feature/your-feature-name
```

### 2. Make Changes

- Keep functions small and focused
- Add docstrings to all functions
- Use type hints
- Follow existing code style

### 3. Write Tests

All new features must have tests:

```python
# tests/test_main.py
def test_your_new_feature():
    result = your_function(input)
    assert result == expected_output
```

### 4. Run Tests Locally

```bash
# Run all tests
pytest tests/ -v

# Run with coverage
pytest tests/ --cov=pubmed_review --cov-report=html

# Run specific test
pytest tests/test_main.py::TestClassName::test_method -v
```

### 5. Test in Dry-Run Mode

```bash
# Test without making API calls
DRY_RUN=true python -m pubmed_review.main
```

### 6. Commit and Push

```bash
git add .
git commit -m "Add feature: description of your changes"
git push origin feature/your-feature-name
```

### 7. Create Pull Request

- Go to GitHub and create a PR
- Describe what you changed and why
- Link any related issues

## Code Style

### Python Style

- Follow PEP 8
- Use meaningful variable names
- Keep lines under 100 characters
- Use f-strings for formatting

### Good Example

```python
def fetch_articles(pmids: list[str]) -> dict[str, Article]:
    """Fetch article data for given PMIDs.

    Args:
        pmids: List of PubMed IDs to fetch

    Returns:
        Dictionary mapping PMID to Article object
    """
    articles = {}
    for pmid in pmids:
        articles[pmid] = fetch_single_article(pmid)
    return articles
```

### Bad Example

```python
def f(x):  # No docstring, unclear name
    d = {}
    for i in x:
        d[i] = g(i)  # What is g?
    return d
```

## Testing Guidelines

### What to Test

✅ **Test:**
- Core business logic
- Data transformations
- Edge cases (empty lists, None values)
- Error handling

❌ **Don't Test:**
- External API calls (mock them instead)
- Simple getters/setters
- Framework code

### Example Test

```python
from unittest.mock import Mock, patch

def test_process_article_with_high_if():
    """Test that High IF articles skip novelty check."""
    article = Article(pmid="123", journal="Nature", ...)
    client = Mock()
    config = {...}
    high_if_list = ["Nature"]

    result = process_article(article, client, config, high_if_list)

    # Should not call novelty LLM for High IF
    client.chat.completions.create.assert_called_once()  # Only summary
    assert result.high_if is True
```

## Common Tasks

### Add a New Feature

1. Write test first (TDD)
2. Implement feature
3. Run tests
4. Update README if needed

### Fix a Bug

1. Write a test that reproduces the bug
2. Fix the bug
3. Verify test passes
4. Add to CHANGELOG (if we had one)

### Update Documentation

- Update README.md for user-facing changes
- Update docstrings for code changes
- Add examples if helpful

## Project Structure

```
pubmed_review/
├── pubmed_review/
│   ├── __init__.py
│   └── main.py          # All core logic (may be split later)
├── tests/
│   ├── __init__.py
│   └── test_main.py     # Unit tests
├── .github/workflows/
│   └── pubmed_review.yml  # CI/CD
├── config.yaml          # User configuration
├── requirements.txt     # Dependencies
└── README.md           # Documentation
```

## Need Help?

- 📖 Check the [README](README.md) for usage info
- 🐛 Open an [Issue](https://github.com/radssk/pubmed_review/issues) for bugs
- 💬 Start a [Discussion](https://github.com/radssk/pubmed_review/discussions) for questions

## Code of Conduct

- Be respectful and inclusive
- Provide constructive feedback
- Help others learn and grow

Thank you for contributing! 🙌

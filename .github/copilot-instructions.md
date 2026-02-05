# AI Coding Agent Instructions for CPSC 404 Questions Repository

## Project Overview

This is a **CPSC 404 Senior Seminar course repository** that manages weekly student question submissions for a Trinity College course on collaboration and open source. The project serves dual purposes:

1. **Course Management**: Collect and grade weekly discussion questions via GitHub pull requests
2. **Educational Tool**: Interactive question picker (`index.html`/`picker.js`) to randomly select questions for class discussion

**Core Pattern**: Students fork the repo, create branches, add markdown question files, and submit PRs—learning real Git workflows while engaging with course readings.

## Architecture & Data Flow

### Question Submission Flow
- **Student Action**: Create `weekXX-{topic}/questions/{lastname}-{firstname}.md` files
- **PR Submission**: Students open PRs to main repository
- **Instructor Review**: `@kousen` (CODEOWNERS) auto-requested as reviewer
- **Merge**: Accepted PRs merge questions into main branch

### Question Picker Tool (Discovery Mechanism)
The picker (`picker.js`) powers `index.html` to enable discussion-neutral question selection:

1. **GitHub API Fetch**: Reads current main branch questions from `/contents/{week}/questions/` endpoint
2. **Filtering**: Extracts only `.md` files via `isQuestionFile()`, filters empty submissions via `isNonEmpty()`
3. **Metadata Parsing**: Extracts author from multiple formats (e.g., `**Student:** Name`, plain `Student: Name`, or falls back to filename)
4. **Cleanup**: `cleanBody()` strips headers (h1/h2) and metadata lines to show only question content
5. **Pool Selection**: `createPool()` implements pick-without-replacement with back/forward navigation history

This design lets instructors discover high-quality questions by randomization rather than first-arrival bias.

## Code Patterns & Conventions

### Markdown Question Format
Every student question follows this exact structure (from [week02-chapter01/questions](week02-chapter01/questions/)):
```markdown
# Week X Question - [Chapter Title]

**Student:** First Last
**Date:** January 28, 2026

## Question
[Substantive question text - typically 1-3 sentences]

## Context (optional)
[Brief context or references to chapter concepts]
```

**Key Convention**: Metadata must be on separate lines; `cleanBody()` regex specifically removes `**Student:**` and `**Date:**` patterns plus `#` and `##` headings.

### File Naming
- **Format**: `lastname-firstname.md` (kebab-case, lowercase)
- **Location**: Always in `weekXX-{topic}/questions/` subdirectory
- **Constraint**: One file per student per week (enforced by PR review)

### Grading Rubric (0-3 Scale)
- **3 (Excellent)**: Insightful synthesis, invites debate, challenges assumptions
- **2 (Good)**: Demonstrates understanding, application-focused
- **1 (Adequate)**: Basic comprehension
- **0 (Insufficient)**: Missing or no evidence of reading

### Week Structure
Pattern: `weekXX-{topic}/` with optional README explaining scope:
- `week02-chapter01`: Single chapter reading (HYBHY Ch 1)
- `week06-dual-reading`: Two readings (HYBHY Ch 4 + ProducingOSS Ch 8)
- `week07-chapter05-async`: Async week (same submission + Moodle analysis)

## Critical Developer Workflows

### Testing the Picker Logic
Run tests (Node.js required):
```bash
node tests/picker.test.js
```

**Test Structure** (`tests/picker.test.js`):
- Unit tests for each pure function: `cleanBody()`, `extractAuthor()`, `weekLabel()`, `createPool()`, etc.
- 422-line test suite covers metadata extraction edge cases (multiple author formats, special chars, empty submissions)
- No external dependencies; tests use only Node.js `assert` module

### Adding a New Week
1. Create folder: `weekXX-{topic}/`
2. Add `weekXX-{topic}/README.md` with reading/due date/key concepts
3. Create `weekXX-{topic}/questions/` subdirectory
4. Update `picker.js` `WEEK_LABELS` object with friendly label for the new week
5. No code changes needed in `picker.js` otherwise—it auto-discovers questions via GitHub API

### Debugging Question Display
The picker uses GitHub raw API (`https://api.github.com/repos/kousen/cpsc404-questions-spring2026/contents`):
- **Check Live API**: `curl "https://api.github.com/repos/kousen/cpsc404-questions-spring2026/contents/week02-chapter01/questions"`
- **Debug `cleanBody()`**: Verify h1/h2 headers and metadata patterns match the regex filters
- **Author Extraction Issues**: Test `extractAuthor()` against submitted filenames and markdown metadata

## Deployment & Hosting

- **Static Site**: `index.html` + `picker.js` + `picker.css` served directly from GitHub (no build step)
- **No Backend**: All GitHub API calls from browser (CORS-enabled public API)
- **Browser Environment**: Picker detects module exports to work in both Node.js (tests) and browser (index.html)

## Integration Points & External Dependencies

- **GitHub API**: Public read-only access to repo contents (no auth needed for public repos)
- **Books**: HYBHY (*Help Your Boss Help You*) and ProducingOSS (*Producing Open Source Software*)
- **Learning Context**: Course mirrors ProducingOSS patterns (forking, branching, PRs, code review)

## Common Patterns to Preserve

1. **Flexible Metadata Parsing**: `extractAuthor()` handles bold (`**Student:**`), plain (`Student:`), and Chinese colons (`：`). **Preserve this tolerance**—students submit inconsistently.

2. **Pure Functions**: `cleanBody()`, `extractAuthor()`, `weekLabel()`, `isQuestionFile()`, `isNonEmpty()` are all stateless. **Keep functions pure** for testability.

3. **History-Based Navigation**: `createPool()` tracks pick history with back/forward pointers. **Preserve the history model**—it enables "undo" in the picker UI.

4. **Graceful Fallbacks**: If metadata can't be parsed, fallback to filename; if label not in `WEEK_LABELS`, auto-generate from folder name. **Maintain this robustness**.

## Questions or Additions?

When adding features or fixing bugs:
- Update both `picker.js` and `tests/picker.test.js` (maintain test parity)
- Preserve GitHub API interface—no backend dependency
- Keep question format simple (markdown metadata + body)
- Run `node tests/picker.test.js` before committing

---

*Last updated: February 2026*

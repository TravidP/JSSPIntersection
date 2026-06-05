# Priority-Aware JSSP AIM Paper Workspace

This folder contains an IEEE conference-style draft paper for the research direction:

**Priority-Aware Constrained Job-Shop Scheduling for Autonomous Intersection Management**

## Build

From this folder, run:

```powershell
latexmk -pdf -interaction=nonstopmode main.tex
```

The main PDF output is `main.pdf`.

## Files

- `main.tex`: paper entrypoint, packages, abstract, title block, and section includes.
- `sections/`: modular manuscript sections.
- `references.bib`: verified central references only.
- `citation_audit.md`: notes on reference cleanup and technical-claim checks.

## Draft Notes

- Author, affiliation, and email are placeholders.
- The paper is written as a formulation paper, not as a completed experimental-results paper.
- Red `TODO` placeholders mark details that need implementation data, calibrated parameters, or supervisor choices.


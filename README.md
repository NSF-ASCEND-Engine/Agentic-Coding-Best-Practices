# Agentic Coding Best Practices

A [Quarto](https://quarto.org) book on developing software with coding agents (Claude Code,
OpenAI Codex, and similar tools).

**Published site:** https://nsf-ascend-engine.github.io/Agentic-Coding-Best-Practices/

## Building locally

Requires Quarto ≥ 1.7 (<https://quarto.org/docs/get-started/>).

```bash
quarto preview     # live-reloading local preview
quarto render      # build static site into _book/
```

## Deployment

Every push to `main` triggers `.github/workflows/publish.yml`, which renders the book and
pushes the HTML to the `gh-pages` branch. GitHub Pages serves that branch.

## Source notes

`FINDINGS.md` holds raw notes that feed the chapters. It is not rendered.

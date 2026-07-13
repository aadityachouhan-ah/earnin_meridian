# earnin_meridian

This is EarnIn's fork of [google/meridian](https://github.com/google/meridian) (`upstream` remote), with LTV/CPIK customizations layered directly into upstream files (no separately-named "custom" module). Currently on `ltv_changes_from_scratch`, version `1.6.0`.

**What changed vs. upstream, authoritatively**: `CHANGELOG.md`'s `[Unreleased]` section. Read that first, don't try to re-derive a diff from the source — it's already maintained and accurate (`media_revenue_per_kpi`, `spend_response_curve_per_channel`, `optimize_marginal_cac_to_rpc`).

The customizations live in:
- `meridian/analysis/optimizer.py` — `BudgetOptimizer.optimize_marginal_cac_to_rpc`, `MarginalCacOptimizationResults`
- `meridian/analysis/analyzer.py` — `Analyzer.spend_response_curve_per_channel`, `Analyzer._inverse_outcome` (channel-specific rpc lookup)
- `meridian/data/input_data.py`, `input_data_builder.py`, `data_frame_input_data_builder.py`, `nd_array_input_data_builder.py`, `load.py` — channel-level LTV as a new `InputData.media_revenue_per_kpi` input
- `meridian/model/context.py`, `meridian/constants.py` — non-revenue-KPI handling

## Where this is consumed

EarnIn's MMM pipeline (`ml-pipelines/marketing/MMM/`, see its `CLAUDE.md`) consumes this via a manually built-and-uploaded wheel on Databricks — **not** an editable/local install. Changes here have zero runtime effect on that pipeline until a wheel is rebuilt and re-uploaded. See `ml-pipelines/marketing/MMM/CLAUDE.md` for the actual build/upload/version-bump workflow before assuming a change is "live" anywhere.

Applied experiments, model-insights notebooks, and dashboards that load trained models via mlflow and `import meridian.analysis.*` directly live in `mmm_experiments` — see its `CLAUDE.md` (`~/Documents/work/EarnIn/Github/mmm_experiments/CLAUDE.md`).

## Local docs vault (Meridian's public methodology docs)

There's a local Obsidian vault mirroring `developers.google.com/meridian/docs` at `~/Documents/Meridian_docs/Meridian-Vault/` — 74 markdown files, git-tracked, refreshed monthly (`~/Documents/Meridian_docs/run.sh`; check `git log` in the vault if something seems stale).

**Prefer it over a live fetch** to `developers.google.com` for methodology/API questions — it's faster, works offline, and this network's corporate TLS inspection makes live fetches to that domain unreliable without a CA-bundle workaround.

**Resolving a doc URL cited in code to its local file**: this codebase already cites specific doc pages inline, e.g. `context.py:821` and `input_data.py:600,722` link to `.../advanced-modeling/unknown-revenue-kpi-custom#...` and `.../advanced-modeling/unknown-revenue-kpi-default#...`. Every vault file's frontmatter has a `source: https://developers.google.com/meridian/docs/<path>` line, so resolve any such citation with:
```
grep -rl "^source:.*<path-fragment>" ~/Documents/Meridian_docs/Meridian-Vault/
```
(Use the frontmatter-anchored form, not a bare substring grep — other pages link *to* the same doc with a `#anchor` suffix that doesn't get wikilinked, so a loose grep returns several false positives.) Example: `grep -rl "^source:.*unknown-revenue-kpi-custom" ~/Documents/Meridian_docs/Meridian-Vault/` → `Applied-Modeling/Priors/Custom-Priors/Set custom priors when outcome is not revenue.md`.

**Vault folder taxonomy**: `Basics/`, `Use-the-Library/` (`Load-the-data/`, `Configure-and-run/`, `Analyze-results/`, `Customize-optimization/`), `Pre-modeling/`, `Applied-Modeling/` (`Priors/`, `Priors/Treatment-Prior-Types/`, `Priors/Default-Priors/`, `Priors/Custom-Priors/`, `The-Meridian-Model/`), `Post-modeling/` (`Assess-model-health/`), `Causal-Inference-Theory/`. `MOC - Meridian Guides.md` is the full index.

**If you're touching X, start here**:
| Working on | Vault pages |
|---|---|
| `optimizer.py` (CPIK-constrained optimization) | `Use-the-Library/Customize-optimization/*`, `Post-modeling/Optimizing with Reach and Frequency.md`, `Post-modeling/Optimizing without Reach and Frequency.md`, `Post-modeling/Scenario planning and future budget optimization.md` |
| `analyzer.py` (ROI/mROI, LTV-aware analysis) | `Post-modeling/Incremental Outcome ROI mROI  Response Curves.md`, `Applied-Modeling/Priors/Treatment-Prior-Types/ROI mROI and Contribution parameterizations.md` |
| `input_data*.py`, `load.py` (channel-level LTV as input) | `Use-the-Library/Load-the-data/*`, `Applied-Modeling/The-Meridian-Model/Input data.md` |
| `context.py`, `constants.py` (non-revenue-KPI handling) | `Applied-Modeling/Priors/Default-Priors/When the KPI is not revenue default priors.md`, `Applied-Modeling/Priors/Custom-Priors/Set custom priors when outcome is not revenue.md`, `Causal-Inference-Theory/` |

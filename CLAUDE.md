# CLAUDE.md

Guidance for Claude Code when working in this repo.

## What this project is

A minimal **Delta Sharing client**. It reads data that was shared with us through
the `jnj-butterfly-share` Delta Share (open / token-based recipient
`bobertx3_local`). The primary asset is the `ai_agent.customers` table.

This is a *consumer* of shared data — there is no Spark cluster or Databricks
workspace dependency on our side. Everything runs locally against the Delta
Sharing REST endpoint using the `delta-sharing` Python library.

## Key facts

- **Connection model:** open sharing. A credential file `config.share` (JSON with
  `endpoint` + `bearerToken`) authenticates the client. It is downloaded once from
  the recipient's activation link in Databricks.
- **Table addressing:** `delta-sharing` uses `<profile>#<share>.<schema>.<table>`.
  Our target is `config.share#jnj-butterfly-share.ai_agent.customers`.
- **Entry point:** `delta_sharing_client.ipynb` (kernel: "Python (jnj-deltashare)").
- **Dependencies:** installed via `%pip` in the notebook (`delta-sharing`, `pandas`, `pyarrow`). Optional `.venv/` for local Jupyter.

## Conventions / guardrails

- **Never commit credentials.** `*.share` and `config.share` are git-ignored. Do
  not print the bearer token or paste the file contents into code, logs, or
  commits. If asked to handle credentials, keep them in `config.share` only.
- `config.share` is **not** in the repo — it must be supplied by the user. Code
  should fail with a clear message if it's missing, not fabricate a path.
- Prefer the pandas connector (`load_as_pandas`) for small reads; mention the
  Spark connector (`load_as_spark`) only when scale warrants it.
- Use the existing `.venv` for running anything: `./.venv/bin/python`.

## Common tasks

- **Run the client:** `./.venv/bin/jupyter notebook delta_sharing_client.ipynb`
- **List shared tables:** `SharingClient("config.share").list_all_tables()`
- **Add a dependency:** add the package to the notebook `%pip install` cell.

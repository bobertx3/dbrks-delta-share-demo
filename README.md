# jnj-deltashare

A little [Delta Sharing](https://delta.io/sharing/) client that reads data shared
with us from the **`franks_cdl`** share on the
[e2-demo-field-eng](https://e2-demo-field-eng.cloud.databricks.com/explore/sharing/shares/franks_cdl?o=1444828305810485)
Databricks workspace.

Delta Sharing / Open Sharing is an open protocol for secure data sharing. We connect as an
**open (token-based) recipient**: Databricks gives the recipient an *activation
link*, which downloads a credential file (`config.share`) containing the sharing
server endpoint and a bearer token. The client reads that file — no Databricks
account required on our side.

## Source share

| | |
|---|---|
| **Workspace** | [e2-demo-field-eng](https://e2-demo-field-eng.cloud.databricks.com) |
| **Share** | [`franks_cdl`](https://e2-demo-field-eng.cloud.databricks.com/explore/sharing/shares/franks_cdl?o=1444828305810485) |
| **Tables** | `ai_agent.customers`, `ai_agent.billing` (and any others added to the share) |

## What's in here

| File | Purpose |
|------|---------|
| `delta_sharing_client.ipynb` | The notebook client: installs deps, connects, lists shared tables, loads the first available table into pandas. |
| `config.share` | **Your credential file — not committed.** You must add this yourself (see below). |
| `.gitignore` | Keeps `*.share` and the venv out of git. |

## Setup

### 1. Get the credential file

In the [e2-demo-field-eng workspace](https://e2-demo-field-eng.cloud.databricks.com),
open Catalog Explorer → **Recipients** → `bobertx3_local`:

1. Under **Token management**, copy/open the **Activation link** in a browser.
2. Click **Download credential file** — you'll get `config.share`.
3. Save it in this folder as `config.share`.

> ⚠️ The activation link is **single-use**, and the token expires. Treat
> `config.share` like a password — it's git-ignored on purpose. If it's lost or
> compromised, rotate the recipient's token in Databricks and re-download.

### 2. Run the notebook

Open `delta_sharing_client.ipynb` in Jupyter or Cursor and **Run All**. The first
code cell installs `delta-sharing`, `pandas`, and `pyarrow` into the active kernel.

Optional: use a local venv and Jupyter:

```bash
python3 -m venv .venv
./.venv/bin/python -m pip install jupyter ipykernel delta-sharing pandas pyarrow
./.venv/bin/jupyter notebook delta_sharing_client.ipynb
```

## How reading a table works

The credential file is referenced together with the table path using the format
`<profile>#<share>.<schema>.<table>`. Share and table names come from
`list_all_tables()` — nothing is hardcoded:

```python
import delta_sharing

profile_file = "config.share"
client = delta_sharing.SharingClient(profile_file)

tables = client.list_all_tables()
for t in tables:
    print(f"{t.share}.{t.schema}.{t.name}")

t = tables[0]
table_url = f"{profile_file}#{t.share}.{t.schema}.{t.name}"
df = delta_sharing.load_as_pandas(table_url)
print(df.head())
```

Useful options:

- `delta_sharing.load_as_pandas(table_url, limit=100)` — peek at the first N rows.
- `delta_sharing.load_as_pandas(table_url, version=N)` — time travel (history must be enabled on the shared table).
- `delta_sharing.SharingClient(profile_file).list_all_tables()` — discover everything shared with this recipient.
- `delta_sharing.load_as_spark(table_url)` — read as a Spark DataFrame (requires PySpark + the `delta-sharing-spark` package).

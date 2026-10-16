<p align="center">
  <img src="assets/icon.png" width="88" alt="Datapia">
</p>

<h1 align="center">Datapia</h1>

<p align="center"><b>Your assistant for data routines.</b><br>
Describe the result in plain English. Datapia writes the code, checks it on your file, and reruns it on next month's file with one click.</p>

<p align="center">
  <a href="https://studio.datapia.app"><b>Open in browser</b></a> ·
  <a href="https://github.com/getdatapia/datapia/releases/latest/download/Datapia-Setup.exe"><b>Download for Windows</b></a> ·
  <a href="https://datapia.app">datapia.app</a>
</p>

---

This repository hosts the **Windows releases** of Datapia and its technical overview. The product
site is [datapia.app](https://datapia.app); the app itself runs at
[studio.datapia.app](https://studio.datapia.app) or as a Windows desktop app (below).

## What it's for

Monthly bank statements, sales exports, timesheets, stock lists: files that arrive every month in
the same shape and need the same cleanup, the same totals and the same chart. In Datapia you build
that routine once, as a chain of small steps. Next month you drop in the new file and the whole
chain reruns. No AI call, no credits, the same result every time.

```
bank-statement-oct.csv
  │
  ├─ 01  Add spending category          ← "Add a Category column from the merchant name"
  ├─ 02  Monthly spending by category   ← "Total per category for each month, plus a bar chart"
  │
  └─ Artifacts: table · chart (PNG) · Excel
```

Every new account starts with this pipeline as a demo, so you can see the whole idea before
bringing your own file.

## How it works

1. **Load a file.** CSV / TSV / TXT, Excel (`.xlsx`, `.xls`) or JSON. The loader detects `;` vs `,`,
   decimal commas (`-76,20`) and Windows-1252 text, which are typical for European bank exports. A
   loader template also finds the table inside a report, skipping title rows above the header and
   totals or notes below it.
2. **Describe a step.** For example, "Remove duplicate transactions" or "Total spending per
   category". Datapia sends the AI a short **data profile**, never the file, and gets back a
   Python function `transform(df) -> df`.
3. **It's checked before you see it.** The code passes an allowlist gate, then runs on your **whole
   file**, locally. If it fails, the error goes back to the AI and the step is regenerated (up to
   4 attempts). You see the row counts in and out, the result table and any artifacts.
4. **Rerun.** A new file under existing steps reruns the whole chain locally: free, offline-capable
   and deterministic.

You can always see and keep the code. Steps are plain pandas. A pipeline exports as a small zip
(the loader plus one `.py` per step, no data) and imports on another machine.

## Your data stays on your computer

| What | Where it goes |
|---|---|
| Your file | **Nowhere.** Read and transformed on your machine. In the browser version, inside the tab (WebAssembly). |
| Results, charts, exports | Your machine only. |
| Data profile (for AI steps) | To the AI model, via Datapia's server: column names and types, basic stats, a few example values. **Never whole rows.** |
| Usage statistics | Counts and outcomes, no file contents or names. No cookies, no IP storage. EU-hosted (PostHog, Sentry). |

**Early access (Earlybird plan).** Earlybird is free. In return, for accounts on this plan we keep a
log of generation requests (the description, the data profile and the generated code) to improve
generations. It never contains your file. Other plans keep only usage statistics. Details are in
the [Privacy Policy](https://datapia.app/#privacy).

## Two ways to run it

|  | Browser | Windows app |
|---|---|---|
| Install | Nothing. Open [studio.datapia.app](https://studio.datapia.app) | [Installer](https://github.com/getdatapia/datapia/releases/latest/download/Datapia-Setup.exe) or [zip](https://github.com/getdatapia/datapia/releases/latest/download/Datapia-Windows.zip) |
| Engine | Python (pandas, matplotlib) on Pyodide / WebAssembly, in a Web Worker | Bundled Python engine (no Python install needed) |
| Pipelines stored | In this browser (IndexedDB) | In your user profile |
| Offline | Yes, after the first load (AI steps need a connection) | Yes (AI steps need a connection) |
| Requirements | Chrome, Edge or Firefox | Windows 10 or 11, 64-bit, 4 GB RAM (8 GB recommended), ~400 MB disk |

Both run the **same engine core**. The browser version is not a cut-down copy.

### Installing on Windows

1. Download **`Datapia-Setup.exe`** from the [latest release](https://github.com/getdatapia/datapia/releases/latest).
2. The installer isn't code-signed yet, so Windows SmartScreen may say *"Windows protected your
   PC"*. Click **More info → Run anyway**.
3. Sign in with your email. You'll get a 6-digit code; there are no passwords.

No admin rights needed. The app checks its version at start and tells you when an update is
required. Prefer not to install? Use **`Datapia-Windows.zip`**: unpack it and run `Datapia.exe`.

## Under the hood

For the technically curious:

- **Step contract.** Every step is `transform(df) -> df` on a pandas DataFrame. `pd`, `np` and `plt`
  are pre-injected. Anything a step writes to its working directory (PNG, HTML, XLSX, CSV) becomes
  an artifact.
- **Transport between steps is Parquet.** Each column has one consistent type, so a mixed-type
  column is a clear error that names the column, not a silent corruption.
- **Allowlist gate.** Step code is parsed (AST) before it runs: imports are limited to data and
  stdlib modules, and `eval` / `exec` / `__import__` and similar are refused.
- **Isolated execution.**
  - Desktop: a warm worker process with pandas pre-loaded; it is killed and restarted on timeout.
  - Step code runs with a stripped environment, so it can't read API keys or host secrets.
  - Browser: Pyodide in a Web Worker.
- **Closed-loop generation.** Generate → gate → run on the full data → feed the traceback back →
  regenerate. "Done" means it ran on your data.
- **Schema review.** If a step changes the columns that later steps depend on, you're asked before
  it's applied.
- **Models.**
  - Hosted tiers: *Swift* and *Balanced* on all plans, *Expert* on Pro.
  - The server picks the concrete model.
  - Credits are weighted by each model's real cost.
- **Templates (no AI, no credits):** cleanup and drop duplicates · tidy text and column names ·
  numbers stored as text (`4.123,80` → 4123.8) · parse dates · fill empty cells · remove incomplete
  rows · monthly totals · totals by category with a chart · top 10 rows · group by first column ·
  sort · missing-values report · describe data · save as Excel.

## Plans

| | Earlybird *(early access)* | Beginner | Pro |
|---|---|---|---|
| Price | **Free** | $9 / mo | $29 / mo |
| Credits / month | 100 | 500 | 3 000 |
| AI pipelines | 10 | 10 | Unlimited |
| Reruns | Unlimited, free | Unlimited, free | Unlimited, free |
| Models | Swift, Balanced | Swift, Balanced | + Expert |
| Code editing, export | View | View | Edit, export |

Earlybird is open for a limited time, and accounts that join keep it after early access closes.
Credits are only spent on AI generation; running and rerunning pipelines never costs anything.

## Releases

Each [release](https://github.com/getdatapia/datapia/releases) contains:

- `Datapia-Setup.exe`: Windows installer (per-user, no admin).
- `Datapia-Windows.zip`: the same app, no installation.

The download links above always point to the latest release. Release notes list what changed.

## Feedback

Found a bad generation, a file that won't load, or a step you wish existed?

- [Open an issue](https://github.com/getdatapia/datapia/issues) (please don't attach private data).
- Email [hello@datapia.app](mailto:hello@datapia.app); [support@datapia.app](mailto:support@datapia.app) for problems.

---

<p align="center"><sub>© 2026 Datapia · <a href="https://datapia.app/#privacy">Privacy</a> · <a href="https://datapia.app/#terms">Terms</a> · <a href="https://datapia.app/#security">Security</a></sub></p>

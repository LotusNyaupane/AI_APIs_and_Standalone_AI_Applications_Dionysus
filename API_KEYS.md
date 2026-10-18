# API Key Setup Guide (llm_sources.py)

## How the tool finds your credentials

`llm_sources.py` resolves settings in this order (command line beats environment):

| Setting | 1st choice (flag) | 2nd choice (env var) | Default |
|---------|-------------------|----------------------|---------|
| API key | `--api-key` | `OPENAI_API_KEY` | none (warning shown) |
| Server  | `--base-url` | `OPENAI_BASE_URL` | `https://api.openai.com/v1` |
| Model   | `--model` | `LLM_SOURCES_MODEL` | `gpt-4o-mini` |

## Step 1 — get a key from a provider

See the free options below. Copy the key (it usually starts with `sk-`, `gsk_`, or similar).

## Step 2 — set the environment variable (pick your OS)

**Linux / macOS (permanent — recommended):**
```bash
echo 'export OPENAI_API_KEY="PASTE-YOUR-KEY-HERE"' >> ~/.bashrc   # zsh users: >> ~/.zshrc
source ~/.bashrc
```

**Windows PowerShell (permanent):**
```powershell
setx OPENAI_API_KEY "PASTE-YOUR-KEY-HERE"
# close this terminal and open a NEW one — setx applies to new sessions only
```

**Windows CMD (current session only):**
```cmd
set OPENAI_API_KEY=PASTE-YOUR-KEY-HERE
```

**Verify:**
```bash
echo $OPENAI_API_KEY        # Linux/macOS/Git-Bash
echo $env:OPENAI_API_KEY    # PowerShell
echo %OPENAI_API_KEY%       # CMD
```

## Step 3 — run the tool

```bash
# OpenAI (default endpoint, uses OPENAI_API_KEY):
python llm_sources.py samples/quarterly_report.pdf -v

# Any other OpenAI-compatible provider: pass --base-url and --model, e.g.
python llm_sources.py samples/quarterly_report.pdf -v \
    --model gemini-3.8-flash \
    --base-url https://generativelanguage.googleapis.com/v1beta/openai
```

## Step 4 — test with zero cost first (no key needed)

The package includes a fake OpenAI-compatible server:
```bash
python tests/mock_openai_server.py 8000        # terminal 1
python llm_sources.py samples/notes.txt --base-url http://127.0.0.1:8000/v1   # terminal 2
```

---

## Free API key options (verified October 2026)

| Provider | Get key at | Free offer | `--base-url` | Example `--model` |
|----------|-----------|------------|--------------|-------------------|
| **Google Gemini** | aistudio.google.com/apikey | Free tier, no credit card; flash models free | `https://generativelanguage.googleapis.com/v1beta/openai` | `gemini-3.8-flash`, `gemini-3.5-flash-lite` |
| **Groq** | console.groq.com/keys | Free forever tier, no credit card, very fast; rate-limited | `https://api.groq.com/openai/v1` | `llama-3.1-8b-instant`, `llama-3.3-70b-versatile`, `openai/gpt-oss-20b` |
| **OpenRouter** | openrouter.ai/keys | ~17 models tagged `(free)`, or the `openrouter/free` auto-router | `https://openrouter.ai/api/v1` | `openrouter/free` |
| **Ollama (local)** | ollama.com | 100% free, runs on your own machine, no key at all | `http://localhost:11434/v1` | `llama3.2`, `qwen2.5` |
| **GitHub Models** | github.com → Settings → Models | Free with a GitHub account, rate-limited | `https://models.github.ai` | `openai/gpt-4o-mini` |

### Recommended for this assignment

```bash
# Groq — fastest sign-up, no card:
export OPENAI_API_KEY="gsk_..."
python llm_sources.py samples/quarterly_report.pdf samples/notes.txt \
    --base-url https://api.groq.com/openai/v1 --model llama-3.1-8b-instant -v

# Gemini — best free quality:
export OPENAI_API_KEY="AIza..."
python llm_sources.py samples/quarterly_report.pdf \
    --base-url https://generativelanguage.googleapis.com/v1beta/openai \
    --model gemini-3.8-flash -v

# Ollama — no key, fully offline:
ollama pull llama3.2
python llm_sources.py samples/sales.csv --base-url http://localhost:11434/v1 --model llama3.2
```

### Free-tier gotchas (and how this tool already helps)

* **Token-per-minute caps.** Groq's free tier allows ~6k–12k tokens/minute per model, so one
  huge request can fail with HTTP 429. Lower the per-request size:
  `--max-total-chars 24000` — the map-reduce strategy then splits big documents into
  several small calls automatically.
* **Daily caps.** Free plans allow roughly 1,000 requests/day (Groq) — the 17-chunk
  map-reduce demo uses 18, so there is plenty of headroom.
* **Data policy.** Free-tier content may be used by the provider for training (Gemini free
  tier, some OpenRouter free models). Do not send confidential documents.
* **Model availability.** Free model IDs change over time; if you get a 404, check the
  provider's model list and pass the new ID with `--model`.
* **Keep the key private.** Use the environment variable (never hardcode it), don't commit
  it to your submission repo, and don't paste it into screenshots. This tool only ever
  sends it in the `Authorization` header and never logs it.

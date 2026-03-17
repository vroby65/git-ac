# git-ac AI Commit Message (Bash + Ollama or tgpt)

A tiny Bash helper that **generates ultra-short Git commit messages** from your staged changes, asks you to confirm/edit, and commits.

- Default engine: **Ollama** with the `gpt-oss:20b-cloud` model (ollama cloud).
- Alternative engine: **tgpt** (CLI chatbot without API keys).
- If generation fails, it falls back to **manual input**.

---

## How it works

1. `git add .`
2. Read the **staged diff** (`git diff --cached`). If empty → exit.
3. Ask an LLM for a **very short** commit message.
4. Show it, then:
   - **Y**: commit,
   - **n**: cancel,
   - **e**: open in `$EDITOR`, then commit.

---

## Requirements

- Git, Bash (Linux/macOS; on Windows use Git Bash or WSL).
- For **Ollama mode**:
  - Install Ollama and have the `mistral` model available. :contentReference[oaicite:0]{index=0}
- For **tgpt mode** (optional):
  - Install **tgpt** (cross-platform CLI; no API keys required). :contentReference[oaicite:1]{index=1}

---

## Install

1. Save the script as `git-ac` in your repo (or somewhere in `PATH`).

2. Make it executable:
   
   ```bash
   chmod +x git-ac
   ```

3. (Optional) choose the editor used by `e`:
   
   ```bash
   export EDITOR=micro   # or nano, geany, vim, ...
   ```

---

## Using Ollama (default)

1. **Install Ollama**
   
   * Linux/macOS:
     
     ```bash
     curl -fsSL https://ollama.com/install.sh | sh
     ```
     
     ([Ollama][1])
   
   * Windows: download the installer from the official site. ([Ollama][2])

2. **Get the model** (pulled automatically on first run):
   
   ```bash
   ollama run mistral "hello"
   ```
   
   ([Ollama][3])

3. Run the script inside your Git repo:
   
   ```bash
   ./git-ac
   ```

---

## Switching to tgpt (optional)

If you prefer **tgpt** instead of Ollama, install it and swap two lines in the script.

### Install tgpt

Pick one method (see repo for others):

* Linux/macOS (install script to `/usr/local/bin`):
  
  ```bash
  curl -sSL https://raw.githubusercontent.com/aandrew-me/tgpt/main/install | bash -s /usr/local/bin
  ```

* Homebrew:
  
  ```bash
  brew install tgpt
  ```

* Go:
  
  ```bash
  go install github.com/aandrew-me/tgpt/v2@latest
  ```

* Windows (PowerShell):
  
  ```powershell
  irm https://raw.githubusercontent.com/aandrew-me/tgpt/refs/heads/main/install-win.ps1 | iex
  ```
  
  ([GitHub][4])

### Minimal code change

Replace the **Ollama** line:

```bash
msg=$(ollama run mistral "Write a very short git commit message from:\n$diff")
```

with the **tgpt** block that also **cleans spinners/fences**:

```bash
raw=$(tgpt "Write ONLY a very short git commit message from:\n$diff")
msg=$(echo "$raw" | sed 's/.*Loading[[:space:]]*//' | sed 's/^[[:space:]]*//' | sed -n '/^```/{n;:a;/^```/q;p;n;ba}')
```

> Notes
> 
> * `tgpt` sometimes prints a “Loading …” spinner or wraps output in \`\`\`\`\` code fences; that one-liner strips both.
> * Keep the prompt **strict** (“ONLY a very short…”) to avoid verbose outputs.

---

## Tips

* Want **even shorter** messages? Soften the prompt to “**ONE line, max 8 words**”.
* Prefer imperative style? Add “**Use imperative tense**” to the prompt.
* Different Ollama model? Replace `mistral` with any installed model, e.g. `gemma:2b`, `llama3.2:1b`. ([GitHub][5])

---

## Troubleshooting

* `No changes to commit.`
  You forgot to stage files. Run `git add -p` or `git add .`.
* `ollama: command not found`
  Install Ollama and/or restart your shell so `PATH` is updated. ([Ollama][1])
* `model not found` or long first run
  Ollama downloads the model on first use; wait for `mistral` to pull. ([Ollama][3])
* `tgpt: command not found`
  Install tgpt via script/Homebrew/Go/Windows installer. ([GitHub][4])

---

## License

MIT (for this script). Ollama, models, and tgpt are licensed separately—see their repos/websites.

```
[1]: https://ollama.com/download?utm_source=chatgpt.com "Download Ollama on Linux"
[2]: https://ollama.com/download/windows?utm_source=chatgpt.com "Download Ollama on Windows"
[3]: https://ollama.com/library/mistral?utm_source=chatgpt.com "mistral"
[4]: https://github.com/aandrew-me/tgpt "GitHub - aandrew-me/tgpt: AI Chatbots in terminal without needing API keys"
[5]: https://github.com/ollama/ollama?utm_source=chatgpt.com "ollama/ollama: Get up and running with OpenAI gpt-oss, ..."
```

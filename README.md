# Paste everything below into your Kali terminal in one go.

set -euo pipefail

echo "[*] Updating apt and installing prerequisites (may ask for sudo)..."
sudo apt update -y
sudo apt install -y adb git python3 python3-pip || true

echo "[*] Installing Python packages (requests, python-dotenv)..."
python3 -m pip install --user requests python-dotenv || python3 -m pip install requests python-dotenv

OUT="$HOME/predator_assist.py"
echo "[*] Writing script to $OUT"

cat > "$OUT" <<'PYCODE'
#!/usr/bin/env python3
# predator_assist.py
# Kali CLI assistant: multi-model adapter + optional adb (Android) access (user-consent only).
# Minimal, robust, no-frills.

import os
import subprocess
import json
import hmac
from pathlib import Path
from datetime import datetime
from dotenv import load_dotenv

load_dotenv()

LOGFILE = Path.home() / "predator_assist_activity.log"

def log_action(action, details=""):
    ts = datetime.utcnow().isoformat() + "Z"
    payload = f"{ts}|{action}|{details}"
    key = os.getenv("PA_SIGN_KEY", "")
    sig = ""
    if key:
        try:
            sig = hmac.new(key.encode(), payload.encode(), "sha256").hexdigest()
        except Exception:
            sig = ""
    try:
        with open(LOGFILE, "a") as f:
            f.write(f"{payload}|{sig}\n")
    except Exception:
        pass

def prompt_yesno(prompt):
    try:
        ans = input(f"{prompt} (y/N): ").strip().lower()
    except EOFError:
        return False
    return ans == "y"

# ------------ Model adapters ------------
import requests

def call_openai(prompt, model="gpt-4o", max_tokens=512):
    api_key = os.getenv("OPENAI_API_KEY")
    if not api_key:
        raise RuntimeError("OPENAI_API_KEY not set in environment")
    url = "https://api.openai.com/v1/chat/completions"
    headers = {"Authorization": f"Bearer {api_key}", "Content-Type":"application/json"}
    body = {
        "model": model,
        "messages": [{"role":"user","content":prompt}],
        "max_tokens": max_tokens
    }
    r = requests.post(url, headers=headers, json=body, timeout=60)
    r.raise_for_status()
    resp = r.json()
    # robust path
    choices = resp.get("choices") or []
    if choices:
        first = choices[0]
        msg = first.get("message") or {}
        text = msg.get("content") or first.get("text") or ""
    else:
        text = ""
    log_action("openai_call", f"model={model} tokens={max_tokens}")
    return text

def call_anthropic(prompt, model="claude-3-opus", max_tokens=512):
    api_key = os.getenv("ANTHROPIC_API_KEY")
    if not api_key:
        raise RuntimeError("ANTHROPIC_API_KEY not set in environment")
    url = "https://api.anthropic.com/v1/complete"
    headers = {"x-api-key": api_key, "Content-Type":"application/json"}
    body = {"model": model, "prompt": prompt, "max_tokens_to_sample": max_tokens}
    r = requests.post(url, headers=headers, json=body, timeout=60)
    r.raise_for_status()
    out = r.json()
    text = out.get("completion","")
    log_action("anthropic_call", f"model={model} tokens={max_tokens}")
    return text

def call_huggingface_inference(prompt, repo_id, api_token=None):
    token = api_token or os.getenv("HUGGINGFACE_API_KEY")
    if not token:
        raise RuntimeError("HUGGINGFACE_API_KEY not set in environment")
    url = f"https://api-inference.huggingface.co/models/{repo_id}"
    headers = {"Authorization": f"Bearer {token}"}
    r = requests.post(url, headers=headers, json={"inputs":prompt}, timeout=120)
    r.raise_for_status()
    out = r.json()
    text = ""
    if isinstance(out, list) and out:
        first = out[0]
        text = first.get("generated_text") or json.dumps(first)
    elif isinstance(out, dict):
        text = out.get("generated_text") or out.get("error") or json.dumps(out)
    log_action("hf_call", f"repo={repo_id}")
    return text

def call_local_llama_cpp(prompt, llm_cmd="./main", model_path=None):
    if not Path(llm_cmd).exists():
        raise RuntimeError(f"llama.cpp binary not found at path: {llm_cmd}")
    if not model_path or not Path(model_path).exists():
        raise RuntimeError("Local model path not provided or not found.")
    # User may need to adapt flags; this is a best-effort invocation.
    cmd = [llm_cmd,

<img src="https://octodex.github.com/images/Professortocat_v2.png" align="right" height="200px" />

Hey @EAOUITTDOS!

Mona here. I'm done preparing your exercise. Hope you enjoy! 💚

Remember, it's self-paced so feel free to take a break! ☕️

[![](https://img.shields.io/badge/Go%20to%20Exercise-%E2%86%92-1f883d?style=for-the-badge&logo=github&labelColor=197935)](https://github.com/EAOUITTDOS/skills-introduction-to-github/issues/1)

---

&copy; 2025 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)


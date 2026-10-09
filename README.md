# Kannada Voice Banking Kiosk

A hands-free bank kiosk for Kannada speakers. The customer speaks Kannada; the system
understands the request, answers in a natural Kannada voice, and fills bank forms by voice.
Branch staff manage it from a web admin console.

## Features
- 7 banking requests: check balance, withdraw, deposit, open account, apply for a loan,
  account information (ATM, PIN, cheque book), interest rates and loan repayment
- 11 bank forms filled entirely by voice, with every answer read back for confirmation
- Spoken Kannada numbers and dates converted to digits and DD/MM/YYYY
- Camera-based customer detection; admin console with conversation history,
  form submissions, customer lookup and voice / speech-engine settings
- Optional Sarvam AI cloud engines with automatic fallback to the local models

## How it works
```
Customer speaks Kannada
 → Voice detection (browser)      start / end of speech
 → Speech-to-text                 Kannada Whisper (CTranslate2)
 → Translation                    IndicTrans2, Kannada → English
 → Intent                         Kannada keywords → English keywords → fine-tuned DistilBERT
 → Decision router                spoken answer, or open the right form
 → Text-to-speech                 Indic Parler-TTS on a separate GPU voice server
```

## Models
| Stage | Model | Trained in this project? |
|---|---|---|
| Speech-to-text | vasista22/whisper-kannada-medium | No (pre-trained, converted to CTranslate2) |
| Translation | AI4Bharat IndicTrans2 (dist-200M, both directions) | No (pre-trained) |
| Intent | distilbert-base-uncased | **Yes, fine-tuned on 301 banking sentences** |
| Voice | AI4Bharat Indic Parler-TTS | No (pre-trained) |

Intent model: 97.8 % accuracy (macro F1 0.979) on 46 held-out test sentences,
against 60.9 % for a keyword baseline.

## Requirements
- Windows 10/11, Python 3.11 or 3.12, Node.js 18+, Git, Microsoft C++ Build Tools
- NVIDIA GPU recommended (4 GB+); runs on CPU, slower
- Free HuggingFace account with access to the two IndicTrans2 models
- A running voice server (Parler-TTS, NVIDIA GPU with about 8 GB) reachable by URL

## Setup
```powershell
setup.bat --skip-models
# NVIDIA GPU only:
.venv\Scripts\python -m pip install torch --index-url https://download.pytorch.org/whl/cu124 --force-reinstall

# Models (one time, needs internet + HuggingFace login)
.venv\Scripts\python -c "from huggingface_hub import login; login()"
$env:TRANSFORMERS_OFFLINE="0"; $env:HF_HUB_OFFLINE="0"; $env:PYTHONPATH=(Get-Location).Path
.venv\Scripts\python backend\stt\convert_models.py --model vasista-medium
.venv\Scripts\python -c "from transformers import AutoModelForSeq2SeqLM, AutoTokenizer; m='ai4bharat/indictrans2-indic-en-dist-200M'; AutoTokenizer.from_pretrained(m, trust_remote_code=True); AutoModelForSeq2SeqLM.from_pretrained(m, trust_remote_code=True)"
.venv\Scripts\python -c "from transformers import AutoModelForSeq2SeqLM, AutoTokenizer; m='ai4bharat/indictrans2-en-indic-dist-200M'; AutoTokenizer.from_pretrained(m, trust_remote_code=True); AutoModelForSeq2SeqLM.from_pretrained(m, trust_remote_code=True)"
.venv\Scripts\python backend\nlu\train.py        # trains the intent model
```
Copy `.env.kiosk.example` to `.env` and set `BANK_TTS_REMOTE_URL`, `BANK_TTS_REMOTE_KEY`
and `BANK_ADMIN_PASS`.

## Run
```powershell
.\scripts\start-kiosk-api.ps1                              # terminal 1
cd frontend; $env:VITE_DEV_HTTP="1"; .\scripts\dev-all.ps1  # terminal 2
```
- Kiosk: http://localhost:5173 (Chrome, one tab, allow the microphone)
- Admin: http://localhost:5174

## Try it
Wait for the green **ಈಗ ಮಾತನಾಡಿ** (speak now), then say:
- **ನನ್ನ ಖಾತೆ ಬ್ಯಾಲೆನ್ಸ್ ಹೇಳಿ** → last 4 digits **ಏಳು ಎಂಟು ಒಂಬತ್ತು ಸೊನ್ನೆ** → **ಹೌದು** (demo account)
- **ಹೊಸ ಖಾತೆ ತೆರೆಯಬೇಕು** → account-opening form by voice
- **ಬಡ್ಡಿ ದರ ಎಷ್ಟು?** → interest rates
- **ಮುಗಿಸು** → end the session

## Tests
```powershell
.venv\Scripts\python -m pytest tests -q
```

## Credits
- Base system: <who built the original system>
- Our contribution: <what this team built, trained, tested and documented>
- Pre-trained models: vasista22/whisper-kannada-medium, AI4Bharat IndicTrans2,
  AI4Bharat Indic Parler-TTS, distilbert-base-uncased (fine-tuned by this team).

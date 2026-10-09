# Kannada Voice Banking Kiosk

A hands-free kiosk where a customer speaks Kannada and the system understands the
request, answers in a Kannada voice, and fills bank forms by voice.

## How it works
Voice detection -> Kannada speech-to-text (Whisper, CTranslate2) -> Kannada->English
translation (IndicTrans2) -> intent (keyword rules + fine-tuned DistilBERT) ->
decision router -> Kannada text-to-speech (Indic Parler-TTS).

## Run
1. `setup.bat --skip-models`, then download / convert the models.
2. Copy `.env.kiosk.example` to `.env` and fill in the keys.
3. `scripts\start-kiosk-api.ps1`, then `cd frontend; .\scripts\dev-all.ps1`.
4. Kiosk: http://localhost:5173   Admin: http://localhost:5174

## Credits
- Base system: <who built the original system>
- Our contribution: <what this team built, trained, tested and documented>
- Pre-trained models: vasista22/whisper-kannada-medium, AI4Bharat IndicTrans2,
  AI4Bharat Indic Parler-TTS, distilbert-base-uncased (fine-tuned by us).

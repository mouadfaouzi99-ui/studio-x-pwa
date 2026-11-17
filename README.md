# STUDIO X

**A Darija-first creative audio studio and AI toolkit, delivered as an installable Progressive Web App.** STUDIO X combines browser-based audio mixing and mastering with Cloudflare Workers AI features for creative chat, image generation, and speech transcription. An optional WhatsApp Business webhook can bring a lightweight audio-and-text assistant into WhatsApp.

The interface is primarily written in Moroccan Darija, while this document describes the project in English. The application is designed to work on desktop and mobile browsers; its static app shell can be cached as a PWA, while AI-backed features require an internet connection and a configured Cloudflare Worker.

## What it does

### Browser-local audio studio

The mixer uses the Web Audio API to load and process a selected audio track in the browser. Its processing chain includes equalization, compression, gain, limiting, reverb, delay, and pitch shifting. Users can start from vocal presets or mastering targets, adjust the controls, preview the result, and render an offline WAV export.

Practice controls add A/B loop points and playback speed adjustment. The speed control changes playback rate and pitch for practice; it is a preview aid and is not included in the exported master. Pitch shifting in the effects chain is a separate control. Vocal presets are starting points, not automatic tuning, a vocoder, or guaranteed audio repair. The displayed loudness estimate is based on sampled RMS and is not a standards-compliant BS.1770 measurement.

The mixer processes and exports the audio locally. It does not send the mixer track to the Worker unless the user separately attaches an audio file to the AI assistant or shares a finished file using the device's native share sheet.

### Creative assistant

The chat assistant is designed for practical creative and audio-workflow advice in Moroccan Darija or French. It can also accept small text files (`.txt`, `.md`, `.csv`, and `.json`); the browser reads the file and includes a limited text excerpt in the chat request. Chat requests are handled by the Worker and sent to Cloudflare Workers AI. The current text model is configured in `src/worker.js`.

For an attached audio file, the Worker sends the audio to its speech-recognition model and passes the resulting transcript to the text assistant. The assistant can suggest next steps, but it does not hear the audio, measure its sound quality, or automatically clean/master it. Audio attachments are limited to 5 MB; text attachments are limited to 2 MB in the interface.

### Image generation

The image tool submits a text prompt to the Worker, which calls the configured Cloudflare Workers AI image model and returns a generated image to the browser. Provider availability, account limits, and charges depend on the Cloudflare account and its current plan; this repository does not promise unlimited or permanently free inference.

### Optional WhatsApp assistant

The Worker includes a WhatsApp Cloud API webhook for text messages, audio transcription, and small text/document files. Supported inbound audio and files are limited to 5 MB. For supported messages, the assistant replies in WhatsApp and sends a `.txt` transcript or content excerpt with its advice. It does not automatically master and send back an audio file.

The WhatsApp webhook is inactive until the owner configures a Meta WhatsApp Business app and adds the required credentials as Cloudflare Worker secrets. WhatsApp charges and Cloudflare AI usage limits may apply. The mixer’s WhatsApp button uses the device’s native share sheet to let the user choose a recipient; it is separate from the inbound webhook workflow.

## Architecture

- `public/` — single-page app, PWA manifest, service worker, and app icons.
- `src/worker.js` — Cloudflare Worker routes for AI services and the optional WhatsApp webhook.
- `wrangler.jsonc` — Wrangler configuration, static asset binding, and Workers AI binding.
- `README.md` — project overview, data-flow notes, and deployment instructions.

The service worker caches the app shell for faster repeat visits and basic offline access to cached static resources. API-backed AI features require a network connection. No model API keys are embedded in the browser application; AI calls are made by the Worker through its Cloudflare binding.

## API routes

| Route | Method | Purpose |
| --- | --- | --- |
| `/api/health` | `GET` | Reports Worker and AI-binding availability. |
| `/api/chat` | `POST` | Sends a bounded conversation to the configured text model. |
| `/api/audio-assist` | `POST` | Accepts an audio upload (up to 5 MB), transcribes it, and returns a transcript plus workflow advice. |
| `/api/image` | `POST` | Generates an image from a text prompt. |
| `/api/whatsapp/status` | `GET` | Reports whether the required WhatsApp configuration is present. |
| `/api/whatsapp/webhook` | `GET`, `POST` | Verifies and receives Meta WhatsApp webhook events when configured. |

The Worker applies a per-IP in-memory request limit to AI POST routes. This is a lightweight runtime safeguard, not a durable global quota or an account-level billing control.

## Run locally

You need Node.js/npm and a Cloudflare account for Worker-backed features.

```bash
npx wrangler@3 dev
```

The browser-local mixer can be exercised without AI, but chat, image generation, and transcription need a working Workers AI binding. Authenticate Wrangler with the Cloudflare account that owns the Worker before deploying. If you use multiple Cloudflare accounts, select the intended account explicitly in your local Wrangler configuration; do not commit credentials or local secret files.

## Deploy to Cloudflare

```bash
npx wrangler@3 login
npx wrangler@3 deploy
```

The deployment uses the static files in `public/`, the Worker entry point in `src/worker.js`, and the `AI` binding declared in `wrangler.jsonc`. Review Cloudflare's current Workers and Workers AI availability, limits, and pricing before enabling public access to AI endpoints.

## Configure the WhatsApp webhook (optional)

1. Create and configure a Meta developer app with the WhatsApp Business Platform.
2. In the Cloudflare dashboard, open Worker `studio-x-pwa` → **Settings → Variables and Secrets**. Add these as **secrets**, not plaintext values in source code:
   - `WHATSAPP_ACCESS_TOKEN`
   - `WHATSAPP_PHONE_NUMBER_ID`
   - `WHATSAPP_VERIFY_TOKEN` — generate a separate random verification string.
   - `META_APP_SECRET`
3. Optionally add `WHATSAPP_GRAPH_VERSION` as a regular variable; the Worker currently defaults to `v27.0`.
4. In Meta Webhooks, configure the callback URL as `https://<your-worker-domain>/api/whatsapp/webhook`, enter the matching verify token, and subscribe to the `messages` field.

The Worker verifies Meta's request signature before processing inbound events. Keep all access tokens and app secrets in Cloudflare's secret store. Never commit them to Git or paste them into issue reports.

## Privacy and usage notes

- Audio loaded only into the mixer stays in the browser for processing and WAV export.
- If you attach audio to the creative assistant, the audio is sent to the Worker for transcription; the transcript and conversation are sent to Workers AI for a response.
- Chat prompts and image prompts are sent to the Worker and its configured AI provider. Do not submit sensitive or confidential content unless you have reviewed the applicable provider terms and data handling.
- WhatsApp messages and supported media pass through Meta's WhatsApp Cloud API and the configured Worker/AI services.
- This repository contains application source and public configuration only. Keep Cloudflare, Meta, and other credentials in secret stores; `.gitignore` excludes common local Wrangler and environment-secret files.

## Limitations

- AI features depend on Cloudflare Workers AI availability, account limits, network access, and any applicable charges.
- Audio speech assistance is transcription-based; it does not perform AI audio repair or return a mastered recording.
- Practice playback speed changes pitch, and that speed setting is not part of the exported WAV.
- The loudness readout is an RMS-based estimate rather than a certified loudness meter.
- WhatsApp automation requires a separately configured Meta Business integration and may incur message charges.

## License

No license is currently included. Unless a license is added, all rights remain with the copyright holder and reuse is not automatically granted.

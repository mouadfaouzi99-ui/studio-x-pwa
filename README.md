# STUDIO X

Static PWA and Cloudflare Worker for browser-local audio mastering, vocal effects, AI chat/image generation, and an optional WhatsApp Cloud API webhook.

## WhatsApp activation

The webhook code is deployed, but it remains inactive until a Meta WhatsApp Business Cloud API app is configured. Do not commit tokens or paste them into chat.

1. Create/configure a Meta developer app with the WhatsApp Business Platform and obtain the phone number ID, access token, and app secret.
2. In the Cloudflare dashboard, open Worker `studio-x-pwa` → Settings → Variables and Secrets. Add these as **secrets**:
   - `WHATSAPP_ACCESS_TOKEN`
   - `WHATSAPP_PHONE_NUMBER_ID`
   - `WHATSAPP_VERIFY_TOKEN` (a new random string that you choose)
   - `META_APP_SECRET`
3. Optional non-secret variable: `WHATSAPP_GRAPH_VERSION` (defaults to `v27.0`).
4. In Meta Webhooks, use `https://studio-x-pwa.studiox-ma.workers.dev/api/whatsapp/webhook`, enter the same verify token, and subscribe to the `messages` field.

The bot can answer text, transcribe inbound audio (up to 5 MB), and read small `.txt`, `.md`, `.csv`, or `.json` documents (up to 5 MB). For supported files it replies with advice and sends a `.txt` report containing the transcript or text excerpt plus that advice. Other file types are acknowledged but not analyzed or stored. From the Studio mixer, the WhatsApp share button can pass an exported mastered WAV to the phone's native share sheet; the user then chooses the WhatsApp recipient. The webhook does not automatically master and return an audio file.

WhatsApp message charges may apply under Meta's pricing rules. AI transcription/chat also uses the Cloudflare Workers AI allowance. The web audio mixer itself runs locally in the browser.

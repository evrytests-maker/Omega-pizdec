# Cumometer Leaderboard

Public leaderboard for the [Cumometer](https://github.com/evrytests-maker/Omega-pizdec-big-dick) SillyTavern extension.
A static page plus three serverless functions, made for Vercel.

## Deploy to Vercel

1. **Add New → Project → Import** this repository. Framework preset **Other**, no build command, root directory as is.
2. **Storage → Marketplace → Upstash Redis → Create → Connect to project.** This adds `KV_REST_API_URL` and
   `KV_REST_API_TOKEN` (`UPSTASH_REDIS_REST_URL` / `UPSTASH_REDIS_REST_TOKEN` also work).
3. Generate the key pair locally (Node 18+):
   ```bash
   node scripts/genkey.mjs
   ```
   Add the printed line as the environment variable **`BOARD_PRIVATE_KEY`** (Settings → Environment Variables).
   Keep it secret.
4. **Redeploy**, then paste the project URL into SillyTavern → Cumometer → Leaderboard → Server URL.

## How submissions work

Nothing is sent until a player presses **Submit** in the extension. The payload is the jar volume and the nickname,
plus a timestamp and a one-way fingerprint of the player's local key (it keeps the nickname theirs; the key never leaves
the browser). It is sealed to this deployment's public key — ephemeral ECDH P-256 → HKDF-SHA256 → AES-256-GCM — so on the
wire there is only `{ v, epk, iv, ct }`. The server rejects tampered ciphertext, replays of older envelopes, and
nicknames claimed by someone else.

## API

- `GET /api/key` → `{ v: 1, key: <public JWK> }`
- `POST /api/submit` `{ v, epk, iv, ct }` → `{ rank }` (one request per 30 s; `409 nick_taken` if the nickname is taken)
- `GET /api/top?limit=50&me=<fingerprint>` → `{ rows: [{ rank, nick, ml }], me, stats: { players, totalMl } }`
- `/` — the leaderboard page (RU/EN, light/dark, refreshes every 30 s).

# GATEKEEPER

**The machine writes the rules. You survive them.**

A 60-second arcade arena in which a language model invents the terms of each run — which
colour sustains you, which colour is fatal, whether you are permitted to stand still. The
arena art is generated per run. Nothing about a run is prewritten.

**Play it:** https://khuzaima3232.github.io/gatekeeper/

## How it uses Pollinations

Everything the game does is a live Pollinations request, all through the unified
`gen.pollinations.ai` API:

| What | Call |
| --- | --- |
| The terms of the run (a JSON ruleset, validated client-side) | `POST gen.pollinations.ai/v1/chat/completions` |
| The arena backdrop, seeded from the ruleset | `GET gen.pollinations.ai/image/{prompt}` |
| The end-of-run verdict line | `POST gen.pollinations.ai/v1/chat/completions` |

Models: `openai/gpt-oss-20b` for the rules and verdict, `openai/gpt-image-1-mini` for the
art — both chosen so a run costs the player roughly 0.0001 Pollen.

## Sign-in: Connect User Wallets (BYOP)

`gen.pollinations.ai` requires an API key, and embedding a raw key in a browser is not
allowed, so the app uses the supported client path instead: **Connect User Wallets**.

1. `signIn()` generates a PKCE verifier, derives the S256 challenge, and sends the player to
   `enter.pollinations.ai/authorize` with this app's publishable App Key as `client_id`.
2. The consent screen shows what is being asked for, and the player approves a small budget
   of **their own** Pollen.
3. The short-lived `?code=` comes back to this page; `completeSignIn()` exchanges it at
   `enter.pollinations.ai/api/oauth/token` with the PKCE verifier (no client secret) for a
   scoped `sk_` key. That key lives in `sessionStorage` only — never `localStorage`, never a
   URL, never a log — and every request after that is `Authorization: Bearer <key>`.

The app owner pays nothing and never sees the player's account; the player can revoke the key
from their dashboard at any time. Generation needs no scope, so the request only asks for
`usage`.

**Setup:** the App Key (`pk_...`) must be registered at `enter.pollinations.ai/keys` with the
Redirect URI `https://khuzaima3232.github.io/gatekeeper/` (an exact match — for local testing,
loopback matches any port). It goes in `AUTH.clientId` at the top of `index.html`.

## The design idea

The Pollinations app catalog holds 873 apps; 52 are games, and nearly all of them are
text-narrative (AI dungeon masters, companion chats, storytelling). The ones that actually get
played are the tight-loop ones. GATEKEEPER goes the other way: a hard 60-second arcade loop in
which the AI is not the storyteller but the **rules-engine**.

The model may only compose the run's terms from a fixed enum of adjudicable mutators:
`only_color`, `dodge_color`, `no_stop`, `reverse`, `shrink`, `ramp`. Its JSON is validated
against that enum, and anything unusable is discarded. If the API is unreachable after three
attempts the arena falls back to a local ruleset and says so, so the game never breaks — but
the terms you get are then marked as local rather than machine-written.

## Play

- Move with **WASD / arrow keys**, or **drag / hold** on touch and pointer.
- 3 strikes ends the run early. Survive 60 seconds to complete it.
- **Daily terms** are seeded from the UTC date, so everyone playing today gets the same
  ruleset; "skip the shared terms" gives you a fresh one.

## Notes

- Single self-contained `index.html`. No build step, no dependencies, no tracking.
- Best score is kept in `localStorage`; the access key is kept in `sessionStorage`.
- Generated art is cached by seed; a failed image request degrades to the plain grid rather
  than breaking the run.

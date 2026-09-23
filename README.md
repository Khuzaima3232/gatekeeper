# GATEKEEPER

**The machine writes the rules. You survive them.**

A 60-second arcade arena in which a language model invents the terms of each run — which
colour sustains you, which colour is fatal, whether you are permitted to stand still. The
arena art is generated per run. Nothing about a run is prewritten.

**Play it:** https://khuzaima3232.github.io/gatekeeper/

## How it uses Pollinations

| What | Endpoint |
| --- | --- |
| The terms of the run (JSON ruleset, validated client-side) | `text.pollinations.ai` |
| The arena backdrop, seeded per ruleset | `image.pollinations.ai` |
| The end-of-run verdict line | `text.pollinations.ai` |

Both are public keyless endpoints, so the app needs no backend, no API key, and no setup.
Every run makes at least two live Pollinations requests, visible in the network panel and in
this source (`TEXT_API` / `IMAGE_API` at the top of `index.html`).

## The design idea

The Pollinations app catalog holds 873 apps; 52 are games, and nearly all of them are
text-narrative (AI dungeon masters, companion chats, storytelling). The ones that actually get
played are the tight-loop ones. GATEKEEPER goes the other way: a hard 60-second arcade loop in
which the AI is not the storyteller but the **rules-engine**.

The model may only compose the run's terms from a fixed enum of adjudicable mutators:
`only_color`, `dodge_color`, `no_stop`, `reverse`, `shrink`, `ramp`. Its JSON is validated
against that enum, and anything unusable is discarded. If the model is unreachable after three
attempts the arena falls back to a local ruleset, so the game never breaks — but the terms you
get are then marked as local rather than machine-written.

## Play

- Move with **WASD / arrow keys**, or **drag / hold** on touch and pointer.
- 3 strikes ends the run early. Survive 60 seconds to complete it.
- **Daily terms** are seeded from the UTC date, so everyone playing today gets the same
  ruleset; "skip the shared terms" gives you a fresh one.

## Notes

- Single self-contained `index.html`. No build step, no dependencies, no tracking.
- Best score is kept in `localStorage`.
- Generated art is cached by seed; a failed image request degrades to the plain grid rather
  than breaking the run.

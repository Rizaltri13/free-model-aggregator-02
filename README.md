[![make.fun · free-model-aggregator-02](https://make.fun/badge/free-model-aggregator-02.svg)](https://make.fun/bounty/free-model-aggregator-02?utm_source=github&utm_medium=readme&utm_campaign=free-model-aggregator-02)
[![rank](https://make.fun/badge/free-model-aggregator-02/Rizaltri13/free-model-aggregator-02/rank.svg)](https://make.fun/bounty/free-model-aggregator-02/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=free-model-aggregator-02)
[![reward](https://make.fun/badge/free-model-aggregator-02/Rizaltri13/free-model-aggregator-02/reward.svg)](https://make.fun/bounty/free-model-aggregator-02?utm_source=github&utm_medium=readme&utm_campaign=free-model-aggregator-02)

# The best live free-model aggregator

One server (`main.ts`, Node 22, standard library only) that serves the
OpenAI Chat Completions API and the Anthropic Messages API — each in JSON and
in server-sent events — and answers every request with a **real** completion
from a free OpenRouter model, dialled through the trial's gateway on the
trial's own key. The server invents nothing: it picks the model, translates
the wire, and hands the provider's answer back.

For every prompt it:

- reads the brief (`MAKEFUN_PROMPT_FILE`): the free model list with the
  house's quality ratings and each model's capabilities (tools, vision, JSON
  mode, context length), and the prompts' manifest;
- registers a catalog naming exactly those free models, once each — the same
  document `node main.ts --list` prints — and answers `GET /v1/models`
  with that same set in whichever wire the auth header names;
- ranks the free models that meet the prompt's requirements by the house
  rating, best first, and dials the best on the gateway's Chat Completions
  route (an Anthropic-shaped request is translated into that body first, and
  the answer translated back);
- puts the dialled model's id — gateway prefix and all — on the answer's
  `model` field and on every streamed chunk, never the provider's own
  spelling of it;
- fails over down the ranking when a provider refuses (a real 429 or 5xx),
  with a second pass when the first spends every candidate, and reports each
  refusal it recovered from in the answer's `x-makefun-upstream-failures`
  header;
- remembers consecutive refusals: a model that refuses twice in a row is
  demoted behind the healthy models for later prompts, and a success clears
  its record;
- exits 0 when the env reports the workload done.

Routing is a pure function of the brief, the requests and the gateway's
answers — no wall clock, no randomness — so a replayed trial makes exactly
the same calls.

## Run

```sh
node main.ts          # serve (trial mode: MAKEFUN_ENV_URL + OPENAI_BASE_URL + MAKEFUN_PROMPT_FILE)
node main.ts --list   # print the catalog document and exit
```

## License

MIT — see `LICENSE`.

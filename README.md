# Petrol Pump Finder

A [neuro-san](https://github.com/cognizant-ai-lab/neuro-san) AAOSA agent network that helps
users find a petrol pump for a given time, fuel type, or location. If no specialized agent
matches the request, it falls back to a general web search agent instead of refusing to answer.

## Files

| File | Purpose |
|---|---|
| `petrol_pump_finder.hocon` | The agent network definition (entry point + sub-agents) |

Place this file in your neuro-san-studio project under `registries/` and register it in
`registries/manifest.hocon` so the server picks it up.

## How it works

`PetrolPumpFinder` is the entry-point agent. It receives the user's query and decides which
sub-agent(s) can answer it, based on each agent's stated hours, fuel types, and location.

```
                        ┌────────────────────┐
                        │  PetrolPumpFinder   │  (entry point)
                        └──────────┬──────────┘
                                   │ routes to best match
        ┌───────────────┬─────────┼─────────┬──────────────┐
        ▼               ▼         ▼          ▼              ▼
 CityFuelStation  HighwayFuelStation  CNGStation  PremiumFuelStation  GlobalSearch
  6am–11pm          24 hours          5am–10pm      24 hours           fallback
  petrol/diesel     petrol/diesel     CNG only      petrol/diesel/     when nothing
                                                     premium fuel       else matches
```

### Sub-agents

- **CityFuelStation** — Sam's City Fuel Station. Open 6 am–11 pm. Petrol and diesel.
- **HighwayFuelStation** — Highway Express Fuels. Open 24 hours. Petrol and diesel.
- **CNGStation** — GreenGas CNG Station. Open 5 am–10 pm. CNG only.
- **PremiumFuelStation** — Titan Premium Fuels. Open 24 hours. Petrol, diesel, and premium/high-octane fuel.
- **GlobalSearch** — Fallback agent. Used only when none of the above can answer (e.g. an
  unfamiliar brand, city, or fuel type). Its response is explicitly labeled as coming from a
  general search rather than one of the network's regular partner stations.

## Example queries

- "Where can I find a petrol pump?"
- "It's 11 pm, where can I get diesel?"
- "Where can I find a CNG station?"
- "Is there a petrol pump on the highway?"
- "Where can I find a Shell station?" → not covered by any sub-agent → routed to `GlobalSearch`

## Current limitation

`GlobalSearch` is currently an **LLM-only agent** — it has no real internet/API access, so its
fallback answers are plausible-sounding rather than fact-checked. To make it perform an actual
web lookup, attach a `CodedTool` (a small Python tool that calls a search API) and reference it
from `GlobalSearch`'s config, the same way `PdfReaderTool` is wired into the PDF Analyst network.

## Verifying it's working

1. Send a query that clearly misses every specialized agent (e.g. an unfamiliar brand name).
2. Check the server logs (`logs/server.log`, `logs/nsflow.log`, `logs/thinking_dir/`) for a
   call to `GlobalSearch`.
3. Or open the nsflow UI (`http://localhost:4173/`) to visually see the agent network graph
   and confirm the call chain reached `GlobalSearch`.
4. Confirm the final answer includes the "this result comes from a general search" disclaimer.

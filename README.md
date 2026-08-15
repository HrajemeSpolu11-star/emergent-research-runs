# emergent-research-runs

Auditní a experimentální repozitář pro projekt Emergent World.

## Proč tento repozitář vznikl

Observatoř (diagnostický nástroj v `emergent-agent/observatory_v2/`)
potřebovala místo pro archivaci kompletních experimentálních běhů --
timeline, snapshoty, telemetrii, Bridge bindings, observations, actions,
consequences, cognition traces, memory changes, causal updates, sensory
traces a auditní závěry -- bez zahlcování hlavních repozitářů
(`emergent-agent`, `emergent-world`, `emergent-bridge`) velkými raw
logy. Založeno 15.8.2026 na pokyn vlastníka projektu.

## Struktura

- `EXPERIMENT_SCHEMA.md` -- přesný formát dat jednoho exportovaného
  experimentu (přečti první, pokud píšeš nebo čteš exportovaná data)
- `AUDIT_PROTOCOL.md` -- co se má nad exportem provést a čím tlačítko
  "ODESLAT K AUDITU" skutečně je (a co NENÍ)
- `DATA_RETENTION.md` -- vlastnictví dat, bezpečnostní hranice,
  GitHub upload workflow
- `ROADMAP.md`, `PROJECT_MEMORY.md`, `CURRENT_STATE.md` -- institucionální
  paměť tohoto repozitáře, česky, podrobně
- `OPEN_SOURCE_REGISTRY.md` -- externí knihovny použité při generování
  exportu (např. ZIP writer), pokud nějaké jsou
- `experiments/` -- jednotlivé exportované experimenty, každý ve
  vlastním adresáři `experiment_<timestamp>_<id>/`

## Vztah k ostatním repozitářům

```
emergent-world    -- simulovaný svět (procedurální generování, 3D/izometrický render)
emergent-agent    -- kognitivní agent, Python, laboratoř MultisensoryLaboratory
emergent-bridge   -- Trusted Epistemic Boundary mezi World a Agent (invarianty I.1-I.12)
emergent-research-runs -- (tento repozitář) archiv experimentů a auditů, žádný běžící kód agenta
```

Tento repozitář neobsahuje běžící kód agenta ani světa -- pouze data
z jejich běhů a dokumentaci k jejich interpretaci.

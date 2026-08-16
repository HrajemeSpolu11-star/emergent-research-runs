# Session sess:publish-endpoint-token-selftest

> Automaticky vygenerováno z reálných dat této session. Žádné číslo níže není
> ilustrační -- vše je spočteno z `manifest.json` a `memory/memory_snapshots/diff.json`.

Agent `agent:live` byl testován 0.0 minut, 5 kroků.
Zaznamenáno 186 telemetrických eventů.

## Paměť: co se změnilo

- Episodická paměť: 0 → 5 záznamů
  (5 nových, 0 odstraněných/konsolidovaných,
  0 změněných).
- Kauzální graf: 0 → 0 hran
  (0 nových, 0 změněných confidence).
- Sémantická paměť: 0 → 0 faktů
  (0 nových).

## Negativní výsledky (co NEfungovalo)

- `interact` byl navržen, ale zamítnut arbitráží 5×
- `wait` byl navržen, ale zamítnut arbitráží 5×

Moduly, které se v tomto běhu **nikdy** neaktivně nepodílely na rozhodnutí
(pouze vypočítány, viz `diagnostics/calculated_consumed_ignored.jsonl`):
AutomaticCurriculum, ChangeDetection, SensorFusion

`SubconsciousGuardian` aktivně zasáhl 5×.
`Planner` fallback (arbitráž nevrátila kandidáta) nastal 0×.

## Úplnost a integrita

- Trace useknuta: NE, běh je kompletní
- Poznámek výzkumníka: 0
- Zásahů výzkumníka: 0
- `agent_commit_sha`: `c33808353cbb239d732aeb17f33ef35174625c5e`
  (`git_tree_clean_at_generation: False`)

## Neimplementované soubory v této session (explicitně, ne tiše chybí)

- `world/environment_changes.jsonl`
- `cognition/predictions.jsonl`
- `subconscious/novelty.jsonl`
- `bridge/action_requests.jsonl`
- `bridge/validation.jsonl`
- `bridge/failures.jsonl`

## Jak ověřit integritu

Pro každý soubor v `manifest.json` -> `checksums_sha256` přepočítej SHA-256 a
porovnej. Pro plnou provenance libovolného rozhodnutí použij
`emergent_agent.research_session.provenance.reconstruct_provenance_chain`
nad `raw/telemetry_chunk_*.jsonl`.

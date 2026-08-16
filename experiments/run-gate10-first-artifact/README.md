# GATE 10 -- první skutečný research artifact (Observatory v0.3.1)

## Co tento experiment je

Diagnostický experiment `diagnostic_causal_activation` (GATE 3 z etapy v0.3.1) --
deterministický scénář, kde agent začíná vedle skutečného objektu ve světě a jeho
skutečná kognitivní telemetrie (`CognitionTelemetryRecorder`) se zaznamenává na
úrovni FULL po celou dobu běhu.

## Klíčové rozlišení: přirozené vs. výzkumnický zásah

**Toto je nejdůležitější věc, kterou má auditor vědět.** Přirozená arbitráž agenta
v tomto běhu **nikdy nezvolila** akci `interact`, přestože byla opakovaně reálně
navržena (viz `manifest.json` -> `causal_activated_at` a
`intervention_summary`). Aby bylo možné pozorovat zbytek řetězce (Bridge resolution,
consequence, prediction error, reward, causal update), byl na posledních
tiku/tikách proveden **explicitní výzkumnický diagnostický zásah**: arbitráž byla
programově přepsána tak, aby vybrala nejlépe skórovaného **již reálně existujícího**
kandidáta na `interact` -- **nikdy nebyla vytvořena syntetická akce, kterou by
provider sám nikdy nenavrhl**.

Tyto tiky jsou v `intervention_log.jsonl` a v `manifest.json` pod
`intervention_summary.researcher_diagnostic_forced_selection_ticks` explicitně
označené. **Nesmí být interpretovány jako autonomní rozhodnutí agenta.**

## Soubory v tomto artefaktu

- `manifest.json` -- kompletní metadata (run_id, agent_id, přesné SHA commitu
  `emergent-agent`, schema/protokol verze, seed, provider, telemetry level,
  start/end tick, úplnost/truncation, chunk checksums)
- `raw_trace_chunk_NNN.jsonl` -- syrová telemetrická trace, rozdělená na chunky
  (viz `manifest.json` -> `chunks` pro SHA-256 každého chunku)
- `intervention_log.jsonl` -- per-tik záznam, zda byl krok přirozený nebo
  výzkumnický zásah (viz výše)
- `README.md` -- tento soubor

## Jak ověřit integritu

Pro každý `raw_trace_chunk_NNN.jsonl` přepočítej SHA-256 a porovnej s
`manifest.json` -> `chunks[i].sha256`. Pro `intervention_log.jsonl` porovnej s
`manifest.json` -> `intervention_log_sha256`. Pokud `manifest.json` ->
`trace_truncated` je `true`, trace je neúplná od tiku
`manifest.json -> last_complete_tick + 1` dál -- to NENÍ chyba, je to
zdokumentovaný, explicitní stav (viz `trace_truncation_reason`).

## Co tento artefakt dokazuje a co ne

**Dokazuje:** že popsaný telemetrický a korelační mechanismus (GATE 1-9 etapy
v0.3.1) skutečně funguje na reálném běhu -- včetně toho, že `CausalGraph` lze
reálně aktivovat a že jeho aktivace je vysledovatelná zpět přes `consumed_event_ids`
až k `AgentDecision` a `World` consequence.

**Nedokazuje:** že agent běžně/autonomně interaguje s objekty -- naopak, dokládá
opak (viz GATE 3 nález o arbitrážním scoring biasu). Nedokazuje nic o chování při
jiném seedu/providerovi/scénáři -- je to jeden konkrétní, deterministický běh.

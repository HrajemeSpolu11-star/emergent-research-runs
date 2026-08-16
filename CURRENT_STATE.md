# CURRENT_STATE.md

## Stav ke dni založení (15.8.2026)

**Fáze:** Repozitář právě založen. Žádný experiment zatím není
exportovaný ani nahraný. `experiments/` adresář zatím neexistuje nebo
je prázdný.

## Co existuje

- Kompletní povinná dokumentace: `README.md`, `ROADMAP.md`,
  `PROJECT_MEMORY.md`, `CURRENT_STATE.md` (tento soubor),
  `EXPERIMENT_SCHEMA.md`, `AUDIT_PROTOCOL.md`, `DATA_RETENTION.md`.
- `OPEN_SOURCE_REGISTRY.md` -- registr externích knihoven (zatím
  žádné externí knihovny nejsou vendorované, viz obsah souboru).

## Co neexistuje / co je otevřené

- Žádný skutečný export z Observatoře zatím nebyl vytvořen a nahrán
  sem -- implementace recorderu (ZAČÍT/POZASTAVIT/UKONČIT ZÁZNAM,
  IndexedDB, export ZIP) v `observatory_v2.html` je oddělený krok
  tohoto úkolu, viz `emergent-agent/CURRENT_STATE.md` pro jeho stav.
- `experiments/` adresář se založí prvním skutečným exportem.

## Navazující zdroje pravdy

Pro aktuální stav SAMOTNÉ Observatoře (recorder, IndexedDB, export,
UI redesign, testy) je zdrojem pravdy `emergent-agent/CURRENT_STATE.md`
a `emergent-agent/ROADMAP.md` -- tento soubor popisuje pouze stav
tohoto archivního repozitáře, ne stav nástroje, který do něj zapisuje.

---

## Aktualizace 15.8.2026 (nejnovější) -- v0.3.1 schema dodatek

`EXPERIMENT_SCHEMA.md` rozšířeno o v1.2.0 dodatek (correlation ID, telemetry režimy,
truncation) -- viz ten soubor pro detaily. Žádný nový experiment nebyl do tohoto repozitáře
nahrán (stejné jako v předchozím zápisu). `AUDIT_PROTOCOL.md` nebyl měněn -- auditní postup
se v této etapě nezměnil, jen datové schéma, které audit zkoumá.

---

## Aktualizace 16.8.2026 -- GATE 10 (v0.3.1): první skutečný research artifact

`experiments/run-gate10-first-artifact/` -- první skutečný exportovaný experiment,
ne demo/placeholder. Obsahuje:
- `manifest.json` -- run_id, agent_id, přesné SHA commitu `emergent-agent`
  (`1067ed9b33037c82f0506f4b9aa33445d5cbc11a`), schema/protokol verze, seed,
  provider, telemetry level, start/end tick, `intervention_summary`
- `raw_trace_chunk_000.jsonl` -- syrová FULL-úrovňová telemetrická trace (445 událostí)
- `intervention_log.jsonl` -- per-tik rozlišení přirozené/výzkumnický zásah
- `README.md` -- český audit summary

**Kritické, explicitně zaznamenané:** posledních 2 ze 12 kroků tohoto běhu byly
`researcher_diagnostic_forced_selection` (viz `manifest.json` ->
`intervention_summary`) -- arbitráž byla programově přepsána, aby vybrala už
reálně navrženého kandidáta na `interact`, protože přirozená arbitráž tuto akci
v tomto scénáři nikdy nezvolila. Tyto kroky NEJSOU autonomní rozhodnutí agenta.

Checksums všech souborů nezávisle ověřeny (viz `emergent-agent/tests/test_gate10_research_artifact.py`).

Toto uzavírá GATE 10 z etapy Observatory v0.3.1 v `emergent-agent`.

---

## Aktualizace 16.8.2026 (oprava) -- GATE 10 provenance bug opraven

Předchozí verze `experiments/run-gate10-first-artifact/` deklarovala
`agent_commit_sha=1067ed9...`, ale byla ve skutečnosti vygenerována kódem
(generátor + intervention_log propojení + testy), který v tu chvíli existoval
jen jako necommitnutá změna, později commitnutá jako `1870d9c`. Nalezeno
vlastníkem při review.

**Oprava:** `emergent-agent` teď má strukturální ochranu
(`generate_research_artifact.build_artifact(require_clean_tree=True)`, výchozí),
která **odmítne** vygenerovat artifact, pokud je pracovní strom špinavý --
`git rev-parse HEAD` sám o sobě tento nesoulad nikdy nezachytí, protože vrátí
poslední commit bez ohledu na necommitnuté změny. Přidán regresní test
(`test_gate10_manifest_sha_matches_actual_head`), který přímo ověřuje
`manifest.agent_commit_sha == git rev-parse HEAD` v okamžiku generování.

**Artifact zde přepsán** nově vygenerovanou verzí z čistého HEAD
`emergent-agent` commit `25568fa63b1fd928622774941fe3b1422b7225f5`
(`git_tree_clean_at_generation: true` v manifestu). Všechny checksums
(chunk SHA-256, intervention_log SHA-256) nezávisle přepočítány a shodují se.

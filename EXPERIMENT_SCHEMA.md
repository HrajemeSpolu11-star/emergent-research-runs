# EXPERIMENT_SCHEMA.md

## Účel tohoto dokumentu

Definuje přesný formát dat, která Observatoř (emergent-agent/observatory_v2)
zapisuje při záznamu experimentu, a přesný formát exportovaného balíčku
uloženého v tomto repozitáři. Tento dokument je zdroj pravdy pro strukturu
dat -- pokud se implementace a tento dokument rozejdou, opravuje se buď
implementace, nebo dokument (explicitně, s poznámkou proč), nikdy se
nenechává rozpor nezaznamenaný.

## Základní invariant: tři úrovně pravdy se nikdy neslévají

Každý zapisovaný event musí nést explicitní příznak, do které kategorie patří:

- `researcher_only: true/false` -- pokud `true`, obsahuje World Ground Truth
  nebo jiná data, která agent nikdy neviděl a nesmí vidět. Tato data smí
  číst pouze researcher/auditor, nikdy se nesmí dostat zpět do
  agent-facing cesty (perception/memory/cognition).
- `agent_visible: true/false` -- pokud `true`, jde o data, která agent
  skutečně dostal (post-EpistemicBoundary, post-Bridge překlad).

Toto přímo navazuje na invariant I.1 z `emergent-bridge/AGENTS.md`
("Žádný World Truth leak") a na existující `EpistemicBoundary.check_leaks()`
mechanismus v `emergent-agent`. Observatoř tento invariant NEPOROVNÁVÁ jen
opakovaně -- ona jej sama musí dodržet ve vlastním datovém modelu.

## Stav pokrytí (aktuální implementace vs. cílový návrh)

Tento schema dokument popisuje CÍLOVÝ, úplný design podle zadání. Aktuální
implementace Observatoře (viz `emergent-agent/observatory_v2/`) v okamžiku
prvního zápisu tohoto souboru pokrývá pouze podmnožinu -- WORLD, BODY,
SENSES (`sensors.vision/smell/sound` po filtraci), BRIDGE (mapování a
confidence) a ACTIONS/CONSEQUENCES. Skutečné kognitivní moduly (perception
pipeline, attention, working/episodic/semantic/autobiographical/procedural
memory, habit memory, belief store, world model, causal model, curiosity,
homeostasis drives, goals, planner, arbiter, experimental drive,
subconscious, guardian, meta-learning, neural/hybrid/adaptive/ensemble
provider, skill discovery/consolidation, replay, imagination/rollout,
prediction/confidence calibration, anomaly detection, self-model) běží
výhradně v Pythonu (`emergent_agent/*`) a v tomto webovém klientovi
NEJSOU reprodukované -- Observatoř v aktuálním stavu je proto nemůže
trasovat. Pole `module_trace` v exportu je pro tuto verzi buď prázdné,
nebo obsahuje pouze moduly, které Observatoř skutečně simuluje (World,
EpistemicBoundary, SensoryTrackAssociation, LocalTargetBridge). Každý
záznam v `module_trace.jsonl` musí mít pole `implemented_in_this_build:
true/false`, aby export nikdy nepředstíral trasování modulu, který
fyzicky neběžel.

## Adresářová struktura jednoho exportu

```
experiment_<timestamp>_<id>/
  manifest.json
  timeline.jsonl
  world_trace.jsonl
  observations.jsonl
  sensory_trace.jsonl
  body_trace.jsonl
  bridge_trace.jsonl
  module_trace.jsonl
  actions.jsonl
  consequences.jsonl
  belief_changes.jsonl
  memory_changes.jsonl
  causal_updates.jsonl
  skill_updates.jsonl
  errors.jsonl
  researcher_notes.md
  summary.md
  checksums.sha256
```

Soubory, pro které aktuální build Observatoře nemá zdroj dat (belief_changes,
memory_changes, causal_updates, skill_updates -- viz sekce výše), se přesto
VŽDY vytvoří, ale jako platný prázdný JSONL (0 řádků) s doprovodnou
poznámkou v `summary.md`, že daný typ trasování není v této verzi
implementovaný. Chybějící soubor by auditor nemohl odlišit od "nic se
nestalo"; prázdný soubor + poznámka v summary je pravdivé.

## `manifest.json`

```json
{
  "experiment_id": "string, UUID v4",
  "created_at": "ISO 8601 UTC timestamp",
  "schema_version": "1.0.0",
  "observatory_version": "v0.2",
  "seed": "integer, world seed",
  "agent_identity": "string nebo null (v0.2 nemá skutečného agenta)",
  "body_identity": "string nebo null",
  "session_identity": "string, UUID v4, unikátní per záznam",
  "commit_sha": {
    "emergent_agent": "string, git SHA nebo null pokud neznámo",
    "emergent_world": "string nebo null",
    "emergent_bridge": "string nebo null"
  },
  "test_protocol": "string, volný popis co se testovalo",
  "environment": {
    "runtime": "browser / node",
    "user_agent": "string nebo null"
  },
  "tick_range": {"first": 0, "last": 0},
  "event_count_by_file": {"world_trace.jsonl": 0, "...": 0},
  "researcher_only_fields_present": true,
  "agent_visible_fields_present": true
}
```

## Obecný formát jednoho řádku v `*.jsonl` souborech

Každý řádek je samostatný validní JSON objekt (JSON Lines formát).
Společná pole přítomná ve VŠECH trace souborech:

```json
{
  "seq": "integer, monotónně rostoucí, globálně unikátní v rámci experimentu",
  "tick": "integer, simulační krok (world.step)",
  "wall_time": "ISO 8601 UTC timestamp zápisu (ne simulačního času)",
  "researcher_only": true,
  "agent_visible": false,
  "provenance": {
    "source_module": "string, např. 'World.act' nebo 'LocalTargetBridge.resolve'",
    "source_tick": "integer"
  }
}
```

Specifická pole podle typu souboru navazují na existující datové
struktury už použité v `observatory_v2.html` a v Pythonu
(`emergent_agent/core/types.py`, `laboratory/world2.py`,
`perception/epistemic_boundary.py`, `perception/track_association.py`,
`bridge/local_target_resolution.py`) -- schema NEVYMÝŠLÍ nová pole tam,
kde už existují ověřená (např. `dx`, `dy`, `distance`, `color`, `shape`
pro vision percepty, `energy`/`hydration`/`integrity` pro tělo).

### `world_trace.jsonl` (researcher_only: true)

Plný World Ground Truth v daném ticku -- pozice agenta, kompletní
`world.objects` (včetně `pos`, `touch`, `effect`, které agent nikdy
nevidí), `world.obstacles`. Odpovídá tomu, co dnes zobrazuje panel
"Absolutní pravda světa" v Observatoři.

### `observations.jsonl` (agent_visible: true)

Observation PO `EpistemicBoundary.filterObservation()` -- to, co agent
skutečně dostal. Nikdy neobsahuje pole mimo allowlist
(`VISION_ALLOWED`/`SMELL_ALLOWED`/`SOUND_ALLOWED`).

### `sensory_trace.jsonl` (obsahuje jak researcher_only, tak agent_visible řádky)

Pro každý smyslový event: syrová hodnota (researcher_only), transformační
řetězec (jaký filtr/Bridge krok proběhl), a finální agent-facing
reprezentace (agent_visible). Toto je jediný soubor, kde se oba typy
řádků mohou objevit vedle sebe -- právě proto, aby šlo auditovat, ŽE
konkrétní pole bylo zablokováno, ne jen že "nějaká data existují".
Odpovídá `EpistemicBoundary.checkLeaks()` diffu už použitému v Observatoři.

### `body_trace.jsonl` (agent_visible: true)

`energy`, `hydration`, `integrity` per tick -- toto je proprioceptivní
stav agenta, legitimně agent_visible dle `EpistemicBoundary` (tělo se
nefiltruje).

### `bridge_trace.jsonl` (researcher_only: true)

Pro každou korelaci: `track_id`, `world_id` (POUZE zde, nikdy jinde
mimo world_trace), `confidence`, `last_seen_step`, `hits`, a pokud
proběhl pokus o `resolve()`: výsledek (`resolved` / `fail_closed`) a
důvod fail-closed stavu (`unknown_track` / `stale` / `low_confidence`).
Odpovídá `LocalTargetBridge.mapping_debug_view()`.

### `module_trace.jsonl`

Viz "Stav pokrytí" výše. Pole:
```json
{
  "module_name": "string",
  "implemented_in_this_build": true,
  "active": true,
  "input_summary": "string nebo null",
  "output_summary": "string nebo null",
  "processing_time_ms": 0,
  "passed_output_to": ["string modul"],
  "received_input_from": ["string modul"],
  "error": null,
  "fallback_used": false,
  "used_in_final_decision": true
}
```

### `actions.jsonl` / `consequences.jsonl`

`actions.jsonl`: co agent/researcher navrhl (`kind`, `target` --
agent-local track_id, nikdy World UUID -- `parameters`).
`consequences.jsonl`: `success`, `message`, delta tělesných hodnot,
a explicitní pole `bridge_resolution` popisující, na jaké (researcher-only)
World UUID se `target` přeložil, pokud vůbec.

### `belief_changes.jsonl`, `memory_changes.jsonl`, `causal_updates.jsonl`, `skill_updates.jsonl`

V aktuální verzi Observatoře prázdné (viz "Stav pokrytí") -- tyto
subsystémy žijí v Pythonu. Schema pro ně je připravené, aby export
zůstal stabilní formát i po budoucím rozšíření (Observatory v0.3+).

### `errors.jsonl`

Cokoliv, co selhalo v samotné Observatoři (ne v simulaci -- simulační
`success:false` patří do `consequences.jsonl`). Např. selhání
IndexedDB zápisu, selhání exportu, neplatný stav recorderu.

## `researcher_notes.md`, `summary.md`

Volný text. `summary.md` navíc MUSÍ obsahovat: co experiment dokazuje
a co nedokazuje (explicitně, aby export nebyl přeceňován při pozdějším
auditu -- viz `AUDIT_PROTOCOL.md`).

## `checksums.sha256`

SHA-256 každého souboru v adresáři, jeden řádek na soubor, formát
kompatibilní s `sha256sum -c`. Slouží k detekci poškození/úpravy
exportu po jeho vytvoření.

## Verzování schématu

Toto je `schema_version: 1.0.0`. Jakákoli změna struktury souborů
vyžaduje novou verzi zapsanou v `manifest.json` a záznam v tomto
souboru pod novou sekcí "Historie verzí" (zatím žádná, toto je první
verze).

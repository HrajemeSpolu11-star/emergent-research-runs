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

---

# DODATEK v1.1.0 (15.8.2026) -- Observatory v0.3: skutečná Python cognition telemetrie

## Nový zdroj dat: `CognitionTelemetryRecorder`

Observatory v0.3 přidává druhý, zcela nezávislý zdroj trace dat vedle v0.2
(World/Body/Senses/Bridge/Actions/Consequences, generovaných JS portem
v prohlížeči). Tento nový zdroj je `emergent_agent/telemetry/
cognition_events.py` -- read-only, opt-in instrumentace **skutečného**
Python `Agent.step()`, ne JS simulace.

**Klíčový rozdíl oproti v0.2 datům:** toto NENÍ živé spojení prohlížeč↔Python.
Prohlížeč nemá jak zavolat běžící Python proces. Data se generují dávkově
skriptem `emergent_agent/experiments/generate_cognition_trace.py`, který
spustí skutečného agenta a exportuje JSON bundle, který Observatoř
(záložka MYSL a odvozené panely) načte jako statickou, nahranou trace.

## Formát `CognitionEvent`

```json
{
  "seq": "integer, monotónně rostoucí v rámci recorderu",
  "tick": "integer, world.step v okamžiku události",
  "source_module": "string, např. 'Perception', 'EpisodicMemory', 'CausalGraph'",
  "event_type": "string, např. 'encode', 'retrieve', 'not_active_this_tick'",
  "visibility": "'agent_visible' | 'researcher_only' | 'internal_agent_state'",
  "input_summary": "dict nebo null -- bezpečně zkrácený souhrn vstupu, NIKDY syrový mutable Python objekt",
  "output_summary": "dict nebo null -- stejně",
  "active": "bool -- byl modul v tomto kroku skutečně aktivní",
  "error": "string nebo null",
  "wall_time": "float (unix timestamp) nebo null -- POUZE metadata, nikdy čteno zpět agentem, nemá vliv na determinismus trajektorie (viz noninterference testy)"
}
```

## Nová klasifikace `visibility` (rozšiřuje `researcher_only`/`agent_visible` z v1.0.0)

Přidána třetí kategorie **`internal_agent_state`**: data, která nejsou
World Truth (agent k nim nemá privilegovaný přístup odjinud), ale zároveň
nejsou "percept" v běžném smyslu -- jde o agentův vlastní vnitřní výpočet
(drives, novelty skóre, arbitrážní skóre, health report). Tato kategorie
existuje proto, že binární `agent_visible`/`researcher_only` z v1.0.0
nedostatečně popisovala tento typ dat -- nejsou to World data (takže ne
`researcher_only` ve smyslu "agent to nikdy neměl"), ale zároveň nejde o
přímý smyslový vjem (takže "agent_visible" by naznačovalo něco jiného).

## Export soubory (rozšíření `experiment_<id>/` adresáře z v1.0.0)

Pro Python cognition telemetrii, oddělené `*_trace.jsonl` soubory per modul
(ne jeden centrální soubor) -- viz `CognitionTelemetryRecorder.to_jsonl_by_module()`.
Aktuálně reálně generované moduly (ověřeno skutečným během, 60 kroků,
seed 42, provider=ensemble): `Perception, Attention, SensorFusion,
ChangeDetection, Curiosity, GoalSystem, EpisodicMemory, HypothesisEngine,
ObjectModel, SubconsciousSystem, SubconsciousGuardian, MetaLearningController,
AutomaticCurriculum, CognitionProvider, ExperimentalDrive, ActionArbiter,
Planner, StuckLoopBreaker (podmíněně), AgentDecision, Bridge, World,
PredictionError, IntrinsicReward, EpisodicMemory(add/decay), WorkingMemory,
ProceduralMemory, BeliefStore, TransitionModel, CausalGraph, HabitMemory,
SkillDiscovery, HierarchicalSkillLibrary, SkillConsolidator,
AutobiographicalMemory, SemanticMemory, OfflineConsolidator, Homeostasis`.

## Nedokázané / chybějící (explicitně, per princip tohoto dokumentu)

- **Žádný modul se nesmí v exportu tvářit jako aktivní, pokud aktivní
  nebyl.** `event_type: "not_active_this_tick"` je POVINNÝ explicitní
  zápis pro moduly, které v daném kroku neběžely (podmíněné moduly jako
  `SemanticMemory`, `CausalGraph`, `Planner` fallback, `OfflineConsolidator`)
  -- nikdy tichá mezera v datech.
- Pokud modul NIKDY nebyl aktivní v celé trace (např. `CausalGraph` v
  demonstrační 60-krokové trace, protože nedošlo k žádné interakci s
  pozorovaným trackem), Observatoř to zobrazuje jako **DATA NEDOSTUPNÁ**,
  ne jako prázdný/nulový graf.
- `SensorFusion`, `ChangeDetection`, `AutomaticCurriculum` se v aktuální
  implementaci `agent.py` počítají, ale jejich výstup nečte žádný cognition
  provider -- telemetrie je zaznamenává jako `not_active_this_tick` se
  zdůvodněním v poznámce, ne jako by ovlivňovaly rozhodování.

## Noninterference garance (nová, klíčová pro tento zdroj dat)

Na rozdíl od v0.2 (JS simulace, žádný "skutečný" agent k narušení),
Python cognition telemetrie se připojuje k reálnému rozhodovacímu procesu.
Proto platí a je automaticky testováno (`tests/test_telemetry_noninterference.py`
v `emergent-agent`):

```
telemetry_recorder=None  ==  telemetry_recorder=CognitionTelemetryRecorder()
```

na úrovni celé trajektorie (akce, odměny, prediction error, world state,
memory counts) při stejném seedu. Měřený reálný overhead (1000 kroků,
provider=ensemble): **+23.7 % relativní čas**, **~10.4 KB/krok** serializovaná
velikost (~99 MB projekce pro 10k kroků -- recorder zatím nemá omezení
velikosti, jde o nevyřešenou škálovací limitaci, ne o chybu).

---

# DODATEK v1.2.0 (15.8.2026) -- Observatory v0.3.1: correlation ID, telemetry režimy, truncation

## Correlation ID (částečně implementováno)

`CognitionEvent` nyní nese `event_id` (`"e:<tick>:<seq>"`, deterministické,
nekonzumuje RNG) a `decision_id` (`"d:<tick>"`, jeden na krok --
`Agent.step()` volá `recorder.begin_decision(tick)` na začátku).
**`parent_event_id` existuje v schématu, ale NENÍ v této fázi
populován** (vždy `None`) -- explicitní propojení výstup-modulu-A →
vstup-modulu-B napříč moduly je samostatný, větší návrhový úkol,
neimplementováno, nefingováno.

## Telemetry režimy (OFF/BASIC/DEBUG/FULL)

`CognitionTelemetryRecorder(level=...)`. OFF = `telemetry_recorder=None`
(nezměněno, 0% overhead). Změřený reálný overhead (1000 kroků, seed 42,
provider=ensemble), oproti OFF:

| režim | overhead | events/krok | bytes/krok |
|-------|----------|-------------|------------|
| BASIC | +6.7 %   | 6.00        | ~1983      |
| DEBUG | +18.3 %  | 37.00       | ~13217     |
| FULL  | +16.9 %  | 37.00       | ~13217     |

**Poctivá poznámka:** DEBUG a FULL v této implementaci produkují
IDENTICKÝ výstup -- rozdíl je zatím jen deklarovaný, ne funkčně
odlišený. To je otevřený nedodělek, ne skrytá vlastnost.
BASIC obsahuje jen `AgentDecision, World, PredictionError,
IntrinsicReward, Bridge` -- minimum pro rekonstrukci vnějšího tvaru
rozhodnutí u dlouhých běhů.

Noninterference ověřena pro všechny úrovně (`test_L_telemetry_levels...`)
-- identická trajektorie jako OFF u všech tří.

## Truncation (`max_events`)

`CognitionTelemetryRecorder(max_events=N)`. Po dosažení limitu se další
události EXPLICITNĚ zahazují (ne tiše) a `recorder.truncated=True` +
`recorder.truncation_reason` popisuje kdy/proč. Export musí toto pole
zahrnout do `manifest.json` (`trace_truncated: bool`) -- schema pole
existuje, export skript (`generate_cognition_trace.py`) zatím
`truncated`/`truncation_reason` do manifestu nekopíruje (otevřený
nedodělek, zapsáno do ROADMAP).

## Explicitně NEIMPLEMENTOVÁNO v této dílčí fázi (v0.3.1, tato aktualizace)

Kvůli velmi omezenému rozpočtu této konkrétní iterace NEBYLO v tomto kroku
provedeno (zůstává v `emergent-agent/ROADMAP.md` jako otevřené):
- Interaktivní klikací TOK INFORMACÍ graf (uzly/hrany s klikatelnými detaily)
- Panel ROZHODNUTÍ (český souhrn jednoho tick/akce)
- Diagnostický experiment cíleně aktivující CausalGraph
- CALCULATED/CONSUMED/IGNORED/NOT ACTIVE jemnější klasifikace (aktuálně jen aktivní/not_active_this_tick)
- Skutečný A/B multi-agent isolation test ve sdíleném World
- Export research run artifact do `emergent-research-runs/experiments/`
- `parent_event_id` populace (viz výše)

---

# DODATEK v1.2.0 (15.8.2026, pozdější) -- Observatory v0.3.1: correlation ID, telemetry režimy, truncation

## Correlation ID

`CognitionEvent` nyní nese `event_id` (`"e:<tick>:<seq>"`) a `decision_id`
(`"d:<tick>"`) -- oba deterministicky odvozené jen z už-existujících
hodnot (tick, seq), NIKDY z náhodného zdroje (`uuid4()` by čerpal z
`os.urandom`, čemuž jsme se záměrně vyhnuli, aby nevznikla ani teoretická
pochybnost o vlivu na determinismus). Všechny události patřící k jednomu
kroku (`agent.step()`) sdílejí stejné `decision_id` -- to je diagnostická
kotva pro rekonstrukci "co všechno patřilo k rozhodnutí v kroku N".

**Nedokončeno, poctivě přiznáno:** pole `parent_event_id` v schématu
existuje, ale v této etapě NENÍ naplňováno (vždy `null`). Explicitní
propojení výstup-modulu-A → vstup-modulu-B napříč moduly je samostatný,
větší návrhový úkol, ne fabrikace prázdného pole.

## Telemetry režimy

`CognitionTelemetryRecorder(level=...)`, čtyři úrovně:

- **OFF** = `telemetry_recorder=None` (beze změny od v0.3 -- nulový overhead, ověřeno)
- **BASIC** = jen `AgentDecision, World, PredictionError, IntrinsicReward, Bridge` (minimum pro dlouhé běhy)
- **DEBUG** = všechny moduly včetně `not_active_this_tick` padding
- **FULL** = totéž jako DEBUG

**Poctivý nález:** DEBUG a FULL v aktuální implementaci produkují
IDENTICKÝ výstup (ověřeno: 500 kroků, oba 18502 událostí). Odlišení
úrovní zůstalo jen na úrovni "zahrnout/nezahrnout not_active padding"
(OFF/BASIC vs DEBUG/FULL), ne na úrovni obsahové hloubky (např. plnější
`input_summary`/`output_summary` pro FULL). Toto je zaznamenaná mezera
pro příští iteraci, ne skrytá.

**Změřený reálný overhead** (500 kroků, provider=ensemble, oproti OFF baseline):
```
BASIC : +4.9 %   (6.0 událostí/krok,  ~1979 B/krok)
DEBUG : +17.0 %  (37.0 událostí/krok, ~13175 B/krok)
FULL  : +19.3 %  (37.0 událostí/krok, ~13175 B/krok)
```

## Omezení růstu dat (truncation)

`CognitionTelemetryRecorder(max_events=N)` -- při dosažení limitu se
další `record()` volání tiše NEZAHODÍ bez záznamu: nastaví se
`recorder.truncated = True` a `recorder.truncation_reason` (např.
`"max_events=100 reached at tick 2"`). Export musí tato pole zahrnout
do `manifest.json` (`trace_truncated: true` + důvod), aby žádný
auditor neinterpretoval neúplnou trace jako úplnou. Ověřeno testem M
(`test_M_truncation_explicit_not_silent`).

**Nedokončeno:** `max_bytes` limit, chunking a rolling buffer (viz
zadání bod 10) NEJSOU implementovány -- pouze `max_events` (počet
událostí). Zaznamenáno jako otevřený bod, ne fabrikováno.

## Noninterference po těchto změnách

Znovu ověřeno (test A + nový test L): telemetrie ON (libovolná úroveň
BASIC/DEBUG/FULL) vs OFF -- trajektorie identická. `begin_decision()`
a `not_active()` gating nepřidávají žádný nový zdroj nedeterminismu.

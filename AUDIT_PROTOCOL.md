# AUDIT_PROTOCOL.md

## Účel

Popisuje, jak se s exportovaným experimentem (viz `EXPERIMENT_SCHEMA.md`)
má nakládat POTÉ, co byl uložen sem do `emergent-research-runs`. Tlačítko
"ODESLAT K AUDITU" v Observatoři pouze PŘIPRAVÍ data v tomto formátu --
samo o sobě nespouští žádnou AI ani lidskou analýzu. Toto je záměrné a
kritické rozlišení: tlačítko nesmí předstírat, že proběhl audit, když
proběhla jen příprava dat.

## Co tlačítko "ODESLAT K AUDITU" skutečně dělá

1. Uzavře aktuální záznam experimentu (pokud ještě běží).
2. Vypočítá `checksums.sha256` pro všechny soubory.
3. Vyplní `manifest.json` (viz schema).
4. Vytvoří ZIP balíček ke stažení / k nahrání.
5. Nastaví stav experimentu v UI na `EXPORTOVÁNO` (ne `AUDITOVÁNO`,
   ne `OVĚŘENO` -- tyto stavy nastává až člověk nebo skutečný audit
   proces, viz níže).

Tlačítko NEVOLÁ žádné AI API, negeneruje žádný "audit report" automaticky.
Pokud v budoucnu vznikne skutečný automatizovaný audit backend, musí to
být zaznamenáno zde jako nová sekce s přesným popisem, co dělá a co ne.

## Stavový automat exportu

```
NAHRÁVÁ SE (recording)
  → POZASTAVENO (paused)
  → UKONČENO (stopped, ještě neexportováno)
  → EXPORTOVÁNO (ZIP vytvořen, checksums spočítány)
  → NAHRÁNO (Claude/vlastník nahrál ZIP do emergent-research-runs)
  → OVĚŘENO (remote verify: soubor na GitHubu odpovídá checksums)
```

Žádný krok se nesmí přeskočit v UI popisu stavu. Pokud Observatoř
neumí sama zapisovat do GitHubu (viz `DATA_RETENTION.md`, sekce o
GitHub tokenu), stavy NAHRÁNO/OVĚŘENO nastavuje až následná ruční
nebo Claude-asistovaná synchronizace, nikdy prohlížeč sám.

## Co má forenzní audit nad exportovanými daty zkontrolovat

Podle zadání, minimálně:

1. **Timeline konzistence** -- `seq` je striktně rostoucí, `tick` je
   monotónní, žádné mezery neodpovídající očekávanému chování recorderu
   (pauza je v pořádku, ale musí být zaznamenaná).
2. **Errors** -- projít `errors.jsonl`, zkontrolovat, že žádná chyba
   nebyla tichá (tj. že se propsala i do `consequences.jsonl` nebo
   `summary.md`, pokud ovlivnila výsledek).
3. **Anomálie** -- statisticky neobvyklé skoky v `body_trace.jsonl`,
   nekonzistence mezi `world_trace` a `observations` mimo očekávaný
   filtr.
4. **World Truth leaks** -- pro KAŽDÝ řádek v `observations.jsonl`
   a `sensory_trace.jsonl` (agent_visible: true řádky) ověřit, že
   neobsahuje žádné pole mimo allowlist definovaný v
   `EXPERIMENT_SCHEMA.md` / `EpistemicBoundary`. Toto je strojově
   automatizovatelná kontrola (stejná logika jako
   `EpistemicBoundary.checkLeaks()`), měla by být prvním krokem
   každého auditu.
5. **Failed bindings** -- v `bridge_trace.jsonl` dohledat všechny
   `fail_closed` výsledky a ověřit, že odpovídající `actions.jsonl`/
   `consequences.jsonl` záznam skutečně reportuje neúspěch (žádná
   akce nesměla "protéct" bez platného Bridge překladu tam, kde ho
   vyžadovala).
6. **Stale tracks** -- v `bridge_trace.jsonl` dohledat mapování, kde
   `last_seen_step` je starší než `max_staleness_steps` v okamžiku
   použití, a ověřit že `resolve()` v takovém případě vrátilo
   fail-closed, ne starou hodnotu.
7. **Module communication** -- pokud `module_trace.jsonl` obsahuje
   záznamy s `implemented_in_this_build: true`, ověřit deklarovaný
   tok `passed_output_to`/`received_input_from` odpovídá tomu, co
   experiment skutečně dělal (cross-check proti `timeline.jsonl`).
8. **Memory mutations / causal changes** -- pokud jsou přítomné
   (Observatory v0.3+), ověřit že každá mutace má provenance a že
   žádná mutace neobsahuje World Truth pole.
9. **Action consequences** -- pro každý řádek v `actions.jsonl` musí
   existovat odpovídající řádek v `consequences.jsonl` se stejným
   `seq`/`tick` párováním; chybějící consequence je chyba exportu,
   ne tichá mezera.
10. **Determinism** -- pokud byl experiment spuštěný se stejným
    `seed` jako dřívější experiment, worldové řádky by měly být
    reprodukovatelné; audit může (ale nemusí mít k dispozici) druhý
    běh pro porovnání.

## Kdo smí auditovat

Claude, ChatGPT, Codex nebo člověk (Michal). Žádný z automatizovaných
nástrojů nemá autoritu prohlásit experiment za "ověřený" bez toho, aby
prošel body 1-10 výše a zapsal nález (i negativní -- "nic nenalezeno")
zpět do `summary.md` daného experimentu nebo do samostatného
`AUDIT_FINDINGS_<experiment_id>.md` v tomto repozitáři.

## Co export experimentu NEDOKAZUJE

Musí být explicitně řečeno (a per-experiment opakováno v `summary.md`
daného běhu), že:

- Úspěšný běh bez zachycených leaků NEDOKAZUJE, že `EpistemicBoundary`
  je vyčerpávající pro všechny budoucí typy World dat -- pouze že
  aktuální implementace nezachytila leak v TOMTO konkrétním běhu.
- Observatory v0.2 nemá skutečného kognitivního agenta -- žádný export
  z této verze NEDOKAZUJE nic o vlastnostech beliefs/memory/causal
  learning, protože tyto subsystémy v tomto buildu neběžely.
- Determinismus jednoho běhu NEDOKAZUJE determinismus napříč platformami
  (viz otevřená otázka v `emergent-agent/ROADMAP.md` o GPU/CPU
  reprodukovatelnosti).

# PROJECT_MEMORY.md

## Založení repozitáře

15.8.2026, na explicitní pokyn vlastníka projektu (Michal), v rámci
úkolu "RESEARCH OBSERVATORY + EXPERIMENT MEMORY + CROSS-REPO TELEMETRY".
Zadání požadovalo samostatný repozitář pro dlouhodobé experimentální
artefakty, aby se nezahlcovaly `emergent-agent`, `emergent-world` ani
`emergent-bridge` velkými raw logy. Vytvořeno přes GitHub API Claude
Sonnetem (hlavní GitHub integrátor projektu), token stejný jako pro
ostatní repozitáře projektu.

## Rozhodnutí a jejich důvody

### Proč JSONL, ne jeden velký JSON

Zadání explicitně specifikuje `.jsonl` soubory (`timeline.jsonl`,
`world_trace.jsonl`, atd.) místo jednoho velkého JSON pole. Důvod: JSONL
lze appendovat řádek po řádku bez nutnosti parsovat a znovu serializovat
celý dosavadní obsah -- důležité pro dlouhé experimenty s velkým počtem
eventů (viz požadavek na test "velmi dlouhý experiment", "vysoký počet
eventů" v zadání).

### Proč jsou belief/memory/causal/skill trace soubory prázdné v této verzi

Observatory v0.2 (viz `emergent-agent/observatory_v2/observatory_v2.html`)
je nezávislý JS port World + EpistemicBoundary + SensoryTrackAssociation +
LocalTargetBridge -- NEobsahuje skutečného kognitivního agenta. Beliefs,
paměť, kauzální učení žijí výhradně v Pythonu
(`emergent_agent/agent.py` + `world_model/*`). Export proto tyto
soubory vytváří jako platné prázdné JSONL s vysvětlující poznámkou,
místo aby je vynechal (chybějící soubor by auditor nemohl odlišit od
"nic se nestalo") nebo je fabrikoval prázdnými/vymyšlenými daty.
Toto je přímé pokračování principu už zavedeného v `observatory_v2.html`
("Rozsah v0.2" poznámka na stránce) a v `emergent-agent/CURRENT_STATE.md`
(zápis z 15.8.2026 o Observatory v0.2).

### Proč Observatoř sama nemá GitHub token

Viz `DATA_RETENTION.md`, sekce "GitHub upload -- bezpečnostní rozhodnutí".
Observatoř běží staticky přes GitHub Pages -- veřejně čitelný klientský
kód. Jakýkoli token vložený do JavaScriptu by byl okamžitě čitelný
komukoli. Export je proto lokální ZIP, nahrání do tohoto repozitáře
provádí Claude přes svůj vlastní bezpečně uložený přístup.

## Otevřené otázky / co zatím není rozhodnuto

- Automatizovaný retention/expiraci mechanismus pro staré experimenty
  -- zatím žádný (viz `DATA_RETENTION.md`).
- Zda a jak přesně bude v budoucnu fungovat automatizovaný (AI) audit
  backend zmíněný jako možnost v `AUDIT_PROTOCOL.md` -- zatím
  neimplementováno, tlačítko "ODESLAT K AUDITU" pouze připravuje data.
- Formát pro Observatory v0.3 (živé beliefs/memory/causal trace) --
  schema v `EXPERIMENT_SCHEMA.md` je připravené (pole existují), ale
  nebylo zatím reálně naplněno daty, protože v0.3 neexistuje.

## Jazyk a hloubka

Tento repozitář následuje stejnou konvenci jako `emergent-agent`:
`ROADMAP.md`, `PROJECT_MEMORY.md`, `CURRENT_STATE.md` se udržují v
češtině, podrobně a raději redundantně než stručně, aby nový chat/
nová osoba dokázala obnovit kontext bez přístupu k původní konverzaci.

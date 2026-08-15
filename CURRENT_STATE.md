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

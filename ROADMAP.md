# ROADMAP.md

## Fáze 1 -- Založení repozitáře a schéma (HOTOVO 15.8.2026)

- [x] Vytvořit repozitář `emergent-research-runs`
- [x] `README.md`, `EXPERIMENT_SCHEMA.md`, `AUDIT_PROTOCOL.md`,
      `DATA_RETENTION.md`, `PROJECT_MEMORY.md`, `CURRENT_STATE.md`,
      `OPEN_SOURCE_REGISTRY.md`

## Fáze 2 -- Recorder v Observatoři (viz emergent-agent repo)

- [ ] Implementace recorderu (ZAČÍT/POZASTAVIT/UKONČIT ZÁZNAM,
      PŘIDAT POZNÁMKU, ULOŽIT SNAPSHOT) v `observatory_v2.html`
- [ ] IndexedDB persistence, obnova po refreshi, zobrazení využité
      velikosti
- [ ] Export do formátu podle `EXPERIMENT_SCHEMA.md` (ZIP)
- [ ] Tlačítko "ODESLAT K AUDITU" podle `AUDIT_PROTOCOL.md`

## Fáze 3 -- První reálný export a nahrání sem

- [ ] První skutečný experiment exportovaný z Observatoře
- [ ] Nahrání Claude přes bezpečný přístup (ne z klientského JS)
- [ ] Ověření checksums po nahrání (`REMOTE VERIFIED`)

## Fáze 4 -- Rozšíření pokrytí (Observatory v0.3+)

- [ ] Skutečný kognitivní agent (Python beliefs/memory/causal) dostupný
      k trasování -- vyžaduje buď backend proces, nebo velký JS port
      (viz otevřená otázka v `emergent-agent/ROADMAP.md`)
- [ ] Naplnění `belief_changes.jsonl`, `memory_changes.jsonl`,
      `causal_updates.jsonl`, `skill_updates.jsonl` skutečnými daty

## Fáze 5 -- Automatizovaný audit (nejisté, budoucnost)

- [ ] Rozhodnout, zda a jak bude fungovat automatizovaný audit backend
      (viz `AUDIT_PROTOCOL.md`, otevřená otázka)

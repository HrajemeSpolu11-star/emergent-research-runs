# DATA_RETENTION.md

## Vlastnictví dat (data ownership)

Všechna data v tomto repozitáři jsou experimentální/syntetická -- pocházejí
z běhu simulovaného `MultisensoryLaboratory` světa a diagnostického
nástroje Observatoř. Neobsahují osobní údaje žádné reálné osoby. Vlastníkem
projektu a těchto dat je Michal (majitel repozitářů `emergent-*`).

## Bezpečnostní hranice (security boundary)

**Kritický invariant, navazující na `emergent-bridge/AGENTS.md` invariant
I.1:** Data v tomto repozitáři SMÍ obsahovat World Ground Truth (World
UUID, plné pozice, `touch`, `effect` pole) v souborech označených
`researcher_only: true` (viz `EXPERIMENT_SCHEMA.md`). To je v pořádku --
tento repozitář je určen výzkumníkovi/auditorovi, ne agentovi.

Co NENÍ v pořádku a nesmí se stát:
- Žádný soubor z tohoto repozitáře se nesmí zpětně načíst do
  agent-facing cesty (perception/cognition) v `emergent-agent` nebo
  `emergent-bridge` runtime. Tento repozitář je čistě WORM-style archiv
  (write-once, read-many) pro forenzní účely, nikdy zdroj dat pro
  běžící agentskou smyčku.
- GitHub token použitý k zápisu do tohoto repozitáře se nikdy nesmí
  objevit v klientském JavaScriptu Observatoře (viz sekce "GitHub
  upload" níže) -- Observatoř běží přes GitHub Pages, tedy veřejně
  čitelná kýmkoli.

## Retention policy

- Experimenty se v tomto repozitáři NEMAŽOU automaticky. Ruční smazání
  (`ODESLAT K AUDITU` opak -- lokální `VYMAZAT LOKÁLNÍ ZÁZNAM` maže
  pouze lokální IndexedDB kopii v prohlížeči, ne cokoliv už nahrané
  sem) provádí vlastník repozitáře nebo Claude na jeho výslovný pokyn.
- Velké raw trace soubory NEPATŘÍ do `emergent-agent`, `emergent-world`
  ani `emergent-bridge` -- to je přesně důvod existence tohoto
  odděleného repozitáře (viz zadání úkolu, bod 2).
- Neexistuje (zatím) automatický retention/expiraci mechanismus (např.
  mazání po N dnech) -- pokud bude potřeba, zapíše se sem jako nová
  sekce s přesným pravidlem a datem zavedení.

## Prohlížečová persistence (IndexedDB) -- vztah k retention

Observatoř používá IndexedDB (ne LocalStorage -- LocalStorage má
limit ~5-10 MB a je synchronní/blokující, nevhodné pro objemné trace
soubory) pro dočasné uložení NEDOKONČENÉHO experimentu v prohlížeči,
aby přežil obnovení stránky. Toto je odlišná vrstva od retention v
tomto Git repozitáři:

- IndexedDB záznam = dočasný, per-prohlížeč, per-zařízení, nikdy
  synchronizovaný automaticky.
- Tento Git repozitář = trvalý, sdílený, verzovaný archiv EXPORTOVANÝCH
  experimentů.

Observatoř musí uživateli jasně zobrazit využitou velikost IndexedDB a
nedovolit neomezený růst bez upozornění (viz požadavek v zadání, bod 11).

## GitHub upload -- bezpečnostní rozhodnutí

Protože Observatoř běží staticky přes GitHub Pages (žádný vlastní
server), nemá bezpečné místo, kam uložit zapisovací GitHub token.
Proto:

- Observatoř klientsky NIKDY neobsahuje GitHub token.
- Export z Observatoře je lokální ZIP stažený do zařízení uživatele.
- Nahrání ZIPu do tohoto repozitáře provádí Claude (přes svůj vlastní,
  bezpečně uložený token mimo klientský kód) na explicitní pokyn
  vlastníka, nebo vlastník sám ručně.
- Observatoř po exportu zobrazuje stav `EXPORTOVÁNO`, nikdy
  `NAHRÁNO`/`OVĚŘENO`, dokud k nahrání/ověření skutečně nedojde (viz
  `AUDIT_PROTOCOL.md`, stavový automat).

## Co se v tomto repozitáři NIKDY neverzuje

- Žádné API klíče, tokeny, credentials.
- Žádná binární data mimo to, co export skutečně vyžaduje (ZIP
  jednotlivých experimentů, jejich JSONL/JSON/MD obsah).

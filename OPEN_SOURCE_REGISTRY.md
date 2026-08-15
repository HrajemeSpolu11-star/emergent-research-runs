# OPEN_SOURCE_REGISTRY.md

## Policy

Stejná politika jako v `emergent-agent/OPEN_SOURCE_REGISTRY.md`: toto je
rozhodovací/výzkumný registr, ne důkaz že závislost je vendorovaná nebo
přijatá. Licenční kompatibilita je tvrdá hranice.

## Aktuální stav

Ke dni založení (15.8.2026) tento repozitář neobsahuje žádný spustitelný
kód s externími závislostmi -- pouze dokumentaci a (v budoucnu) statická
exportovaná data.

Pokud recorder v Observatoři (`emergent-agent/observatory_v2.html`)
použije pro generování ZIP souborů externí JS knihovnu (např. pro
kompresi), musí být zde zaznamenána před nasazením do produkce, se
stejnými náležitostmi jako v `emergent-agent/OPEN_SOURCE_REGISTRY.md`:
přesný projekt/revize, licence/SPDX, zdrojová URL, použité
soubory/komponenty, úpravy, udržovanost.

V první implementaci (viz `emergent-agent/CURRENT_STATE.md` pro aktuální
stav recorderu) je preferovaná vlastní minimální implementace ZIP
writeru (store/bez komprese) přímo v projektu, aby se předešlo závislosti
-- pokud se to změní, zapíše se sem.

---
# SPDX-License-Identifier: CC-BY-SA-4.0
#  Copyright (C) 2020 Casper Meijn <casper@meijn.net>
#  This work is licensed under the Creative Commons Attribution-ShareAlike 4.0 International License. 
#  To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/4.0/ or 
#  send a letter to Creative Commons, PO Box 1866, Mountain View, CA 94042, USA.
title: "BUVA Q-Stream afzuiging deel 2"
date: 2024-02-25T00:00:00Z
categories:
- Home Assistant
---
Twee jaar geleden schreef ik in [deel 1]({{< ref "/posts/buva-q-stream.md" >}}) over mijn BUVA Q-Stream afzuiging. Inmiddels is deze setup deels veranderd. Ik heb de schakeling op het breadboard vervangen door een zelfgemaakte printplaat en de luchtvochtigheid sensor is vervangen door een koppeling met de cv-ketel.

<!--more-->

## Problemen met vorige versie

De vorige versie werkte goed, maar had zijn gebreken. Af en toe viel de verbinding tussen de afzuiging en Home Assistant weg, na een reboot van de ESP kwam de verbinding weer terug. Uiteindelijk ben ik er achter gekomen dat de voeding van 15 volt het probleem is. Volgens de documentatie van het ESP32 bordje dat ik gebruik is een voeding van 15 volt toegestaan, maar de chip naast de voedingspin werd erg warm. Mijn conclusie is dat de ESP32 oververhit raakt met een hoge voedingsspanning. Ik ga dit oplossen door de spanning terug te brengen naar 5 volt. Hiervoor gebruik ik een LM7805, omdat ik deze nog in de kast heb liggen.

Een ander probleem was de batterij van de luchtvochtigheid sensor. Deze liep in 2 maanden leeg en elke keer vervangen vind ik geen goede oplossing. Ik heb dit opgelost door mijn cv-ketel uit te lezen. Als de ketel 1 minuut warm water levert, dan ben ik waarschijnlijk aan het douchen. Ik heb de koppeling met de cv-ketel gemaakt door middel van de [OpenTherm Gateway](https://www.nodo-shop.nl/nl/aanbevolen/211-opentherm-gateway.html). Ik ga daar in dit verhaal niet verder op in.

Verder kon mijn vorige schakeling slechts twee standen aan (0 volt en 5 volt). Ik wil dit uitbreiden naar iedere spanning tussen de 0 en 10 volt. De meeste microcontrollers hebben een DAC, hiermee kan je een analoge spanning maken. Voor de ESP32 is dit een spanning tussen 0 en de 3,3 volt. Dus ik heb een versterker nodig om de spanning te vermenigvuldigen met een factor 3.

## Historie

In mijn nieuwbouwwoning zit een BUVA Q-Stream afzuiging. Deze wordt met een afstandsbediening kastje bediend. Het kastje hing in de woonkamer, wordt met 240 volt gevoed en stuurt draadloos het signaal naar de afzuiging. De afzuiging zelf bevindt zich op zolder en zuigt in de keuken, badkamer en de zolder aan.

De afstandsbediening heeft een paar modi die de afzuiging slim kunnen schakelen. De meeste modi zijn tijd gebaseerd en ruim configureerbaar. Echter, de bediening is erg slecht: de knopjes registreren je vinger slecht en de tijd instellen moet je per minuut doorheen klikken. Plus het apparaat vergeet al zijn instellingen als het spanningsloos is geweest.

Wij gebruikte het kastje enkel in handmatige modus. Meestal staat hij op stand 1 (zacht op de achtergrond) en na het douchen op stand 3 (redelijke kracht).

Het probleem met handmatig is: ik vergeet de afzuiging aan te zetten voor het douchen en mijn vrouw vergeet het uit te zetten. Dus besluit ik de bediening van de afzuiging (met een ingewikkelde oplossing) simpel te maken.

## Afzuiging aansturen

In de [handleiding](https://moam.info/installatiehandleiding-q-stream-buva_5a2edc321723dd74b6322777.html) van de afzuiging is te vinden dat het zowel via de originele bediening als via een 0-10 volt signaal bedient kan worden.

Op de printplaat zitten drie DIP-switches. De eerste is voor het inschakelen van de 0-10 volt sturing.

- Omlaag is voor de originele bediening
- Omhoog is voor de 0-10 volt aansturing

![Binnenkant afzuiging](/afzuiging-binnenkant.jpg)

Ik heb experimenteel vastgesteld welke spanning gelijk staat aan iedere stand van de originele bediening:

| Origineel | 0-10 volt |
|-----------|-----------|
| Stand 1   | 0 volt    |
| Stand 2   | 2 volt    |
| Stand 3   | 5 volt    |
| Stand 4   | 7 volt    |

## Printplaat

Het wordt lastig om mijn nieuwe ideeën toe te voegen aan de bestaande schakeling. Ik heb wel eens gehoord van goedkope productie van PCB's, dus ik wil mijn eigen printplaat maken! Tijdens mijn opleiding elektrotechniek heb ik wel eens een printplaat ontworpen. Tegenwoordig bestaat er een uitgebreid open-source pakket om dit te doen: [KiCad](https://flathub.org/apps/org.kicad.KiCad). Ik ben gekomen tot het volgende schema:

![Schema afzuiging](/afzuiging2-schema.webp)

Links staat de ESP32 devkit. Bovenin de voedingsstekker met de LM7805 voor de juiste spanning. Rechts staan de versterkers met een uitgangsconnector.

Ik heb de PCB besteld op https://jlcpcb.com/ voor €10 kreeg ik 5 PCBs, thuisbezorgd. De componenten had ik nog liggen of los besteld. Het is via lokale webwinkels verkrijgbaar of op AliExpress.

Componenten:

- ESP32 devkit V1
- LM7805, throughhole
- NE5532 comperator
- Enkele 10K en 30K weerstanden
- Enkele 0.1uF condensatoren
- 3 connectoren

De bron van de schema's en de gerber bestanden voor het bestellen zijn beschikbaar op: https://github.com/caspermeijn/pcb/tree/main/afzuiging

![Print afzuiging](/afzuiging2-print.jpg)

De schroefgaten op de print komen overeen met de ongebruikte gaten in de BUVA. Ik heb de kabeltjes via de buitenrand van de kast laten lopen.

![Overzicht afzuiging](/afzuiging2-overzicht.jpg)

## ESPHome

Voor de software op de ESP32 heb ik gekozen voor [ESPHome](https://esphome.io/). Dit is fantastische software. Je maakt een YAML-configuratie bestand en er wordt een custom firmware voor je gemaakt. Het heeft geweldige integratie met Home Assistant, waardoor bediening direct beschikbaar is. Het updaten van de firmware gaat via de HA interface met een enkele klik.

Ik heb het volgende blok aan de standaard ESPHome config toegevoegd om de gewenste functionaliteit te krijgen:

```
output:
  - platform: esp32_dac
    pin: GPIO25
    id: dac1_output

fan:
  - platform: speed
    output: dac1_output
    name: "Afzuiging Fan"
```

Na een eenmalige installatie van de firmware via USB, worden updates via wifi geïnstalleerd. De afzuiging is als ventilator beschikbaar in Home Assistant. Nu kan je een automatisering maken die een sensor leest en de afzuiging aanzet.

![Afzuiging in Home Assistant](/afzuiging2-home-assistant.png)

## Conclusie

Als je behoorlijk wat tijd besteed aan het ontwerpen van een printplaat, de componenten verzamelt, de print soldeert en een configuratie bestand maakt, dan kan je een BUVA afzuiging aansturen via Home Assistant.

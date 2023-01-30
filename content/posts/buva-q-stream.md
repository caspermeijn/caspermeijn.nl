---
# SPDX-License-Identifier: CC-BY-SA-4.0
#  Copyright (C) 2021 Casper Meijn <casper@meijn.net>
#  This work is licensed under the Creative Commons Attribution-ShareAlike 4.0 International License. 
#  To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/4.0/ or 
#  send a letter to Creative Commons, PO Box 1866, Mountain View, CA 94042, USA.
title: "BUVA Q-Stream afzuiging via Home Assistant"
date: 2022-01-03T12:30:00+01:00
categories:
- Home Assistant
---

In mijn nieuwbouw woning zit een BUVA Q-Stream afzuiging. Deze wordt met een afstandsbediening kastje bediend. Het kastje hing in de woonkamer, wordt met 240 volt gevoed en stuurt draadloos het signaal naar de afzuiging. De afzuiging zelf bevind zich op zolder en zuigt in de keuken, badkamer en de zolder aan. 

De afstandsbediening heeft een paar modi die de afzuiging slim kunnen schakelen. De meeste zijn tijd gebaseerd en ruim configureerbaar. Echter de bediening is erg slecht: de knopjes registreren je vinger slecht en de tijd instellen moet je per minuut doorheen klikken. Plus het apparaat vergeet al zijn instellingen als het spanningsloos is geweest.

Wij gebruikte het kastje enkel in handmatige modus. Meestal staat hij op stand 1 (zacht op de achtergrond) en na het douchen op stand 3 (redelijke kracht). 

Het probleem met handmatig is: ik vergeet de afzuiging aan te zetten voor het douchen en mijn vrouw vergeet het uit te zetten. Dus besluit ik de bediening van de afzuiging (met een ingewikkelde oplossing) simpel te maken.

## Luchtvochtigheid

Ik heb een [Shelly H&T](https://shelly.cloud/products/shelly-humidity-temperature-smart-home-automation-sensor/) aangeschaft. Deze sensor meet de luchtvochtigheid en temperatuur en rapporteert deze via WiFi. Het werkt op batterijen en staat boven op de badkamerkast.

[Home Assistant](https://www.home-assistant.io/integrations/shelly/) heeft goede integratie voor Shelly apparaten. Het apparaat wordt bijna vanzelf gevonden. Het is wel nog nodig om het Home Assistent ip-adres in de Shelly configuratie te zetten (zoals beschreven in de documentatie). 

## Afzuiging aansturen

In de [handleiding](https://moam.info/installatiehandleiding-q-stream-buva_5a2edc321723dd74b6322777.html) van de afzuiging is te vinden dat het zowel via de originele bediening als via een 0-10 volt signaal bedient kan worden. 

Op de printplaat zitten drie DIP switches. De eerste is voor het inschakelen van de 0-10 volt sturing.
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

## Schakeling

De afzuiging bied een 15 volt voeding aan en verwacht een 0-10 volt signaal terug. Ik heb met een pwm geëxperimenteerd, maar dit was geen succes. Daarna ben ik met Zener diodes aan de slag gegaan. Het plan was om met meerdere mosfets de verschillende spannings niveaus te kunnen kiezen, maar ook dit was geen success.

Aangezien wij altijd stand 1 en 3 gebruiken heb ik er voor gekozen om met een 5 volt Zener diode de juiste spanning te maken en deze met een mosfet te schakelen. Hierdoor kan je tussen 0 en 5 volt schakelen. Helaas had ik een tweede mosfet nodig, omdat het stuursignaal slechts 3,3 volt is.

![Schema afzuiging](/afzuiging-schema.svg)

Het stuursignaal komt van een [ESP32 Devkit V1](https://github.com/playelek/pinout-doit-32devkitv1) met [Tasmota](https://tasmota.github.io/docs/) software. Als Tasmota geïnstalleerd is, dan fungeert het in eerste instantie als een WiFi AP waar je mee kan verbinden. Via de website van het apparaat kies je een WiFi netwerk, daarna verbind het met de gekozen WiFi. Op het nieuwe IP adres kan je het verder configureren. In dit geval koos ik voor `Relay_i` (inverted, omdat door de dubbele mosfet het stuursignaal andersom werkt). Vervolgens stel je MQTT in met Home Assistent als server en dan wordt het vanzelf beschikbaar als schakelaar binnen Home Assistant.

![Schakeling afzuiging](/afzuiging-schakeling.jpg)

## Automatisering

Ik heb in Home Assistent een automatiseringsregel gemaakt:
> Als de luchtvochtigheid gelijk is aan 100% dan moet de afzuiging aan. Als de luchtvochtigheid voor een half uur onder de 100% zit, dan mag deze weer uit.

Ik heb de grafische configurator gebruikt om dit te maken, maar de code is makkelijker te delen:

```yaml
id: '1639934542987'
alias: Badkamer luchtvochtigheid
description: ''
trigger:
  - platform: numeric_state
    entity_id: sensor.shelly_badkamer_humidity
    above: '99'
condition: []
action:
  - type: turn_on
    device_id: 6da598939eaf5659ca063cc92ea0d43c
    entity_id: switch.afzuiging
    domain: switch
  - wait_for_trigger:
      - platform: numeric_state
        entity_id: sensor.shelly_badkamer_humidity
        for:
          hours: 0
          minutes: 30
          seconds: 0
        below: '99'
  - type: turn_off
    device_id: 6da598939eaf5659ca063cc92ea0d43c
    entity_id: switch.afzuiging
    domain: switch
mode: single
```

## Conclusie

Na maanden fantaseren en meerdere dagen experimenteren, hoef ik nooit meer om de afzuiging te denken als ik ga douchen.

Totdat de batterij op is, of de Raspberry Pi er mee stopt, of het WiFi bereik net te weinig blijkt. Maar dat is een probleem voor future-me.

![Overzicht afzuiging](/afzuiging-overzicht.jpg)

---
# SPDX-License-Identifier: CC-BY-SA-4.0
#  Copyright (C) 2024 Casper Meijn <casper@meijn.net>
#  This work is licensed under the Creative Commons Attribution-ShareAlike 4.0 International License. 
#  To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/4.0/ or 
#  send a letter to Creative Commons, PO Box 1866, Mountain View, CA 94042, USA.
title: "Rust Meets C++"
date: 2024-04-15T00:00:00Z
categories:
- Rust
- Meetup
---

Afgelopen week was ik bij een meetup op het kantoor van [True Legends IT Professionals](https://www.linkedin.com/company/truelegends-it-professionals/) met als thema "Rust meets C++". Deze meetup had als onderwerp: Hoe integreer je Rust in een bestaande C++ codebase?

Als ik terugdenk aan de vragen die daar gesteld werden valt mij één ding op: de meeste vragen gingen over memory management, terwijl Rust die zorg juist grotendeels wegneemt.

TLDR: Ik werk nu zes maanden als fulltime Rust developer en ik heb in die tijd bijna niet over memory management nagedacht.

<!--more-->

Sinds zes maanden ben ik gedetacheerd via True Legends als Full Time Rust Developer. Dit is mijn eerste baan waarin ik voornamelijk in de programmeertaal Rust werk. Hiervoor heb ik professioneel wel eens een Rust side project gedaan en ik heb uitgebreide hobby ervaring met Rust.

Ik ervaar de taal Rust als heel betrouwbaar. Als ik een stuk code geschreven heb, de compiler keurt het goed en ik heb een paar unit testen voor de happy flow, dan weet ik 95% zeker dat deze code goed is. In C++ heb ik de behoefte om de code nog eens aan de tand te voelen, een paar gekke edge cases te verzinnen of `printf` statements toe te voegen om mezelf te overtuigen dat het goed is.

Er zijn een aantal onderwerpen die mij een gevoel van betrouwbaarheid geven, maar ik wil het hier over memory management hebben. Tijdens de meetup merkte ik dat de focus hier namelijk op lag, terwijl ik er de afgelopen 6 maanden juist weinig aan heb gedacht.

Rust maakt memory management een non-issue. Er zijn meer talen die dit doen door memory management uit handen te nemen, maar Rust geeft je de controle als je het nodig hebt. In Java is bijvoorbeeld bijna alles een object, objecten worden op de heap gealloceerd en de garbage collector ruimt het voor je op. In Python en JavaScript wordt alles voor je gedaan, je maakt objecten en arrays zonder er bij na te denken en je hoopt dat de browser genoeg geheugen beschikbaar heeft (dat is natuurlijk gechargeerd).

In Rust ben je zelf verantwoordelijk voor het memory management, maar het biedt tools die het makkelijk maakt. Bijvoorbeeld als je een functie maakt, dan moet uit de functie signatuur duidelijk zijn wie eigenaar is van de data. Als dat niet duidelijk is, dan keurt de compiler je code af. Die signatuur kan je zo simpel en complex maken als je zelf wilt.

De volgende methodes om aan memory management te doen gebruik ik veel in de praktijk:

1. Als het gaat om een klein type, dan kan de data gekopieerd worden. Met een klein type bedoel ik een type dat weinig geheugen in gebruik neemt. Bijvoorbeeld een integer of een struct met een paar floats. De compiler weet dat het een type mag kopiëren als het de `Copy` trait implementeert. De primitieve types (zoals `i8` of `usize`) en veel standaard library types implementeren de `Copy` trait. Voor je eigen types kan je dat automatisch doen door middel van `#[derive(Copy)]`. Zo'n type kan je gewoon als parameter meegeven aan een functie. De compiler snapt dan dat het domweg de bytes mag kopiëren naar de stack van de functie.

2. Voor grotere types is het inefficiënt om deze te kopiëren, dan gebruik ik een reference. Bijvoorbeeld een struct met honderd integers of een lange tekst gebruiken 100+ bytes in geheugen en dit kopiëren is (relatief) veel werk. In Rust kan je een reference naar zo'n type maken. Een reference is een pointer waarvan de Rust compiler kan garanderen dat het naar geldige data wijst. De compiler gebruikt hiervoor het concept van een lifetime. Dit is te complex om nu uit te leggen, maar de samenvatting is: door middel van static code analysis bewijst de compiler dat de reference alleen gebruikt wordt als de pointer geldig is. Lifetimes kunnen heel complex worden op het moment dat je een reference voor langere tijd wilt bewaren. Soms is dit het waard voor extra performance, maar meestal gebruik ik references in de makkelijke gevallen.

3. In geval van een complexe lifetime is de makkelijkste oplossing om de data op de heap te zetten. Door een type in een `Box` te zetten, kies je er voor om het type in de heap te plaatsen. Toen ik nieuw was in Rust vond ik dit doodzonde, als ik gewoon wat meer tijd besteed aan dit lifetime probleem, dan kan ik het met een reference oplossen. Totdat ik besefte dat in Java alle objecten op de heap staan en in C++ gebruik je de heap bij het `new` operator. Je zou kunnen zeggen dat Rust efficiënter is, want standaard staan alle types op de stack, totdat je kiest voor de heap. Als ik nu een lifetime situatie tegenkom die ik niet makkelijk kan oplossen, dan kies ik voor een `Box`. Als later blijkt dat de performance niet goed is, dan kunnen we dat altijd nog optimaliseren.

Veel code die ik schrijf zit niet in het kritische pad en daarom maakt het mij niet echt uit hoe het memory management van de data werkt. Ik kies dan wat er in dat scenario het makkelijkste is. De compiler garandeert vervolgens dat het goed is. Een aantal mooie trucs die de compiler automatische voor mij doet zijn:

- RAII is heel gebruikelijk binnen Rust, de compiler snapt wanneer zaken uit scope gaan en ruimt deze op. Bijvoorbeeld de `Box` die ik eerder noemde, wordt automatische opgeruimd als deze out of scope gaat. Daardoor kan je geen memory leaken (zoals in C++ als je `delete` vergeet) en is er geen garbage collector nodig (zoals in Java). Hetzelfde principe geldt voor bijvoorbeeld geopende files en Mutex locks.

- Als de compiler ziet dat ik een object voor de laatste keer gebruik, dan kiest het ervoor om dat object te verplaatsen. Bijvoorbeeld, als ik een object om de stack maak en deze enkel gebruik in een functie aanroep, dan kan kiest de compiler er voor om die data niet te kopiëren, maar om het te verplaatsen met een `move` operatie. Deze optimalisatie wordt automatisch toegepast.

- Een aantal van de genoemde methodes voor memory management zijn thread safe, maar belangrijker: ze zijn nooit "thread unsafe". Met "thread unsafe" bedoel ik dat je een fout kan maken waardoor er problemen ontstaan. De compiler zal je hier altijd op wijzen en ziet dit als error. Als je data probeert te gebruiken vanaf meerdere threads, dan voorkomt de compiler dat. Gebruik een `Mutex` of `RwLock` om dat op te lossen. Ook deze types kunnen niet verkeerd gebruikt worden: je kan nooit per ongeluk bij de data achter het lock komen.

Oké, dit is een lang verhaal geworden. Mijn punt is: Rust heeft de simpele memory management problemen opgelost. Ik hoef er niet over na te denken. Ik kom natuurlijk nog steeds moeilijkere problemen tegenkomen, maar dan helpt de compiler je met het voorkomen van fouten. Ik ervaar dat ik meer bezig ben het domein probleem als ik in Rust programmeer. Dat is fijn, want dan kan ik het probleem van mijn klant oplossen in plaats van bezig te zijn met memory management.

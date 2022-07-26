---
title: HTML - översikt
description: Läs om hur AEM stöder HTML mallspråk (HTL) för att skapa ett produktivt webbramverk på företagsnivå. HTML ökar säkerheten och gör det möjligt för HTML-utvecklare utan Java-kunskaper att bättre delta i AEM projekt.
exl-id: 5d06ff25-d681-4b95-8375-c28a8364eb7e
source-git-commit: 7b53eff0652f650ffb8caae0e69aa349b5c548eb
workflow-type: tm+mt
source-wordcount: '663'
ht-degree: 0%

---

# Översikt {#overview}

Syftet med HTML Template Language (HTL), som stöds av Adobe Experience Manager (AEM), är att erbjuda ett mycket produktivt webbramverk på företagsnivå som ökar säkerheten och gör det möjligt för HTML-utvecklare utan Java-kunskaper att bättre delta i AEM projekt.

HTML mallspråk är det rekommenderade och rekommenderade serversidesmallsystemet för HTML i AEM. HTML introducerades med AEM 6.0 och ersätter JSP (JavaServer Pages). För webbutvecklare som behöver bygga robusta företagswebbplatser kan mallspråket HTML bidra till ökad säkerhet och effektivare utveckling.

## Ökad säkerhet {#increased-security}

HTML-mallspråket ökar säkerheten för webbplatser som använder det i implementeringen, jämfört med JSP och de flesta andra mallsystem, eftersom HTML automatiskt kan använda rätt kontextmedveten escape-konvertering för alla variabler som ska skrivas ut i presentationslagret. HTML gör detta möjligt eftersom det förstår HTML-syntaxen, och använder den kunskapen för att justera den nödvändiga escape-konverteringen för uttryck utifrån deras position i koden. Detta resulterar t.ex. i uttryck som placeras i `href` eller `src` attribut som ska undantas på ett annat sätt än uttryck som placeras i andra attribut, eller någon annanstans.

Även om samma resultat kan uppnås med mallspråk som JSP, måste utvecklaren manuellt se till att rätt escape-konvertering tillämpas på varje variabel. Eftersom ett enda utelämnande eller fel på den tillämpade escape-konverteringen potentiellt är tillräckligt för att orsaka en XSS-säkerhetslucka (cross-site scripting) beslutade vi oss för att automatisera den här åtgärden med HTML. Vid behov kan utvecklare fortfarande ange olika escape-värden för uttrycken, men med HTML är standardbeteendet troligare att det motsvarar det önskade beteendet, vilket minskar risken för fel.

## Förenklad utveckling {#simplified-development}

HTML-mallspråket är enkelt att lära sig och dess funktioner är avsiktligt begränsade för att säkerställa att det är enkelt och rakt framåt. Den har också kraftfulla mekanismer för att strukturera markeringar och anropa logik, samtidigt som den alltid tvingar fram strikt åtskillnad mellan markeringar och logik. Själva HTML är standard HTML5 eftersom den använder uttryck och dataattribut för att kommentera koden med önskat dynamiskt beteende, vilket innebär att den inte bryter markeringens giltighet utan gör den läsbar. Observera att utvärderingen av uttrycken och dataattributen görs helt på serversidan och inte visas på klientsidan, där önskat JavaScript-ramverk kan användas utan att störa.

Tack vare dessa funktioner kan utvecklare av HTML utan Java-kunskap och med liten produktspecifik kunskap redigera HTML-mallar, vilket gör att de kan vara en del av utvecklingsteamet och effektivisera samarbetet med Java-utvecklare i full hög. Och tvärtom gör det att Java-utvecklare kan fokusera på backend-koden utan att behöva oroa sig för HTML.

## Minskade kostnader {#reduced-costs}

Ökad säkerhet, förenklad utveckling och förbättrat teamsamarbete, ger minskad AEM, kortare time-to-market och lägre total ägandekostnad.

Det som har observerats när webbplatsen Adobe.com implementeras på nytt med mallspråket HTML är att kostnaden och varaktigheten för projektet kan minskas med cirka 25 %.

![Effektiv ökning och kostnadsminskning](assets/chlimage_1.png)

Diagrammet ovan visar följande effektivitetsförbättringar som kan göras möjliga av HTML:

* **HTML / CSS / JS:** Eftersom utvecklarna av HTML kan redigera HTML-mallar direkt behöver inte gränssnittsdesignen längre implementeras separat från det AEM projektet, utan kan implementeras direkt på de faktiska AEM komponenterna. Detta minskar antalet svåra iterationer med Java-utvecklare i full-stack.
* **JSP / HTML:** Eftersom HTML inte kräver någon Java-kunskap och är enkel att skriva, kan alla utvecklare med HTML expertis redigera mallarna.
* **Java:** Tack vare det tydliga och lättanvända Use-API:t som tillhandahålls av HTL har gränssnittet med affärslogiken klargjorts, vilket även gynnar Java-utvecklingen generellt.

**Läs nästa:**

* [Komma igång med mallspråket HTML](getting-started.md)

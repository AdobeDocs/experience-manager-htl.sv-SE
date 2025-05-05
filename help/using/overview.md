---
title: HTML - översikt
description: Upptäck hur AEM stöder HTML (HTML Template Language) för att skapa ett produktivt webbramverk på företagsnivå som förbättrar säkerheten. Med det här ramverket kan utvecklare av HTML utan Java-kunskaper delta bättre i AEM projekt.
exl-id: 5d06ff25-d681-4b95-8375-c28a8364eb7e
source-git-commit: c6bb6f0954ada866cec574d480b6ea5ac0b51a3f
workflow-type: tm+mt
source-wordcount: '645'
ht-degree: 0%

---


# Ökning {#overview}

HTML Template Language (HTL), som stöds av Adobe Experience Manager (AEM), har som mål att tillhandahålla ett högproduktivt webbramverk på företagsnivå som förbättrar säkerheten. Det gör också att utvecklare av HTML utan Java-kunskaper kan delta bättre i AEM projekt.

[HTML-mallspråket introducerades i AEM 6.0](history.md) och rekommenderas som serversidesmallsystem för HTML i AEM. För webbutvecklare som behöver bygga robusta företagswebbplatser kan mallspråket HTML bidra till ökad säkerhet och effektivare utveckling.

## Ökad säkerhet {#increased-security}

HTML (HTML Template Language) förbättrar webbplatsens säkerhet genom att automatiskt tillämpa kontextmedveten escape-konvertering på alla utdatavariabler, vilket gör den säkrare än de flesta andra mallsystem. HTML gör detta tillvägagångssätt möjligt eftersom det förstår HTML-syntaxen, och använder den kunskapen för att justera den nödvändiga escape-konverteringen för uttryck utifrån deras position i koden. Den här metoden kan leda till att uttryck som placerats i attributen `href` eller `src` undantas på ett annat sätt än uttryck som placerats i andra attribut eller någon annanstans.

Även om samma resultat kan uppnås med mallspråk som JSP, måste utvecklaren manuellt se till att rätt escape-konvertering tillämpas på varje variabel. Eftersom ett enda utelämnande eller fel i den tillämpade escape-konverteringen potentiellt är tillräckligt för att orsaka en XSS-säkerhetslucka (cross-site scripting) beslutade Adobe att automatisera denna uppgift med HTML. Vid behov kan utvecklare fortfarande ange en annan escape-konvertering för uttrycken, men med HTML är standardbeteendet troligare att det motsvarar det önskade beteendet, vilket minskar risken för fel.

## Förenklad utveckling {#simplified-development}

HTML-mallspråket är enkelt att lära sig och dess funktioner är avsiktligt begränsade för att säkerställa att det är enkelt och rakt framåt. Den har också kraftfulla mekanismer för att strukturera markeringar och anropa logik, samtidigt som den alltid tvingar fram strikt åtskillnad mellan markeringar och logik. HTL är standard för HTML5, där uttryck och dataattribut används för att kommentera koden med dynamiskt beteende. Detta tillvägagångssätt bevarar giltigheten och läsbarheten för koden. Utvärderingen av uttrycken och dataattributen görs helt på serversidan och är inte synlig på klientsidan, där ett önskat JavaScript-ramverk kan användas utan att störa.

Tack vare dessa funktioner kan utvecklare av HTML utan Java-kunskaper redigera HTML-mallar, integrera i utvecklingsteamet och effektivisera samarbetet med Java-utvecklare i full hög. Och vice versa låter Java-utvecklare fokusera på backend-koden utan att behöva oroa sig för HTML.

## Minskade kostnader {#reduced-costs}

Ökad säkerhet, förenklad utveckling och förbättrat teamsamarbete, innebär minskad insats AEM projekt, kortare time-to-market och lägre total ägandekostnad.

Om du implementerar om Adobe.com webbplats med HTML Template Language har det visat sig att projektkostnaderna och varaktigheten minskar med cirka 25 %.

![Öka och minska kostnaderna effektivt](assets/chlimage_1.png)

Diagrammet ovan visar följande effektivitetsförbättringar som kan göras möjliga av HTML:

* **HTML/CSS/JS:** HTML-utvecklare kan redigera HTML-mallar direkt så att gränssnittsdesignen kan implementeras direkt i AEM komponenter, vilket eliminerar behovet av separat implementering. Detta arbetssätt minskar antalet svåra iterationer med Java-utvecklare som använder en hel hög.
* **JSP/HTL:** Eftersom HTML inte kräver någon Java-kunskap och är rakt fram att skriva, kan utvecklare med HTML expertis redigera mallarna.
* **Java:** Tack vare det tydliga och lättanvända Use-API:t som tillhandahålls av HTL, har gränssnittet med affärslogiken klargjorts, vilket även gynnar Java-utvecklingen generellt.

## Videointroduktion {#video}

Följande video från en [AEM Gems-session](https://experienceleague.adobe.com/sv/docs/events/experience-manager-gems-recordings/gems2014/aem-introduction-to-htl) ger en översikt över syftet med HTML samt exempel på implementering.

>[!VIDEO](https://video.tv.adobe.com/v/19504/?quality=9)

Observera att videon refererar till HTML med [dess tidigare namn, Sightly](history.md).

## Nästa steg {#next-steps}

Nu när du känner till målen och fördelarna med HTML kan du komma igång med språket. Se [Komma igång med HTML-mallspråket](getting-started.md).

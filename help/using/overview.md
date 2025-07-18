---
title: HTML - översikt
description: Upptäck hur AEM stöder HTML (HTML Template Language) för att skapa ett produktivt webbramverk på företagsnivå som förbättrar säkerheten. Med det här ramverket kan HTML-utvecklare utan Java-kunskaper delta bättre i AEM Projects.
exl-id: 5d06ff25-d681-4b95-8375-c28a8364eb7e
source-git-commit: addc69e4b4e56a9b1c5f91ce9af26fa2d326d981
workflow-type: tm+mt
source-wordcount: '677'
ht-degree: 0%

---


# Ökning {#overview}

>[!TIP]
>
>**Har du övervägt Edge Delivery Services för AEM?**
>
>Du kan fortsätta använda metoderna som beskrivs i det här dokumentet för befintliga projekt. För nya projekt rekommenderar Adobe att du använder [Edge Delivery Services.](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/edge-delivery/overview)

HTML mallspråk (HTL), som stöds av Adobe Experience Manager (AEM), har som mål att tillhandahålla ett högproduktivt webbramverk på företagsnivå som förbättrar säkerheten. Det gör det även möjligt för HTML-utvecklare utan Java-kunskaper att delta bättre i AEM Projects.

[HTML Template Language, som introducerades i AEM 6.0](history.md), är det rekommenderade serversidesmallsystemet för HTML i AEM. För webbutvecklare som behöver bygga robusta företagswebbplatser kan HTML mallspråk bidra till ökad säkerhet och effektivare utveckling.

## Ökad säkerhet {#increased-security}

HTML mallspråk (HTL) förbättrar webbplatsens säkerhet genom att automatiskt tillämpa kontextmedveten escape-konvertering på alla utdatavariabler, vilket gör den säkrare än de flesta andra mallsystem. HTML gör detta tillvägagångssätt möjligt eftersom det förstår HTML-syntax och använder den kunskapen för att justera den flytning som krävs för uttryck baserat på deras position i koden. Den här metoden kan leda till att uttryck som placerats i attributen `href` eller `src` undantas på ett annat sätt än uttryck som placerats i andra attribut eller någon annanstans.

Även om samma resultat kan uppnås med mallspråk som JSP, måste utvecklaren manuellt se till att rätt escape-konvertering tillämpas på varje variabel. Eftersom ett enda utelämnande eller fel i den tillämpade escape-konverteringen potentiellt är tillräckligt för att orsaka en XSS-säkerhetslucka (cross-site scripting) beslutade Adobe att automatisera denna uppgift med HTML. Vid behov kan utvecklare fortfarande ange en annan escape-konvertering för uttrycken, men med HTML är standardbeteendet troligare att det motsvarar det önskade beteendet, vilket minskar risken för fel.

## Förenklad utveckling {#simplified-development}

HTML mallspråk är enkelt att lära sig och dess funktioner är avsedda att vara enkla och enkla att använda. Den har också kraftfulla mekanismer för att strukturera markeringar och anropa logik, samtidigt som den alltid tvingar fram strikt åtskillnad mellan markeringar och logik. HTL är standard-HTML5, där uttryck och dataattribut används för att kommentera koden med dynamiskt beteende. Detta tillvägagångssätt bevarar giltigheten och läsbarheten för koden. Utvärderingen av uttrycken och dataattributen görs helt på serversidan och är inte synlig på klientsidan, där ett önskat JavaScript-ramverk kan användas utan att störa.

Med dessa funktioner kan HTML-utvecklare utan Java-kunskaper redigera HTML-mallar, integrera i utvecklingsteamet och effektivisera samarbetet med Java-utvecklare i full skala. Och vice versa låter Java-utvecklare fokusera på backend-koden utan att behöva oroa sig för HTML.

## Minskade kostnader {#reduced-costs}

Ökad säkerhet, förenklad utveckling och förbättrat teamsamarbete innebär minskad insats i AEM Projects, snabbare time to market (TTM) och lägre total ägandekostnad.

Om du implementerar om Adobe.com webbplats med HTML Template Language har det visat sig att projektkostnaderna och varaktigheten minskar med cirka 25 %.

![Öka och minska kostnaderna effektivt](assets/chlimage_1.png)

Diagrammet ovan visar följande effektivitetsförbättringar som kan göras möjliga av HTML:

* **HTML/CSS/JS:** HTML-utvecklare kan redigera HTML-mallar direkt, vilket innebär att gränssnittsdesign kan implementeras direkt i AEM-komponenter, vilket eliminerar behovet av separat implementering. Detta arbetssätt minskar antalet svåra iterationer med Java-utvecklare som använder en hel hög.
* **JSP/HTL:** Eftersom HTML inte kräver någon Java-kunskap och är enkel att skriva, kan alla utvecklare med HTML expertis redigera mallarna.
* **Java:** Tack vare det tydliga och lättanvända Use-API:t som tillhandahålls av HTL, har gränssnittet med affärslogiken klargjorts, vilket även gynnar Java-utvecklingen generellt.

## Videointroduktion {#video}

Följande video från en [AEM Gems-session](https://experienceleague.adobe.com/en/docs/events/experience-manager-gems-recordings/gems2014/aem-introduction-to-htl) ger en översikt över syftet med HTML samt exempel på implementering.

>[!VIDEO](https://video.tv.adobe.com/v/19504/?quality=9)

Observera att videon refererar till HTML med [dess tidigare namn, Sightly](history.md).

## Nästa steg {#next-steps}

Nu när du känner till målen och fördelarna med HTML kan du komma igång med språket. Se [Komma igång med HTML mallspråk](getting-started.md).

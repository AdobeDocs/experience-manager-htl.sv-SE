---
title: HTL Use-API
description: Det finns två API:er för HTL – Java Use-API och JavaScript Use-API
translation-type: tm+mt
source-git-commit: d7efae3d1b4d1bc22c63c21f544a99bf0ae4b3c9
workflow-type: tm+mt
source-wordcount: '183'
ht-degree: 6%

---


# HTL Use-API {#htl-use-api}

HTML uppmuntrar till åtskilda bekymmer genom att affärslogiken inte får blandas med markeringar. Affärslogik kan implementeras via Use-API.

I följande tabell visas fördelar och nackdelar med respektive API.

|  | **Java Use-API** | **JavaScript Use-API** |
|--- |--- |--- |
| **Fördelar** | <ul><li>Snabbare</li><li>Kan inspekteras med en felsökare</li><li>Enkelt att enhetstesta</li></ul> | <ul><li>Kan ändras av gränssnittsutvecklare</li><li>Finns inuti komponenten och håller vylogiken för en komponent nära motsvarande mall</li></ul> |
| **Nackdelar** | <ul><li>Kan inte ändras av gränssnittsutvecklare</li></ul> | <ul><li>Långsammare</li><li>Ingen felsökare (än)</li><li>Svårare till enhetstest</li></ul> |

För sidkomponenter bör du använda en blandad modell, där all modelllogik finns i Java, vilket ger tydliga API:er som inte är baserade på något som händer i vyn (dvs. i komponenterna). AEM innehåller bra standardmodeller som Page eller Resource API som kan användas i de flesta fall.

All visningslogik som är specifik för en komponent ska placeras i den komponenten som JavaScript, eftersom den tillhör den komponenten.

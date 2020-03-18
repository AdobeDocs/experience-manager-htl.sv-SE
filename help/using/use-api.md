---
title: HTL Use-API
seo-title: Adobe HTML Use-API
description: Det finns två API:er för HTL – Java Use-API och JavaScript Use-API
seo-description: Det finns två API:er för Adobe HTL – Java Use-API och JavaScript Use-API
uuid: ab44aa5c-ce7e-40b9-97fb-e86c6a28405c
contentOwner: User
products: SG_EXPERIENCEMANAGER/HTL
topic-tags: html-template-language
content-type: reference
discoiquuid: 89004426-eb59-4b63-913f-51bf98662773
mwpw-migration-script-version: 2017-10-12T21 46 58.665-0400
translation-type: tm+mt
source-git-commit: 5cbaf9c747acf748d12559c2c8e3aba4600cf9a4

---


# HTL Use-API {#htl-use-api}

I följande tabell visas en översikt över för- och nackdelar med varje API.

|  | **Java Use-API** | **JavaScript Use-API** |
|--- |--- |--- |
| **Proffs** | <ul><li>snabbare</li><li>kan inspekteras med en felsökare</li><li>enkla att utföra</li></ul> | <ul><li>kan ändras av gränssnittsutvecklare</li><li>finns inuti komponenten och håller vylogiken för en komponent nära dess motsvarande mall</li></ul> |
| **Kon** | <ul><li>kan inte ändras av gränssnittsutvecklare</li></ul> | <ul><li>långsam</li><li>ingen felsökare (ännu)</li><li>hårdare mot enhetstest</li></ul> |


För sidkomponenter bör du använda en blandad modell, där all modelllogik finns i Java, vilket ger tydliga API:er som inte är baserade på något som händer i vyn (dvs. i komponenterna). AEM innehåller bra standardmodeller som Page eller Resource API som kan användas i de flesta fall.

All visningslogik som är specifik för en komponent ska placeras i den komponenten som JavaScript, eftersom den tillhör den komponenten.

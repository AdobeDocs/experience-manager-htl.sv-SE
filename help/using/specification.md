---
title: HTML-specifikationen
description: Mer detaljerad syntaxinformation finns i HTML-specifikationen.
exl-id: c0657476-4db6-4fad-ad87-9252b5003237
source-git-commit: addc69e4b4e56a9b1c5f91ce9af26fa2d326d981
workflow-type: tm+mt
source-wordcount: '134'
ht-degree: 0%

---


# HTML-specifikationen {#htl-specification}

HTML Template Language (HTL) är det rekommenderade serverplatsmallsystemet för HTML.

## HTML-lager {#layers}

Du kan definiera HTML i AEM med ett antal lager.

1. **[HTL-specifikation](https://github.com/adobe/htl-spec)** - HTML är en öppen källkodsbaserad, plattformsoberoende specifikation som alla kan implementera. Dess specifikationer behålls i GitHub-repo.
1. **[Flytta HTML-skriptmotor](https://sling.apache.org/documentation/bundles/scripting/scripting-htl.html)** - `Sling`-projektet har skapat referensimplementeringen av HTML, som används av AEM. Dokumentationen för projektet `Sling` behålls.
1. **[AEM-tillägg](aem-extensions.md)** - AEM bygger ovanpå `Sling` HTML-skriptmotorn och ger utvecklare praktiska funktioner som är specifika för AEM. Dessa tillägg dokumenteras som en del av den här dokumentationsuppsättningen.

Följ länkarna ovan för att se den dedikerade dokumentationen för alla lager av HTML som används av AEM.

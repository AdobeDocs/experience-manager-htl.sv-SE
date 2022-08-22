---
title: Historik för HTML
description: För användare som har AEM länge ger det här dokumentet bakgrunden i HTML, hur den ersätter JSP och namnändringen från Sightly.
exl-id: 00985b35-2130-4946-959a-0a09a34a0f05
source-git-commit: 5ab1275c984135fe946f36905bbc979cf19edd80
workflow-type: tm+mt
source-wordcount: '542'
ht-degree: 0%

---


# Historik för HTML {#history-of-htl}

För användare som har AEM länge ger det här dokumentet bakgrunden i HTML, hur den ersätter JSP och namnändringen från Sightly.

## Tidigare Känd som låg {#sightly}

HTML (HTML Template Language) är det rekommenderade och rekommenderade serversidesmallsystemet för HTML i Adobe Experience Manager. Den ersätter JSP (JavaServer Pages) som den användes i tidigare versioner av AEM.

## HTL över JSP {#htl-over-jsp}

Vi rekommenderar att du använder HTML mallspråk i nya AEM-projekt eftersom det ger flera fördelar jämfört med JSP. Men för befintliga projekt är en migrering bara meningsfull om den beräknas vara mindre ansträngning än att underhålla de befintliga JSP:erna under de kommande åren.

Men att gå över till HTML är inte nödvändigtvis ett alternativ som är helt eller inget, eftersom komponenter skrivna i HTML är kompatibla med komponenter skrivna i JSP eller ESP. Det innebär att befintliga projekt utan problem kan använda HTML för nya komponenter, samtidigt som JSP för befintliga komponenter behålls.

Även i samma komponent kan HTML-filer användas tillsammans med JSP och ESP. I följande exempel visas **rad 1** hur du inkluderar en HTML-fil från en JSP-fil, och **rad 2** hur en JSP-fil kan inkluderas från en HTML-fil:

```xml
<cq:include script="template.html"/>
<sly data-sly-include="template.jsp"/>
```

## Vanliga frågor {#frequently-asked-questions}

Detta är några frågor som ofta ställs av erfarna AEM utvecklare som inte har HTML.

### Har HTML några begränsningar som JSP inte har? {#limitations}

HTML har inga begränsningar jämfört med JSP i den meningen att det som kan göras med JSP också ska vara möjligt med HTML. HTL är dock i flera avseenden striktare än JSP. Vad du kan göra allt med en enda JSP-fil, kan behöva separeras till en Java-klass eller en JavaScript-fil för att kunna nås i HTML. Detta är dock i allmänhet önskvärt för att säkerställa en bra avvägning mellan logiken och markeringen.

### Stöder HTML JSP-taggbibliotek? {#tag-libraries}

Nej, men enligt [Läser in klientbibliotek](getting-started.md#loading-client-libraries) -avsnittet i dokumentet Komma igång innehåller programsatserna för mall och anrop ett liknande mönster.

### Kan HTML-funktionerna utökas i ett AEM projekt? {#extended}

Nej, det kan de inte. HTL har kraftfulla tilläggsfunktioner för återanvändning av logik ( [Use-API](#use-api-for-accessing-logic)) och för kod (mallen och anropsprogramsatserna), som kan användas för att modularisera koden för projekt.

### Vilka är de viktigaste fördelarna med HTML jämfört med JSP? {#benefits}

Säkerhet och projekteffektivitet är de viktigaste fördelarna som finns i [Översikt.](overview.md)

### Kommer JSP så småningom att försvinna? {#go-away}

Det finns inga planer på detta.

## Vad heter du? {#what-is-in-a-name}

I AEM 6.0 och 6.1 kallades HTML **Snäll**. Adobe har ändrat namn till **HTML mallspråk** eller **HTL** för att klargöra vad produktspecifikationen gäller och för att anpassa sig till Adobe allmänna riktlinjer för namngivning. Den här namnändringen gällde från och med augusti 2016 och gäller AEM version 6.0 och framåt.

>[!NOTE]
>
>Den här namnändringen påverkar inte koden eller API:t, och kompatibiliteten påverkas därför inte. Mer information finns här: [i den här videon.](https://helpx.adobe.com/experience-manager/how-to/announce-htl.html)

För att få veta mer om HTML och en bra plats att börja på är vår tjänsteman [Komma igång med HTML-guiden (HTML Templating Language).](overview.md)

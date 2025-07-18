---
title: Historik för HTML
description: För användare som använder AEM länge ger det här dokumentet bakgrunden i HTML, hur den ersätter JSP och namnändringen från Sightly.
exl-id: 00985b35-2130-4946-959a-0a09a34a0f05
source-git-commit: addc69e4b4e56a9b1c5f91ce9af26fa2d326d981
workflow-type: tm+mt
source-wordcount: '530'
ht-degree: 0%

---


# Historik för HTML {#history-of-htl}

För användare som använder AEM länge ger det här dokumentet bakgrunden i HTML, hur den ersätter JSP och namnändringen från Sightly.

## Tidigare Känd som låg {#sightly}

HTML Template Language (HTL) är det rekommenderade serversidesmallsystemet för HTML i Adobe Experience Manager. Det ersätter JSP (JavaServer Pages) som i tidigare versioner av AEM.

## HTL över JSP {#htl-over-jsp}

Adobe rekommenderar att du använder HTML mallspråk för nya AEM-projekt. Orsaken är att det ger flera fördelar jämfört med JSP. Men för befintliga projekt är en migrering bara meningsfull om den beräknas vara mindre ansträngning än att underhålla de befintliga JSP:erna under de kommande åren.

Att gå över till HTML är inte nödvändigtvis ett alternativ som bara innehåller ingenting, eftersom komponenter skrivna i HTML är kompatibla med komponenter skrivna i JSP eller ESP. Detta innebär att befintliga projekt kan använda HTML för nya komponenter utan problem, samtidigt som JSP för befintliga komponenter behålls.

Även i samma komponent kan HTML-filer användas tillsammans med JSP och ESP. I följande exempel visas på **rad 1** hur du tar med en HTML-fil från en JSP-fil och på **rad 2** hur en JSP-fil kan inkluderas från en HTML-fil:

```xml
<cq:include script="template.html"/>
<sly data-sly-include="template.jsp"/>
```

## Vanliga frågor {#frequently-asked-questions}

Erfarna AEM-utvecklare som inte använder HTML ställer ofta följande frågor:

### Har HTML några begränsningar som JSP inte har? {#limitations}

HTML har inga begränsningar jämfört med JSP i den meningen att det som kan göras med JSP också ska vara möjligt med HTML. HTL är dock i flera avseenden striktare än JSP. Vad som kan uppnås i en enda JSP-fil kan behöva separeras till en Java-klass eller en JavaScript-fil för att kunna nås i HTML. Denna metod är dock i allmänhet önskvärd för att säkerställa en bra avvägning mellan logik och påläggning.

### Stöder HTML JSP-taggbibliotek? {#tag-libraries}

Nej. Som du kan se i avsnittet [Läser in klientbibliotek](getting-started.md#loading-client-libraries) i dokumentet Komma igång, erbjuder mallen och anropsprogramsatserna ett liknande mönster.

### Kan HTML-funktionerna utökas i ett AEM-projekt? {#extended}

Nej. HTL har kraftfulla tilläggsmekanismer för återanvändning av logik ([Use-API](#use-api-for-accessing-logic)) och kod (mallen och anropsprogramsatserna), som kan användas för att modularisera koden för projekt.

### Vilka är de viktigaste fördelarna med HTML jämfört med JSP? {#benefits}

Säkerhet och projekteffektivitet är de viktigaste fördelarna, som beskrivs i [Översikt](overview.md).

### Kommer JavaServer Pages (JSP) att försvinna? {#go-away}

Nej. Det finns inga planer på att avbryta JSP.

## Vad heter du? {#what-is-in-a-name}

I AEM 6.0 och 6.1 anropades HTML **Sightly**. Adobe gav det nya namnet **HTML Template Language** eller **HTL** för att förtydliga vad specifikationen gäller och för att anpassa den till Adobe allmänna riktlinjer för namngivning. Den här namnändringen gällde från och med augusti 2016 och gäller för AEM version 6.0 och framåt.

>[!NOTE]
>
>Den här namnändringen påverkar inte koden eller API:t, och kompatibiliteten påverkas därför inte.

<!-- LINK IS 404
For more information, watch [this announcement video](https://helpx.adobe.com/experience-manager/how-to/announce-htl.html). -->

Mer information om HTML finns i [Komma igång med HTML-handboken (HTML Templating Language)](overview.md).

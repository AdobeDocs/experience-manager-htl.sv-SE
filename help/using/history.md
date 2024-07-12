---
title: Historik för HTML
description: För användare som har AEM länge ger det här dokumentet bakgrunden i HTML, hur den ersätter JSP och namnändringen från Sightly.
exl-id: 00985b35-2130-4946-959a-0a09a34a0f05
source-git-commit: 88edbd2fd66de960460df5928a3b42846d32066b
workflow-type: tm+mt
source-wordcount: '544'
ht-degree: 0%

---


# Historik för HTML {#history-of-htl}

För användare som har AEM länge ger det här dokumentet bakgrunden i HTML, hur den ersätter JSP och namnändringen från Sightly.

## Tidigare Känd som låg {#sightly}

HTML (HTML Template Language) är det rekommenderade och rekommenderade serversidesmallsystemet för HTML i Adobe Experience Manager. Den ersätter JSP (JavaServer Pages) som den användes i tidigare versioner av AEM.

## HTL över JSP {#htl-over-jsp}

Vi rekommenderar att du använder HTML mallspråk i nya AEM-projekt eftersom det ger flera fördelar jämfört med JSP. Men för befintliga projekt är en migrering bara meningsfull om den beräknas vara mindre ansträngning än att underhålla de befintliga JSP:erna under de kommande åren.

Men att gå över till HTML är inte nödvändigtvis ett alternativ som är helt eller inget, eftersom komponenter skrivna i HTML är kompatibla med komponenter skrivna i JSP eller ESP. Det innebär att befintliga projekt utan problem kan använda HTML för nya komponenter, samtidigt som JSP för befintliga komponenter behålls.

Även i samma komponent kan HTML-filer användas tillsammans med JSP och ESP. I följande exempel visas på **rad 1** hur en HTML-fil från en JSP-fil ska inkluderas och på **rad 2** hur en JSP-fil kan inkluderas från en HTML-fil:

```xml
<cq:include script="template.html"/>
<sly data-sly-include="template.jsp"/>
```

## Vanliga frågor {#frequently-asked-questions}

Detta är några frågor som ofta ställs av erfarna AEM utvecklare som inte har HTML.

### Har HTML några begränsningar som JSP inte har? {#limitations}

HTML har inga begränsningar jämfört med JSP i den meningen att det som kan göras med JSP också ska vara möjligt med HTML. HTL är dock i flera avseenden striktare än JSP. Vad man kan göra med en enda JSP-fil kan man behöva separera den till en Java-klass eller en JavaScript-fil för att kunna göra den möjlig i HTML. Detta är dock i allmänhet önskvärt för att säkerställa en bra avvägning mellan logiken och markeringen.

### Stöder HTML JSP-taggbibliotek? {#tag-libraries}

Nej, men som det visas i avsnittet [Läser in klientbibliotek](getting-started.md#loading-client-libraries) i dokumentet Komma igång, erbjuder mallen och anropsprogramsatserna ett liknande mönster.

### Kan HTML-funktionerna utökas i ett AEM projekt? {#extended}

Nej, det kan de inte. HTL har kraftfulla tilläggsmekanismer för återanvändning av logik ([Use-API](#use-api-for-accessing-logic)) och kod (mallen och anropsprogramsatserna), som kan användas för att modularisera koden för projekt.

### Vilka är de viktigaste fördelarna med HTML jämfört med JSP? {#benefits}

Säkerhet och projekteffektivitet är de viktigaste fördelarna som beskrivs i [Översikt.](overview.md)

### Kommer JSP så småningom att försvinna? {#go-away}

Det finns inga planer på detta.

## Vad heter du? {#what-is-in-a-name}

I AEM 6.0 och 6.1 kallades HTL för **Sightly**. Adobe gav den ett nytt namn till **HTML-mallspråk** eller **HTML** för att förtydliga vad specifikationen är till för och anpassa den till Adobe:s allmänna riktlinjer för namngivning. Den här namnändringen gällde från och med augusti 2016 och gäller AEM version 6.0 och framåt.

>[!NOTE]
>
>Den här namnändringen påverkar inte koden eller API:t, och kompatibiliteten påverkas därför inte. Mer information finns i [den här videon.](https://helpx.adobe.com/experience-manager/how-to/announce-htl.html)

Om du vill veta mer om HTML och en bra plats att börja på är vår officiella [Komma igång med HTML-handbok (HTML Templating Language).](overview.md)

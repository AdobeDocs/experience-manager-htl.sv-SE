---
title: Komma igång med HTML
description: HTML som stöds av AEM ersätter JSP som det rekommenderade och rekommenderade serversidesmallsystemet för HTML i AEM.
translation-type: tm+mt
source-git-commit: f7e46aaac2a4b51d7fa131ef46692ba6be58d878
workflow-type: tm+mt
source-wordcount: '2471'
ht-degree: 0%

---


# Komma igång med HTML {#getting-started-with-htl}

HTML-mallspråket (HTL) som stöds av Adobe Experience Manager (AEM) är det rekommenderade och rekommenderade serversidesmallsystemet för HTML i AEM. Den ersätter JSP (JavaServer Pages) som den användes i tidigare versioner av AEM.

>[!NOTE]
>
>Om du vill köra de flesta exempel som finns på den här sidan kan du använda en körningsmiljö som heter [Read Eval Print Loop](https://github.com/Adobe-Marketing-Cloud/aem-htl-repl).

## HTL över JSP {#htl-over-jsp}

Du bör använda HTML-mallspråket i nya AEM eftersom det ger flera fördelar jämfört med JSP. Men för befintliga projekt är en migrering bara meningsfull om den beräknas vara mindre ansträngning än att underhålla de befintliga JSP:erna under de kommande åren.

Men att gå över till HTML är inte nödvändigtvis ett alternativ som är helt eller inget, eftersom komponenter skrivna i HTML är kompatibla med komponenter skrivna i JSP eller ESP. Det innebär att befintliga projekt utan problem kan använda HTML för nya komponenter, samtidigt som JSP för befintliga komponenter behålls.

Även i samma komponent kan HTML-filer användas tillsammans med JSP och ESP. I följande exempel visas på **rad 1** hur en HTML-fil ska inkluderas från en JSP-fil och på **rad 2** hur en JSP-fil kan inkluderas från en HTML-fil:

```xml
<cq:include script="template.html"/>
<sly data-sly-include="template.jsp"/>
```

### Vanliga frågor och svar{#frequently-asked-questions}

Innan vi börjar med HTML-mallspråket börjar vi med att besvara några frågor som rör JSP kontra HTML-ämnet.

**Har HTML några begränsningar som JSP inte har?** - HTML har inga begränsningar jämfört med JSP i den meningen att det som kan göras med JSP också ska vara möjligt med HTML. HTL är dock striktare genom design än JSP i flera aspekter, och vad som kan uppnås i en enda JSP-fil kan behöva separeras till en Java-klass eller en JavaScript-fil för att kunna nås i HTML. Detta är dock i allmänhet önskvärt för att säkerställa en bra avvägning mellan logiken och markeringen.

**Stöder HTML JSP-taggbibliotek?** - Nej, men som visas i avsnittet  [Läser in klientbibliotek ](getting-started.md#loading-client-libraries) erbjuder  [mallen och ](block-statements.md#template-call) callstatements ett liknande mönster.

**Kan HTML-funktionerna utökas i ett AEM projekt?** Nej, det kan de inte. HTML har kraftfulla tilläggsfunktioner för återanvändning av logik - [Use-API](getting-started.md#use-api-for-accessing-logic) - och av kod ([template &amp; call](block-statements.md#template-call)-programsatserna), som kan användas för att modularisera koden för projekt.

**Vilka är de viktigaste fördelarna med HTML jämfört med JSP?** - Säkerhet och projekteffektivitet är de viktigaste fördelarna som finns i  [översikten](overview.md).

**Kommer JSP så småningom att försvinna?** - För närvarande finns det inga planer på detta.

## Grundläggande begrepp för HTML {#fundamental-concepts-of-htl}

HTML-mallspråket använder ett uttrycksspråk för att infoga innehållsdelar i den återgivna koden, och HTML5-dataattribut för att definiera satser över kodblock (som villkor eller iterationer). När HTML kompileras till Java-servrar utvärderas både uttrycken och HTML-dataattributen helt på serversidan och inget visas i den resulterande HTML-koden.

### Block och uttryck {#blocks-and-expressions}

Här är ett första exempel som kan finnas som i en **`template.html`**-fil:

```xml
<h1 data-sly-test="${properties.jcr:title}">
    ${properties.jcr:title}
</h1>
```

Två olika typer av syntaxer kan urskiljas:

* **[Blockprogramsatser](block-statements.md)**  - För villkorlig visning av  **&lt;h1>** används ett  [`data-sly-test`](block-statements.md#test) HTML5-dataattribut. HTL innehåller flera sådana attribut, som gör att du kan koppla beteenden till ett HTML-element, och alla har `data-sly` som prefix.

* **[Uttrycksspråk](expression-language.md)**  - HTML-uttryck avgränsas med tecken  `${` och  `}`. Vid körning utvärderas dessa uttryck och deras värde matas in i den utgående HTML-strömmen.

På de två länkarna ovan finns en detaljerad lista över funktioner som är tillgängliga för syntax.

### SLY-elementet {#the-sly-element}

Ett centralt koncept för HTML är att erbjuda möjligheten att återanvända befintliga HTML-element för att definiera blocksatser, vilket gör att man slipper lägga in ytterligare avgränsare för att definiera var satsen börjar och slutar. Den här diskreta kommenteringen av koden för att omvandla en statisk HTML till en fungerande dynamisk mall ger fördelen att inte bryta giltigheten för HTML-koden och därmed fortfarande visa den korrekt, även som statiska filer.

Ibland kanske det inte finns något befintligt element på exakt den plats där en blockprogramsats måste infogas. I sådana fall är det möjligt att infoga ett särskilt SLY-element som automatiskt tas bort från utdata när de kopplade blockprogramsatserna körs och innehållet visas i enlighet med detta.

Följande exempel:

```xml
<sly data-sly-test="${properties.jcr:title && properties.jcr:description}">
    <h1>${properties.jcr:title}</h1>
    <p>${properties.jcr:description}</p>
</sly>
```

kommer att generera något som följande HTML-kod, men bara om det finns både en **`jcr:title`**- och en **`jcr:description`**-egenskap definierad och om ingen av dem är tom:

```xml
<h1>MY TITLE</h1>
<p>MY DESCRIPTION</p>
```

En sak som du bör tänka på är att endast använda SLY-elementet när inget befintligt element kunde ha kommenterats med blockprogramsatsen, eftersom SLY-element inte ändrar värdet som språket erbjuder för att inte ändra den statiska HTML-koden när den blir dynamisk.

Om det föregående exemplet till exempel skulle ha kapslats redan i ett DIV-element skulle det tillagda SLY-elementet vara stötande:

```xml
<div>
    <sly data-sly-test="${properties.jcr:title && properties.jcr:description}">
        <h1>${properties.jcr:title}</h1>
        <p>${properties.jcr:description}</p>
    </sly>
</div>
```

och DIV-elementet kunde ha kommenterats med följande villkor:

```xml
<div data-sly-test="${properties.jcr:title && properties.jcr:description}">
    <h1>${properties.jcr:title}</h1>
    <p>${properties.jcr:description}</p>
</div>
```

### HTML-kommentarer {#htl-comments}

I följande exempel visas en HTML-kommentar på **rad 1** och en HTML-kommentar på **rad 2**:

```xml
<!--/* An HTL Comment */-->
<!-- An HTML Comment -->
```

HTML-kommentarer är HTML-kommentarer med en extra JavaScript-liknande syntax. Hela HTML-kommentaren, och allt inom den, ignoreras helt av processorn och tas bort från utdata.

Innehållet i vanliga HTML-kommentarer skickas dock och uttrycken i kommentaren utvärderas.

HTML-kommentarer får inte innehålla HTML-kommentarer och vice versa.

### Specialkontexter {#special-contexts}

För att kunna använda HTML på bästa sätt är det viktigt att du är väl införstådd med konsekvenserna av att det baseras på HTML-syntaxen.

### Element- och attributnamn {#element-and-attribute-names}

Uttryck kan bara placeras i HTML-text eller -attributvärden, men inte i elementnamn eller attributnamn, eller så är det inte längre giltig HTML. Om du vill ange elementnamn dynamiskt kan du använda programsatsen [`data-sly-element`](block-statements.md#element) för de önskade elementen och för att dynamiskt ange attributnamn, även om du anger flera attribut samtidigt, kan du använda programsatsen [`data-sly-attribute`](block-statements.md#attribute).

```xml
<h1 data-sly-element="${myElementName}" data-sly-attribute="${myAttributeMap}">...</h1>
```

### Kontexter utan blocksatser {#contexts-without-block-statements}

Eftersom HTML använder dataattribut för att definiera blockprogramsatser är det inte möjligt att definiera sådana blockprogramsatser i följande sammanhang, och bara uttryck kan användas där:

* HTML-kommentarer
* Skriptelement
* Formatelement

Orsaken till detta är att innehållet i dessa kontexter är text och inte HTML, och HTML-element som ingår betraktas som enkla teckendata. Utan riktiga HTML-element kan inte heller attributen **`data-sly`** köras.

Det kan låta som en stor begränsning, men det är önskat eftersom HTML-mallspråket inte ska missbrukas för att generera utdata som inte är HTML. Avsnittet [Använd-API för att komma åt logik](getting-started.md#use-api-for-accessing-logic) nedan visar hur ytterligare logik kan anropas från mallen, som kan användas om den behövs för att förbereda komplexa utdata för dessa kontexter. Ett enkelt sätt att skicka data från bakänden till ett front end-skript är till exempel att ha komponentens logik att generera en JSON-sträng, som sedan kan placeras i ett dataattribut med ett enkelt HTL-uttryck.

I följande exempel visas beteendet för HTML-kommentarer, men i skript- eller formatelement observeras samma beteende:

```xml
<!--
    The title is: ${properties.jcr:title}
    <h1 data-sly-test="${properties.jcr:title}">${properties.jcr:title}</h1>
-->
```

kommer att visa något som följande HTML:

```xml
<!--
    The title is: MY TITLE
    <h1 data-sly-test="MY TITLE">MY TITLE</h1>
-->
```

### Explicit kontext krävs {#explicit-contexts-required}

Som förklaras i avsnittet [Automatisk kontextmedveten Escaping](getting-started.md#automatic-context-aware-escaping) nedan är ett av målen med HTML att minska riskerna med att införa XSS-problem (cross-site scripting) genom att automatiskt tillämpa kontextmedveten escape-konvertering på alla uttryck. HTML kan automatiskt identifiera sammanhanget för uttryck som placerats i HTML-kod, men inte syntaxen för infogad JavaScript eller CSS, och därför förlitar sig utvecklaren på att explicit ange exakt vilket sammanhang som ska användas för sådana uttryck.

Eftersom inte rätt escape-resultat används i XSS-sårbarheter, tar HTL därför bort utdata från alla uttryck som finns i skript- och formatkontexter när kontexten inte har deklarerats.

Här är ett exempel på hur du anger kontext för uttryck som är placerade i skript och format:

```xml
<script> var trackingID = "${myTrackingID @ context='scriptString'}"; </script>
<style> a { font-family: "${myFont @ context='styleString'}"; } </style>
```

Mer information om hur du styr escape-konverteringen finns i avsnittet [Visningssammanhang för uttrycksspråk](expression-language.md#display-context).

### Lyftbegränsningar för specialkontexter {#lifting-limitations-of-special-contexts}

I de specialfall där det behövs för att kringgå begränsningarna för skript, format och kommentarskontexter, är det möjligt att isolera deras innehåll i en separat HTML-fil. Allt som finns i dess egen fil tolkas av HTML som ett vanligt HTML-fragment, vilket glömmer det begränsade sammanhang som det kan ha inkluderats från.

Se avsnittet [Arbeta med mallar på klientsidan](getting-started.md#working-with-client-side-templates) längre ned för ett exempel.

>[!CAUTION]
>
>Den här tekniken kan medföra XSS-säkerhetsluckor (cross-site scripting) och säkerhetsaspekterna bör noggrant undersökas om detta används. Det finns vanligtvis bättre sätt att implementera samma sak än att förlita sig på den här metoden.

## Allmänna funktioner för HTML {#general-capabilities-of-htl}

I det här avsnittet går du snabbt igenom de allmänna funktionerna i HTML-mallspråket.

### Använd-API för att komma åt logik {#use-api-for-accessing-logic}

Titta på följande exempel:

```xml
<p data-sly-use.logic="logic.js">${logic.title}</p>
```

Följande `logic.js` JavaScript-fil som körs på servern placerades bredvid den:

```javascript
use(function () {
    return {
        title: currentPage.getTitle().substring(0, 10) + "..."
    };
});
```

Eftersom HTML-mallspråket inte tillåter att kod blandas i koden finns det i stället en tilläggsmekanism för Use-API som gör att koden enkelt kan köras från mallen.

I exemplet ovan används JavaScript som körs på servern för att korta ned titeln till 10 tecken, men det kan också ha använt Java-kod för att göra det genom att ange ett fullständigt kvalificerat Java-klassnamn. I allmänhet bör affärslogik helst byggas in i Java, men när komponenten behöver vissa vyspecifika ändringar från Java API:t kan det vara praktiskt att använda en del JavaScript som körs på servern för att göra det.

Mer information om detta i följande avsnitt:

* Avsnittet i [`data-sly-use`-satsen](block-statements.md#use) förklarar allt som kan göras med den satsen.
* På sidan [Use-API](use-api.md) finns information som kan hjälpa dig att välja mellan att skriva logiken i Java eller i JavaScript.
* För att beskriva logiken bör sidorna [JavaScript Use-API](use-api-javascript.md) och [Java Use-API](use-api-java.md) vara till hjälp.

### Automatisk kontextmedveten Escaping {#automatic-context-aware-escaping}

Titta på följande exempel:

```xml
<p data-sly-use.logic="logic.js">
    <a href="${logic.link}" title="${logic.title}">
        ${logic.text}
    </a>
</p>
```

I de flesta mallspråk skulle det här exemplet kunna skapa en XSS-säkerhetslucka (cross-site scripting), eftersom attributet `href` måste vara specifikt URL-escape även om alla variabler automatiskt är HTML-escape. Detta utelämnande är ett av de vanligaste felen, eftersom det är mycket lätt att glömma och det är svårt att hitta på ett automatiserat sätt.

För att underlätta med det undviker HTML-mallspråket automatiskt varje variabel efter sammanhanget som variabeln placeras i. Detta är möjligt tack vare att HTML förstår HTML-syntaxen.

Anta följande `logic.js`-fil:

```javascript
use(function () {
    return {
        link:  "#my link's safe",
        title: "my title's safe",
        text:  "my text's safe"
    };
});
```

Det inledande exemplet ger sedan följande utdata:

```xml
<p>
    <a href="#my%20link%27s%20safe" title="my title&#39;s safe">
        my text&#39;s safe
    </a>
</p>
```

Observera hur de två attributen har escape-konverterats eftersom HTML vet att attributen `href` och `src` måste escape-konverteras för URI-kontext. Om URI:n startade med **`javascript:`** skulle attributet ha tagits bort helt, om inte sammanhanget uttryckligen ändrades till något annat.

Mer information om hur du styr escape-konverteringen finns i avsnittet [Visningssammanhang för uttrycksspråk](expression-language.md#display-context).

### Automatisk borttagning av tomma attribut {#automatic-removal-of-empty-attributes}

Titta på följande exempel:

```xml
<p class="${properties.class}">some text</p>
```

Om värdet för egenskapen `class` är tomt tas hela `class`-attributet bort automatiskt från utdata.

Detta är möjligt igen, eftersom HTML förstår HTML-syntaxen och därför bara kan visa attribut med dynamiska värden om deras värde inte är tomt. Detta är mycket praktiskt eftersom det inte går att lägga till ett villkorsblock runt attribut, vilket skulle ha gjort markeringen ogiltig och oläslig.

Dessutom gäller den typ av variabel som placerats i uttrycket:

* **Sträng:**
   * **inte tom:** Anger strängen som attributvärde.
   * **tom:** Tar bort attributet helt.

* **Number:** Anger värdet som attributvärde.

* **Boolesk:**
   * **true:** Visar attributet utan värde (som ett booleskt HTML-attribut)
   * **false:** Tar bort attributet helt och hållet.

Här är ett exempel på hur ett booleskt uttryck skulle kunna tillåta kontroll av ett booleskt HTML-attribut:

```xml
<input type="checkbox" checked="${properties.isChecked}"/>
```

För att ställa in attribut kan [`data-sly-attribute`](block-statements.md#attribute)-satsen också vara användbar.

## Vanliga mönster med HTML {#common-patterns-with-htl}

I det här avsnittet beskrivs några vanliga scenarier och hur du bäst löser dem med HTML-mallspråket.

### Läser in klientbibliotek {#loading-client-libraries}

I HTML läses klientbibliotek in via en hjälpmall från AEM, som du kommer åt via [`data-sly-use`](block-statements.md#use). Det finns tre tillgängliga mallar i den här filen som kan anropas via [`data-sly-call`](block-statements.md#template-call):

* **`css`** - Läser bara in CSS-filerna för de refererade klientbiblioteken.
* **`js`** - Läser bara in JavaScript-filer från de refererade klientbiblioteken.
* **`all`** - Läser in alla filer i de refererade klientbiblioteken (både CSS och JavaScript).

Varje hjälpmall förväntar sig ett **`categories`**-alternativ för att referera till de önskade klientbiblioteken. Det alternativet kan antingen vara en array med strängvärden eller en sträng som innehåller en kommaseparerad värdelista.

Här är två korta exempel:

### Läser in flera klientbibliotek fullständigt samtidigt {#loading-multiple-client-libraries-fully-at-once}

```xml
<sly data-sly-use.clientlib="/libs/granite/sightly/templates/clientlib.html"
     data-sly-call="${clientlib.all @ categories=['myCategory1', 'myCategory2']}"/>
```

### Referera till ett klientbibliotek i olika avsnitt på en sida {#referencing-a-client-library-in-different-sections-of-a-page}

```xml
<!doctype html>
<html data-sly-use.clientlib="/libs/granite/sightly/templates/clientlib.html">
    <head>
        <!-- HTML meta-data -->
        <sly data-sly-call="${clientlib.css @ categories='myCategory'}"/>
    </head>
    <body>
        <!-- page content -->
        <sly data-sly-call="${clientlib.js @ categories='myCategory'}"/>
    </body>
</html>
```

I det andra exemplet ovan, om HTML-elementen **`head`** och **`body`** placeras i olika filer, måste sedan **`clientlib.html`**-mallen läsas in i varje fil som behöver den.

Avsnittet i [mallen och anropet](block-statements.md#template-call)-programsatserna innehåller mer information om hur du deklarerar och anropar sådana mallar.

### Skicka data till klienten {#passing-data-to-the-client}

Det bästa och mest eleganta sättet att skicka data till klienten i allmänhet, men ännu mer med HTML, är att använda dataattribut.

I följande exempel visas hur logiken (som också kan skrivas i Java) kan användas för att på ett mycket enkelt sätt serialisera JSON-objektet som ska skickas till klienten, som sedan enkelt kan placeras i ett dataattribut:

```xml
<!--/* template.html file: */-->
<div data-sly-use.logic="logic.js" data-json="${logic.json}">...</div>
```

```javascript
/* logic.js file: */
use(function () {
    var myData = {
        str: "foo",
        arr: [1, 2, 3]
    };

    return {
        json: JSON.stringify(myData)
    };
});
```

Därifrån är det enkelt att föreställa sig hur ett JavaScript på klientsidan kan komma åt attributet och tolka JSON igen. Detta skulle till exempel vara motsvarande JavaScript som ska placeras i ett klientbibliotek:

```javascript
var elements = document.querySelectorAll("[data-json]");
for (var i = 0; i < elements.length; i++) {
    var obj = JSON.parse(elements[i].dataset.json);
    //console.log(obj);
}
```

### Arbeta med mallar på klientsidan {#working-with-client-side-templates}

Ett specialfall, där tekniken som förklaras i avsnittet [Lyftbegränsningar för specialkontexter](getting-started.md#lifting-limitations-of-special-contexts) kan användas på ett legitimt sätt, är att skriva klientsidesmallar (till exempel Handlebars) som finns i **script**-element. Anledningen till att den här tekniken kan användas i så fall är att elementet **script** inte innehåller JavaScript som förväntat, utan ytterligare HTML-element. Här är ett exempel på hur det skulle fungera:

```xml
<!--/* template.html file: */-->
<script id="entry-section" type="text/template"
    data-sly-include="entry-section.html"></script>

<!--/* entry-section.html file: */-->
<div class="entry-section">
    <h2 data-sly-test="${properties.entrySectionTitle}">
        ${properties.entrySectionTitle}
    </h2>
    {{#each entry}}<h3><a href="{{link}}">{{title}}</a></h3>{{/each}}
</div>
```

Som visas ovan kan koden som inkluderas i **`script`**-elementet innehålla HTML-blockprogramsatser och uttrycken behöver inte ange explicita kontexter eftersom innehållet i Handlebars-mallen har isolerats i sin egen fil. I det här exemplet visas även hur HTML-kod som körs på servern (som i elementet **`h2`**) kan blandas med ett mallspråk som körs på klientsidan, som Handlebars (visas i elementet **`h3`**).

En modernare teknik skulle emellertid vara att använda HTML-elementet **`template`** i stället, eftersom fördelen då skulle vara att det inte behövs för att isolera innehållet i mallarna i separata filer.

**Läs nästa:**

* [Uttrycksspråk](expression-language.md)  - för att i detalj lära dig vad som kan göras i HTML-uttryck.
* [Blockprogramsatser](block-statements.md)  - för att identifiera alla blockprogramsatser som är tillgängliga i HTML och hur du använder dem.

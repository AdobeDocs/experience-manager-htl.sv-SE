---
title: Komma igång med HTML
description: Läs om HTML, det rekommenderade och rekommenderade serverbaserade mallsystemet för HTML i AEM, och förstå de viktigaste begreppen för språket och dess grundläggande konstruktion.
exl-id: c95eb1b3-3b96-4727-8f4f-d54e7136a8f9
source-git-commit: a496d23277902a5cd573a6a8af770f27b0269f05
workflow-type: tm+mt
source-wordcount: '2077'
ht-degree: 0%

---


# Komma igång med HTML {#getting-started-with-htl}

HTML Template Language (HTL) är det rekommenderade serversidesmallsystemet för HTML i Adobe Experience Manager. Precis som i alla mallsystem på serversidan i HTML definierar en HTML-fil utdata som skickas till webbläsaren genom att ange själva HTML, viss grundläggande presentationslogik och variabler som ska utvärderas vid körning.

Det här dokumentet ger en översikt över syftet med HTML samt en introduktion till grundläggande begrepp och konstruktioner för språket.

>[!TIP]
>
>**Har du övervägt Edge Delivery Services för AEM?**
>
>Du kan fortsätta använda metoderna som beskrivs i det här dokumentet för befintliga projekt. För nya projekt rekommenderar Adobe att du använder [Edge Delivery Services.](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/edge-delivery/overview)

>[!TIP]
>
>I det här dokumentet presenteras syftet med HTML och en översikt över dess grundläggande struktur och begrepp. Om du har frågor om specifik syntax kan du läsa [HTML-specifikationen](specification.md).

## HTML-lager {#layers}

I AEM definierar ett antal lager HTML.

1. **[HTL-specifikation](specification.md)** - HTML är en öppen källkodsbaserad, plattformsoberoende specifikation som alla kan implementera.
1. **[`Sling`HTML-skriptmotor](specification.md)** - `Sling`-projektet har skapat referensimplementeringen av HTML, som används av AEM.
1. **[AEM-tillägg](specification.md)** - AEM bygger ovanpå `Sling` HTML-skriptmotorn och ger utvecklare praktiska funktioner som är specifika för AEM.

Denna HTML-dokumentation fokuserar på att använda HTML för att utveckla AEM-lösningar. Det berör alla tre lager och kopplar externa resurser efter behov.

## Grundläggande begrepp i HTML {#fundamental-concepts-of-htl}

I HTML Template Language används ett uttrycksspråk för att infoga innehållsdelar i den återgivna koden, och HTML5-dataattribut för att definiera satser över kodblock (som villkor eller iterationer). När HTML kompileras till Java-servrar utvärderas både uttrycken och HTML-dataattributen helt på serversidan och inget visas i den resulterande HTML.

>[!TIP]
>
>Om du vill köra de flesta exempel som finns på den här sidan kan du använda en körningsmiljö som kallas [Läs utvärderingsloop](https://github.com/adobe/aem-htl-repl).

### Block och uttryck {#blocks-and-expressions}

Här är ett första exempel som kan finnas som i en `template.html`-fil:

```xml
<h1 data-sly-test="${properties.jcr:title}">
    ${properties.jcr:title}
</h1>
```

Två olika typer av syntaxer kan särskiljas:

* **Blockprogramsatser** - Om du vill visa elementet `<h1>` villkorligt använder du ett `data-sly-test` HTML5-dataattribut. HTL innehåller flera sådana attribut, som tillåter att beteenden kopplas till ett HTML-element, och alla har prefixet `data-sly`.
* **Uttrycksspråk** - Tecknen `${` och `}` avgränsar HTML-uttryck. Vid körning utvärderas dessa uttryck och deras värde injiceras i den utgående HTML-strömmen.

Mer information om båda syntaxerna finns i [HTML-specifikationen](specification.md).

### Elementet SLY {#the-sly-element}

Ett centralt koncept för HTML är att erbjuda möjligheten att återanvända befintliga HTML-element för att definiera blocksatser. Med den här återanvändningen undviker du behovet av att infoga ytterligare avgränsare för att definiera var satsen börjar och slutar. När du kommenterar markeringen omvandlas statiska HTML diskret till dynamiska mallar utan att HTML-giltigheten bryts, vilket säkerställer korrekt visning även som statiska filer.

Ibland kanske det inte finns något befintligt element på exakt den plats där en blockprogramsats måste infogas. I så fall kan du infoga ett särskilt `sly`-element. Det här elementet tas automatiskt bort från utdata när de kopplade blockprogramsatserna körs och innehållet visas i enlighet med detta.

Följande exempel:

```xml
<sly data-sly-test="${properties.jcr:title && properties.jcr:description}">
    <h1>${properties.jcr:title}</h1>
    <p>${properties.jcr:description}</p>
</sly>
```

Skapar något som följande HTML, men bara om det finns både en `jcr:title`- och en `jcr:description`-egenskap definierad och om ingen av dem är tom:

```xml
<h1>MY TITLE</h1>
<p>MY DESCRIPTION</p>
```

En sak som du bör tänka på är att bara använda elementet `sly` när inget befintligt element kunde ha kommenterats med blockprogramsatsen. Orsaken är att `sly`-element hindrar det värde som språket erbjuder från att inte ändra den statiska HTML när den blir dynamisk.

Om det föregående exemplet till exempel skulle ha kapslats i ett `div`-element skulle det tillagda `sly`-elementet vara stötande:

```xml
<div>
    <sly data-sly-test="${properties.jcr:title && properties.jcr:description}">
        <h1>${properties.jcr:title}</h1>
        <p>${properties.jcr:description}</p>
    </sly>
</div>
```

Och elementet `div` kan ha kommenterats med villkoret:

```xml
<div data-sly-test="${properties.jcr:title && properties.jcr:description}">
    <h1>${properties.jcr:title}</h1>
    <p>${properties.jcr:description}</p>
</div>
```

### HTML-kommentarer {#htl-comments}

I följande exempel visas en HTML-kommentar på den första raden och en HTML-kommentar på den andra raden.

```xml
<!--/* An HTL Comment */-->
<!-- An HTML Comment -->
```

HTML-kommentarer är HTML-kommentarer med en ytterligare JavaScript-liknande syntax. Processorn ignorerar helt hela HTML-kommentaren och allt i den och tar bort den från utdata.

Innehållet i vanliga HTML-kommentarer skickas dock vidare och uttrycken i kommentaren utvärderas.

HTML kommentarer får inte innehålla HTML-kommentarer och vice versa.

### Särskilda sammanhang {#special-contexts}

För att kunna använda HTML på bästa sätt är det viktigt att du är väl införstådd med konsekvenserna av att det baseras på HTML syntax.

Mer information finns i avsnittet [Visningskontext](https://github.com/adobe/htl-spec/blob/1.4/SPECIFICATION.md#121-display-context) i HTML-specifikationen.

### Element- och attributnamn {#element-and-attribute-names}

Uttryck kan bara placeras i HTML text- eller attributvärden, men inte i elementnamn eller attributnamn, eller så är de inte längre giltiga i HTML. Om du vill ange elementnamn dynamiskt kan programsatsen `data-sly-element` användas på de önskade elementen och om du vill ange attributnamn dynamiskt, även om du anger flera attribut samtidigt, kan programsatsen `data-sly-attribute` användas.

```xml
<h1 data-sly-element="${myElementName}" data-sly-attribute="${myAttributeMap}">...</h1>
```

### Kontexter utan blocksatser {#contexts-without-block-statements}

Eftersom HTML använder dataattribut för att definiera blockprogramsatser är det inte möjligt att definiera sådana blockprogramsatser i följande sammanhang, och bara uttryck kan användas där:

* HTML kommentarer
* Skriptelement
* Formatelement

Anledningen till detta är att innehållet i dessa kontexter är text och inte HTML, och att de innehåller HTML-element skulle betraktas som enkla teckendata. Utan riktiga HTML-element kan inte heller `data-sly`-attribut köras.

Detta kan låta som en betydande begränsning. Det är dock att föredra eftersom HTML mallspråk endast ska generera giltiga HTML-utdata. Avsnittet [Använd-API för att komma åt logik](#use-api-for-accessing-logic) nedan visar hur ytterligare logik kan anropas från mallen, som kan användas om den behövs för att förbereda komplexa utdata för dessa kontexter. Om du vill skicka data från bakänden till ett front-end-skript genererar du en JSON-sträng med komponentens logik och placerar den i ett dataattribut med ett enkelt HTML-uttryck.

I följande exempel visas beteendet för HTML-kommentarer, men i skript- eller formatelement observeras samma beteende:

```xml
<!--
    The title is: ${properties.jcr:title}
    <h1 data-sly-test="${properties.jcr:title}">${properties.jcr:title}</h1>
-->
```

Ger ut något som följande HTML:

```xml
<!--
    The title is: MY TITLE
    <h1 data-sly-test="MY TITLE">MY TITLE</h1>
-->
```

### Explicit kontext krävs {#explicit-contexts-required}

Som förklaras i avsnittet [Automatisk kontextmedveten Escaping](#automatic-context-aware-escaping) nedan är ett av målen med HTML att minska riskerna med att införa XSS-problem (cross-site scripting) genom att automatiskt tillämpa kontextmedveten escape-konvertering på alla uttryck. HTL identifierar sammanhanget för uttryck i HTML-kod men analyserar inte infogad JavaScript eller CSS, så utvecklare måste ange den exakta kontexten för dessa uttryck.

Eftersom inte rätt escape-resultat används i XSS-sårbarheter, tar HTL därför bort utdata från alla uttryck som finns i skript- och formatkontexter när kontexten inte har deklarerats.

Här är ett exempel på hur du anger kontext för uttryck som är placerade i skript och format:

```xml
<script> var trackingID = "${myTrackingID @ context='scriptString'}"; </script>
<style> a { font-family: "${myFont @ context='styleString'}"; } </style>
```

Mer information om hur du styr escape-konverteringen finns i avsnittet [Visningssammanhang för uttrycksspråk](https://github.com/adobe/htl-spec/blob/master/SPECIFICATION.md#121-display-context) i HTML-specifikationerna.

## Allmänna funktioner för HTML {#general-capabilities-of-htl}

Här går vi igenom de allmänna funktionerna i HTML Template Language.

### Använd-API för att komma åt logik {#use-api-for-accessing-logic}

Java Use-API:t för HTML Template Language (HTL) ger en HTML-fil åtkomst till hjälpmetoder i en anpassad Java-klass via `data-sly-use`. Med den här processen kan all komplex affärslogik kapslas in i Java-koden, medan HTML-koden endast hanterar direkt markeringsproduktion.

Mer information finns i dokumentet [HTL Java Use-API](java-use-api.md).

### Automatisk kontextmedveten Escaping {#automatic-context-aware-escaping}

Titta på följande exempel:

```xml
<p data-sly-use.logic="logic.js">
    <a href="${logic.link}" title="${logic.title}">
        ${logic.text}
    </a>
</p>
```

I de flesta mallspråk skulle det här exemplet kunna skapa en XSS-säkerhetslucka (cross-site scripting), eftersom attributet `href` måste vara specifikt URL-escape även om alla variabler automatiskt är HTML-escape. Detta utelämnande är ett av de vanligaste felen, eftersom det är lätt att glömma och det är svårt att hitta på ett automatiserat sätt.

För att underlätta kan HTML mallspråk automatiskt undgå varje variabel i det sammanhang som variabeln placeras i. Detta är möjligt tack vare att HTML förstår HTML syntax.

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

Det inledande exemplet ger följande utdata:

```xml
<p>
    <a href="#my%20link%27s%20safe" title="my title&#39;s safe">
        my text&#39;s safe
    </a>
</p>
```

Observera hur de två attributen har escape-konverterats eftersom HTML vet att attributen `href` och `src` måste escape-konverteras för URI-kontext. Om URI:n startade med `javascript:` skulle attributet också ha tagits bort helt, om inte kontexten uttryckligen ändrades till något annat.

Mer information om hur du styr escape-konverteringen finns i avsnittet [Visningssammanhang för uttrycksspråk](https://github.com/adobe/htl-spec/blob/master/SPECIFICATION.md#121-display-context) i HTML-specifikationerna.

### Automatisk borttagning av tomma attribut {#automatic-removal-of-empty-attributes}

Titta på följande exempel:

```xml
<p class="${properties.class}">some text</p>
```

Om värdet för egenskapen `class` råkar vara tomt tar HTML mallspråk automatiskt bort hela attributet `class` från utdata.

Återigen är den här processen möjlig eftersom HTML förstår HTML-syntax och därför bara kan visa attribut med dynamiska värden om deras värde inte är tomt. Orsaken är mycket praktisk eftersom den undviker att lägga till ett villkorsblock runt attribut, vilket skulle ha gjort markeringen ogiltig och oläslig.

Dessutom gäller den typ av variabel som placerats i uttrycket:

* **Sträng:**
   * **inte tom:** Anger strängen som ett attributvärde.
   * **tom:** Tar bort attributet helt.

* **Number:** Anger värdet som ett attributvärde.

* **Boolean:**
   * **true:** Visar attributet utan värde (som ett booleskt HTML-attribut)
   * **false:** Tar bort attributet helt.

Här är ett exempel på hur ett booleskt uttryck skulle tillåta kontroll av ett booleskt HTML-attribut:

```xml
<input type="checkbox" checked="${properties.isChecked}"/>
```

För att ställa in attribut kan programsatsen `data-sly-attribute` också vara användbar.

## Vanliga mönster med HTML {#common-patterns-with-htl}

I det här avsnittet presenteras några vanliga scenarier. Här beskrivs hur du bäst löser dessa scenarier med HTML mallspråk.

### Läser in klientbibliotek {#loading-client-libraries}

I HTML läses klientbibliotek in via en hjälpmall från AEM, som du kommer åt via `data-sly-use`. Det finns tre mallar i den här filen som kan anropas via `data-sly-call`:

* **`css`** - Läser bara in CSS-filer för de refererade klientbiblioteken.
* **`js`** - Läser bara in JavaScript-filerna för de refererade klientbiblioteken.
* **`all`** - Läser in alla filer i de refererade klientbiblioteken (både CSS och JavaScript).

Varje hjälpmall förväntar sig ett `categories`-alternativ för att referera till de önskade klientbiblioteken. Det alternativet kan antingen vara en array med strängvärden eller en sträng som innehåller en kommaavgränsad värdeslista.

Här följer två korta exempel.

#### Läsa in flera klientbibliotek helt samtidigt {#loading-multiple-client-libraries-fully-at-once}

```xml
<sly data-sly-use.clientlib="/libs/granite/sightly/templates/clientlib.html"
     data-sly-call="${clientlib.all @ categories=['myCategory1', 'myCategory2']}"/>
```

#### Referera till ett klientbibliotek i olika avsnitt på en sida {#referencing-a-client-library-in-different-sections-of-a-page}

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

I det här exemplet måste mallen `head` läsas in i varje fil som kräver den om elementen `body` och `clientlib.html` finns i separata filer.

Avsnittet om mallen och anropssatserna i [HTML-specifikationen](specification.md) innehåller mer information om hur du deklarerar och anropar sådana mallar.

### Skicka data till klienten {#passing-data-to-the-client}

Det bästa och mest eleganta sättet att skicka data till klienten i allmänhet, men ännu mer med HTML, är att använda `data`-attribut.

I följande exempel visas hur du serialiserar ett objekt till JSON (även möjligt i Java) för att skicka det till klienten. Den kan sedan enkelt placeras i ett `data`-attribut:

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

Därifrån är det enkelt att föreställa sig hur en klientside JavaScript kan komma åt attributet och tolka JSON igen. Detta skulle vara den motsvarande JavaScript som ska placeras i ett klientbibliotek, till exempel:

```javascript
var elements = document.querySelectorAll("[data-json]");
for (var i = 0; i < elements.length; i++) {
    var obj = JSON.parse(elements[i].dataset.json);
    //console.log(obj);
}
```

### Arbeta med mallar på klientsidan {#working-with-client-side-templates}

Ett specialfall, där tekniken som förklaras i avsnittet [Lyftbegränsningar för specialkontexter](#lifting-limitations-of-special-contexts) kan användas på ett korrekt sätt, är att skriva klientsidesmallar (till exempel Handlebars) som finns i `scrip` -element. Anledningen till att den här tekniken kan användas i så fall är att elementet `script` inte innehåller JavaScript som förväntat, utan ytterligare HTML-element. Här är ett exempel på hur det skulle fungera:

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

`script`-elementets kod kan innehålla HTML-blocksatser utan explicita kontexter, eftersom innehållet i Handlebars-mallen isoleras i sin egen fil. I det här exemplet visas även hur HTML-kod som körs på servern (som i elementet `h2`) kan blandas med ett mallspråk som körs på klientsidan, som Handlebars (visas i elementet `h3`).

En modernare teknik alls skulle dock vara att använda elementet `template` i stället, eftersom fördelen då skulle vara att det inte behövs för att isolera innehållet i mallarna i separata filer.

### Upphävande av begränsningar i specialsammanhang {#lifting-limitations-of-special-contexts}

I de specialfall där det behövs för att kringgå begränsningarna för skript, format och kommentarskontexter, är det möjligt att isolera deras innehåll i en separat HTML-fil. HTML tolkar allt i sin egen fil som ett vanligt HTML-fragment och ignorerar alla begränsningar som fanns där det inkluderades.

Ett exempel finns i avsnittet [Arbeta med klientmallar](#working-with-client-side-templates) längre ned.

>[!CAUTION]
>
>Den här tekniken kan medföra XSS-problem (cross-site scripting). Säkerhetsaspekterna bör noggrant undersökas om denna metod används. Det finns vanligtvis bättre sätt att implementera samma sak än att förlita sig på den här metoden.

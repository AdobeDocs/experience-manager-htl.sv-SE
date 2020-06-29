---
title: HTML-uttrycksspråk
description: HTML-mallspråket använder ett uttrycksspråk för att komma åt de datastrukturer som innehåller de dynamiska elementen i HTML-utdata.
translation-type: tm+mt
source-git-commit: ee712ef61018b5e05ea052484e2a9a6b12e6c5c8
workflow-type: tm+mt
source-wordcount: '1848'
ht-degree: 0%

---


# HTML-uttrycksspråk {#htl-expression-language}

HTML-mallspråket använder ett uttrycksspråk för att komma åt de datastrukturer som innehåller de dynamiska elementen i HTML-utdata. Uttrycken avgränsas med tecken `${` och `}`. För att undvika felformaterad HTML kan uttryck bara användas i attributvärden, i elementinnehåll eller i kommentarer.

```xml
<!-- ${component.path} -->
<h1 class="${component.name}">
    ${properties.jcr:title}
</h1>
```

Uttryck kan föregås av ett `\` tecken som `\${test}` återges `${test}`.

>[!NOTE]
>
>Om du vill testa exemplen som finns på den här sidan kan du använda en körningsmiljö som kallas [Read Eval Print Loop](https://github.com/Adobe-Marketing-Cloud/aem-sightly-repl) .

Uttryckssyntaxen innehåller [variabler](#variables), [litteraler](#literals), [operatorer](#operators) och [alternativ](#options):

## Variabler {#variables}

Variabler är behållare som lagrar datavärden eller objekt. Variabelnamnen kallas identifierare.

Utan att behöva ange något ger HTML åtkomst till alla objekt som var vanliga i JSP efter att ha inkluderat `global.jsp`. På sidan [Globala objekt](global-objects.md) visas en lista med alla objekt som HTL ger åtkomst till.

### Egenskapsåtkomst {#property-access}

Det finns två sätt att komma åt egenskaper för variabler, med punktnotation eller med hakparentesnotation:

```xml
${currentPage.title}  
${currentPage['title']} or ${currentPage["title"]}
```

I de flesta fall bör den enklare punktnotationen vara att föredra, och hakparentesnotationen bör användas för att komma åt egenskaper som innehåller ogiltiga identifierartecken eller för att komma åt egenskaper dynamiskt. Följande två kapitel innehåller information om dessa två fall.

De åtkomliga egenskaperna kan vara funktioner, men att skicka argument stöds inte, så bara funktioner som inte förväntar sig argument kan nås, som get-metoder. Detta är en önskad begränsning, som är avsedd att minska mängden logik som är inbäddad i uttryck. Vid behov kan programsatsen användas för att skicka parametrar till logiken. [`data-sly-use`](block-statements.md#use)

I exemplet ovan visas också att Java-get-funktioner, som `getTitle()`exempelvis, kan nås utan att föregå `get`och genom att gemena till det efterföljande tecknet.

### Giltiga ID-tecken {#valid-identifier-characters}

Variabelnamnen, som kallas identifierare, följer vissa regler. De måste börja med en bokstav (`A`-`Z` och `a`-`z`) eller ett understreck (`_`), och efterföljande tecken kan också vara siffror (`0`-`9`) eller kolon (`:`). Unicode-bokstäver som `å` och `ü` kan inte användas i identifierare.

Eftersom kolontecknet (`:`) är vanligt i AEM-egenskapsnamn bör det understrykas att det är ett lämpligt identifierartecken:

`${properties.jcr:title}`

hakparenteser kan användas för att komma åt egenskaper som innehåller ogiltiga identifierartecken, som blankstegstecknet i exemplet nedan:

`${properties['my property']}`

### Åtkomst till medlemmar dynamiskt {#accessing-members-dynamically}

```xml
${properties[myVar]}
```

### Tillåten hantering av null-värden {#permissive-handling-of-null-values}

```xml
${currentPage.lastModified.time.toString}
```

## Explicita värden {#literals}

En litteral är en notation som representerar ett fast värde.

### Boolesk {#boolean}

Boolean representerar en logisk enhet och kan ha två värden: `true`och `false`.

`${true} ${false}`

### Nummer {#numbers}

Det finns bara en taltyp: positiva heltal. Andra talformat, som flyttal, stöds i variabler, men kan inte uttryckas som litteraler.

`${42}`

### Strängar {#strings}

Strängar representerar textdata och kan vara enkla eller dubbla citattecken:

`${'foo'} ${"bar"}`

Förutom vanliga tecken kan du använda följande specialtecken:

* `\\` Backslash-tecken
* `\'` Enkelt citattecken (eller apostrof)
* `\"` Dubbelt citattecken
* `\t` Tabb
* `\n` Ny rad
* `\r` Radretur
* `\f` Formulärfeed
* `\b` Backsteg
* `\uXXXX` Unicode-tecknet som anges med de fyra hexadecimala siffrorna XXXX.\
   Några användbara escape-sekvenser med unicode är:

   * `\u0022` for `"`
   * `\u0027` for `'`

För tecken som inte listas ovan visas ett fel före ett omvänt snedstreck.

Här är några exempel på hur du använder strängescape:

```xml
<p>${'it\'s great, she said "yes!"'}</p>
<p title="${'it\'s great, she said \u0022yes!\u0022'}">...</p>
```

som resulterar i följande utdata eftersom HTML använder kontextspecifik escape-konvertering:

```xml
<p>it&#39;s great, she said &#34;yes!&#34;</p>
<p title="it&#39;s great, she said &#34;yes!&#34;">...</p>
```

### Arrayer {#arrays}

En array är en sorterad uppsättning värden som kan refereras till med ett namn och ett index. Elementtyperna kan blandas.

```xml
${[1,2,3,4]}
${myArray[2]}
```

Arrayer är användbara för att ge en lista med värden från mallen.

```xml
<ul data-sly-list="${[1,2,3,4]}">
  <li>${item}</li>
</ul>
```

## Operatorer {#operators}

### Logiska operatorer {#logical-operators}

De här operatorerna används vanligtvis med booleska värden, men precis som i JavaScript returnerar de i själva verket värdet för en av de angivna operanderna, så när de används med icke-booleska värden kan de returnera ett icke-booleskt värde.

Om ett värde kan konverteras till `true`kallas värdet sann. Om ett värde kan konverteras till `false`kallas värdet falskt. Värden som kan konverteras till `false` är odefinierade variabler, null-värden, talet noll och tomma strängar.

#### Logiskt NOT {#logical-not}

`${!myVar}` returnerar `false` om dess enda operand kan konverteras till `true`, annars returneras `true`.

Det här kan till exempel användas för att invertera ett testvillkor, som att bara visa ett element om det inte finns några underordnade sidor:

```xml
<p data-sly-test="${!currentPage.hasChild}">current page has no children</p>
```

#### Logiskt OCH {#logical-and}

`${varOne && varTwo}` återgår `varOne` om den är falsk, annars returneras `varTwo`.

Den här operatorn kan användas för att testa två villkor samtidigt, som att verifiera att det finns två egenskaper:

```xml
<div data-sly-test="${properties.jcr:title && properties.jcr:description}">
    <h1>${properties.jcr:title}</h1>
    <p>${properties.jcr:description}</p>
</div>
```

Den logiska AND-operatorn kan också användas för att villkorligt visa HTML-attribut, eftersom HTML tar bort attribut med dynamiskt angivna värden som utvärderas till false, eller till en tom sträng. Så i exemplet nedan visas bara `class` attributet om `logic.showClass` är sann och om den `logic.className` finns och inte är tom:

```xml
<div class="${logic.showClass && logic.className}">...</div>
```

#### Logiskt ELLER {#logical-or}

`${varOne || varTwo}` returnerar `varOne` om den är sann, annars returneras `varTwo`.

Den här operatorn kan användas för att testa om ett av två villkor är uppfyllt, som att kontrollera om det finns minst en egenskap:

```xml
<div data-sly-test="${properties.jcr:title || properties.jcr:description}">...</div>
```

När den logiska OR-operatorn returnerar den första variabeln som är sann, kan den också användas för att tillhandahålla reservvärden.

Visa HTML-attribut på ett villkorligt sätt eftersom HTML tar bort attribut med värden som anges av uttryck som utvärderas till false eller till en tom sträng. I exemplet nedan visas **`properties.jcr:`** titeln om den finns och inte är tom, annars visas den **`properties.jcr:description`** om den finns och inte är tom. Annars visas meddelandet&quot;ingen rubrik eller beskrivning har angetts&quot;:

```xml
<p>${properties.jcr:title || properties.jcr:description || "no title or description provided"}</p>
```

### Villkorlig (ternär) operator {#conditional-ternary-operator}

`${varCondition ? varOne : varTwo}` returnerar `varOne` om `varCondition` är sann, annars returneras `varTwo`.

Den här operatorn kan vanligtvis användas för att definiera villkor i uttryck, som att visa ett annat meddelande baserat på sidans status:

```xml
<p>${currentPage.isLocked ? "page is locked" : "page can be edited"}</p>
```

>[!TIP]
>
>Eftersom kolontecken även tillåts i identifierare är det bäst att separera de ternära operatorerna med ett tomt utrymme för att tolken ska bli tydlig:

```xml
<p>${properties.showDescription ? properties.jcr:description : properties.jcr:title}</p>
```

### Jämförelseoperatorer {#comparison-operators}

Likhets- och olikhetsoperatorer stöder bara operander som är av identiska typer. När typerna inte matchar visas ett fel.

* Strängar är lika när de har samma teckensekvens.
* Tal är lika när de har samma värde
* Booleaner är lika om båda är `true` eller båda är `false`.
* Null- eller undefined-variabler är lika med sig själva och med varandra.

`${varOne == varTwo}` returnerar `true` om `varOne` och `varTwo` är lika.

`${varOne != varTwo}` returnerar `true` om `varOne` och `varTwo` inte är lika.

Relationsoperatorer stöder bara operander som är tal. För alla andra typer visas ett fel.

`${varOne > varTwo}` returnerar `true` om `varOne` är större än `varTwo`.

`${varOne < varTwo}` returneras `true` om `varOne` är mindre än `varTwo`.

`${varOne >= varTwo}` returnerar `true` om `varOne` är större eller lika med `varTwo`.

`${varOne <= varTwo}` returneras `true` om `varOne` är mindre eller lika med `varTwo`.

### Gruppera parenteser {#grouping-parentheses}

Grupperingsoperatorn `()` styr utvärderingens prioritet i uttryck.

`${varOne && (varTwo || varThree)}`

## Alternativ {#options}

Uttrycksalternativen kan agera på uttrycket och ändra det, eller fungera som parametrar när de används tillsammans med blocksatser.

Allt efter `@` är ett alternativ:

```xml
${myVar @ optOne}
```

Alternativen kan ha ett värde, som kan vara en variabel eller en litteral:

```xml
${myVar @ optOne=someVar}
${myVar @ optOne='bar'}
${myVar @ optOne=10}
${myVar @ optOne=true}
```

Flera alternativ avgränsas med kommatecken:

```xml
${myVar @ optOne, optTwo=bar}
```

Parametriska uttryck som bara innehåller alternativ är också möjliga:

```xml
${@ optOne, optTwo=bar}
```

### Strängformatering {#string-formatting}

Alternativ som ersätter de uppräknade platshållarna, {*n*}, med motsvarande variabel:

```xml
${'Page {0} of {1}' @ format=[current, total]}
```

## URL-hantering {#url-manipulation}

Det finns en ny uppsättning URL-ändringar.

Se följande exempel på deras användning:

Lägger till html-tillägget i en bana.

```xml
<a href="${item.path @ extension = 'html'}">${item.name}</a>
```

Lägger till HTML-tillägget och en väljare i en bana.

```xml
<a href="${item.path @ extension = 'html', selectors='products'}">${item.name}</a>
```

Lägger till HTML-tillägget och ett fragment (#value) till en bana.

```xml
<a href="${item.path @ extension = 'html', fragment=item.name}">${item.name}</a>
```

Detta `@extension` fungerar i alla scenarier och kontrollerar om tillägget ska läggas till eller inte.

```xml
${ link @ extension = 'html' }
```

### Tal-/datumformatering {#number-date-formatting}

HTML tillåter intern formatering av tal och datum utan att behöva skriva egen kod. Detta stöder även tidszon och nationella inställningar.

I följande exempel visas att formatet anges först och sedan det värde som behöver formateras:

```xml
<h2>${ 'dd-MMMM-yyyy hh:mm:ss' @
           format=currentPage.lastModified,
           timezone='PST',
           locale='fr'}</h2>

<h2>${ '#.00' @ format=300}</h2>
```

>[!NOTE]
>
>Fullständig information om det format du kan använda finns i [HTML-specifikationen](https://github.com/Adobe-Marketing-Cloud/htl-spec/blob/master/SPECIFICATION.md).

### Internationalisering {#internationalization}

Översätter strängen till språket i den aktuella *källan* (se nedan) med den aktuella [ordlistan](https://docs.adobe.com/content/help/en/experience-manager-65/developing/components/internationalization/i18n-translator.html). Om ingen översättning hittas används den ursprungliga strängen.

```xml
${'Page' @ i18n}
```

Tipsalternativet kan användas för att ge en kommentar till översättare och ange i vilket sammanhang texten ska användas:

```xml
${'Page' @ i18n, hint='Translation Hint'}
```

Standardkällan för språket är `resource`, vilket innebär att texten översätts till samma språk som innehållet. Detta kan ändras till `user`, vilket innebär att språket hämtas från webbläsarens språkområde eller från den inloggade användarens språkområde:

```xml
${'Page' @ i18n, source='user'}
```

Om du anger en explicit språkinställning åsidosätts källinställningarna:

```xml
${'Page' @ i18n, locale='en-US'}
```

Om du vill bädda in variabler i en översatt sträng kan du använda formatalternativet:

```xml
${'Page {0} of {1}' @ i18n, format=[current, total]}
```

### Matriskoppling {#array-join}

Som standard visas kommaavgränsade värden (utan mellanrum) i HTML när en array visas som text.

Använd alternativet join för att ange en annan avgränsare:

```xml
${['one', 'two'] @ join='; '}
```

### Visningssammanhang {#display-context}

Visningssammanhanget för ett HTML-uttryck refererar till dess plats i HTML-sidans struktur. Om uttrycket till exempel visas på plats som skulle skapa en textnod när den återgavs, sägs det vara i ett `text` sammanhang. Om det hittas inom värdet för ett attribut, sägs det vara i ett `attribute` sammanhang och så vidare.

Med undantag för skript- (JS) och stilkontexter (CSS) identifierar HTML automatiskt uttryckens kontext och undviker dem på rätt sätt för att förhindra XSS-säkerhetsproblem. För skript och CSS måste det önskade sammanhanget anges explicit. Dessutom kan sammanhangsbeteendet anges explicit i alla andra fall där du vill åsidosätta det automatiska beteendet.

Här finns tre variabler i tre olika sammanhang:

* `properties.link` ( `uri` kontext)
* `properties.title` (`attribute` kontext)
* `properties.text` (`text` kontext)

HTML kommer att kringgå dessa olika typer i enlighet med säkerhetskraven i deras respektive sammanhang. Ingen explicit kontextinställning krävs i vanliga fall som den här:

```xml
<a href="${properties.link}" title="${properties.title}">${properties.text}</a>
```

För att skapa en säker utdatamarkering (d.v.s. där själva uttrycket utvärderas till HTML) används `html` kontexten:

```xml
<div>${properties.richText @ context='html'}</div>
```

Explicit kontext måste anges för formatkontexter:

```xml
<span style="color: ${properties.color @ context='styleToken'};">...</span>
```

Explicit kontext måste anges för skriptkontexter:

```xml
<span onclick="${properties.function @ context='scriptToken'}();">...</span>
```

Escaping- och XSS-skydd kan också stängas av:

```xml
<div>${myScript @ context='unsafe'}</div>
```

### Kontextinställningar {#context-settings}

| Kontext | När ska användas | Vad det gör |
|--- |--- |--- |
| `text` | Standard för innehåll inuti element | Kodar alla HTML-specialtecken. |
| `html` | För säker utskrift | Filtrerar HTML så att den uppfyller AntiSamy-policyreglerna och tar bort det som inte matchar reglerna. |
| `attribute` | Standard för attributvärden | Kodar alla HTML-specialtecken. |
| `uri` | Så här visar du länkar och sökvägar: Standard för href- och src-attributvärden | Validerar URI för att skriva som ett href- eller src-attributvärde. Inget returneras om valideringen misslyckas. |
| `number` | Visa tal | Validerar URI för att innehålla ett heltal, returnerar noll om valideringen misslyckas. |
| `attributeName` | Standard för attributnamn utan data när attributnamn anges | Validerar attributnamnet, returnerar ingenting om valideringen misslyckas. |
| `elementName` | Standard för element som är sparade i datan | Validerar elementnamnet, returnerar ingenting om valideringen misslyckas. |
| `scriptToken` | För JS-identifierare, literala tal eller literala strängar | Validerar JavaScript-token, returnerar ingenting om valideringen misslyckas. |
| `scriptString` | Inom JS-strängar | Kodar tecken som skulle bryta ut ur strängen. |
| `scriptComment` | I JS-kommentarer | Validerar JavaScript-kommentaren och returnerar ingenting om valideringen misslyckas. |
| `styleToken` | För CSS-identifierare, tal, dimensioner, strängar, hexadecimala färger eller funktioner. | Validerar CSS-token, returnerar ingenting om valideringen misslyckas. |
| `styleString` | Inom CSS-strängar | Kodar tecken som skulle bryta ut ur strängen. |
| `styleComment` | I CSS-kommentarer | Validerar CSS-kommentaren, returnerar ingenting om valideringen misslyckas. |
| `unsafe` | Bara om ingen av ovanstående utför jobbet | Inaktiverar flytning och XSS-skydd fullständigt. |

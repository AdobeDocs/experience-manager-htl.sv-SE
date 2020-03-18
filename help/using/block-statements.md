---
title: HTML-blocksatser
seo-title: HTML-blocksatser
description: HTML Template Language (HTL) block statements are custom data attributes added directly to existing HTML.
seo-description: 'HTML Template Language (HTL) block statements are custom data attributes added directly to existing HTML. '
uuid: 0624fb6e-6989-431b-aabc-1138325393f1
contentOwner: User
products: SG_EXPERIENCEMANAGER/HTL
topic-tags: html-template-language
content-type: reference
discoiquuid: 58aa6ea8-1d45-4f6f-a77e-4819f593a19d
mwpw-migration-script-version: 2017-10-12T21 46 58.665-0400
translation-type: tm+mt
source-git-commit: afc29cbad83caeb549097da3fc33fd9147f1157a

---


# HTL Block Statements {#htl-block-statements}

HTML Template Language (HTL) block statements are custom `data` attributes added directly to existing HTML. This allows easy and unobtrusive annotation of a prototype static HTML page, converting it to a functioning dynamic template without breaking the validity of the HTML code.

## sly element {#sly-element}

The **&lt;sly> element** does not get displayed in the resulting HTML and can be used instead of the data-sly-unwrap. The goal of the &lt;sly> element is to make it more obvious that the element is not outputted. If you want you can still use data-sly-unwrap.

```xml
<sly data-sly-test.varone="${properties.yourProp}"/>
```

As with data-sly-unwrap, try to minimize the use of this.

## use {#use}

**`data-sly-use`**: Initializes a helper object (defined in JavaScript or Java) and exposes it through a variable.

Initialize a JavaScript object, where the source file is located in the same directory as the template. Note that the filename must be used:

```xml
<div data-sly-use.nav="navigation.js">${nav.foo}</div>
```

Initialize a Java class, where the source file is located in the same directory as the template. Note that the classname must be used, not the file name:

```xml
        <div data-sly-use.nav="Navigation">${nav.foo}</div>
```

Initiera en Java-klass, där den klassen installeras som en del av ett OSGi-paket. Note that its fully-qualified class name must be used:

```xml
        <div data-sly-use.nav="org.example.Navigation">${nav.foo}</div>
```

Parametrar kan skickas till initieringen med *alternativ*. Generally this feature should only be used by HTL code that is itself within a `data-sly-template` block:

```xml
<div data-sly-use.nav="${'navigation.js' @parentPage=currentPage}">${nav.foo}</div>
```

Initiera en annan HTML-mall som sedan kan anropas med **`data-sly-call`**:

```xml
<div data-sly-use.nav="navTemplate.html" data-sly-call="${nav.foo}"></div>
```

>[!NOTE]
>
>For more information on the Use-API, see:
>
>* [Java Use-API](use-api-java.md)
>* [JavaScript Use-API](use-api-javascript.md)
>



## unwrap {#unwrap}

**`data-sly-unwrap`**: Removes the host element from the generated markup while retaining its content. Detta gör att element som krävs som en del av HTML-presentationslogiken kan uteslutas, men som inte är önskade i det faktiska resultatet.

However, this statement should be used sparingly. I allmänhet är det bättre att hålla HTML-koden så nära den avsedda utdatakoden som möjligt. När du lägger till HTML-blockprogramsatser kan du med andra ord försöka så mycket som möjligt för att bara kommentera den befintliga HTML-koden, utan att införa nya element.

Den här

```xml
<p data-sly-use.nav="navigation.js">Hello World</p>
```

skapar

```xml
<p>Hello World</p>
```

Detta

```xml
<p data-sly-use.nav="navigation.js" data-sly-unwrap>Hello World</p>
```

skapar

```xml
Hello World
```

Du kan också villkorligt dela upp ett element:

```xml
<div class="popup" data-sly-unwrap="${isPopup}">content</div>
```

## text {#text}

**`data-sly-text`**: Ersätter innehållet i värdelementet med den angivna texten.

Den här

```xml
<p>${properties.jcr:description}</p>
```

likvärdigt med

```xml
<p data-sly-text="${properties.jcr:description}">Lorem ipsum</p>
```

Båda visar värdet för **`jcr:description`** som stycketext. Fördelen med den andra metoden är att den tillåter den diskreta kommenteringen av HTML samtidigt som det statiska platshållarinnehållet behålls från den ursprungliga designern.

## attribute {#attribute}

**data-sly-attribute**: Lägger till attribut i värdelementet.

Den här

```xml
<div title="${properties.jcr:title}"></div>
```

likvärdigt med

```xml
<div title="Lorem Ipsum" data-sly-attribute.title="${properties.jcr:title}"></div>
```

Båda ställer in attributet till `title` värdet för **`jcr:title`**. Fördelen med den andra metoden är att den tillåter den diskreta kommenteringen av HTML samtidigt som det statiska platshållarinnehållet behålls från den ursprungliga designern.

Attribut tolkas från vänster till höger, där förekomsten längst till höger av ett attribut (antingen literal eller definierad via **`data-sly-attribute`**) har företräde framför instanser av samma attribut (definierat antingen bokstavligt eller via **`data-sly-attribute`**) som definierats till vänster.

Observera att ett attribut (antingen **`literal`** eller inställt via **`data-sly-attribute`**) vars värde *utvärderas* till den tomma strängen kommer att tas bort i den slutliga koden. Ett undantag till den här regeln är att ett *literalt* attribut som är inställt på en tom *sträng* bevaras. Exempel:

```xml
<div class="${''}" data-sly-attribute.id="${''}"></div>
```

skapar,

```xml
<div></div>
```

men

```xml
<div class="" data-sly-attribute.id=""></div>
```

skapar,

```xml
<div class=""></div>
```

Om du vill ange flera attribut skickar du ett mappningsobjekt med nyckelvärdepar som motsvarar attributen och deras värden. Anta till exempel att

```xml
attrMap = {
    title: "myTitle",
    class: "myClass",
    id: "myId"
}
```

Sedan

```xml
<div data-sly-attribute="${attrMap}"></div>
```

skapar,

```xml
<div title="myTitle" class="myClass" id="myId"></div>
```

## element {#element}

**`data-sly-element`**: Ersätter elementnamnet för värdelementet.

Exempel:

```xml
<h1 data-sly-element="${titleLevel}">text</h1>
```

Ersätter **`h1`** värdet med värdet för **`titleLevel`**.

Av säkerhetsskäl accepteras endast följande elementnamn i `data-sly-element` :

```xml
a abbr address article aside b bdi bdo blockquote br caption cite code col colgroup
data dd del dfn div dl dt em figcaption figure footer h1 h2 h3 h4 h5 h6 header i ins
kbd li main mark nav ol p pre q rp rt ruby s samp section small span strong sub 
sup table tbody td tfoot th thead time tr u var wbr
```

XSS-skyddet måste vara inaktiverat ( `@context='unsafe'`) för att andra element ska kunna anges.

## test {#test}

**`data-sly-test`:**Värdelementet och dess innehåll tas bort villkorligt. Ett värde som`false`tar bort elementet. ett värde som`true`behåller elementet.

Elementet och dess innehåll återges till exempel bara om `p` är `isShown` `true`:

```xml
<p data-sly-test="${isShown}">text</p>
```

Resultatet av ett test kan tilldelas en variabel som kan användas senare. Detta används vanligtvis för att konstruera &quot;if else&quot;-logik eftersom det inte finns någon explicit else-sats:

```xml
<p data-sly-test.abc="${a || b || c}">is true</p>
<p data-sly-test="${!abc}">or not</p>
```

När variabeln har angetts har den ett globalt omfång i HTML-filen.

Nedan följer några exempel på hur du jämför värden:

```xml
<div data-sly-test="${properties.jcr:title == 'test'}">TEST</div>
<div data-sly-test="${properties.jcr:title != 'test'}">NOT TEST</div>

<div data-sly-test="${properties['jcr:title'].length > 3}">Title is longer than 3</div>
<div data-sly-test="${properties['jcr:title'].length >= 0}">Title is longer or equal to zero </div> 

<div data-sly-test="${properties['jcr:title'].length > aemComponent.MAX_LENGTH}">
    Title is longer than the limit of ${aemComponent.MAX_LENGTH}
</div>
```

## upprepa {#repeat}

Med data-sly-repeat kan du *upprepa* ett element flera gånger baserat på den angivna listan.

```xml
<div data-sly-repeat="${currentPage.listChildren}">${item.name}</div>
```

Detta fungerar på samma sätt som en datavänlig lista, förutom att du inte behöver ett behållarelement.

I följande exempel visas att du även kan referera till *objektet* för attribut:

```xml
<div data-sly-repeat="${currentPage.listChildren}" data-sly-attribute.class="${item.name}">${item.name}</div>
```

## list {#list}

**`data-sly-list`**: Upprepar innehållet i host-elementet för varje uppräkningsbar egenskap i det angivna objektet.

Här är en enkel slinga:

```xml
<dl data-sly-list="${currentPage.listChildren}">
    <dt>index: ${itemList.index}</dt>
    <dd>value: ${item.title}</dd>
</dl>
```

Följande standardvariabler är tillgängliga inom listans omfång:

**`item`**: Det aktuella objektet i iterationen.

**`itemList`**: Objekt som innehåller följande egenskaper:

**`index`**: nollbaserad räknare ( `0..length-1`).

**`count`**: en-baserad räknare ( `1..length`).

`first`: `true` om det aktuella objektet är det första objektet.

**`middle`**: `true` om det aktuella objektet varken är det första eller det sista objektet.

**`last`**: `true` om det aktuella objektet är det sista objektet.

**`odd`**: `true` om `index` är udda.

**`even`**: `true` om `index` är jämnt.

Genom att definiera en identifierare för `data-sly-list` -programsatsen kan du byta namn på **`itemList`** - och `item` -variablerna. **`item`** blir **** `<variable>`*** och **`itemList`** blir **`*<variable>*List`**.

```xml
<dl data-sly-list.child="${currentPage.listChildren}">
    <dt>index: ${childList.index}</dt>
    <dd>value: ${child.title}</dd>
</dl>
```

Du kan också komma åt egenskaper dynamiskt:

```xml
<dl data-sly-list.child="${myObj}">
    <dt>key: ${child}</dt>
    <dd>value: ${myObj[child]}</dd>
</dl>
```

## resurs {#resource}

**`data-sly-resource`**: Inkluderar resultatet av återgivningen av den angivna resursen via segmentupplösningen och återgivningsprocessen.

En enkel resurs är:

```xml
<article data-sly-resource="path/to/resource"></article>
```

Alternativ tillåter ett antal ytterligare varianter:

Ändra resurssökvägen:

```xml
<article data-sly-resource="${ @ path='path/to/resource'}"></article>
<article data-sly-resource="${'resource' @ prependPath='my/path'}"></article>
<article data-sly-resource="${'my/path' @ appendPath='resource'}"></article>
```

Lägg till (eller ersätt) en väljare:

```xml
<article data-sly-resource="${'path/to/resource' @ selectors='selector'}"></article>
```

Lägga till, ersätta eller ta bort flera väljare:

```xml
<article data-sly-resource="${'path/to/resource' @ selectors=['s1', 's2']}"></article>
```

Lägg till en väljare till de befintliga:

```xml
<article data-sly-resource="${'path/to/resource' @ addSelectors='selector'}"></article>
```

Ta bort några väljare från de befintliga:

```xml
<article data-sly-resource="${'path/to/resource' @ removeSelectors='selector1'}"></article>
```

Ta bort alla väljare:

```xml
<article data-sly-resource="${'path/to/resource' @ removeSelectors}"></article>
```

Åsidosätter resursens resurstyp:

```xml
<article data-sly-resource="${'path/to/resource' @ resourceType='my/resource/type'}"></article>
```

Ändrar WCM-läge:

```xml
<article data-sly-resource="${'path/to/resource' @ wcmmode='disabled'}"></article>
```

Som standard är AEM-dekorationstaggarna inaktiverade, med alternativet decorationTagName kan du hämta dem tillbaka och med cssClassName kan du lägga till klasser i det elementet.

```xml
<article data-sly-resource="${'path/to/resource' @ decorationTagName='span',
cssClassName='className'}"></article>
```

>[!NOTE]
>
>AEM erbjuder tydlig och enkel logik som styr de dekorationstaggar som omsluter de inkluderade elementen. Mer information finns i [Dekoration Tag](https://helpx.adobe.com/experience-manager/6-4/sites/developing/using/decoration-tag.html) (Dekoration Tag) i dokumentationen för komponenter under utveckling.

## include {#include}

**`data-sly-include`**: Ersätter innehållet i host-elementet med koden som genereras av den angivna HTML-mallfilen (HTL, JSP, ESP osv.) när den bearbetas av motsvarande mallmotor. Återgivningssammanhanget för den *inkluderade filen* kommer inte att omfatta den aktuella HTML-kontexten (den för *inkluderingsfilen*). För att HTML-filer ska kunna inkluderas måste därför aktuell **`data-sly-use`** upprepas i den inkluderade filen (i så fall är det oftast bättre att använda **`data-sly-template`** och `data-sly-call`)

En enkel sak:

```xml
<section data-sly-include="path/to/template.html"></section>
```

JSP kan inkluderas på samma sätt:

```xml
<section data-sly-include="path/to/template.jsp"></section>
```

Med alternativen kan du ändra filens sökväg:

```xml
<section data-sly-include="${ @ file='path/to/template.html'}"></section>
<section data-sly-include="${'template.html' @ prependPath='my/path'}"></section>
<section data-sly-include="${'my/path' @ appendPath='template.html'}"></section>
```

Du kan också ändra WCM-läge:

```xml
<section data-sly-include="${'template.html' @ wcmmode='disabled'}"></section>
```

## mall och samtal {#template-call}

`data-sly-template`: Definierar en mall. Värdelementet och dess innehåll genereras inte av HTML

`data-sly-call`: Anropar en mall som är definierad med en datamall. Innehållet i den anropade mallen (eventuellt parametriserat) ersätter innehållet i värdelementet i anropet.

Definiera en statisk mall och anropa den sedan:

```xml
<template data-sly-template.one>blah</template>
<div data-sly-call="${one}"></div>
```

Definiera en dynamisk mall och anropa den sedan med parametrar:

```xml
<template data-sly-template.two="${ @ title}"><h1>${title}</h1></template>
<div data-sly-call="${two @ title=properties.jcr:title}"></div>
```

Mallar som finns i en annan fil kan initieras med `data-sly-use`. Observera att i det här fallet `data-sly-use` och `data-sly-call` även kan placeras på samma element:

```xml
<div data-sly-use.lib="templateLib.html">
    <div data-sly-call="${lib.one}"></div>
    <div data-sly-call="${lib.two @ title=properties.jcr:title}"></div>
</div>
```

Mallrekursion stöds:

```xml
<template data-sly-template.nav="${ @ page}">
    <ul data-sly-list="${page.listChildren}">
        <li>
            <div class="title">${item.title}</div>
            <div data-sly-call="${nav @ page=item}" data-sly-unwrap></div>
        </li>
    </ul>
</template>
<div data-sly-call="${nav @ page=currentPage}" data-sly-unwrap></div>
```

## i18n och språkområdesobjekt {#i-n-and-locale-objects}

När du använder i18n och HTML kan du nu även skicka anpassade språkområdesobjekt.

```xml
${'Hello World' @ i18n, locale=request.locale}
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

## HTML-funktioner som stöds i AEM 6.3 {#htl-features-supported-in-aem}

Följande nya HTML-funktioner stöds i Adobe Experience Manager (AEM) 6.3:

### Tal-/datumformatering {#number-date-formatting}

AEM 6.3 har stöd för intern formatering av siffror och datum, utan att behöva skriva egen kod. Detta stöder även tidszon och nationella inställningar.

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

### datadriven användning med resurser {#data-sly-use-with-resources}

Detta gör att du kan hämta resurser direkt i HTML med databaserad användning och kräver inte att du skriver kod för att hämta resursen.

Exempel:

```xml
<div data-sly-use.product=“/etc/commerce/product/12345”>
  ${ product.title }
</div>
```

### Request-attributes {#request-attributes}

I det *datavänliga inkluderings* - och *datavänliga resursläget* kan du nu skicka *requestAttributes* för att använda dem i det mottagande HTML-skriptet.

På så sätt kan du skicka in parametrar korrekt i skript eller komponenter.

```xml
<sly data-sly-use.settings="com.adobe.examples.htl.core.hashmap.Settings" 
        data-sly-include="${ 'productdetails.html' @ requestAttributes=settings.settings}" />
```

Java-kod i klassen Settings används för att skicka requestAttributes:

```xml
public class Settings extends WCMUsePojo {

  // used to pass is requestAttributes to data-sly-resource
  public Map<String, Object> settings = new HashMap<String, Object>();

  @Override
  public void activate() throws Exception {
    settings.put("layout", "flex");
  }
}
```

Via en delmodell kan du till exempel använda värdet för de angivna requestAttributes.

I det här exemplet injiceras *layout* via kartan från klassen Use:

```xml
@Model(adaptables=SlingHttpServletRequest.class)
public class ProductSettings {
  @Inject @Optional @Default(values="empty")
  public String layout;

}
```

### Korrigera för @extension {#fix-for-extension}

Tillägget @fungerar i alla scenarier i AEM 6.3 innan du kan få ett resultat som *www.adobe.com.html* och kontrollerar även om du vill lägga till tillägget eller inte.

```xml
${ link @ extension = 'html' }
```

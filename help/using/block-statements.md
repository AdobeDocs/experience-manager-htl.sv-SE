---
title: HTML-blocksatser
description: HTML-blocksatser (Template Language) är anpassade dataattribut som läggs till direkt i befintlig HTML.
exl-id: a517dcef-ab7a-4d4c-a1a9-2e57aad034f7
source-git-commit: 89b9e89254f341e74f1a5a7b99735d2e69c8a91e
workflow-type: tm+mt
source-wordcount: '1555'
ht-degree: 1%

---

# HTML-blocksatser {#htl-block-statements}

HTML-blocksatser (Template Language) är anpassade `data`-attribut som läggs till direkt i befintlig HTML. Detta gör det enkelt och smidigt att kommentera en prototyplös statisk HTML-sida och konvertera den till en fungerande dynamisk mall utan att HTML-kodens giltighet bryts.

## Blockera översikt {#overview}

HTML-blockplugin-program definieras av `data-sly-*`-attribut som angetts för HTML-element. Elementen kan ha en avslutande tagg eller vara självavslutande. Attribut kan ha värden (som kan vara statiska strängar eller uttryck) eller helt enkelt vara booleska attribut (utan värde).

```xml
<tag data-sly-BLOCK></tag>                                 <!--/* A block is simply consists in a data-sly attribute set on an element. */-->
<tag data-sly-BLOCK/>                                      <!--/* Empty elements (without a closing tag) should have the trailing slash. */-->
<tag data-sly-BLOCK="string value"/>                       <!--/* A block statement usually has a value passed, but not necessarily. */-->
<tag data-sly-BLOCK="${expression}"/>                      <!--/* The passed value can be an expression as well. */-->
<tag data-sly-BLOCK="${@ myArg='foo'}"/>                   <!--/* Or a parametric expression with arguments. */-->
<tag data-sly-BLOCKONE="value" data-sly-BLOCKTWO="value"/> <!--/* Several block statements can be set on a same element. */-->
```

Alla utvärderade `data-sly-*`-attribut tas bort från den genererade koden.

### ID {#identifiers}

En blockprogramsats kan också följas av en identifierare:

```xml
<tag data-sly-BLOCK.IDENTIFIER="value"></tag>
```

Identifieraren kan användas av blockprogramsatsen på olika sätt. Här är några exempel:

```xml
<!--/* Example of statements that use the identifier to set a variable with their result: */-->
<div data-sly-use.navigation="MyNavigation">${navigation.title}</div>
<div data-sly-test.isEditMode="${wcmmode.edit}">${isEditMode}</div>
<div data-sly-list.child="${currentPage.listChildren}">${child.properties.jcr:title}</div>
<div data-sly-template.nav>Hello World</div>

<!--/* The attribute statement uses the identifier to know to which attribute it should apply it's value: */-->
<div data-sly-attribute.title="${properties.jcr:title}"></div> <!--/* This will create a title attribute */-->
```

Identifierare på den översta nivån är inte skiftlägeskänsliga (eftersom de kan anges med HTML-attribut som inte är skiftlägeskänsliga), men alla deras egenskaper är skiftlägeskänsliga.

## Tillgängliga blocksatser {#available-block-statements}

Det finns ett antal blocksatser tillgängliga. När det används för samma element definierar följande prioritetslista hur blockprogramsatser utvärderas:

1. `data-sly-template`
1. `data-sly-set`, `data-sly-test`, `data-sly-use`
1. `data-sly-call`
1. `data-sly-text`
1. `data-sly-element`,  `data-sly-include`,  `data-sly-resource`
1. `data-sly-unwrap`
1. `data-sly-list`, `data-sly-repeat`
1. `data-sly-attribute`

När två blockprogramsatser har samma prioritet, är deras utvärderingsordning från vänster till höger.

### use {#use}

`data-sly-use` initierar ett hjälpobjekt (definieras i JavaScript eller Java) och visar det genom en variabel.

Initiera ett JavaScript-objekt, där källfilen finns i samma katalog som mallen. Observera att filnamnet måste användas:

```xml
<div data-sly-use.nav="navigation.js">${nav.foo}</div>
```

Initiera en Java-klass, där källfilen finns i samma katalog som mallen. Observera att klassnamnet måste användas, inte filnamnet:

```xml
<div data-sly-use.nav="Navigation">${nav.foo}</div>
```

Initiera en Java-klass, där den klassen installeras som en del av ett OSGi-paket. Observera att det fullständiga klassnamnet måste användas:

```xml
<div data-sly-use.nav="org.example.Navigation">${nav.foo}</div>
```

Parametrar kan skickas till initieringen med alternativ. I allmänhet bör den här funktionen endast användas av HTML-kod som i sin tur finns i ett `data-sly-template`-block:

```xml
<div data-sly-use.nav="${'navigation.js' @parentPage=currentPage}">${nav.foo}</div>
```

Initiera en annan HTML-mall som sedan kan anropas med `data-sly-call`:

```xml
<div data-sly-use.nav="navTemplate.html" data-sly-call="${nav.foo}"></div>
```

>[!NOTE]
>
>Mer information om Use-API finns i:
>
>* [Java Use-API](use-api-java.md)
>* [JavaScript Use-API](use-api-javascript.md)


#### datadriven användning med resurser {#data-sly-use-with-resources}

Detta gör att resurser kan hämtas direkt i HTML med `data-sly-use` och kräver inte att du skriver kod för att få den.

Till exempel:

```xml
<div data-sly-use.product=“/etc/commerce/product/12345”>
  ${ product.title }
</div>
```

>[!TIP]
>
>Se även avsnittet [Sökväg krävs inte alltid.](#path-not-required)

### unwrap {#unwrap}

`data-sly-unwrap` tar bort värdelementet från den genererade koden samtidigt som innehållet behålls. Detta gör att element som krävs som en del av HTML-presentationslogiken kan uteslutas, men som inte är önskade i det faktiska resultatet.

Denna programsats bör dock användas sparsamt. I allmänhet är det bättre att hålla HTML-koden så nära den avsedda utdatakoden som möjligt. När du lägger till HTML-blockprogramsatser kan du med andra ord försöka så mycket som möjligt för att bara kommentera den befintliga HTML-koden, utan att införa nya element.

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

### uppsättning {#set}

`data-sly-set` definierar en ny identifierare med ett fördefinierat värde.

```xml
<span data-sly-set.profile="${user.profile}">Hello, ${profile.firstName} ${profile.lastName}!</span>
<a class="profile-link" href="${profile.url}">Edit your profile</a>
```

### text {#text}

`data-sly-text` ersätter innehållet i dess host-element med den angivna texten.

Den här

```xml
<p>${properties.jcr:description}</p>
```

likvärdigt med

```xml
<p data-sly-text="${properties.jcr:description}">Lorem ipsum</p>
```

Båda visar värdet `jcr:description` som stycketext. Fördelen med den andra metoden är att den tillåter den diskreta kommenteringen av HTML samtidigt som det statiska platshållarinnehållet behålls från den ursprungliga designern.

### attribute {#attribute}

`data-sly-attribute` lägger till attribut i värdelementet.

Den här

```xml
<div title="${properties.jcr:title}"></div>
```

likvärdigt med

```xml
<div title="Lorem Ipsum" data-sly-attribute.title="${properties.jcr:title}"></div>
```

Båda ställer in attributet `title` på värdet `jcr:title`. Fördelen med den andra metoden är att den tillåter den diskreta kommenteringen av HTML samtidigt som det statiska platshållarinnehållet behålls från den ursprungliga designern.

Attribut tolkas från vänster till höger, med instansen längst till höger av ett attribut (antingen literal eller definierad via `data-sly-attribute`) som har företräde framför instanser av samma attribut (definierat antingen bokstavligt eller via `data-sly-attribute`) som definierats till vänster.

Observera att ett attribut (antingen `literal` eller inställt via `data-sly-attribute`) vars värde utvärderas till den tomma strängen kommer att tas bort i den slutliga koden. Ett undantag till den här regeln är att ett literalt attribut som är inställt på en literal tom sträng bevaras. Till exempel,

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

### element {#element}

`data-sly-element` ersätter värdelementets elementnamn.

Till exempel,

```xml
<h1 data-sly-element="${titleLevel}">text</h1>
```

Ersätter `h1` med värdet `titleLevel`.

Av säkerhetsskäl accepterar `data-sly-element` bara följande elementnamn:

```xml
a abbr address article aside b bdi bdo blockquote br caption cite code col colgroup
data dd del dfn div dl dt em figcaption figure footer h1 h2 h3 h4 h5 h6 header i ins
kbd li main mark nav ol p pre q rp rt ruby s samp section small span strong sub
sup table tbody td tfoot th thead time tr u var wbr
```

XSS-skyddet måste vara inaktiverat ( `@context='unsafe'`) för att andra element ska kunna anges.

### test {#test}

`data-sly-test` tar bort värdelementet och dess innehåll. Värdet `false` tar bort elementet; värdet `true` behåller elementet.

Elementet `p` och dess innehåll återges till exempel bara om `isShown` är `true`:

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

### upprepa {#repeat}

Med `data-sly-repeat` kan du upprepa ett element flera gånger baserat på den angivna listan.

```xml
<div data-sly-repeat="${currentPage.listChildren}">${item.name}</div>
```

Detta fungerar på samma sätt som `data-sly-list`, förutom att du inte behöver något behållarelement.

I följande exempel visas att du även kan referera till *objektet* för attribut:

```xml
<div data-sly-repeat="${currentPage.listChildren}" data-sly-attribute.class="${item.name}">${item.name}</div>
```

### list {#list}

`data-sly-list` upprepar innehållet i host-elementet för varje uppräkningsbar egenskap i det angivna objektet.

Här är en enkel slinga:

```xml
<dl data-sly-list="${currentPage.listChildren}">
    <dt>index: ${itemList.index}</dt>
    <dd>value: ${item.title}</dd>
</dl>
```

Följande standardvariabler är tillgängliga inom listans omfång:

* `item`: Det aktuella objektet i iterationen.
* `itemList`: Objekt som innehåller följande egenskaper:
* `index`: nollbaserad räknare (  `0..length-1`).
* `count`: en-baserad räknare (  `1..length`).
* `first`:  `true` om det aktuella objektet är det första objektet.
* `middle`:  `true` om det aktuella objektet varken är det första eller det sista objektet.
* `last`:  `true` om det aktuella objektet är det sista objektet.
* `odd`:  `true` om  `index` är udda.
* `even`:  `true` om  `index` är jämnt.

Om du definierar en identifierare för `data-sly-list`-satsen kan du byta namn på variablerna `itemList` och `item`. `item` kommer att bli  `<variable>` och  `itemList` bli  `<variable>List`.

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

### resurs {#resource}

`data-sly-resource` innehåller resultatet av återgivningen av den angivna resursen via segmentupplösningen och återgivningsprocessen.

En enkel resurs är:

```xml
<article data-sly-resource="path/to/resource"></article>
```

#### Sökväg krävs inte alltid {#path-not-required}

Observera att du inte behöver använda en sökväg med `data-sly-resource` om du redan har resursen. Om du redan har resursen kan du använda den direkt.

Följande är till exempel korrekt.

```xml
<sly data-sly-resource="${resource.path @ decorationTagName='div'}"></sly>
```

Följande är dock helt godtagbart.

```xml
<sly data-sly-resource="${resource @ decorationTagName='div'}"></sly>
```

Vi rekommenderar att du använder resursen direkt när det är möjligt på grund av följande orsaker.

* Om du redan har resursen är det onödigt att lösa om med sökvägen.
* Om du använder sökvägen när du redan har resursen kan det leda till oväntade resultat eftersom Sling-resurser kan kapslas in eller vara syntetiska och inte anges på den angivna sökvägen.

#### Alternativ {#resource-options}

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

Som standard är de AEM dekorationstaggarna inaktiverade, med alternativet decorationTagName kan du hämta dem tillbaka och med cssClassName kan du lägga till klasser i det elementet.

```xml
<article data-sly-resource="${'path/to/resource' @ decorationTagName='span',
cssClassName='className'}"></article>
```

>[!NOTE]
>
>AEM erbjuder tydlig och enkel logik som styr de dekorationstaggar som omsluter de inkluderade elementen. Mer information finns i [Dekoration Tag](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/implementing/developing/full-stack/components-templates/decoration-tag.html) i dokumentationen för komponenter under utveckling.

### include {#include}

`data-sly-include` ersätter innehållet i host-elementet med koden som genereras av den angivna HTML-mallfilen (HTL, JSP, ESP osv.) när den bearbetas av motsvarande mallmotor. Återgivningssammanhanget för den inkluderade filen kommer inte att omfatta den aktuella HTML-kontexten (den för den inkluderade filen). För att HTML-filer ska kunna inkluderas måste därför aktuell `data-sly-use` upprepas i den inkluderade filen (i så fall är det oftast bättre att använda `data-sly-template` och `data-sly-call`)

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

### Request-attributes {#request-attributes}

I `data-sly-include` och `data-sly-resource` kan du skicka `requestAttributes` för att använda dem i det mottagande HTL-skriptet.

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

Via en delmodell kan du till exempel använda värdet för den angivna `requestAttributes`.

I det här exemplet injiceras layout via kartan från klassen Use:

```xml
@Model(adaptables=SlingHttpServletRequest.class)
public class ProductSettings {
  @Inject @Optional @Default(values="empty")
  public String layout;

}
```

### mall och samtal {#template-call}

Mallblock kan användas som funktionsanrop: i sin deklaration kan de hämta parametrar som sedan kan skickas när de anropas. De tillåter även rekursion.

`data-sly-template` definierar en mall. Värdelementet och dess innehåll genereras inte av HTML

`data-sly-call` anropar en mall som har definierats med en datamall. Innehållet i den anropade mallen (eventuellt parametriserat) ersätter innehållet i värdelementet i anropet.

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

Mallar som finns i en annan fil kan initieras med `data-sly-use`. Observera att i det här fallet kan `data-sly-use` och `data-sly-call` också placeras i samma element:

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

## sly Element {#sly-element}

HTML-taggen `<sly>` kan användas för att ta bort det aktuella elementet, så att bara dess underordnade element kan visas. Dess funktionalitet liknar blockelementet `data-sly-unwrap`:

```xml
<!--/* This will display only the output of the 'header' resource, without the wrapping <sly> tag */-->
<sly data-sly-resource="./header"></sly>
```

Även om den inte är en giltig HTML 5-tagg kan `<sly>`-taggen visas i det slutliga resultatet med `data-sly-unwrap`:

```xml
<sly data-sly-unwrap="${false}"></sly> <!--/* outputs: <sly></sly> */-->
```

Målet med `<sly>`-elementet är att göra det tydligare att elementet inte är utdata. Om du vill kan du fortfarande använda `data-sly-unwrap`.

Precis som med `data-sly-unwrap`, försök att minimera användningen av detta.

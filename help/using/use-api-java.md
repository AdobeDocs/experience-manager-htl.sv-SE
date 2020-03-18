---
title: HTL Java Use-API
seo-title: HTL Java Use-API
description: 'Java Use-API:t för HTML-mallspråket (HTL) aktiverar en HTL-fil för att komma åt hjälpmetoder i en anpassad Java-klass. '
seo-description: 'Java Use-API:t för HTML-mallspråket (HTL) aktiverar en HTL-fil för att komma åt hjälpmetoder i en anpassad Java-klass. '
uuid: b340f8f7-a193-45c8-aa39-5c6e2c0194ea
contentOwner: User
products: SG_EXPERIENCEMANAGER/HTL
topic-tags: html-template-language
content-type: reference
discoiquuid: 126ebc9d-5f7b-47a4-aea2-c8840d34864c
mwpw-migration-script-version: 2017-10-12T21 46 58.665-0400
translation-type: tm+mt
source-git-commit: 6de5ed20e4463c0c2e804e24cb853336229a7c1f

---


# HTL Java Use-API{#htl-java-use-api}

HTML-mallspråkets Java Use-API (Java Use-API) ger en HTML-fil åtkomst till hjälpmetoder i en anpassad Java-klass. På så sätt kan all komplex affärslogik kapslas in i Java-koden, medan HTML-koden endast hanterar direkt markeringsproduktion.

## Ett enkelt exempel {#a-simple-example}

Vi börjar med en HTML-komponent som *inte* har någon användningsklass. Den består av en enda fil, `/apps/my-example/components/info.html`

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html}

```xml
<div>
    <h1>${properties.title}</h1>
    <p>${properties.description}</p>
</div>
```

Vi lägger även till innehåll för den här komponenten som ska återges på **`/content/my-example/`**:

### `http://localhost:4502/content/my-example.json` {#http-localhost-content-my-example-json}

```java
{
    "sling:resourceType": "my-example/component/info",
    "title": "My Example",
    "description": "This Is Some Example Content."
}
```

När det här innehållet öppnas körs HTML-filen. I HTML-koden använder vi kontextobjektet **`properties`** för att komma åt den aktuella resursens `title` och `description` visa dem. HTML-utdata blir:

### `view-source:http://localhost:4502/content/my-example.html` {#view-source-http-localhost-content-my-example-html}

```xml
<div>
    <h1>My Example</h1>
    <p>This Is Some Example Content.</p>
</div>
```

### Lägga till en användningsklass {#adding-a-use-class}

Komponenten **info** behöver inte någon use-klass för att utföra sin (mycket enkla) funktion. Det finns fall där du behöver göra saker som inte kan göras i HTML och där du behöver en användningsklass. Tänk på följande:

>[!NOTE]
>
>*En use-klass ska bara användas när något inte kan göras i enbart HTML.*

Anta till exempel att du vill att `info` komponenten ska visa resursens `title` - och **`description`** -egenskaper, men alla med gemener. Eftersom HTML inte har någon metod för att sänka rabattsträngar behöver du en use-class. Vi kan göra detta genom att lägga till en Java-användarklass och ändra **`info.html`** enligt följande:

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-1}

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### `/apps/my-example/component/info/Info.java` {#apps-my-example-component-info-info-java}

```java
package apps.my_example.components.info;
 
import com.adobe.cq.sightly.WCMUsePojo;
 
public class Info extends WCMUsePojo {
    private String lowerCaseTitle;
    private String lowerCaseDescription;
 
    @Override
    public void activate() throws Exception {
        lowerCaseTitle = getProperties().get("title", "").toLowerCase();
        lowerCaseDescription = getProperties().get("description", "").toLowerCase();
    }
 
    public String getLowerCaseTitle() {
        return lowerCaseTitle;
    }
 
    public String getLowerCaseDescription() {
        return lowerCaseDescription;
    }
}
```

I följande avsnitt går vi igenom de olika delarna av koden.

### Java-klass, lokal kontra bunt {#local-vs-bundle-java-class}

Java-klassen kan installeras på två sätt: **lokal** eller **bunt**. *I det här exemplet används en lokal installation.*

I en lokal installation placeras Java-källfilen bredvid HTML-filen i samma databasmapp. Källan kompileras automatiskt vid behov. Inget separat kompilerings- eller paketeringssteg krävs.

I en paketinstallation måste Java-klassen kompileras och distribueras i ett OSGi-paket med hjälp av den vanliga AEM-paketdistributionsmekanismen (se [Paketerad Java-klass](#bundled-java-class)).

>[!NOTE]
>
>En **lokal Java use-class** rekommenderas när use-klassen är specifik för komponenten i fråga.
>
>En **bunt Java use-class** rekommenderas när Java-koden implementerar en tjänst som är tillgänglig från flera HTML-komponenter.

### Java-paketet är en databassökväg {#java-package-is-repository-path}

När en lokal installation används måste paketnamnet för use-klassen matcha databasmappens namn, där eventuella bindestreck i sökvägen ersätts med understreck i paketnamnet.

I det här fallet **`Info.java`** finns på **`/apps/my-example/components/info`** så att paketet är **`apps.my_example.components.info`** :

### `/apps/my-example/component/info/Info.java` {#apps-my-example-component-info-info-java-1}

```java
package apps.my_example.components.info;
 
import com.adobe.cq.sightly.WCMUsePojo;
 
public class Info extends WCMUsePojo {

   ...

}
```

>[!NOTE]
>
>Att använda bindestreck i namn på databasobjekt rekommenderas i AEM-utveckling. Däremot är bindestreck ogiltiga i Java-paketnamn. Därför måste **alla bindestreck i databassökvägen konverteras till understreck i paketnamnet**.

### Utöka `WCMUsePojo`{#extending-wcmusepojo}

Det finns många sätt att införliva en Java-klass med HTML (se Alternativ till `WCMUsePojo`), men det enklaste är att utöka `WCMUsePojo` klassen:

#### `/apps/my-example/component/info/Info.java` {#apps-my-example-component-info-info-java-2}

```java
package apps.my_example.components.info;
 
import com.adobe.cq.sightly.WCMUsePojo;
 
public class Info extends WCMUsePojo {
    
    ...
}
```

### Initierar klassen {#initializing-the-class}

När use-klassen utökas från **`WCMUsePojo`**, initieras metoden genom att åsidosätta **`activate`** :

### /apps/my-example/component/info/Info.java {#apps-my-example-component-info-info-java-3}

```java
...

public class Info extends WCMUsePojo {
    private String lowerCaseTitle;
    private String lowerCaseDescription;
 
    @Override
    public void activate() throws Exception {
        lowerCaseTitle = getProperties().get("title", "").toLowerCase();
        lowerCaseDescription = getProperties().get("description", "").toLowerCase();
    }

...

}
```

### Kontext {#context}

Vanligtvis används metoden [activate](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html) för att beräkna och lagra (i medlemsvariabler) de värden som behövs i din HTML-kod utifrån det aktuella sammanhanget (till exempel aktuell begäran och resurs).

Klassen ger åtkomst till samma uppsättning kontextobjekt som finns i en HTML-fil (se `WCMUsePojo` Globala objekt [](global-objects.md)).

I en klass som utökas **`WCMUsePojo`** kan kontextobjekt nås *via namn* med

[`<T> T get(String name, Class<T> type)`](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html)

Du kan också komma åt ofta använda kontextobjekt direkt med lämplig **bekvämlighetsmetod**:

|  |  |
|---|---|
| [PageManager](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/PageManager.html) | [getPageManager()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getPageManager()) |
| [Sida](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/Page.html) | [getCurrentPage()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getCurrentPage()) |
| [Sida](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/Page.html) | [getResourcePage()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResourcePage()) |
| [ValueMap](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ValueMap.html) | [getPageProperties()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getPageProperties()) |
| [ValueMap](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ValueMap.html) | [getProperties()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getProperties()) |
| [Designer](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/designer/Designer.html) | [getDesigner()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getDesigner()) |
| [Design](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/designer/Design.html) | [getCurrentDesign()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getCurrentDesign()) |
| [Format](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/designer/Style.html) | [getCurrentStyle()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getCurrentStyle()) |
| [Komponent](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/components/Component.html) | [getComponent()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getComponent()) |
| [ValueMap](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ValueMap.html) | [getInheritedProperties()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse#getInheritedProperties.html()) |
| [Resurs](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/Resource.html) | [getResource()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResource()) |
| [ResursResolver](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ResourceResolver.html) | [getResourceResolver()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResourceResolver()) |
| [SlingHttpServletRequest](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletRequest.html) | [getRequest()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getRequest()) |
| [SlingHttpServletResponse](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletResponse.html) | [getResponse()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResponse()) |
| [SlingScriptHelper](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/scripting/SlingScriptHelper.html) | [getSlingScriptHelper()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getSlingScriptHelper()) |

### Getter-metoder {#getter-methods}

När use-class har initierats körs HTML-filen. Under den här scenen kommer HTML vanligtvis att dra in status för olika medlemsvariabler i use-klassen och återge dem för presentation.

Om du vill ge åtkomst till dessa värden inifrån HTML-filen måste du definiera egna get-metoder i use-klassen **enligt följande namnkonvention**:

* En formulärmetod **`getXyz`** visar en objektegenskap med namnet ***xyz*** i HTML-filen.

I följande exempel resulterar metoderna **`getTitle`** och **`getDescription`** resultatet i objektegenskaperna **`title`** och **`description`** blir tillgängliga i HTML-filens kontext:

### `/apps/my-example/component/info/Info.java` {#apps-my-example-component-info-info-java-4}

```java
...

public class Info extends WCMUsePojo {

    ...
 
    public String getLowerCaseTitle() {
        return lowerCaseTitle;
    }
 
    public String getLowerCaseDescription() {
        return lowerCaseDescription;
    }
}
```

### dataanvändarvänligt attribut {#data-sly-use-attribute}

Attributet **`data-sly-use`** används för att initiera use-klassen i din HTML-kod. I vårt exempel deklarerar attributet `data-sly-use` att vi vill använda klassen **`Info`**. Vi kan bara använda det lokala namnet på klassen eftersom vi använder en lokal installation (när Java-källfilen har placerats i samma mapp som HTML-filen). Om vi använde en paketinstallation måste vi ange det fullständiga klassnamnet (se [Use-class Bundle Install](#LocalvsBundleJavaClass)).

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-2}

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### Lokal identifierare {#local-identifier}

Identifieraren &#39;**`info`**&#39; (efter punkten i **`data-sly-use.info`**) används i HTML-filen för att identifiera klassen. Identifierarens omfång är globalt i filen, efter att den har deklarerats. Den är inte begränsad till elementet som innehåller `data-sly-use` programsatsen .

### `/apps/my-example/component/info/info.html`{#apps-my-example-component-info-info-html-3}

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### Hämta egenskaper {#getting-properties}

Identifieraren `info` används sedan för att komma åt objektegenskaperna **`title`** och **`description`** som exponerades via get-metoderna `Info.getTitle` och **`Info.getDescription`**.

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-4}

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### Output {#output}

När vi öppnar **`/content/my-example.html`** den returneras följande HTML:

### `view-source:http://localhost:4502/content/my-example.html` {#view-source-http-localhost-content-my-example-html-1}

```xml
<div>
    <h1>my example</h1>
    <p>this is some example content.</p>
</div>
```

## Förutom grunderna {#beyond-the-basics}

I det här avsnittet kommer vi att införa ytterligare funktioner som går utöver det enkla exemplet ovan:

* Skicka parametrar till en use-klass.
* Paketerad Java-klass.
* Alternativ till `WCMUsePojo`

### Parametrar skickas {#passing-parameters}

Parametrar kan skickas till en use-class vid initiering. Vi kan till exempel göra något så här:

### `/content/my-example/component/info/info.html` {#content-my-example-component-info-info-html}

```xml
<div data-sly-use.info="${'Info' @ text='Some text'}">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
    <p>${info.upperCaseText}</p>
</div>
```

Här skickar vi en parameter med namnet **`text`**. Klassen use får då den sträng som vi hämtar och visar resultatet med `info.upperCaseText`. Här är den justerade use-klassen:

### `/apps/my-example/component/info/Info.java` {#apps-my-example-component-info-info-java-5}

```java
package apps.my_example.components.info;
 
import com.adobe.cq.sightly.WCMUsePojo;
 
public class Info extends WCMUsePojo {
    
    ...

    private String reverseText;
    
    @Override
    public void activate() throws Exception {

        ...
        
        String text = get("text", String.class);
        reverseText = new StringBuffer(text).reverse().toString();

    }
 
    public String getReverseText() {
        return reverseText;
    }

    ...
}
```

Parametern nås via `WCMUsePojo` metoden

[ `<T> T get(String paramName, Class<T> type)`](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html)

I det här fallet, programsatsen

`get("text", String.class)`

Strängen kastas sedan om och visas via metoden

**`getReverseText()`**

### Skicka bara parametrar från data-sly-template {#only-pass-parameters-from-data-sly-template}

Ovanstående exempel är tekniskt korrekt, men det är faktiskt inte särskilt vettigt att skicka ett värde från HTML för att initiera en use-klass, när värdet i fråga är tillgängligt i körningskontexten för HTML-koden (eller, trivialt, är värdet statiskt, som ovan).

Orsaken är att use-klassen alltid har tillgång till samma körningskontext som HTML-koden. Detta utgör en importpunkt för bästa praxis:

>[!NOTE]
>
>Du bör bara skicka en parameter till en use-klass när use-klassen används i en **data-sly-template** -fil som själv anropas från en annan HTL-fil med parametrar som måste skickas vidare.

Låt oss till exempel skapa en separat `data-sly-template` fil tillsammans med vårt befintliga exempel. Vi ska ringa den nya filen `extra.html`. Den innehåller ett **`data-sly-template`** block med namnet **`extra`**:

### `/apps/my-example/component/info/extra.html` {#apps-my-example-component-info-extra-html}

```xml
<template data-sly-template.extra="${@ text}"
          data-sly-use.extraHelper="${'ExtraHelper' @ text=text}">
  <p>${extraHelper.reversedText}</p>
</template>
```

Mallen **`extra`** tar en parameter **`text`**. Sedan initierar den Java use-class `ExtraHelper` med det lokala namnet **`extraHelper`** och skickar värdet för template-parametern **`text`** som use-class-parameter **`text`**.

Mallens brödtext hämtar egenskapen `extraHelper.reversedText` (som under huven faktiskt anropar `ExtraHelper.getReversedText()`) och visar det värdet.

Vi anpassar också vår befintliga **`info.html`** metod så att den nya mallen kan användas:

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-5}

```xml
<div data-sly-use.info="Info" 
     data-sly-use.extra="extra.html">
    
  <h1>${info.lowerCaseTitle}</h1>
  <p>${info.lowerCaseDescription}</p>
    
  <div data-sly-call="${extra.extra @ text=properties.description}"></div>

</div>
```

Filen innehåller `info.html` nu två **`data-sly-use`** programsatser, den ursprungliga som importerar **`Info`** Java-klassen och en ny som importerar mallfilen under det lokala namnet `extra`.

Observera att vi kunde ha placerat mallblocket inuti **`info.html`** filen för att undvika det andra, **`data-sly-use`** men en separat mallfil är vanligare och mer återanvändbar.

Klassen används som tidigare och anropar dess get-metoder **`Info`** och **`getLowerCaseTitle()`** genom deras motsvarande HTL-egenskaper `getLowerCaseDescription()` och `info.lowerCaseTitle` **`info.lowerCaseDescription`**.

Sedan utför vi en åtgärd **`data-sly-call`** i mallen **`extra`** och skickar värdet `properties.description` som parameter **`text`**.

Klassen Java use `Info.java` ändras för att hantera den nya textparametern:

### `/apps/my-example/component/info/ExtraHelper.java` {#apps-my-example-component-info-extrahelper-java}

```java
package apps.my_example.components.info;
 
import com.adobe.cq.sightly.WCMUsePojo;
 
public class ExtraHelper extends WCMUsePojo {
    private String reversedText;
    ...
    
    @Override
    public void activate() throws Exception {
        String text = get("text", String.class);      
        reversedText = new StringBuilder(text).reverse().toString();

        ...
    }
 
    public String getReversedText() {
        return reversedText;
    }
}
```

Parametern **`text`** hämtas med **`get("text", String.class)`**, värdet återförs och görs tillgänglig som HTL-objekt `reversedText` via get-metoden `getReversedText()`.

### Paketerad Java-klass {#bundled-java-class}

Med en paketklass måste klassen kompileras, paketeras och distribueras i AEM med hjälp av den vanliga OSGi-paketdistributionsmekanismen. Till skillnad från en lokal installation bör **paketdeklarationen** för use-klassen namnges normalt:

### `/apps/my-example/component/info/Info.java` {#apps-my-example-component-info-info-java-6}

```java
package org.example.app.components;
 
import com.adobe.cq.sightly.WCMUsePojo;
 
public class Info extends WCMUsePojo {
    ...
}
```

och programsatsen måste referera till det `data-sly-use` fullständiga klassnamnet **, i motsats till bara det lokala klassnamnet:

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-6}

```xml
<div data-sly-use.info="org.example.app.components.info.Info">
  <h1>${info.title}</h1>
  <p>${info.description}</p>
</div>
```

### Alternativ till `WCMUsePojo`{#alternatives-to-wcmusepojo}

Det vanligaste sättet att skapa en Java-klass är att utöka `WCMUsePojo`. Det finns dock ett antal andra alternativ. För att förstå de här varianterna hjälper det att förstå hur HTML- `data-sly-use` satsen fungerar under huven.

Anta att du har följande `data-sly-use` programsats:

**`<div data-sly-use.`** `localName`**`="`** `UseClass`**`">`**

Systemet bearbetar programsatsen på följande sätt:

(1)

* Om det finns en lokal fil med *UseClass.java* i samma katalog som HTML-filen försöker du kompilera och läsa in den klassen. Om du lyckas går du till (2).

* Annars tolkar du *UseClass* som ett fullständigt kvalificerat klassnamn och försöker läsa in det från OSGi-miljön. Om du lyckas går du till (2).
* Annars tolkar du *UseClass* som en sökväg till en HTML- eller JavaScript-fil och läser in den filen. Om det lyckas går du till (4).

(2)

* Försök att anpassa den aktuella versionen **`Resource`** till *`UseClass.`* Om det lyckas går du till (3).

* Annars kan du försöka anpassa den aktuella **`Request`** till *`UseClass`*. Om det lyckas går du till (3).

* Annars försöker du instansiera *`UseClass`* med en nollargumentskonstruktor. Om det lyckas går du till (3).

(3)

* Bind det nyanpassade eller skapade objektet till namnet i HTML-koden *`localName`*.
* If *`UseClass`* implements [ `io.sightly.java.api.Use`](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/io/sightly/java/api/Use.html) then call the `init` method, passing the current execution context (in the form of a `javax.scripting.Bindings` object).

(4)

* Om *`UseClass`* är en sökväg till en HTML-fil som innehåller en `data-sly-template`mall förbereder du mallen.

* Om *`UseClass`* är en sökväg till en JavaScript-användningsklass förbereder du klassen use (se [JavaScript Use-API](use-api-javascript.md)).

Några viktiga punkter om ovanstående beskrivning:

* Alla klasser som kan anpassas från `Resource`, anpassas från `Request`eller som har en nollargumentskonstruktor kan vara användningsklass. Klassen behöver inte utöka `WCMUsePojo` eller implementera `Use`.

* Men om use-klassen *implementerar* så anropas dess `Use`**`init`** metod automatiskt med den aktuella kontexten, så att du kan placera initieringskoden där som är beroende av den kontexten.

* En användningsklass som utökar `WCMUsePojo` är bara ett specialfall när det gäller implementering **`Use`**. It provides the convenience context methods and its **`activate`** method is automatically called from `Use.init`.

### Directly Implement Interface Use {#directly-implement-interface-use}

Det vanligaste sättet att skapa en use-klass är att utöka `WCMUsePojo`men det går också att implementera själva `[io.sightly.java.api.Use](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/io/sightly/java/api/Use.html)`gränssnittet direkt.

Gränssnittet `Use` definierar bara en metod:

`[public void init(javax.script.Bindings bindings)](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/io/sightly/java/api/Use#init(javax.script.Bindings))`

The **`init`** method will be called on initialization of the class with a **`Bindings`** object that holds all the context objects and any parameters passed into the use-class.

All additional functionality (such as the equivalent of `WCMUsePojo.getProperties()`) must be implmented explicitly using the ` [javax.script.Bindings](http://docs.oracle.com/javase/7/docs/api/javax/script/Bindings.html)` object. Exempel:

### `Info.java` {#info-java}

```java
import io.sightly.java.api.Use;

public class MyComponent implements Use {
   ...
    @Override
    public void init(Bindings bindings) {

        // All standard objects/binding are available
        Resource resource = (Resource)bindings.get("resource");
        ValueMap properties = (ValueMap)bindings.get("properties"); 
        ...

        // Parameters passed to the use-class are also available
        String param1 = (String) bindings.get("param1");
    }
    ...
}
```

Det viktigaste när du implementerar gränssnittet själv i stället för att utöka `Use` `WCMUsePojo` är när du vill använda en underklass till en redan befintlig klass som use-class.

### Adaptable from Resource {#adaptable-from-resource}

Another option is to use a helper class that is adaptable from **`org.apache.sling.api.resource.Resource`**.

Let&#39;s say you need to write an HTL script that displays the mimetype of a DAM asset. In this case you know that when your HTL script is called, it will be within the context of a **`Resource`** that wraps a JCR **`Node`** with nodetype **`dam:Asset`**.

You know that a **`dam:Asset`** node has a structure like this:

### Repository Structure {#repository-structure}

```java
{
  "content": {
    "dam": {
      "geometrixx": {
        "portraits": {
          "jane_doe.jpg": {
            ...
          "jcr:content": {
            ...
            "metadata": {
              ...
            },
            "renditions": {
              ...
              "original": {
                ...
                "jcr:content": {
                  "jcr:primaryType": "nt:resource",
                  "jcr:lastModifiedBy": "admin",
                  "jcr:mimeType": "image/jpeg",
                  "jcr:lastModified": "Fri Jun 13 2014 15:27:39 GMT+0200",
                  "jcr:data": ...,
                  "jcr:uuid": "22e3c598-4fa8-4c5d-8b47-8aecfb5de399"
                }
              },
              "cq5dam.thumbnail.319.319.png": {
                  ...
              },
              "cq5dam.thumbnail.48.48.png": {
                  ...
              },
              "cq5dam.thumbnail.140.100.png": {
                  ...
              }
            }
          }  
        }
      }
    }
  }
}
```

Here we show the asset (a JPEG image) that comes with a default install of AEM as part of the example project geometrixx. The asset is called **`jane_doe.jpg`** and its mimetype is **`image/jpeg`**.

To access the asset from within HTL, you can declare ` [com.day.cq.dam.api.Asset](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/granite/asset/api/Asset.html)` as the class in the **`data-sly-use`** statement: and then use a get method of **`Asset`** to retrieve the desired information. Exempel:

### `mimetype.html` {#mimetype-html}

```xml
<div data-sly-use.asset="com.day.cq.dam.api.Asset">
  <p>${asset.mimeType}</p>
</div>
```

The `data-sly-use` statement directs HTL to adapt the current **`Resource`** to an **`Asset`** and give it the local name **`asset`**. It then calls the **`getMimeType`** method of `Asset` using the HTL getter shorthand: `asset.mimeType`.

### Adaptable from Request {#adaptable-from-request}

It is also possible to emply as a use-class any class that is adaptable from **` [org.apache.sling.api.SlingHttpServletRequest](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletRequest.html)`**

As with the above case of a use-class adaptable from `Resource`, a use-class adaptable from [`SlingHttpServletRequest`](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletRequest.html) can be specified in the `data-sly-use` statement. Upon execution the current request will be adapted to the class given and the resulting object will be made available within HTL.

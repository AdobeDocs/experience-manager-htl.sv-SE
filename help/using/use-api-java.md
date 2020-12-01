---
title: HTL Java Use-API
description: 'Java Use-API:t för HTML-mallspråket (HTL) aktiverar en HTL-fil för att komma åt hjälpmetoder i en anpassad Java-klass. '
translation-type: tm+mt
source-git-commit: ee712ef61018b5e05ea052484e2a9a6b12e6c5c8
workflow-type: tm+mt
source-wordcount: '2558'
ht-degree: 1%

---


# HTL Java Use-API {#htl-java-use-api}

HTML-mallspråkets Java Use-API (Java Use-API) ger en HTML-fil åtkomst till hjälpmetoder i en anpassad Java-klass via `data-sly-use`. På så sätt kan all komplex affärslogik kapslas in i Java-koden, medan HTML-koden endast hanterar direkt markeringsproduktion.

Ett Java Use-API-objekt kan vara en enkel POJO som initieras av en viss implementering via POJO:s standardkonstruktor.

Use-API-POJO:erna kan också visa en offentlig metod, som kallas init, med följande signatur:

```java
    /**
     * Initializes the Use bean.
     *
     * @param bindings All bindings available to the HTL scripts.
     **/
    public void init(javax.script.Bindings bindings);
```

Mappningen `bindings` kan innehålla objekt som ger kontext till det HTML-skript som körs för tillfället och som Use-API-objektet kan använda för bearbetningen.

## Ett enkelt exempel {#a-simple-example}

Vi börjar med en HTML-komponent som inte har någon användningsklass. Den består av en enda fil, `/apps/my-example/components/info.html`

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html}

```xml
<div>
    <h1>${properties.title}</h1>
    <p>${properties.description}</p>
</div>
```

Vi lägger också till en del innehåll för den här komponenten som ska återges på `/content/my-example/`:

### `http://<host>:<port>/content/my-example.json` {#http-localhost-content-my-example-json}

```java
{
    "sling:resourceType": "my-example/component/info",
    "title": "My Example",
    "description": "This Is Some Example Content."
}
```

När det här innehållet öppnas körs HTML-filen. I HTML-koden använder vi kontextobjektet `properties` för att komma åt den aktuella resursens `title` och `description` och visa dem. HTML-utdata blir:

### `view-source:http://<host>:<port>/content/my-example.html` {#view-source-http-localhost-content-my-example-html}

```xml
<div>
    <h1>My Example</h1>
    <p>This Is Some Example Content.</p>
</div>
```

### Lägga till en Use-Class {#adding-a-use-class}

Komponenten **info** behöver inte någon use-klass för att utföra sin (mycket enkla) funktion. Det finns fall där du behöver göra saker som inte kan göras i HTML och där du behöver en användningsklass. Tänk på följande:

>[!NOTE]
>
>En use-klass ska bara användas när något inte kan göras i enbart HTML.

Anta till exempel att du vill att `info`-komponenten ska visa egenskaperna `title` och `description` för resursen, men alla med gemener. Eftersom HTML inte har någon metod för att sänka rabattsträngar behöver du en use-class. Vi kan göra detta genom att lägga till en Java-användarklass och ändra `info.html` enligt följande:

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

### Java-klass, lokal kontra paket {#local-vs-bundle-java-class}

Java-klassen kan installeras på två sätt: **local** eller **bundle**. I det här exemplet används en lokal installation.

I en lokal installation placeras Java-källfilen bredvid HTML-filen i samma databasmapp. Källan kompileras automatiskt vid behov. Inget separat kompilerings- eller paketeringssteg krävs.

I en paketinstallation måste Java-klassen kompileras och distribueras i ett OSGi-paket med hjälp av AEM vanliga distributionsmekanismen (se [Paketerad Java-klass](#bundled-java-class)).

>[!NOTE]
>
>En **lokal Java use-class** rekommenderas när use-klassen är specifik för den aktuella komponenten.
>
>En **Java use-class**-paketklass rekommenderas när Java-koden implementerar en tjänst som är tillgänglig från flera HTML-komponenter.

### Java-paketet är databassökvägen {#java-package-is-repository-path}

När en lokal installation används måste paketnamnet för use-klassen matcha databasmappens namn, där eventuella bindestreck i sökvägen ersätts med understreck i paketnamnet.

I det här fallet finns `Info.java` på `/apps/my-example/components/info` så paketet är `apps.my_example.components.info`:

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
>Du bör använda bindestreck i namn på databasobjekt när du AEM. Däremot är bindestreck ogiltiga i Java-paketnamn. Därför måste **alla bindestreck i databassökvägen konverteras till understreck i paketnamnet**.

### Utöka `WCMUsePojo` {#extending-wcmusepojo}

Det finns många sätt att införliva en Java-klass med HTML (se Alternativ till `WCMUsePojo`), men det enklaste är att utöka klassen `WCMUsePojo`:

#### `/apps/my-example/component/info/Info.java` {#apps-my-example-component-info-info-java-2}

```java
package apps.my_example.components.info;

import com.adobe.cq.sightly.WCMUsePojo;

public class Info extends WCMUsePojo

    ...
}
```

### Initierar klassen {#initializing-the-class}

När klassen use utökas från `WCMUsePojo`, initieras den genom att åsidosätta metoden `activate`:

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

Metoden [activate](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUsePojo.html) används vanligtvis för att beräkna och lagra (i medlemsvariabler) de värden som behövs i din HTML-kod utifrån den aktuella kontexten (till exempel aktuell begäran och resurs).

Klassen `WCMUsePojo` ger åtkomst till samma uppsättning kontextobjekt som finns i en HTML-fil (se [Globala objekt](global-objects.md)).

I en klass som utökar `WCMUsePojo` kan kontextobjekt nås via namn med

[`<T> T get(String name, Class<T> type)`](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUsePojo.html)

Du kan även komma åt ofta använda kontextobjekt direkt med lämplig **bekvämlighetsmetod**:

|  |  |
|---|---|
| [PageManager](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/PageManager.html) | [getPageManager()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getPageManager()) |
| [Sida](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/Page.html) | [getCurrentPage()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getCurrentPage()) |
| [Sida](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/Page.html) | [getResourcePage()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResourcePage()) |
| [ValueMap](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ValueMap.html) | [getPageProperties()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getPageProperties()) |
| [ValueMap](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ValueMap.html) | [getProperties()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getProperties()) |
| [Designer](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/designer/Designer.html) | [getDesigner()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getDesigner()) |
| [Design](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/designer/Design.html) | [getCurrentDesign()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getCurrentDesign()) |
| [Format](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/designer/Style.html) | [getCurrentStyle()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getCurrentStyle()) |
| [Komponent](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/day/cq/wcm/api/components/Component.html) | [getComponent()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getComponent()) |
| [ValueMap](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ValueMap.html) | [getInheritedProperties()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getInheritedProperties) |
| [Resurs](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/Resource.html) | [getResource()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResource()) |
| [ResursResolver](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/resource/ResourceResolver.html) | [getResourceResolver()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResourceResolver()) |
| [SlingHttpServletRequest](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletRequest.html) | [getRequest()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getRequest()) |
| [SlingHttpServletResponse](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletResponse.html) | [getResponse()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getResponse()) |
| [SlingScriptHelper](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/scripting/SlingScriptHelper.html) | [getSlingScriptHelper()](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html#getSlingScriptHelper()) |

### Getter-metoder {#getter-methods}

När use-class har initierats körs HTML-filen. Under den här scenen kommer HTML vanligtvis att dra in status för olika medlemsvariabler i use-klassen och återge dem för presentation.

Om du vill ge åtkomst till dessa värden inifrån HTML-filen måste du definiera egna get-metoder i use-klassen enligt följande namnkonvention:

* En metod i formatet `getXyz` visar en objektegenskap med namnet `xyz` i HTML-filen.

I följande exempel resulterar metoderna `getTitle` och `getDescription` i att objektegenskaperna `title` och `description` blir tillgängliga i HTML-filens kontext:

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

### dataanvändarattributet {#data-sly-use-attribute}

Attributet `data-sly-use` används för att initiera use-klassen i din HTML-kod. I vårt exempel deklarerar attributet `data-sly-use` att vi vill använda klassen `Info`. Vi kan bara använda det lokala namnet på klassen eftersom vi använder en lokal installation (när Java-källfilen har placerats i samma mapp som HTML-filen). Om vi använde en paketinstallation måste vi ange det fullständiga klassnamnet.

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-2}

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### Lokal identifierare {#local-identifier}

Identifieraren `info` (efter punkten i `data-sly-use.info`) används i HTML-filen för att identifiera klassen. Identifierarens omfång är globalt i filen, efter att den har deklarerats. Den är inte begränsad till elementet som innehåller `data-sly-use`-satsen.

### `/apps/my-example/component/info/info.html`{#apps-my-example-component-info-info-html-3}

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### Hämtar egenskaper {#getting-properties}

Identifieraren `info` används sedan för att komma åt objektegenskaperna `title` och `description` som exponerades med get-metoderna `Info.getTitle` och `Info.getDescription`.

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-4}

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### Utdata {#output}

När vi nu använder `/content/my-example.html` returneras följande HTML:

### `view-source:http://<host>:<port>/content/my-example.html` {#view-source-http-localhost-content-my-example-html-1}

```xml
<div>
    <h1>my example</h1>
    <p>this is some example content.</p>
</div>
```

## Bortom grunderna {#beyond-the-basics}

I det här avsnittet kommer vi att presentera ytterligare funktioner som går utöver det enkla exemplet ovan:

* Skicka parametrar till en use-klass.
* Paketerad Java-klass.
* Alternativ till `WCMUsePojo`

### Skickar parametrar {#passing-parameters}

Parametrar kan skickas till en use-class vid initiering. Vi kan till exempel göra något så här:

### `/content/my-example/component/info/info.html` {#content-my-example-component-info-info-html}

```xml
<div data-sly-use.info="${'Info' @ text='Some text'}">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
    <p>${info.upperCaseText}</p>
</div>
```

Här skickar vi en parameter med namnet `text`. Klassen use får då den sträng som vi hämtar och visar resultatet med `info.upperCaseText`. Här är den justerade use-klassen:

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

Parametern nås via metoden `WCMUsePojo` [`<T> T get(String paramName, Class<T> type)`](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/com/adobe/cq/sightly/WCMUse.html)

I det här fallet ska följande anges:

`get("text", String.class)`

Strängen kastas sedan om och visas med metoden:

`getReverseText()`

### Skicka bara parametrar från data-sly-template {#only-pass-parameters-from-data-sly-template}

Ovanstående exempel är tekniskt korrekt, men det är faktiskt inte särskilt vettigt att skicka ett värde från HTML för att initiera en use-klass, när värdet i fråga är tillgängligt i körningskontexten för HTML-koden (eller, trivialt, är värdet statiskt, som ovan).

Orsaken är att use-klassen alltid har tillgång till samma körningskontext som HTML-koden. Detta utgör en importpunkt för bästa praxis:

>[!NOTE]
>
>Du bör bara skicka en parameter till en use-class när use-class används i en `data-sly-template`-fil som själv anropas från en annan HTML-fil med parametrar som måste skickas vidare.

Låt oss till exempel skapa en separat `data-sly-template`-fil tillsammans med vårt befintliga exempel. Vi anropar den nya filen `extra.html`. Den innehåller ett `data-sly-template`-block med namnet `extra`:

### `/apps/my-example/component/info/extra.html` {#apps-my-example-component-info-extra-html}

```xml
<template data-sly-template.extra="${@ text}"
          data-sly-use.extraHelper="${'ExtraHelper' @ text=text}">
  <p>${extraHelper.reversedText}</p>
</template>
```

Mallen `extra` har en enda parameter, `text`. Sedan initieras Java use-class `ExtraHelper` med det lokala namnet `extraHelper` och värdet för template-parametern `text` skickas som use-class-parametern `text`.

Mallens brödtext hämtar egenskapen `extraHelper.reversedText` (som under huven faktiskt anropar `ExtraHelper.getReversedText()`) och visar det värdet.

Vi anpassar också vår befintliga `info.html` så att den använder den nya mallen:

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-5}

```xml
<div data-sly-use.info="Info"
     data-sly-use.extra="extra.html">

  <h1>${info.lowerCaseTitle}</h1>
  <p>${info.lowerCaseDescription}</p>

  <div data-sly-call="${extra.extra @ text=properties.description}"></div>

</div>
```

Filen `info.html` innehåller nu två `data-sly-use`-satser, den ursprungliga som importerar Java-klassen och en ny som importerar mallfilen under det lokala namnet `extra`.`Info`

Observera att vi kunde ha placerat mallblocket inuti `info.html`-filen för att undvika den andra `data-sly-use`, men en separat mallfil är vanligare och mer återanvändbar.

Klassen `Info` används som tidigare och anropar dess get-metoder `getLowerCaseTitle()` och `getLowerCaseDescription()` genom deras motsvarande HTL-egenskaper `info.lowerCaseTitle` och `info.lowerCaseDescription`.

Sedan gör vi en `data-sly-call` till mallen `extra` och skickar värdet `properties.description` som parametern `text`.

Java use-class `Info.java` ändras för att hantera den nya textparametern:

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

Parametern `text` hämtas med `get("text", String.class)`, värdet återförs och görs tillgänglig som HTL-objekt `reversedText` via get `getReversedText()`.

### Paketerad Java-klass {#bundled-java-class}

Med en paketklass måste klassen kompileras, paketeras och distribueras i AEM med den vanliga OSGi-paketdistributionsmekanismen. Till skillnad från en lokal installation ska paketdeklarationen **för use-klassen** namnges normalt:

### `/apps/my-example/component/info/Info.java` {#apps-my-example-component-info-info-java-6}

```java
package org.example.app.components;

import com.adobe.cq.sightly.WCMUsePojo;

public class Info extends WCMUsePojo {
    ...
}
```

och `data-sly-use`-programsatsen måste referera till det fullständiga, kvalificerade klassnamnet, i motsats till bara det lokala klassnamnet:

### `/apps/my-example/component/info/info.html` {#apps-my-example-component-info-info-html-6}

```xml
<div data-sly-use.info="org.example.app.components.info.Info">
  <h1>${info.title}</h1>
  <p>${info.description}</p>
</div>
```

### Alternativ till `WCMUsePojo` {#alternatives-to-wcmusepojo}

Det vanligaste sättet att skapa en Java use-klass är att utöka `WCMUsePojo`. Det finns dock ett antal andra alternativ. För att förstå de här varianterna hjälper det att förstå hur HTML-satsen `data-sly-use` fungerar under huven.

Anta att du har följande `data-sly-use`-sats:

**`<div data-sly-use.`** `localName`**`="`**`UseClass`**`">`**

Systemet bearbetar programsatsen på följande sätt:

(1)

* Om det finns en lokal fil `UseClass.java` i samma katalog som HTML-filen försöker du kompilera och läsa in den klassen. Om du lyckas går du till (2).
* Annars tolkar du `UseClass` som ett fullständigt kvalificerat klassnamn och försöker läsa in det från OSGi-miljön. Om du lyckas går du till (2).
* Annars tolkar du `UseClass` som en sökväg till en HTML- eller JavaScript-fil och läser in den filen. Om det lyckas går du till (4).

(2)

* Försök att anpassa aktuell `Resource` till `UseClass`. Om det lyckas går du till (3).
* Annars kan du försöka anpassa aktuell `Request` till `UseClass`. Om det lyckas går du till (3).
* Annars försöker du instansiera `UseClass` med en nollargumentskonstruktor. Om det lyckas går du till (3).

(3)

* Bind det nyanpassade eller skapade objektet till namnet `localName` i HTML.
* Om `UseClass` implementerar [`io.sightly.java.api.Use`](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/io/sightly/java/api/Use.html) anropar du metoden `init` och skickar den aktuella körningskontexten (i form av ett `javax.scripting.Bindings`-objekt).

(4)

* Om `UseClass` är en sökväg till en HTML-fil som innehåller `data-sly-template` förbereder du mallen.
* Om `UseClass` är en sökväg till en JavaScript-klass ska du förbereda use-class (se [JavaScript Use-API](use-api-javascript.md)).

Några viktiga punkter om ovanstående beskrivning:

* Alla klasser som kan anpassas från `Resource`, anpassas från `Request` eller som har en nollargumentskonstruktor kan vara en use-klass. Klassen behöver inte utöka `WCMUsePojo` eller implementera `Use`.
* Om klassen *inte* implementerar `Use` anropas metoden `init` automatiskt med den aktuella kontexten, så att du kan placera initieringskoden där som är beroende av den kontexten.
* En use-klass som utökar `WCMUsePojo` är bara ett specialfall när `Use` implementeras. Den innehåller bekväma kontextmetoder och dess `activate`-metod anropas automatiskt från `Use.init`.

### Använd gränssnittet {#directly-implement-interface-use} direkt

Det vanligaste sättet att skapa en use-klass är att utöka `WCMUsePojo`, men det går också att implementera gränssnittet [`io.sightly.java.api.Use`](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/io/sightly/java/api/Use.html) direkt.

Gränssnittet `Use` definierar bara en metod:

[`public void init(javax.script.Bindings bindings)`](https://helpx.adobe.com/experience-manager/6-2/sites/developing/using/reference-materials/javadoc/io/sightly/java/api/Use#init(javax.script.Bindings))

Metoden `init` anropas vid initiering av klassen med ett `Bindings`-objekt som innehåller alla kontextobjekt och alla parametrar som skickats till klassen use.

Alla ytterligare funktioner (till exempel motsvarigheten till `WCMUsePojo.getProperties()`) måste implementeras explicit med [`javax.script.Bindings`](http://docs.oracle.com/javase/7/docs/api/javax/script/Bindings.html)-objektet. Till exempel:

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

Huvudfallet när du implementerar gränssnittet `Use` i stället för att utöka `WCMUsePojo` är när du vill använda en underklass till en befintlig klass som use-class.

### Kan anpassas från resursen {#adaptable-from-resource}

Ett annat alternativ är att använda en hjälpklass som kan anpassas från `org.apache.sling.api.resource.Resource`.

Säg att du måste skriva ett HTML-skript som visar mimetypen för en DAM-resurs. I det här fallet vet du att när ditt HTML-skript anropas, kommer det att finnas i kontexten för en `Resource` som omsluter en JCR `Node` med nodetype `dam:Asset`.

Du vet att en `dam:Asset`-nod har följande struktur:

### Databasstruktur {#repository-structure}

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

Här visas resursen (en JPEG-bild) som medföljer en standardinstallation av AEM som en del av exempelprojektgeometrixen. Resursen kallas `jane_doe.jpg` och dess mimeType är `image/jpeg`.

Om du vill komma åt resursen inifrån HTML kan du deklarera [`com.day.cq.dam.api.Asset`](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/com/adobe/granite/asset/api/Asset.html) som klassen i `data-sly-use`-satsen och sedan använda get-metoden `Asset` för att hämta den önskade informationen. Till exempel:

### `mimetype.html` {#mimetype-html}

```xml
<div data-sly-use.asset="com.day.cq.dam.api.Asset">
  <p>${asset.mimeType}</p>
</div>
```

Programsatsen `data-sly-use` instruerar HTML att anpassa aktuell `Resource` till en `Asset` och ge den det lokala namnet `asset`. Sedan anropas metoden `getMimeType` för `Asset` med hjälp av HTL-kortkommandot: `asset.mimeType`.

### Anpassa från begäran {#adaptable-from-request}

Det går också att använda alla klasser som kan anpassas från [`org.apache.sling.api.SlingHttpServletRequest`](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletRequest.html) som användningsklass

Precis som med fallet ovan för en användningsklass som kan anpassas från `Resource`, kan en användningsklass som kan anpassas från [`SlingHttpServletRequest`](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/org/apache/sling/api/SlingHttpServletRequest.html) anges i `data-sly-use`-satsen. Vid körning anpassas den aktuella begäran till den klass som anges och det resulterande objektet görs tillgängligt i HTML-koden.

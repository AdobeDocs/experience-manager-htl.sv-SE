---
title: HTL Java Use-API
description: Med HTL Java Use-API:t kan en HTML-fil få åtkomst till hjälpmetoder i en anpassad Java-klass.
exl-id: 9a9a2bf8-d178-4460-a3ec-cbefcfc09959
source-git-commit: ebeac25c38b81c92011c163c7860688f43547a7d
workflow-type: tm+mt
source-wordcount: '1137'
ht-degree: 0%

---


# HTL Java Use-API {#htl-java-use-api}

Med HTL Java Use-API:t kan en HTML-fil få åtkomst till hjälpmetoder i en anpassad Java-klass.

## Användningsfall {#use-case}

Använd-API:t för HTL Java gör att en HTML-fil kan komma åt hjälpmetoder i en anpassad Java-klass via `data-sly-use`. Med den här funktionen kan all komplex affärslogik kapslas in i Java-koden, medan HTML-koden endast hanterar direkt markeringsproduktion.

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

## Ett exempel {#a-simple-example}

I det här exemplet visas användningen av Use-API.

>[!NOTE]
>
>Det här exemplet är förenklat bara för att illustrera hur det används. I en produktionsmiljö rekommenderar Adobe att du använder [segmenteringsmodeller](https://sling.apache.org/documentation/bundles/models.html).

Börja med en HTL-komponent, som kallas `info` och som inte har någon use-klass. Den består av en enda fil, `/apps/my-example/components/info.html`

```xml
<div>
    <h1>${properties.title}</h1>
    <p>${properties.description}</p>
</div>
```

Lägg nu till innehåll för den här komponenten som ska återges på `/content/my-example/`:

```xml
{
    "sling:resourceType": "my-example/component/info",
    "title": "My Example",
    "description": "This Is Some Example Content."
}
```

När det här innehållet öppnas körs HTML-filen. I HTML-koden används kontextobjektet `properties` för att komma åt den aktuella resursens `title` och `description` och visa dem. Utdatafilen `/content/my-example.html` är:

```html
<div>
    <h1>My Example</h1>
    <p>This Is Some Example Content.</p>
</div>
```

### Lägga till en användningsklass {#adding-a-use-class}

Komponenten `info` som den ser ut behöver ingen use-klass för att utföra sin enkla funktion. Det finns fall där du behöver göra saker som inte kan göras i HTML och där du behöver en användningsklass. Tänk på följande:

>[!NOTE]
>
>En use-klass ska bara användas när något inte kan göras i enbart HTML.

Anta till exempel att du vill att komponenten `info` ska visa egenskaperna `title` och `description` för resursen, men alla med gemener. Eftersom HTML inte har någon metod för att sänka radering av strängar, behöver du en use-klass genom att lägga till en Java use-class och ändra `/apps/my-example/component/info/info.html` enligt följande:

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

Dessutom skapas `/apps/my-example/component/info/Info.java`.

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

Mer information finns i [Java-dokumenten för `com.adobe.cq.sightly.WCMUsePojo`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html).

Nu går vi igenom de olika delarna av koden.

### Java-klass, lokal kontra bunt {#local-vs-bundle-java-class}

Java-klassen kan installeras på två sätt:

* **Lokal** - I en lokal installation placeras Java-källfilen bredvid HTML-filen i samma databasmapp. Källan kompileras automatiskt vid behov. Inget separat kompilerings- eller paketeringssteg krävs.
* **Paket** - I en paketinstallation måste Java-klassen kompileras och distribueras i ett OSGi-paket med hjälp av AEM standarddistributionsmekanism (se avsnittet [Paketerad Java-klass](#bundled-java-class)).

Om du vill veta vilken metod du ska använda när bör du tänka på följande två saker:

* En **lokal Java use-class** rekommenderas när use-klassen är specifik för den aktuella komponenten.
* En **Java-användarklass** som ingår i paketet rekommenderas när Java-koden implementerar en tjänst som är tillgänglig från flera HTML-komponenter.

I det här exemplet används en lokal installation.

### Java-paketet är en databassökväg {#java-package-is-repository-path}

När du använder en lokal installation måste paketnamnet för use-klassen matcha databasmappens plats. Ersätt eventuella bindestreck i sökvägen med understreck i paketnamnet.

I det här fallet finns `Info.java` på `/apps/my-example/components/info` så paketet är `apps.my_example.components.info`:

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

### Utökar `WCMUsePojo` {#extending-wcmusepojo}

Det finns flera sätt att införliva en Java-klass med HTML, men det enklaste är att utöka klassen `WCMUsePojo`. I det här exemplet `/apps/my-example/component/info/Info.java`:

```java
package apps.my_example.components.info;

import com.adobe.cq.sightly.WCMUsePojo;

public class Info extends WCMUsePojo

    ...
}
```

### Initierar klassen {#initializing-the-class}

När use-klassen utökas från `WCMUsePojo` utförs initieringen genom att metoden `activate` åsidosätts, i det här fallet i `/apps/my-example/component/info/Info.java`

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

Metoden [activate](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html) används vanligtvis för att beräkna och lagra (i medlemsvariabler) de värden som behövs i din HTML-kod utifrån det aktuella sammanhanget (till exempel aktuell begäran och resurs).

Klassen `WCMUsePojo` ger åtkomst till samma uppsättning kontextobjekt som finns i en HTML-fil (se dokumentet [Globala objekt.](global-objects.md))

I en klass som utökar `WCMUsePojo` kan du komma åt kontextobjekt med hjälp av deras namn:

[`<T> T get(String name, Class<T> type)`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html)

Du kan också komma åt vanliga sammanhangsobjekt direkt med den bekvämlighetsmetod som anges i den här tabellen.

| Objekt | Praktisk metod |
|---|---|
| [`PageManager`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/day/cq/wcm/api/PageManager.html) | [`getPageManager()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getPageManager--) |
| [`Page`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/day/cq/wcm/api/Page.html) | [`getCurrentPage()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getCurrentPage--) |
| [`Page`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/day/cq/wcm/api/Page.html) | [`getResourcePage()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getResourcePage--) |
| [`ValueMap`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/org/apache/sling/api/resource/ValueMap.html) | [`getPageProperties()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getPageProperties--) |
| [`ValueMap`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/org/apache/sling/api/resource/ValueMap.html) | [`getProperties()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getProperties--) |
| [`Designer`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/day/cq/wcm/api/designer/Designer.html) | [`getDesigner()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getDesigner--) |
| [`Design`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/day/cq/wcm/api/designer/Design.html) | [`getCurrentDesign()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getCurrentDesign--) |
| [`Style`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/day/cq/wcm/api/designer/Style.html) | [`getCurrentStyle()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getCurrentStyle--) |
| [`Component`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/day/cq/wcm/api/components/Component.html) | [`getComponent()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getComponent--) |
| [`ValueMap`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/org/apache/sling/api/resource/ValueMap.html) | [`getInheritedProperties()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getInheritedPageProperties--) |
| [`Resource`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/org/apache/sling/api/resource/Resource.html) | [`getResource()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getResource--) |
| [`ResourceResolver`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/org/apache/sling/api/resource/ResourceResolver.html) | [`getResourceResolver()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getResourceResolver--) |
| [`SlingHttpServletRequest`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/org/apache/sling/api/SlingHttpServletRequest.html) | [`getRequest()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getRequest--) |
| [`SlingHttpServletResponse`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/org/apache/sling/api/SlingHttpServletResponse.html) | [`getResponse()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getResponse--) |
| [`SlingScriptHelper`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/org/apache/sling/api/scripting/SlingScriptHelper.html) | [`getSlingScriptHelper()`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html#getSlingScriptHelper--) |

### Getter-metoder {#getter-methods}

När klassen use har initierats körs HTML-filen. Under den här fasen hämtar HTML vanligtvis statusen för olika medlemsvariabler i use-klassen och återger dem för presentation.

Om du vill ge åtkomst till dessa värden inifrån HTML-filen måste du definiera egna get-metoder i use-klassen enligt följande namnkonvention:

* En metod i formatet `getXyz` visar i HTML-filen att en objektegenskap med namnet `xyz` finns.

I följande exempelfil `/apps/my-example/component/info/Info.java` resulterar metoderna `getTitle` och `getDescription` i att objektegenskaperna `title` och `description` blir tillgängliga i HTML-filens kontext.

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

### `data-sly-use`-attribut {#data-sly-use-attribute}

Attributet `data-sly-use` används för att initiera use-klassen i din HTML-kod. I det här exemplet deklarerar attributet `data-sly-use` att klassen `Info` används. I det här fallet kan du bara använda det lokala namnet på klassen eftersom du använder en lokal installation (när du har placerat Java-källfilen i samma mapp som HTML-filen). Om du använde en paketinstallation måste du ange det fullständiga, kvalificerade klassnamnet.

Observera användningen i det här `/apps/my-example/component/info/info.html`-exemplet.

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### Lokal identifierare {#local-identifier}

Identifieraren `info` (efter punkten i `data-sly-use.info`) används i HTML-filen för att identifiera klassen. Identifierarens omfång är globalt i filen, efter att den har deklarerats. Den är inte begränsad till elementet som innehåller programsatsen `data-sly-use`.

Observera användningen i det här `/apps/my-example/component/info/info.html`-exemplet.

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### Hämtar egenskaper {#getting-properties}

Identifieraren `info` används sedan för att komma åt objektegenskaperna `title` och `description` som exponerades via get-metoderna `Info.getTitle` och `Info.getDescription`.

Observera användningen i det här `/apps/my-example/component/info/info.html`-exemplet.

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### Utdata {#output}

När `/content/my-example.html` används returneras nu följande `/content/my-example.html`-fil.

```xml
<div>
    <h1>my example</h1>
    <p>this is some example content.</p>
</div>
```

>[!NOTE]
>
>Det här exemplet förenklades för att illustrera hur det används. I en produktionsmiljö rekommenderar Adobe att du använder [segmenteringsmodeller](https://sling.apache.org/documentation/bundles/models.html).

## Förutom grunderna {#beyond-the-basics}

I det här avsnittet introduceras ytterligare funktioner som går utöver det enkla exempel som beskrivs ovan.

* Skicka parametrar till en use-klass
* Paketerad Java-klass

### Parametrar skickas {#passing-parameters}

Parametrar kan skickas till en use-class vid initiering.

Mer information finns i dokumentationen för Sling [HTL Scripting Engine.](https://sling.apache.org/documentation/bundles/scripting/scripting-htl.html#passing-parameters-to-java-use-objects)

### Paketerad Java-klass {#bundled-java-class}

Med en paketerad användningsklass måste klassen kompileras, paketeras och distribueras i AEM med den vanliga OSGi-paketdistributionsmekanismen. Till skillnad från en lokal installation bör paketdeklarationen för use-klass namnges normalt på samma sätt som i det här `/apps/my-example/component/info/Info.java`-exemplet.

```java
package org.example.app.components;

import com.adobe.cq.sightly.WCMUsePojo;

public class Info extends WCMUsePojo {
    ...
}
```

Programsatsen `data-sly-use` måste referera till det fullständiga klassnamnet, i motsats till bara det lokala klassnamnet som i det här `/apps/my-example/component/info/info.html`-exemplet.

```xml
<div data-sly-use.info="org.example.app.components.info.Info">
  <h1>${info.title}</h1>
  <p>${info.description}</p>
</div>
```

---
title: HTL Java Use-API
description: Med HTL Java Use-API:t kan en HTML-fil få åtkomst till hjälpmetoder i en anpassad Java-klass.
exl-id: 9a9a2bf8-d178-4460-a3ec-cbefcfc09959
source-git-commit: 83f07cab5e2f4604701708f6a1a4bc19e3b54107
workflow-type: tm+mt
source-wordcount: '1505'
ht-degree: 0%

---


# HTL Java Use-API {#htl-java-use-api}

Med HTL Java Use-API:t kan en HTML-fil få åtkomst till hjälpmetoder i en anpassad Java-klass.

## Användningsfall {#use-case}

Använd-API:t för HTL Java möjliggör för en HTML-fil att komma åt hjälpmetoder i en anpassad Java-klass via `data-sly-use`. På så sätt kan all komplex affärslogik kapslas in i Java-koden, medan HTML-koden endast hanterar direkt markeringsproduktion.

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

The `bindings` kartan kan innehålla objekt som ger kontext till det HTML-skript som körs för tillfället och som Use-API-objektet kan använda för sin bearbetning.

## Ett exempel {#a-simple-example}

I det här exemplet visas användningen av Use-API.

>[!NOTE]
>
>Det här exemplet är förenklat för att visa hur det används. I en produktionsmiljö bör du använda [Sling-modeller.](https://sling.apache.org/documentation/bundles/models.html)

Vi börjar med en HTML-komponent, som kallas `info`, som inte har någon use-klass. Den består av en enda fil, `/apps/my-example/components/info.html`

```xml
<div>
    <h1>${properties.title}</h1>
    <p>${properties.description}</p>
</div>
```

Vi lägger även till innehåll för den här komponenten som ska återges på `/content/my-example/`:

```xml
{
    "sling:resourceType": "my-example/component/info",
    "title": "My Example",
    "description": "This Is Some Example Content."
}
```

När det här innehållet öppnas körs HTML-filen. I HTML-koden använder vi kontextobjektet `properties` för att komma åt den aktuella resursens `title` och `description` och visa dem. Utdatafilen `/content/my-example.html` blir:

```html
<div>
    <h1>My Example</h1>
    <p>This Is Some Example Content.</p>
</div>
```

### Lägga till en användningsklass {#adding-a-use-class}

The `info` som den ser ut behöver ingen use-klass för att utföra sin enkla funktion. Det finns fall där du behöver göra saker som inte kan göras i HTML och där du behöver en användningsklass. Tänk på följande:

>[!NOTE]
>
>En use-klass ska bara användas när något inte kan göras i enbart HTML.

Anta till exempel att du vill ha `info` -komponenten som ska visas `title` och `description` resursens egenskaper, dock med gemener. Eftersom HTML inte har någon metod för att sänka rabattsträngar behöver du en use-class. Vi kan göra detta genom att lägga till en Java-klass och ändra `/apps/my-example/component/info/info.html` enligt följande:

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

Dessutom skapar vi `/apps/my-example/component/info/Info.java`.

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

Se [Javadocs for `com.adobe.cq.sightly.WCMUsePojo`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html) för mer information.

Nu går vi igenom de olika delarna av koden.

### Java-klass, lokal kontra bunt {#local-vs-bundle-java-class}

Java-klassen kan installeras på två sätt:

* **Lokal** - I en lokal installation placeras Java-källfilen bredvid HTML-filen i samma databasmapp. Källan kompileras automatiskt vid behov. Inget separat kompilerings- eller paketeringssteg krävs.
* **Paket** - I en paketinstallation måste Java-klassen kompileras och distribueras i ett OSGi-paket med hjälp av AEM standardmetod (se avsnittet [Paketerad Java-klass](#bundled-java-class)).

Om du vill veta vilken metod du ska använda när bör du tänka på följande två saker:

* A **lokal Java use-class** rekommenderas när use-klassen är specifik för komponenten i fråga.
* A **bundle Java use-class** rekommenderas när Java-koden implementerar en tjänst som är tillgänglig från flera HTML-komponenter.

I det här exemplet används en lokal installation.

### Java-paketet är en databassökväg {#java-package-is-repository-path}

När en lokal installation används måste paketnamnet för use-klassen matcha databasmappens namn, där eventuella bindestreck i sökvägen ersätts med understreck i paketnamnet.

I detta fall `Info.java` finns på `/apps/my-example/components/info` så paketet är `apps.my_example.components.info`:

```java
package apps.my_example.components.info;

import com.adobe.cq.sightly.WCMUsePojo;

public class Info extends WCMUsePojo {

   ...

}
```

>[!NOTE]
>
>Du bör använda bindestreck i namn på databasobjekt när du AEM. Däremot är bindestreck ogiltiga i Java-paketnamn. Av denna anledning **alla bindestreck i databassökvägen måste konverteras till understreck i paketnamnet**.

### Utöka `WCMUsePojo` {#extending-wcmusepojo}

Det finns många sätt att införliva en Java-klass med HTML, men det enklaste är att utöka `WCMUsePojo` klassen. Till exempel `/apps/my-example/component/info/Info.java`:

```java
package apps.my_example.components.info;

import com.adobe.cq.sightly.WCMUsePojo;

public class Info extends WCMUsePojo

    ...
}
```

### Initierar klassen {#initializing-the-class}

När klassen use utökas från `WCMUsePojo`, initieringen utförs genom att åsidosätta `activate` i det här fallet `/apps/my-example/component/info/Info.java`

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

Vanligtvis är [activate](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html) -metoden används för att beräkna och lagra (i medlemsvariabler) de värden som behövs i din HTML-kod baserat på den aktuella kontexten (till exempel aktuell begäran och resurs).

The `WCMUsePojo` -klassen ger åtkomst till samma uppsättning kontextobjekt som finns i en HTML-fil (se dokumentet [Globala objekt.](global-objects.md))

I en klass som utökas `WCMUsePojo`, kan kontextobjekt kommas åt med hjälp av namn

[`<T> T get(String name, Class<T> type)`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/com/adobe/cq/sightly/WCMUsePojo.html)

Du kan också komma åt ofta använda kontextobjekt direkt med den bekvämlighetsmetod som anges i den här tabellen.

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

När use-class har initierats körs HTML-filen. Under den här scenen kommer HTML vanligtvis att dra in status för olika medlemsvariabler i use-klassen och återge dem för presentation.

Om du vill ge åtkomst till dessa värden inifrån HTML-filen måste du definiera egna get-metoder i use-klassen enligt följande namnkonvention:

* En metod i formuläret `getXyz` visar en objektegenskap med namnet `xyz`.

I följande exempelfil `/apps/my-example/component/info/Info.java`, metoderna `getTitle` och `getDescription` resultatet i objektegenskaperna `title` och `description` som är tillgängliga i HTML-filens sammanhang.

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

The `data-sly-use` -attributet används för att initiera use-klassen i din HTML-kod. I vårt exempel `data-sly-use` -attributet deklarerar att vi vill använda klassen `Info`. Vi kan bara använda det lokala namnet på klassen eftersom vi använder en lokal installation (när Java-källfilen har placerats i samma mapp som HTML-filen). Om vi använde en paketinstallation måste vi ange det fullständiga klassnamnet.

Observera hur detta används `/apps/my-example/component/info/info.html` exempel.

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### Lokal identifierare {#local-identifier}

Identifierare `info` (efter punkt in `data-sly-use.info`) används i HTML-filen för att identifiera klassen. Identifierarens omfång är globalt i filen, efter att den har deklarerats. Den är inte begränsad till elementet som innehåller `data-sly-use` -programsats.

Observera hur detta används `/apps/my-example/component/info/info.html` exempel.

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### Hämtar egenskaper {#getting-properties}

Identifierare `info` används sedan för att komma åt objektegenskaperna `title` och `description` som exponerades via get-metoder `Info.getTitle` och `Info.getDescription`.

Observera hur detta används `/apps/my-example/component/info/info.html` exempel.

```xml
<div data-sly-use.info="Info">
    <h1>${info.lowerCaseTitle}</h1>
    <p>${info.lowerCaseDescription}</p>
</div>
```

### Utdata {#output}

Nu när vi öppnar `/content/my-example.html` returnerar följande `/content/my-example.html` -fil.

```xml
<div>
    <h1>my example</h1>
    <p>this is some example content.</p>
</div>
```

>[!NOTE]
>
>Det här exemplet förenklades för att visa hur det används. I en produktionsmiljö bör du använda [Sling-modeller.](https://sling.apache.org/documentation/bundles/models.html)

## Förutom grunderna {#beyond-the-basics}

I det här avsnittet introduceras ytterligare funktioner som går utöver det enkla exempel som beskrivs ovan.

* Skicka parametrar till en use-klass
* Paketerad Java-klass

### Parametrar skickas {#passing-parameters}

Parametrar kan skickas till en use-class vid initiering.

Mer information finns i Sling [Dokumentation för HTL Scripting Engine.](https://sling.apache.org/documentation/bundles/scripting/scripting-htl.html#passing-parameters-to-java-use-objects)

### Paketerad Java-klass {#bundled-java-class}

Med en paketerad användningsklass måste klassen kompileras, paketeras och distribueras i AEM med den vanliga OSGi-paketdistributionsmekanismen. I motsats till en lokal installation bör paketdeklarationen för use-klass namnges normalt på samma sätt som i detta `/apps/my-example/component/info/Info.java` exempel.

```java
package org.example.app.components;

import com.adobe.cq.sightly.WCMUsePojo;

public class Info extends WCMUsePojo {
    ...
}
```

Och `data-sly-use` -programsatsen måste referera till det fullständiga klassnamnet, i motsats till bara det lokala klassnamnet som i det här `/apps/my-example/component/info/info.html` exempel.

```xml
<div data-sly-use.info="org.example.app.components.info.Info">
  <h1>${info.title}</h1>
  <p>${info.description}</p>
</div>
```

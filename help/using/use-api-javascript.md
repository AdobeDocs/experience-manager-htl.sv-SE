---
title: HTL JavaScript Use-API
seo-title: HTL JavaScript Use-API
description: HTML-mallens språk - HTML - JavaScript Use-API aktiverar en HTML-fil för att komma åt hjälpkod som skrivits i JavaScript.
seo-description: HTML-mallens språk - HTML - JavaScript Use-API aktiverar en HTML-fil för att komma åt hjälpkod som skrivits i JavaScript.
uuid: 7ab34b10-30ac-44d6-926b-0234f52e5541
contentOwner: User
products: SG_EXPERIENCEMANAGER/HTL
topic-tags: html-template-language
content-type: reference
discoiquuid: 18871af8-e44b-4eec-a483-fcc765dae58f
mwpw-migration-script-version: 2017-10-12T21 46 58.665-0400
translation-type: tm+mt
source-git-commit: bd1962e25d152be4f1608d0a83d8d5b3e728b4aa

---


# HTL JavaScript Use-API {#htl-javascript-use-api}

Med HTML-mallens språk (HTL) JavaScript Use-API kan en HTML-fil få åtkomst till hjälpkod som skrivits i JavaScript. På så sätt kan all komplex affärslogik kapslas in i JavaScript-koden, medan HTML-koden endast hanterar direkt kodproduktion.

## Ett enkelt exempel {#a-simple-example}

Vi definierar en komponent, `info`som finns på

**`/apps/my-example/components/info`**

Den innehåller två filer:

* **`info.js`**: en JavaScript-fil som definierar use-klassen.
* `info.html`: en HTML-fil som definierar komponenten `info`. Den här koden använder funktionerna i `info.js` via API:t.

### /apps/my-example/component/info/info.js {#apps-my-example-component-info-info-js}

```java
"use strict";
use(function () {
    var info = {};    
    info.title = resource.properties["title"];
    info.description = resource.properties["description"];    
    return info;
});
```

### /apps/my-example/component/info/info.html {#apps-my-example-component-info-info-html}

```xml
<div data-sly-use.info="info.js">
    <h1>${info.title}</h1>
    <p>${info.description}</p>
</div>
```

Vi skapar också en innehållsnod som använder **`info`** komponenten på

**`/content/my-example`**, med egenskaper:

* `sling:resourceType = "my-example/component/info"`
* `title = "My Example"`
* `description = "This is some example content."`

Här är den resulterande databasstrukturen:

### Databasstruktur {#repository-structure}

```java
{
  "apps": {
    "my-example": {
      "components": {
        "info": {
          "info.html": {
            ...
          }, 
          "info.js": {
            ...
          }
        }
      }
    }
 },     
 "content": {
    "my-example": {
      "sling:resourceType": "my-example/component/info",
      "title": "My Example",
      "description": "This is some example content."
    }
  }
}
```

Överväg följande komponentmall:

```xml
<section class="component-name" data-sly-use.component="component.js">
    <h1>${component.title}</h1>
    <p>${component.description}</p>
</section>
```

Motsvarande logik kan skrivas med följande ***JavaScript på*** serversidan, som finns i en `component.js` fil bredvid mallen:

```
use(function () {
    var Constants = {
        DESCRIPTION_PROP: "jcr:description",
        DESCRIPTION_LENGTH: 50
    };
 
    var title = currentPage.getNavigationTitle() || currentPage.getTitle() || currentPage.getName();
    var description = properties.get(Constants.DESCRIPTION_PROP, "").substr(0, Constants.DESCRIPTION_LENGTH);
 
    return {
        title: title,
        description: description
    };
});
```

Detta försöker hämta `title` från olika källor och beskär beskrivningen till 50 tecken.

## Beroenden {#dependencies}

Tänk dig att vi har en verktygsklass som redan är utrustad med smarta funktioner, som standardlogiken för navigeringsrubriken eller som klipper en sträng till en viss längd:

```
use(['../utils/MyUtils.js'], function (utils) {
    var Constants = {
        DESCRIPTION_PROP: "jcr:description",
        DESCRIPTION_LENGTH: 50
    };
 
    var title = utils.getNavigationTitle(currentPage);
    var description = properties.get(Constants.DESCRIPTION_PROP, "").substr(0, Constants.DESCRIPTION_LENGTH);
 
    return {
        title: title,
        description: description
    };
});
```

## Utöka {#extending}

Beroendemönstret kan också användas för att utöka logiken i en annan komponent (som vanligtvis är `sling:resourceSuperType` den aktuella komponenten).

Föreställ dig att den överordnade komponenten redan tillhandahåller `title`och vi vill lägga till en **`description`** också:

```
use(['../parent-component/parent-component.js'], function (component) {
    var Constants = {
        DESCRIPTION_PROP: "jcr:description",
        DESCRIPTION_LENGTH: 50
    };
 
    component.description = utils.shortenString(properties.get(Constants.DESCRIPTION_PROP, ""), Constants.DESCRIPTION_LENGTH);
 
    return component;
});
```

## Skicka parametrar till en mall {#passing-parameters-to-a-template}

När det gäller programsatser som kan vara oberoende av komponenter kan det vara användbart att skicka parametrar till associerat Use-API. **`data-sly-template`**

I vår komponent anropar vi en mall som finns i en annan fil:

```xml
<section class="component-name" data-sly-use.tmpl="template.html" data-sly-call="${tmpl.templateName @ page=currentPage}"></section>
```

Det här är mallen som finns i `template.html`:

```xml
<template data-sly-template.templateName="${@ page}" data-sly-use.tmpl="${'template.js' @ page=page, descriptionLength=50}">
    <h1>${tmpl.title}</h1>
    <p>${tmpl.description}</p>
</template>
```

Motsvarande logik kan skrivas med följande JavaScript på ***serversidan*** , som finns i en `template.js` fil bredvid mallfilen:

```
use(function () {
    var Constants = {
        DESCRIPTION_PROP: "jcr:description"
    };
 
    var title = this.page.getNavigationTitle() || this.page.getTitle() || this.page.getName();
    var description = this.page.getProperties().get(Constants.DESCRIPTION_PROP, "").substr(0, this.descriptionLength);
 
    return {
        title: title,
        description: description
    };
});
```

De parametrar som skickas anges för `this` nyckelordet.

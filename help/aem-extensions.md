---
title: AEM
description: AEM erbjuder tillägg av HTML-specifikationen för att AEM som utvecklare.
exl-id: d78cb84d-f958-45e2-9c6c-df86a68277d5
source-git-commit: 151b40ee204f1afa7a772afd64cbac4790c02cc2
workflow-type: tm+mt
source-wordcount: '308'
ht-degree: 0%

---

# AEM {#aem-extensions}

Liknar [Apache Sling-tillägg för HTML-specifikationen,](https://sling.apache.org/documentation/bundles/scripting/scripting-htl.html#extensions-of-the-htl-specification-1) AEM erbjuder ytterligare uttrycksalternativ som gör det lite enklare att arbeta med AEM koncept direkt i HTML-skripten.

## i18n {#i18n}

Samma [tre ytterligare alternativ](https://sling.apache.org/documentation/bundles/scripting/scripting-htl.html#i18n) som i Apache Sling kan användas tillsammans med `i18n`:

* `locale`
* `hint`
* `basename`

I AEM [Stöd för internationalisering](https://experienceleague.adobe.com/docs/experience-manager-65/developing/components/internationalization/i18n-dev.html) för HTML implementeras med hjälp av API:t från `com.day.cq.i18n` paket.

## data-sly-include {#data-sly-include}

AEM `data-sly-include` kan ta ytterligare `wcmmode` alternativ som styr [WCM-läge](https://developer.adobe.com/experience-manager/reference-materials/cloud-service/javadoc/com/day/cq/wcm/api/WCMMode.html) för det inkluderade skriptet. De tillåtna värdena är namnen på de tillgängliga enum-konstanterna.

## resurssnål {#data-sly-resource}

Förutom banor och `Resources`, `data-sly-resource` blockelement kan också fungera med [`Maps`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Map.html) eller [`Records`.](https://github.com/apache/sling-org-apache-sling-scripting-sightly-runtime/blob/master/src/main/java/org/apache/sling/scripting/sightly/Record.java) Med båda inriktningarna `resourceName` Strängegenskapen måste anges. Dess värde används för att skapa en [Syntetisk resurs](https://www.javadoc.io/doc/org.apache.sling/org.apache.sling.api/latest/org/apache/sling/api/resource/SyntheticResource.html) som kommer att inkluderas i återgivningssammanhanget. Resten av egenskaperna från `Record` eller `Map` som skickades till `data-sly-resource` kommer att användas som vanligt `Resource` egenskaper. Om `sling:resourceType` egenskapen saknas i kartan, resurstypen antas vara antingen värdet för `resourceType` [uttrycksalternativ](https://github.com/adobe/htl-spec/blob/1.4/SPECIFICATION.md#229-resource) eller resurstypen för den aktuella resursen som styr återgivningen.

Följande mappnings-/postegenskaper tillgängliga i skriptomfånget som `map`:

```javascript
{
    resourceName: "myText",
    "sling:resourceType": "core/wcm/components/text/v2/text",
    "text": "Hello World!"
}
```

Och med följande kod:

```html
<div class="outer" data-sly-resource="${map}"></div>
```

Följande utdata förväntas:

```html
<div class="outer">
    <div class="myText">
        <div data-cmp-data-layer="{&quot;text-e58d65c472&quot;:{&quot;@type&quot;:&quot;core/wcm/components/text/v2/text&quot;,&quot;xdm:text&quot;:&quot;<p>Hello world!</p>&quot;}}" id="text-e58d65c472" class="cmp-text">
            <p>Hello world!</p>
        </div>
  </div>
</div>
```

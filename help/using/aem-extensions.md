---
title: AEM Extensions
description: AEM erbjuder tillägg av HTML-specifikationen till AEM för att underlätta för dig som utvecklare.
exl-id: d78cb84d-f958-45e2-9c6c-df86a68277d5
index: false
source-git-commit: a496d23277902a5cd573a6a8af770f27b0269f05
workflow-type: tm+mt
source-wordcount: '228'
ht-degree: 0%

---


# AEM Extensions {#aem-extensions}

På samma sätt som [Apache Sling-tilläggen i HTML-specifikationen](https://sling.apache.org/documentation/bundles/scripting/scripting-htl.html#extensions-of-the-htl-specification-1) har AEM ytterligare uttrycksalternativ som gör det lite enklare att arbeta med AEM-koncept direkt i HTML-skripten.

## i18n {#i18n}

Samma [tre ytterligare alternativ](https://sling.apache.org/documentation/bundles/scripting/scripting-htl.html#i18n) som i Apache Sling kan användas tillsammans med `i18n`:

* `locale`
* `hint`
* `basename`

I AEM implementeras emellertid [internationaliseringsstödet](https://experienceleague.adobe.com/sv/docs/experience-manager-65/content/implementing/developing/components/internationalization/i18n-dev) för HTML med hjälp av API:t från paketet `com.day.cq.i18n`.

## `data-sly-include` {#data-sly-include}

I AEM kan `data-sly-include` ta ytterligare ett `wcmmode`-alternativ som styr [WCM-läget ](https://developer.adobe.com/experience-manager/reference-materials/cloud-service/javadoc/com/day/cq/wcm/api/WCMMode.html) för det inkluderade skriptet. De tillåtna värdena är namnen på de tillgängliga enum-konstanterna.

## `data-sly-resource` {#data-sly-resource}

Förutom sökvägar och `Resources` kan blockelementet `data-sly-resource` även fungera med [`Maps`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Map.html) eller [`Records`](https://github.com/apache/sling-org-apache-sling-scripting-sightly-runtime/blob/master/src/main/java/org/apache/sling/scripting/sightly/Record.java). Med båda metoderna måste strängegenskapen `resourceName` anges. Dess värde används för att skapa en [syntetisk resurs](https://www.javadoc.io/doc/org.apache.sling/org.apache.sling.api/latest/org/apache/sling/api/resource/SyntheticResource.html) som ingår i återgivningssammanhanget. Resten av egenskaperna från `Record` eller `Map` som skickas till `data-sly-resource` används som normala `Resource`-egenskaper. Om egenskapen `sling:resourceType` saknas i kartan antas resurstypen vara antingen värdet för `resourceType` [expression option](https://github.com/adobe/htl-spec/blob/1.4/SPECIFICATION.md#229-resource) eller resurstypen för den aktuella resursen som styr återgivningen.

Följande mappnings-/postegenskaper som är tillgängliga i skriptomfånget har angetts som `map`:

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

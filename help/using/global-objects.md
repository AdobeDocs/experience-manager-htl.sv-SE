---
title: Globala HTML-objekt
description: Lär dig mer om uppräkningsbara objekt, Java-baserade objekt och JavaScript-baserade objekt i HTML.
exl-id: ca590b92-f1b3-4e44-a04a-a2c10dff256f
source-git-commit: c6bb6f0954ada866cec574d480b6ea5ac0b51a3f
workflow-type: tm+mt
source-wordcount: '164'
ht-degree: 1%

---


# Globala HTML-objekt {#htl-global-objects}

Utan att behöva ange något ger HTML tillgång till många objekt som är användbara för utvecklaren. De här objekten är utöver alla objekt som kan introduceras via [Use-API](java-use-api.md).

>[!NOTE]
>
>För utvecklare som är bekanta med JSP-utveckling i AEM, ger HTL åtkomst till alla objekt som var vanliga i JSP efter att `global.jsp` inkluderats.

## Antal objekt {#enumerable-objects}

Dessa objekt ger smidig åtkomst till ofta använd information. Deras innehåll kan nås med punktnotation och kan itereras igenom med `data-sly-list` eller `data-sly-repeat`.

| Variabelnamn | Beskrivning | Bakåt av |
|--- |--- |--- |
| `properties` | Lista över egenskaper för den aktuella resursen | [`org.apache.sling.api.resource.ValueMap`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/org/apache/sling/api/resource/ValueMap.html) |
| `pageProperties` | Lista över sidegenskaper för den aktuella sidan | [`org.apache.sling.api.resource.ValueMap`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/org/apache/sling/api/resource/ValueMap.html) |
| `inheritedPageProperties` | Lista över ärvda sidegenskaper för den aktuella sidan | [`org.apache.sling.api.resource.ValueMap`](https://developer.adobe.com/experience-manager/reference-materials/6-5/javadoc/org/apache/sling/api/resource/ValueMap.html) |

## Java-baserade objekt {#java-backed-objects}

Motsvarande Java-objekt har bakomliggande baksidor.

| Variabelnamn | Beskrivning |
|---|---|
| `component` | `com.day.cq.wcm.api.components.Component` |
| `componentContext` | `com.day.cq.wcm.api.components.ComponentContext` |
| `currentContentPolicy` | `com.day.cq.wcm.api.policies.ContentPolicy` |
| `currentContentPolicyProperties` | `com.day.cq.wcm.api.policies.ContentPolicy` |
| `currentDesign` | `com.day.cq.wcm.api.designer.Design` |
| `currentNode` | `javax.jcr.Node` |
| `currentPage` | `com.day.cq.wcm.api.Page` |
| `currentSession` | `javax.servlet.http.HttpSession` |
| `currentStyle` | `com.day.cq.wcm.api.designer.Style` |
| `designer` | `com.day.cq.wcm.api.designer.Designer` |
| `editContext` | `com.day.cq.wcm.api.components.EditContext` |
| `log` | `org.slf4j.Logger` |
| `out` | `java.io.PrintWriter` |
| `pageManager` | `com.day.cq.wcm.api.PageManager` |
| `reader` | `java.io.BufferedReader` |
| `request` | `org.apache.sling.api.SlingHttpServletRequest` |
| `resolver` | `org.apache.sling.api.resource.ResourceResolver` |
| `resource` | `org.apache.sling.api.resource.Resource` |
| `resourceDesign` | `com.day.cq.wcm.api.designer.Design` |
| `resourcePage` | `com.day.cq.wcm.api.Page` |
| `response` | `org.apache.sling.api.SlingHttpServletResponse` |
| `sling` | `org.apache.sling.api.scripting.SlingScriptHelper` |
| `slyWcmHelper` | `com.adobe.cq.sightly.WCMScriptHelper` |
| `wcmmode` | `com.adobe.cq.sightly.SightlyWCMMode` |
| `xssAPI` | `com.adobe.granite.xss.XSSAPI` |

## JavaScript-baserade objekt {#javascript-backed-objects}

Det går att backa HTML-logiken med JavaScript. Den rekommenderade eller rekommenderade metoden är att använda [segmenteringsmodeller](https://sling.apache.org/documentation/bundles/models.html).

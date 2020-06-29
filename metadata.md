---
product: Adobe Experience Manager
git-repo: https://github.com/AdobeDocs/experience-manager-htl.en
index: y
solution-title: Lär dig mer och support för HTML
solution-hub-url: https://docs.adobe.com/content/help/en/experience-manager-cloud-service/sites/home.html
getting-started-title: Komma igång med utveckling för AEM
getting-started-url: https://docs.adobe.com/content/help/en/experience-manager-cloud-service/core-concepts/home.html
tutorials-title: AEM-självstudiekurser
tutorials-url: https://docs.adobe.com/content/help/en/experience-manager-learn/cloud-service/overview.html
translation-type: tm+mt
source-git-commit: d3426d87dce09ac34ff1aca431ff2bfad2f7134a
workflow-type: tm+mt
source-wordcount: '139'
ht-degree: 17%

---


# Metadata för intern användning

Metadata i GitHub-redigeringssystemet är hierarkiska och definieras så här i ökande prioritetsnivåer.

1. metadata.md
1. TillC
1. Artikel

Metadata som definieras i filen metadata.md gäller för hela repon, men kan åsidosättas på ToC- och artikelnivå. Alla åsidosättningar av metadata bör göras på lägsta möjliga nivå.

Metadata i filen experience-manager-core-components.en repo är det minsta nödvändiga.

metadata.md

* `product`
* `git-repo`
* `index: y`
* `solution-title`
* `solution-hub-url`
* `getting-started-title`
* `getting-started-url`
* `tutorials-title`
* `tutorials-url`

ToCS

* `sub-product`
* `user-guide-title`

Artikel

* `title`
* `description`
* `index: n` (endast för tidigare versioner av komponenter)

Mer information om metadata finns i den [interna redigeringsguiden.](https://docs.adobe.com/help/en/collaborative-doc-instructions/collaboration-guide/markdown/metadata.html#solution-metadata)

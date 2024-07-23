---
solution: Experience Manager
type: Documentation
product: adobe experience manager
git-repo: https://github.com/AdobeDocs/experience-manager-htl.sv-SE
index: y
recommendations: noDisplay
source-git-commit: 22f62868df0fcfc558e5d62434dde843a9f3ca83
workflow-type: tm+mt
source-wordcount: '81'
ht-degree: 0%

---


# Metadata för intern användning

I GitHub-redigeringssystemet definieras metadata hierarkiskt, med ökande nivåer av prejudikat enligt följande:

1. metadata.md
1. TillC
1. Artikel

De metadata som definieras i filen metadata.md gäller för hela repon, men kan åsidosättas på ToC- och artikelnivå. Alla åsidosättningar av metadata bör göras på lägsta möjliga nivå.

Metadata i `experience-manager-core-components.en`-svaret är det minsta nödvändiga.

metadata.md

* `product`
* `git-repo`
* `index: y`

Används inte längre:

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


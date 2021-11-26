---
title: Istruzioni online
permalink: index.html
layout: home
ms.openlocfilehash: ba3d2a5154e5e9b08e750f7d9ced8b62048dd66d
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 11/16/2021
ms.locfileid: "132625849"
---
# <a name="content-directory"></a>Directory contenuto

Questa repository contiene le esercitazioni pratiche per il corso Microsoft [AI-102 Progettazione e implementazione di una soluzione di Microsoft Azure per intelligenza artificiale](https://docs.microsoft.com/learn/certifications/courses/ai-102t00) e i relativi [moduli autogestiti su Microsoft Learn](https://aka.ms/AzureLearn_AIEngineer). Gli esercizi si integrano con i materiali di formazione e permettono di fare pratica con le tecnologie descritte.

Per completare gli esercizi, sarà necessaria una sottoscrizione di Microsoft Azure. Se non è stata fornita dal docente, è possibile iscriversi a una versione di valutazione gratuita in [https://azure.microsoft.com](https://azure.microsoft.com).

## <a name="labs"></a>Lab

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions'" %}
| Modulo | Lab |
| --- | --- | 
{% for activity in labs  %}| {{ activity.lab.module }} | [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}


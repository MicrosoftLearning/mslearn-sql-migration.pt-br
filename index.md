---
title: Exercícios do Azure OpenAI
permalink: index.html
layout: home
---

# Exercícios de migração do SQL Server

Os exercícios a seguir foram projetados para acompanhar os módulos no roteiro de aprendizado [Migrar cargas de trabalho do SQL Server para o SQL do Azure](https://learn.microsoft.com/training/paths/migrate-sql-workloads-azure/) no Microsoft Learn.

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %} {% for activity in labs  %}
- [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) {% endfor %}
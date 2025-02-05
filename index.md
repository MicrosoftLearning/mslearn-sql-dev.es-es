---
title: Instrucciones hospedadas en línea
permalink: index.html
layout: home
---

# Ejercicios de desarrollo de Azure SQL

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %} {% for activity in labs  %} [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) {% endfor %}



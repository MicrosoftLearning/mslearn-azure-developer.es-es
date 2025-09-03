---
title: Ejercicios para desarrolladores de Azure
permalink: index.html
layout: home
---

## Información general

Los siguientes ejercicios se han diseñado para brindarle una experiencia práctica de aprendizaje en la que explorará las tareas comunes que realizan los desarrolladores al compilar e implementar soluciones en Microsoft Azure.

> **Nota**: para completar los ejercicios, necesitarás una suscripción a Azure en la que tengas permisos y cuota suficientes para aprovisionar los recursos de Azure y modelos de IA generativa necesarios. Si aún no tienes una, puedes registrarte para obtener una [cuenta de Azure](https://azure.microsoft.com/free). 

Algunos ejercicios pueden tener requisitos adicionales o diferentes. Estos contendrán una sección **Antes de empezar** específica de ese ejercicio.

## Áreas temáticas
{% assign exercises = site.pages | where_exp:"page", "page.url contains '/instructions'" %} {% assign grouped_exercises = exercises | group_by: "lab.topic" %}

<ul>
{% for group in grouped_exercises %}
<li><a href="#{{ group.name | slugify }}">{{ group.name }}</a></li>
{% endfor %}
</ul>

{% for group in grouped_exercises %}

## <a id="{{ group.name | slugify }}"></a>{{ group.name }} 

{% for activity in group.items %} [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) <br/> {{ activity.lab.description }}

---

{% endfor %} <a href="#overview">Volver al principio</a> {% endfor %}


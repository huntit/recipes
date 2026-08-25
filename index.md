---
title: Recipes repository.
---

Nicely formatted for import into MealBoard app.

{% assign cards = site.pages | where: "recipe", true | sort: "title" %}
{% assign legacy = site.static_files | where_exp: "f", "f.extname == '.html'" | sort: "basename" %}

<ul class="recipe-index">
{% for p in cards %}
  <li>
    <a href="{{ p.url | relative_url }}"><strong>{{ p.title }}</strong></a>
    {% if p.meat %}<em>{{ p.meat }}{% if p.serves %} · serves {{ p.serves }}{% endif %}</em>{% endif %}
    {% if p.blurb %}<br>{{ p.blurb }}{% endif %}
  </li>
{% endfor %}

{% for f in legacy %}
  <li>
    <a href="{{ f.path | relative_url }}">
      {{ f.basename | replace: '-', ' ' | capitalize }}
    </a>
  </li>
{% endfor %}
</ul>

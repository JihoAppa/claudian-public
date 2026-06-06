---
layout: default
title: "Claudian Public Research"
---

<header class="masthead">
  <h1>{{ site.title }}</h1>
  <p>{{ site.description }}</p>
</header>

<main>
{% assign sections = site.research | group_by: "section" %}
{% for section in sections %}
  <section>
    <h2>{{ section.name }}</h2>
    <ol class="article-list">
    {% for item in section.items %}
      <li>
        <a href="{{ site.baseurl }}{{ item.url }}">{{ item.title }}</a>
        {% if item.summary %}<span>{{ item.summary }}</span>{% endif %}
      </li>
    {% endfor %}
    </ol>
  </section>
{% endfor %}
</main>

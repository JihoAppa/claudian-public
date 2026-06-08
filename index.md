---
layout: default
title: "Claudian Public Research"
---

<header class="masthead">
  <div>
    <p class="folio-kicker">Curated pop research</p>
    <h1>{{ site.title }}</h1>
    <p>{{ site.description }}</p>
  </div>
  <div class="pop-mark" aria-label="Published research count">
    <span>published</span>
    <strong>{{ site.research.size }}</strong>
    <span>notes</span>
  </div>
</header>

<main>
{% if site.research.size == 0 %}
  <section class="empty-state">
    <h2>아직 공개된 원고가 없습니다</h2>
  </section>
{% else %}
{% assign sections = site.research | group_by: "section" %}
{% for section in sections %}
  <section class="workflow">
    <h2>{{ section.name }}</h2>
    <ol class="article-list">
    {% for item in section.items %}
      <li>
        <div>
          <a href="{{ item.url | relative_url }}">{{ item.title }}</a>
          {% if item.summary %}<span>{{ item.summary }}</span>{% endif %}
        </div>
      </li>
    {% endfor %}
    </ol>
  </section>
{% endfor %}
{% endif %}
</main>

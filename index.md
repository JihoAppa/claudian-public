---
layout: default
title: "Claudian Public Research"
---

<header class="masthead">
  <div>
    <p class="folio-kicker">Curated pop research</p>
    <h1>{{ site.title }}</h1>
    <p>{{ site.description }} 승인된 원고만 공개되는 작은 연구 서가입니다.</p>
  </div>
  <div class="pop-mark" aria-label="Approved manuscripts count">
    <span>approved</span>
    <strong>{{ site.research.size }}</strong>
    <span>manuscripts</span>
  </div>
</header>

<main>
{% if site.research.size == 0 %}
  <section class="empty-state">
    <h2>No manuscripts published</h2>
    <p>아직 공개 승인된 원고가 없습니다. 이 페이지는 빈 상태를 의도적으로 유지하며, 충분히 검토되고 큐레이션된 글만 Jekyll collection에 들어갑니다.</p>
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
          <a href="{{ site.baseurl }}{{ item.url }}">{{ item.title }}</a>
          {% if item.summary %}<span>{{ item.summary }}</span>{% endif %}
        </div>
      </li>
    {% endfor %}
    </ol>
  </section>
{% endfor %}
{% endif %}
</main>

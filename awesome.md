---
layout: page
title: Awesome
permalink: /awesome/
---

收錄一些常用的工具 & 網站，有時真的會忘記

<div>
{% for awesome in site.data.awesome %}
    <h2>{{ awesome.name }}</h2>
    <ul>
    {% for item in awesome.items %}
        <li>
            <strong>{{ item.name }}</strong>：<a href="{{ item.url }}" target="_blank" rel="noopener">{{ item.url }}</a>
            {% if item.description %}
                <br/>{{ item.description }}
            {% endif %}
            {% if item.news %}
                <br/>
                {% for news in item.news %}
                    <a href="{{ news }}" target="_blank" rel="noopener">[新聞]</a>
                {% endfor %}
            {% endif %}
        </li>
    {% endfor %}
    </ul>
{% endfor %}
</div>

---
layout: page
title: Backend
description: "收錄知名的後端資源"
---

{{ page.description | escape }}

<div>
{% for backend in site.data.backend %}
    <h2>{{ backend.name }}</h2>
    <ul>
    {% assign sortedItems = backend.items | sort_natural: 'name' %}
    {% for item in sortedItems %}
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
---
layout: page
title: Learning
description: "收錄學習資源"
visible: true
---

{{ page.description | escape }}

<div>
{% for learning in site.data.learning %}
    <h2>{{ learning.name }}</h2>
    <ul>
    {% assign sortedItems = learning.items | sort_natural: 'name' %}
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
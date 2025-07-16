---
layout: page
title: Taiwan
description: "收錄跟 Taiwan 🇹🇼 相關的線上資源"
---

{{ page.description | escape }}

<div>
{% for taiwan in site.data.taiwan %}
    <h2>{{ taiwan.name }}</h2>
    <ul>
    {% assign sortedItems = taiwan.items | sort_natural: 'name' %}
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
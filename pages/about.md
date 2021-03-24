---
layout: page
title: About
description: 我不过像你像他像那野草野花
keywords: hq234
comments: true
menu: 关于
permalink: /about/
---

时间无言，如此这般

## 联系

<ul>
{% for website in site.data.social %}
<li>{{website.sitename }}：<a href="{{ website.url }}" target="_blank">@{{ website.name }}</a></li>
{% endfor %}
{% if site.url contains 'hq234.top' %}
<li>
<br />
<img style="height:395px;width:632px;border:1px solid lightgrey;" src="https://github.com/hq234/markdown_image/blob/main/theme1/avatar.jpg?raw=true" alt="avatar" />
</li>
{% endif %}
</ul>



## Skill Keywords

{% for skill in site.data.skills %}
### {{ skill.name }}
<div class="btn-inline">
{% for keyword in skill.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}

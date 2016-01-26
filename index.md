---
layout: home
---

<div class="index-content Spring">
    <div class="section">
        <ul class="artical-cate">
            <li class="on" style="text-align:left"><a href="/"><span>Spring</span></a></li>
        </ul>

        <div class="cate-bar"><span id="cateBar"></span></div>

        <ul class="artical-list">
        {% for post in site.categories.spring %}
            <li>
                <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
                <div class="title-desc">{{ post.description }}</div>
            </li>
        {% endfor %}
        </ul>
    </div>
    <div class="aside">
    </div>
</div>

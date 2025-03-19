---
layout: page
title: 代码覆盖率系列文章
titlebar: coverage
subtitle: <span class="mega-octicon octicon-flame"></span>&nbsp;&nbsp; 代码覆盖率系列教程
menu: coverage
css: ['blog-page.css']
permalink: /coverage
keywords: 代码覆盖率,JaCoCo
---

<div class="row">

    <div class="col-md-12">

        <ul id="posts-list">
            {% for post in site.posts %}
                {% if post.category=='coverage'  or post.keywords contains 'coverage' or post.keywords contains 'coverage' %}
                <li class="posts-list-item">
                    <div class="posts-content">
                        <span class="posts-list-meta">{{ post.date | date: "%Y-%m-%d" }}</span>
                        <a class="posts-list-name bubble-float-left" href="{{ site.url }}{{ post.url }}">{{ post.title }}</a>
                        <span class='circle'></span>
                    </div>
                </li>
                {% endif %}
            {% endfor %}
        </ul> 

        <!-- Pagination -->
        {% include pagination.html %}

        <!-- Comments -->
       <div class="comment">
         {% include comments.html %}
       </div>
    </div>

</div>
<script>
    $(document).ready(function(){

        // Enable bootstrap tooltip
        $("body").tooltip({ selector: '[data-toggle=tooltip]' });

    });
</script>
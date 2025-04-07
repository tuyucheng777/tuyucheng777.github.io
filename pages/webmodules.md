---
layout: page
title: Web框架系列文章
titlebar: webmodules
subtitle: <span class="mega-octicon octicon-flame"></span>&nbsp;&nbsp; Web框架系列教程
menu: webmodules
css: ['blog-page.css']
permalink: /webmodules
keywords: Jakarta EE、Ratpack、Jersey
---

<div class="row">

    <div class="col-md-12">

        <ul id="posts-list">
            {% for post in site.posts %}
                {% if post.category=='webmodules'  or post.keywords contains 'webmodules' or post.keywords contains 'webmodules' %}
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
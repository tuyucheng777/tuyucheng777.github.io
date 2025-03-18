---
layout: page
title: 自动化测试系列文章
titlebar: automatedtest
subtitle: <span class="mega-octicon octicon-flame"></span>&nbsp;&nbsp; 自动化测试系列教程
menu: automatedtest
css: ['blog-page.css']
permalink: /automatedtest
keywords: Selenium,Playwright,自动化测试
---

<div class="row">

    <div class="col-md-12">

        <ul id="posts-list">
            {% for post in site.posts %}
                {% if post.category=='automatedtest'  or post.keywords contains 'automatedtest' or post.keywords contains 'automatedtest' %}
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
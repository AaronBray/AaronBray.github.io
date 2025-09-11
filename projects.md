---
layout: page
title: projects
---

<div class="page-content wc-container">
	<div class="post">
		<h1>Tags</h1>  
		<ul>
			{% for tag in site.tags %}
			<li><a href="{{ '/projects/' | append:tag[0] | relative_url }}">{{ tag[0] }}</a></li>
			{% endfor %}
		</ul>
	</div>
</div>


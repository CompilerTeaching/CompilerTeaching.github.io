{% assign type = include.type %}

{% capture projects  %}
{% for project in site.projects %}
{% if true != project.done or project.done == nil %}
{% capture project_type %}{{ project.type }}{% endcapture %}
{% if project_type contains type %}
###{{ project.title }}
{% if project.prerequisites %}
<p><b>Prerequisites:</b> {{ project.prerequisites}}</p>
{% endif %}
{{ project.content }}
<hr />
{% endif %}
{% endif %}
{% endfor %}
{% endcapture %}
{% if projects.size > 20  %}
{% include contents.markdown %}
{{ projects }}
{% else %}
I currently have no {{ page.title}}.
{% endif %}


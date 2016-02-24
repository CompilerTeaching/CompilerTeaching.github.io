---
layout: default
title: People
published: false
---

{% comment %}
The people.html inclusion will list all of the people if no role is specified.  If you set the style to 'verbose' then it will put people one per line with description text alongside their photos, otherwise it will just show names and photos.
{% endcomment %}

{% include contents.markdown %}

Principal Investigators (verbose format)
----------------------------------------

{% include people.html role='PI' style='verbose' %}

Students (default format)
-------------------------

{% include people.html role='PhD' %}

Everyone (default format)
-------------------------

{% include people.html %}

Everyone (brief format)
-----------------------

{% include people.html style='brief' %}

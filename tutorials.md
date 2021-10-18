---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: page
title: Tutorials
permalink: /tutorials/
---

{% for tutorial in site.tutorials %}
  <h4><a href="{{tutorial.url}}">{{tutorial.title}}</a>
{% if tutorial.level == 0 %}
  ⚪⚪⚪⚪⚪ <span style="color:green">Beginner
{% elsif tutorial.level == 1 %}
  ⚫⚪⚪⚪⚪ <span style="color:green">Beginner
{% elsif tutorial.level == 2 %}
  ⚫⚫⚪⚪⚪ <span style="color:orange">Advanced
{% elsif tutorial.level == 3 %}
  ⚫⚫⚫⚪⚪ <span style="color:orange">Advanced
{% elsif tutorial.level == 4 %}
  ⚫⚫⚫⚫⚪ <span style="color:red">Expert
{% elsif tutorial.level == 5 %}
  ⚫⚫⚫⚫⚫ <span style="color:red">Expert
{% endif %} </span></h4>
  <ul>
  {% for lesson in tutorial.lessonsContent %}
    <li>{{ lesson }}</li>
  {% endfor %}
  </ul>
  <br />
{% endfor %}

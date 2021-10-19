---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: page
title: Tutorials
permalink: /tutorials/
---

<style>
  blockquote {
    color:black;
    background: #f9f9f9;
    font-style:normal;
    padding: 1em;
    -webkit-border-radius:10px;
    -moz-border-radius:10px;
    border-radius:10px;
    border:2px solid #a9a9a9;
  }
</style>

{% for tutorial in site.tutorials %}
  <blockquote>
  <h4><a href="{{tutorial.url}}">{{tutorial.title}}</a>
{% if tutorial.level == 0 %}
  ⚪⚪⚪⚪⚪ <span style="color:purple">All levels
{% elsif tutorial.level == 1 %}
  ⚫⚪⚪⚪⚪ <span style="color:green">Beginner
{% elsif tutorial.level == 2 %}
  ⚫⚫⚪⚪⚪ <span style="color:green">Beginner
{% elsif tutorial.level == 3 %}
  ⚫⚫⚫⚪⚪ <span style="color:orange">Advanced
{% elsif tutorial.level == 4 %}
  ⚫⚫⚫⚫⚪ <span style="color:orange">Advanced
{% elsif tutorial.level == 5 %}
  ⚫⚫⚫⚫⚫ <span style="color:red">Expert
{% endif %} </span></h4>
  <ul>
  {% for lesson in tutorial.lessonsContent %}
    <li>{{ lesson }}</li>
  {% endfor %}
  </ul>
  </blockquote>
{% endfor %}

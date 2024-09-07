---
layout: page
title: Lecture Notes
description: Lecture Notes written by the TAs!
has_children: true
---

# Lecture Notes

These lecture notes are written by the TAs. They aim to supplement and expand upon the course slides.

{% for lecture in site.lectures %}

- [{{lecture.title}}]({{site.baseurl}}{{ lecture.url }})

{% endfor %}

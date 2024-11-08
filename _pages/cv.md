---
layout: archive
title: "CV"
permalink: /cv/
author_profile: true
redirect_from:
  - /resume
---
{% include base_path %}

Education
=========

* PhD in Computer Science, National University of Singapore, 2020 - Current
* M.S. in Energy Systems, Skolkovo Institute of Science and Technology
* B.S. in Finance and Mathematics, New York University

Publications
============

<ul>{% for post in site.publications reversed %}
    {% include archive-single-cv.html %}
  {% endfor %}</ul>

Teaching
========

<ul>{% for post in site.teaching reversed %}
    {% include archive-single-cv.html %}
  {% endfor %}</ul>

Service
=======

* SIGPLAN Long-Term Mentoring Committee operation team, Nov 2022 - Current
* NUS School of Computing Graduate Student Association of Computing, Jan 2021 - May 2024
* Artifact evaluation: APLAS 2022, VMCAI 2021
* Student volunteering: ESEC/FSE 2022, ICFP 2020

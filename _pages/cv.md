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
======
* BS in Computer Science, Texas State University, 2006
* MS in Computer Science, The University of Texas at Dallas, 2008
* PhD in Computer Science, The University of Texas at Dallas, 2010

Work experience
======
* 2018 - Present
  The University of West Florida
  * Assistant Professor of Computer Science

* 2015 - Present
  * First Principle Innovators
  * Chief Architect

* 1996 - 2017
	* US Navy/Navy Reserve
	* Submarine warfare, Navy diver (SCUBA), and Information Dominance Warfare Officer qualified
	* Retired LT

Publications
======
  <ul>{% for post in site.publications %}
    {% include archive-single-cv.html %}
  {% endfor %}</ul>

Talks
======
  <ul>{% for post in site.talks %}
    {% include archive-single-talk-cv.html %}
  {% endfor %}</ul>

Teaching
======
  <ul>{% for post in site.teaching %}
    {% include archive-single-cv.html %}
  {% endfor %}</ul>

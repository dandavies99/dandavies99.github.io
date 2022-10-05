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

* PhD in Sustainable Chemical Technologies, University of Bath, 2018
* MRes in Sustainable Chemical Technologies, University of Bath, 2015
* MChem in Chemistry, University of Bath, 2014

Work experience
======

* 2021-present: Imperial College London (Senior Research Software Engineer)
  * Developing software to enable research across a range of disciplines
  * Scoping and writing proposals for new software projects
  * Teaching graduate students about software development and version control

* 2019-2021: University College London (Postdoctoral Research Fellow)
  * Investigating mixed-anion compounds for energy generation
  * Writing software and machine learning models for fast materials discovery

* 2018-2019: Imperial College London (EPSRC Doctoral Prize Fellowship)
  * Developed data-driven materials tools in python to design better battery cathodes
  * Investigated anion redox activity in battery cathodes

* April 2018: Lawrence Berkeley National Laboratory, CA (postgraduate placement)
  * Applied machine learning techniques to large materials databases

* 2012-2013: Merk KGaA, Southampton (undergraduate placement)
  * Fabrication of nanowire-based biosensors and organic infrared sensors

Skills
======

* Software engineering
  * Expert in Python
  * Webapp development with Django and Flask
  * Git version control
  * Unit tests, continuous integration and deployment
  * Containerisation with Docker
  * Cloud computing on Azure
  * Experience with FORTRAN and Julia
* Machine learning and data science
  * Pandas, Scikit-learn, MongoDB and SQL (postgres, sqlite, timescaledb), TensorFlow
  * Implement supervised learning models such as random forest, GBR and SVM
* Agile
  * Experience working in an agile environment
  * Working knowledge of Scrum and Kanban
  * Use of Jira, Aha! and GitHub Projects
* Chemistry
  * Expert user of quantum chemistry codes VASP and GPAW
  * Manage automated calculations with Fireworks and Atomate
  * Developer of SMACT code and expert user of Pymatgen, Matminer, ASE and OpenBabel
* Writing
  * Lead author on publications in high-impact journals
  * Experience writing proposals for HPC access at national (tier 1) and international (tier 0) levels
  * Regularly write proposals and progress reports overseeing research software projects

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
  
Service and leadership
======

* Member of Royal Society of Chemistry and part of Solid State Chemistry subdivision
* Reviewer for various journals including Materials Horizons, Chemical Science, Machine Learning: Science and Technology and Journal of Chemical Physics

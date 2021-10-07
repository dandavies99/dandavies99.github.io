---
permalink: /
title: "About me"
excerpt: "About me"
author_profile: true
redirect_from: 
  - /about/
  - /about.html
---

I'm a senior research software engineer based within the [Research Computing Service](https://www.imperial.ac.uk/admin-services/ict/self-service/research-support/rcs/) of Imperial College London. I work with researchers from across the college to produce performant and reliable software solutions to cutting-edge research challenges. I joined the team in August 2021 and have so far worked on a range of projects using [Python GUIs](https://github.com/ImperialCollegeLondon/python-gui-template) and the [Django](https://www.djangoproject.com/) Python web framework.

Software
======

A lot of my work has involved developing codes that can be used in computational materials discovery. I'm a lead developer of [SMACT](https://github.com/wmd-group/smact) (Semiconducting Materials by Analogy and Chemical Thoery) which aides in the taming of the vast search space for high-order inorganic materials. 

More recently, I have been involved in the development of [Thermoplotter](https://github.com/SMTG-UCL/ThermoPlotter) and [Surfaxe](https://github.com/SMTG-UCL/Surfaxe), which are designed to simplify what are traditionally complicated and error-prone sets of calculations and analysis. 



Research Background
======

My background is in computational chemistry. I completed an MChem at the University of Bath and stayed on to carry out an MRes, then PhD on [_"Materials discovery using chemical heuristics and high-throughput calculations"_](https://dandavies99.github.io/files/my_thesis.pdf) under the supervision of [Prof. Aron Walsh](https://wmd-group.github.io). I was awarded a [Doctoral Prize Fellowship](https://epsrc.ukri.org/skills/students/dta/doctoralprize/) at Imperial College London before moving to the chemistry department of UCL within the group of [Prof. David Scanlon](http://davidscanlon.com).

Searching the materials hyperspace
------

My research focused on the discovery of new inorganic materials for clean energy. Application areas included solar absorbers, transparent conductors, thermoelectrics, photocatalysts and battery cathodes. 

A typical computational materials screening process is: **1.** generate a sensible, and often very large, search space of hypothetical compositions or compounds (see the [SMACT](https://github.com/wmd-group/smact) code) **2.** use chemical heuristics, machine learning, or other data-driven filters to significantly narrow down the search space **3.** subject top candidates to accurate first-principles methods to predict their properties. I developed models and tools that help with all three steps. 

Materials data and machine learning
---

Trying to predict stable compounds with target properties using existing materials data is an enjoyable challenge. Traditionally, this has been done using chemical heuristic, which certainly [come in useful](https://dandavies99.github.io/publication/2018_chemsci), but [more recently](https://dandavies99.github.io/publication/2019_chemmater) I have relied more upon supervised machine learning models.

A combination of python packages (pandas, scikit-learn, pymatgen, smact...) and a selection of queryable databases of materials properties (The Materials Project, The Computational Materials Repository...) usually get the job done. I have also been fortunate to collaborate with experimental groups who can make our imagined compounds with awesome properties a reality. 

For a wider overview, you can read our quick-start guide on [_Machine Learning for Molecular and Materials Science_](https://dandavies99.github.io/publication/2018_nature), which appeared in Nature in 2018. 

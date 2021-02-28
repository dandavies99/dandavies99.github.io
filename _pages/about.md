---
permalink: /
title: "About me"
excerpt: "About me"
author_profile: true
redirect_from: 
  - /about/
  - /about.html
---

My name is Dan and I'm a computational materials chemist, currently based in the chemistry department of UCL within the group of [Prof. David Scanlon](http://davidscanlon.com). I completed a Chemistry MRes at the University of Bath, then stayed on to carry out my PhD on [_"Materials discovery using chemical heuristics and high-throughput calculations"_](https://dandavies99.github.io/files/my_thesis.pdf) under the supervision of [Prof. Aron Walsh](https://wmd-group.github.io). I was then awarded a [Doctoral Prize Fellowship](https://epsrc.ukri.org/skills/students/dta/doctoralprize/) at Imperial College London. 

About my research
======

Searching the materials hyperspace
------

My research focuses on the discovery of new inorganic materials for clean energy. Application areas have so far included solar absorbers, transparent conductors, thermoelectrics, photocatalysts and battery cathodes. 

The workflow of a typical study is often something like: **1.** generate a sensible, and often very large, search space of hypothetical compositions or compounds (see the [SMACT](https://github.com/wmd-group/smact) code) **2.** Use chemical heuristics, machine learning, or other data-driven filters to significantly narrow down the search space **3.** subject top candidates to accurate first-principles methods to predict their properties.

Software
-----

A lot of my work has involved developing codes that can be used in the computational materials discovery process. I'm a lead developer of [SMACT](https://github.com/wmd-group/smact) (Semiconducting Materials by Analogy and Chemical Thoery) which aides in the taming of the vast search space for high-order inorganic materials. 

More recently, I have been involved in the development of [Thermoplotter](https://github.com/SMTG-UCL/ThermoPlotter) and [Surfaxe](https://github.com/SMTG-UCL/Surfaxe), which are designed to simplify what are traditionally complicated and error-prone calculations and analysis.  

Materials data and machine learning
---

The challenge I find most interesting is trying to predict stable compounds with target properties using existing materials data. Traditionally, this has been done using chemical heuristic, which certainly [come in useful](https://dandavies99.github.io/publication/2018_chemsci), but [more recently](https://dandavies99.github.io/publication/2019_chemmater) I have relied more upon supervised machine learning models.

A combination of python packages (pandas, scikit-learn, pymatgen, smact...) and a selection of queryable databases of materials properties (The Materials Project, The Computational Materials Repository...) usually get the job done. I have also been fortunate to collaborate with many experimental groups who can make our imagined compounds with awesome properties a reality. 

For a wider overview, you can read our quick-start guide on [_Machine Learning for Molecular and Materials Science_](https://dandavies99.github.io/publication/2018_nature), which appeared in Nature in 2018. 



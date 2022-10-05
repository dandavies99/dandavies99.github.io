---
title: "Surfaxe: Automated First-Principles Calcualtions of Surface Properties"
excerpt: "A package to assist with reliable and systematic calcualtions of surface properties <br/><img src='/images/surfaxe_header.png' width=100>"
collection: portfolio
---

Calculating surface properties of materials using density functional theory (DFT) is significantly more challenging (or at least a lot more fiddly) than calculating bulk properties. Extra considerations such as slab thickness, amount of vacuum, surface termination and surface dipole all need to factored in to get reliable answers.

[Surfaxe](https://github.com/SMTG-UCL/surfaxe) is designed to simplify this process, and contains modules for generating surfaces automatically, anlysing raw data from DFT calculations, and producing clear plots of results. It is reliant on the [pymatgen](https://pymatgen.org/) package for generating slabs and reading/writing calculation input/output files.

Take a look at the docs [here](https://surfaxe.readthedocs.io/en/latest/).

![surfaxe example](/images/bond_analysis_plot.png)

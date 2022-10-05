---
title: "Paricia: Hydroclimatic Data Management Platform" 
excerpt: "A Django webapp to manage hydroclimatic time-series data <br/><center><img src='/images/paricia-schema.png' width=400></center>"
collection: portfolio
---

[Paricia](https://github.com/ImperialCollegeLondon/paricia) is a Django webapp that allows users to upload and store hydroclimatic time-series data, and then perform analysis on the data. It is currently being developed in collaboration with the Imperial College London [Department of Civil and Environmental Engineering](https://www.imperial.ac.uk/civil-engineering) and the group of [Prof. Wouter Buytaert](https://www.imperial.ac.uk/people/w.buytaert) who is collecting data from monitoring stations in the Andes and Himalayas.

The webapp leverages the [TimescaleDB](https://www.timescale.com/) extension to Postgres for efficent handling of time-series data. A React frontend to provide a user-friendly interface for data upload and analysis is currently in development in collaboration with [Kathmandu Living Labs](https://www.kathmandulivinglabs.org/).

![Paricia schema](/images/paricia-schema.png)

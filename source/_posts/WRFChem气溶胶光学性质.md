---
title: WRFChem气溶胶光学性质
date: 2022-10-25 09:18:14
toc: true
tags: WRF-Chem
categories:
  - model
excerpt: WRF/Chem 模式，气溶胶方案，光学性质模拟检验
---

# WRFChem中的气溶胶方案

- sectional approach: ```MOSAIC``` (Model for Simulating Aerosol Interactions and Chemistry)
- modal approach: ```MADE``` (Modal Aerosol Dynamics Model for Europe), ```MAM``` (Modal Aerosol Model from CAM5)
- bulk approach: ```GOCART``` (Goddard Global Ozone Chemistry Aerosol Radiation and Transport model)

# 不同方案的验证

- **MOSAIC**
  - ***Barnard et al. (2010) and Lennartson et al. (2018)***  analyze the diurnal variation of AOD
  - ***Ghan et al. (2001)***  evaluated the radiative impact of including coupled aerosol–cloud–radiation processes.
  - Other studies assessing the representation of aerosol optical properties and their uncertainties using MOSAIC together with other schemes, mainly MADE ***Zhao et al., 2010, 2011, 2013; Balzarini et al., 2015; Yang et al., 2018; Saide et al., 2020***
- **GOCART**
  - **LeGrand et al., 2019** compared the Air Force Weather Agency (AFWA) dust emission scheme within GOCART to other dust emission schemes available in WRF-Chem and their skills for representing AOD
  -  In this contribution, the need for tuning the model in order to get a reasonable simulation of AOD for each location and/or event was pointed out based on the results of Bian et al. (2011), Dipu et al. (2013), Kumar et al. (2014), Jish Prakash et al. (2015), Zhang et al. (2015), Kalenderski and Stenchikov (2016), and Hu et al. (2020), among others. 


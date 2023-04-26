---
title: WRF/Chem模式运行说明
date: 2022-10-25 09:18:14
toc: true
tags: WRF-Chem
categories:
  - model
excerpt: WRF/Chem 模式运行流程
---

# 目录说明

- ```shell
  程序和脚本在 /home/xdxie/data/wrfchemdata/beijing201612
  只需要复制 presource  scripts  WPS WRFChem-3.9.1-revised这几个文件夹
  
  WPS/WRFChem-3.9.1-revised: 主程序
  （WRF文件夹是WRF391 default版本，WRFChem-3.9.1-revised是WRF391李楠组版本，修改了SO4/GLY，见Li et al, EI，2022）
  
  scripts：运行脚本
  
  presource：运行前准备程序和文件
  	presource/domain：地形文件和namelist文件，需要在运行之前生成geo文件，并将相关文件复制到该目录
  	presource/*emiss：生成排放文件的程序
  	presource/mozbc：生成边界条件的程序
  
  data：结果数据文件
  	data/camchem：全球模式提供边界条件（需要自己下载对应的模拟时段数据，见下面说明）
  	data下其他文件夹为对应case的结果文件,以basecase为例，包括
  		data/basecase/predata：排放、fnl链接（需要手动创建fnl目录，并将需要的fnl数据链接到该目录下）
  		data/basecase/runwrfchem：wrf运行目录
  		data/basecase/combine：处理后的wrfout文件
          data/basecase/results：提取的文本文件
          data/basecase/figures：出图文件
  ```

# 运行流程

- 第一次运行需要设置domain和生成geo_em文件，WRF培训时有介绍

- 之后的所有运行步骤都在autorun.wrfchem.sh这个脚本中

  ```shell
  step1_wps: 运行ungrib.exe和metgrid.exe
  step2_real_w_o_chem: 运行不带化学过程的real.exe,这一步namelist中chem_opt要设置为0
  step3_emission： 生成排放，这一步中人为排放部分需要在admin上运行，需要自己安装matlab
  step4_real_w_chem： 运行带化学过程的real.exe
  step5_mozbc: 使用camchem数据添加边界文件
  step6_wrf： 运行wrf.exe
  step7_wrf_from_existing_case：当step1-5不需要重新运行时，快速开始其他case的运行
  
  ```

  

# 注意事项

- 下载camchem数据，路径```https://www.acom.ucar.edu/cam-chem/cam-chem.shtml```， 根据你的研究区域和时间下载对应文件，如```lat,10.0,60.0 lon,60.0,160.0 ```
- ```chem_opt=203: SAPRC99_MOSAIC_8BIN_VBS2_AQ_KPP; SAPRC99 Chemistry with MOSAIC using KPP library. The MOSAIC aerosols uses 8 sectional aerosol bins and includes volatility basis set (VBS) for organic aerosol evolution. Also include aqueous phase chemistry.``` 在李楠的版本中，该选项对应MOSAIC_4bin
- 安装时候的问题：configure之后需要修改下configure.wrf文件，mpif90改成mpiifort
- 处理排放问题：人为排放处理需要matlab，必须在admin上运行
- 边界条件处理，mozbc，目前使用camchem， domain1：ic、bc都需要，其他domain，只需要ic， 运行wrf.exe时必须设置```have_bcs_chem = .true. ```




---
title: CMAQ代码说明
date: 2022-10-25 09:18:14
toc: true
tags: CMAQ
categories:
  - model
excerpt: CMAQ代码说明，代码结构分析
---



## 代码结构分析

#### drive.F 主程序

1. 一些数组说明：
   1. CGRID,指针，指向PCGRID，所有物种瞬时浓度的存储数组
   2. AGRID平均浓度的数组，输出时在wr_aconc.F文件中判断是否是输出物种和输出层数
   3. SGRID输出到CONC文件的瞬时物种浓度的数组，即CGRID的子集
2. 依次运行的模块
   1. GRID_INIT()：网格定义
   2. CGRID_SPCS_INIT()：物种定义
   3. PAGRD_INIT()：PA模块网格定义
   4. PCGRID_INIT()：定义物种PCGRID数组 
   5. CONC_DEFN() | A_CONC_DEFN()：获取输出的CONC、ACONC文件对应的物种名称
   6. WVEL_INIT()：读取环境变量“CTM_WVEL” ，是否输出沉降速度
   7. INITSCEN()：输出IC到CONC文件
   8. PA_INIT()：PA模块初始化，创建输出文件并初始化数组
   9. FLCHECK()：文件header信息检查
   10. 分配相关数组的内存：AGRID、RCONC、CFLAG、ASTEP
   11. ADVSTEP()：计算时步
   12. SCIPROC()：理化过程计算程序
3. 调用的子程序
   1. GRID_CONF.F  网格设置，会调用HGRD_DEFN.F和VGRD_DEFN.F， 其中，HGRD读取"GRIDDESC"文件，VGRD读取MET_CRO_3D文件中的垂直信息
   2. CGRID_SPCS.F 物种设置，读取GC_nml，AE_nml等物种定义文件
   3. PAGRD_DEFN.F 读取环境变量中的一些设置，PA模块有关
   4. PCGRID_DEFN.F 定义物种数组，PCGRID, 这是CGRID指针的目标，即CGRID指向PCGRID
   5. STD_CONC.F、AVG_CONC.F  定义物种名称CONC_SPCS、AVG_SPCS
   6. initscen.F 输出IC到CONC文件
   7. pa_init.F、flcheck.F
   8. advstep.F 根据CFD条件（Courant-number conditions）计算时步，TSTEP(2)指运行的时间步长，TSTEP(1)指输出的时间步长（run脚本设置，在initscen.F读取），NREPS指输出时间步长中的时步数量，NREPS=TSTEP(1)/TSTEP(2)
   9. sciproc.F 理化过程

#### SCIPROC程序说明

##### CHEM: grdriver.F

1. 第一次运行需要smvgear初始化 GRVARS_INIT、GRINIT、JSPARSE
2. 排放：默认不需要
3. **定义MET_CRO_3D文件的网格的起始位置 （注意boxmodel待修改） SUBHFILE(MET_CRO_3D)**
4. FIND_DEGRADED：确定degradation（降解）的物种
5. **从气象文件中读取TA, QV, DENS, PRES**
6. **PHOT.F 计算光解速率-这里会读取气象数据**, 主要流程：
   1. 初次运行：
      1. SUBHFILE: GRID_CRO_2D，MET_CRO_2D，MET_CRO_3D
      2. LOAD_REF_DATA()：读取CSQY_DATA文件中的数据
      3. 从GRID_CRO_2D读取纬度 LAT
      4. 判断landuse类型，计算FRACTION_LANDUSE；判断是否有SEAICE变量；定义一些数组
      5. 读取经度LON, 计算CONLAT, SINLAT
      6. 读取地面高度HT
      7. O3和NO2指针
   2. 读取一些变量：SNOCOV (snow cover)
   3. 太阳相关参数计算，输入经纬度、年份和天，SOLEFM3
   4. 计算太阳高度角：GETZEN2
   5. 判断是否是夜晚 DARK
   6. 读取变量：ZH、ZF、DENS、TA、PRES、WBAR、CLDT、CLDB、CFRAC
   7. AERO_PHOTDATA：计算气溶胶特性
   8. NEW_OPTICS：计算晴空光解速率
   9. NEW_OPTICS：计算有云情况下光解速率
7. 进入block循环：
   1. PA_IRR_CKBLK、PA_IRR_BLKSTRT
   2. CALCKS：计算反应速率
   3. INIT_DEGRADE
   4. SMVGEAR：化学模块积分
   5. FINAL_DEGRADE

##### AERO: aero_driver.F

1. 第一次运行，初始化：
   1. **打开气象文件**
   2. 打开能见度文件 OPVIS: opvis.F
   3. 打开气溶胶参数(粒径和标准差)文件OPDIAM: opdiam.F
   4. **定义MET_CRO_3D文件的网格的起始位置 （注意boxmodel待修改） SUBHFILE(MET_CRO_3D)**
2. **读取气象：PRES、TA、QV、DENS、GSW**
3. 进入格点循环：
   1. 从CGRID数组中提取相关数据：EXTRACT_AERO、EXTRACT_PRECURSOR、EXTRACT_SOA
   2. **计算气溶胶过程 AEROPROC： aero_subs.F**
      1. GETPAR: getpar.f 计算M3, mass, 气溶胶密度，直径
      2. ORGAER： SOA传统生成路径， SOA_DEFN.F
      3. POAAGE：POA氧化，poaage.F
      4. HETCHEM：非均相反应（N2O5, sulfate, GLY, MGLY），hetchem.f
      5. GETPAR
      6. VOLINORG：NPF和SNA+Cl的气粒转化
      7. coagulation：GETCOAGS
      8. GETPAR
   3. 更新CGRID数组：UPDATE_AERO、UPDATE_PRECURSOR、UPDATE_SOA
   4. 计算每个模态中PM2.5的占比：INLET25： aero_subs.F
   5. 输出：
      1. 计算能见度相关 GETVISBY：aero_subs.F
      2. 计算不同模态 M2 and M3值
      3. 粒径分布 计算直径和标准差 GETPAR
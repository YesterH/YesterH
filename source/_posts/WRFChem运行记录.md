---
title: WRF/Chem模式运行记录
date: 2022-10-25 09:18:14
toc: true
tags: WRF-Chem
categories:
  - model
excerpt: 黑碳混合态个例分析WRF/Chem 模式运行记录（20161211~20161219）
---

## 运行历史记录

1. 第一次运行
   - 参数化方案选择组里常用，和Li et al., 2023, EI一致
   - chembdy没有打开，只有初始场
   - d02模拟结果和观测对比有偏差，后面几天低估明显

<img src="WRFChem运行记录/image-20230403094932996.png" alt="image-20230403094932996" style="zoom: 80%;" />

2. 第二次运行(调整不同参数)
   - 改变模型版本（使用revised版本,copy from linan）,其他不变（basecase_RM）
     - 修改模型版本后，SO4, nh4, SOA浓度增加，其他组分不变
     - <img src="WRFChem运行记录/image-20230403214043383.png" alt="image-20230403214043383" style="zoom:80%;" />
     - ![image-20230403214630526](WRFChem运行记录/image-20230403214630526.png)
   - chembdy打开(have_bcs_chem=.ture.)、obsgrid打开（basecase_OBS）
   - 上两种改变同时进行（basecase_RMOBS）
3. 存在问题：
   - 气象模拟不对，温度和RH偏差很大
     - ![image-20230404112534091](WRFChem运行记录/image-20230404112534091.png)
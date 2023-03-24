---
title: WRF/Chem
date: 2022-10-25 09:18:14
toc: true
tags: WRF
categories:
  - model
excerpt: WRF/Chem 模式
---

# 安装

- 问题1：configure之后需要修改下configure.wrf文件，mpif90改成mpiifort
- 

# 运行

## 排放处理

- 关于wildfire排放

  - 在```Registry/registry.chem```中，有物种定义
  - ```package   biomassb      biomass_burn_opt==1             -          state:mean_fct_agtf,mean_fct_agef,mean_fct_agsv,mean_fct_aggr,firesize_agtf,firesize_agef,firesize_agsv,firesize_aggr;ebu:ebu_no,ebu_no2,ebu_co,ebu_co2,ebu_eth,ebu_hc3,ebu_hc5,ebu_hc8,ebu_ete,ebu_olt,ebu_oli,ebu_pm25,ebu_pm10,ebu_dien,ebu_iso,ebu_api,ebu_lim,ebu_tol,ebu_csl,ebu_hcho,ebu_ald,ebu_ket,ebu_macr,ebu_ora1,ebu_ora2,ebu_so2,ebu_nh3,ebu_oc,ebu_bc,ebu_sulf,ebu_dms;ebu_in:ebu_in_no,ebu_in_no2,ebu_in_co,ebu_in_co2,ebu_in_eth,ebu_in_hc3,ebu_in_hc5,ebu_in_hc8,ebu_in_ete,ebu_in_olt,ebu_in_oli,ebu_in_pm25,ebu_in_pm10,ebu_in_dien,ebu_in_iso,ebu_in_api,ebu_in_lim,ebu_in_tol,ebu_in_csl,ebu_in_hcho,ebu_in_ald,ebu_in_xyl,ebu_in_ket,ebu_in_macr,ebu_in_ora1,ebu_in_ora2,ebu_in_so2,ebu_in_nh3,ebu_in_oc,ebu_in_bc,ebu_in_sulf,ebu_in_dms,ebu_in_ash```
  - ```ebu```表示最终模式需要的物种；```ebu_in``` 表示排放文件中的物种（```wrffirechemi_d01```文件中，物种名称为```ebu_in_no```）

- 数据处理流程

  1. 数据读取程序为``` share/mediation_integrate.F ```, 数据读到```ebu_in```数组中（此为推测），如下：

  - ```fortran
    	CALL med_auxinput_in ( grid, ialarm, config_flags )
        WRITE ( message , FMT='(A,i3,A,i3)') 'Input data processed for aux input ',&
        	ialarm - first_auxinput + 1, ' for domain ',grid%id
    ```

  2. 在```chem/emissions_driver.F```程序中调用 ```do_plumerisefire```将```ebu_in```赋值给```ebu```, 

     - 涉及子程序```chem/module_plumerise1.f90```
     - ```do_plumerisefire```是根据```namelist.input```中的```plumerisefire_frq```来设置的，每隔一段时间运行一次

     - ```fortran
              if( do_plumerisefire )then
                 CALL wrf_debug(15,'fire emissions: calling biomassb')
                 write(0,*)ktau,stepfirepl
                 call plumerise_driver (id,ktau,dtstep,                           &
                  ebu,ebu_in,                                                     &
                  mean_fct_agtf,mean_fct_agef,mean_fct_agsv,mean_fct_aggr,        &
                  firesize_agtf,firesize_agef,firesize_agsv,firesize_aggr,        &
                  config_flags, t_phy,moist,                                      &
                  rho_phy,vvel,u_phy,v_phy,p_phy,                                 &
                  emis_ant,z_at_w,z,config_flags%scale_fire_emiss,                &
                  ids,ide, jds,jde, kds,kde,                                      &
                  ims,ime, jms,jme, kms,kme,                                      &
                  its,ite, jts,jte, kts,kte                                       )
               endif
       ```

  3. 在```chem/emissions_driver.F```程序中调用 ```add_emis_burn```：

     - ```fortran
            CASE (BIOMASSB,BIOMASSB_MOZC,BIOMASSB_MOZ,BIOMASSB_GHG)
              CALL wrf_debug(15,'fire emissions: adding biomassb emissions')
              call add_emis_burn(id,dtstep,ktau,dz8w,rho_phy,chem,&
                   julday,gmt,xlat,xlong,t_phy,p_phy,                           &
                   ebu,config_flags%chem_opt,0,config_flags%biomass_burn_opt,     &
                   num_chem,ids,ide, jds,jde, kds,kde,                                   &
                   ims,ime, jms,jme, kms,kme,                                   &
                   its,ite, jts,jte, kts,kte                                    )
         
       ```

     - 注意：```chem/module_add_emiss_burn.F```中将排放中的物种分配给不同的化学机制

     - ```fortran
             emiss_select:  SELECT CASE(chem_opt)
             CASE (RADM2,RACM_KPP,RACM_MIM_KPP,SAPRC99_KPP,SAPRC99_MOSAIC_4BIN_VBS2_KPP, &
                  SAPRC99_MOSAIC_8BIN_VBS2_AQ_KPP, SAPRC99_MOSAIC_8BIN_VBS2_KPP)
                 do j=jts,jte
                 do i=its,ite
                  do k=kts,kte
               conv_rho=r_q*4.828e-4/rho_phy(i,k,j)*dtstep/60./dz8w(i,k,j)
               chem(i,k,j,p_csl)  =  chem(i,k,j,p_csl)                        &
                                +ebu(i,k,j,p_ebu_csl)*conv_rho
               chem(i,k,j,p_iso)  = chem(i,k,j,p_iso)                         &
                                +ebu(i,k,j,p_ebu_iso)*conv_rho
               chem(i,k,j,p_no)   = chem(i,k,j,p_no)                          &
                                +ebu(i,k,j,p_ebu_no)*conv_rho
         
       ```

  4. 注意，在```emissions_driver.F```中，由于不同化学机制的物种不一样，需要对读入的排放数据进行匹配（mapping），

  5. 存在问题，```saprc99```机制中并没有正确读入野火排放数据，```VOC```的mapping是和其他机制一样，比如```chem/module_add_emiss_burn.F```文件中，```saprc99```机制中并没有```HC3```物种，只有```ALK```，所以```p_hc3=1```,读入的```ebu(p_ebu_hc3)```排放赋值给了一个不存在的物种！！！
  
  6. 需要修改，参考```Simulating Air Pollution in the Severe Fires Event during 2015 El-Niño in Indonesia using WRF-Chem```

# MOSAIC气溶胶模块

- mosaic bin粒径范围设置, 在```module_mosaic_driver.F```程序中

  - ```fortran
    !
    !   set section size arrays
    !
            do itype = 1, ntype_aer
          nhi = nsize_aer(itype)
          dlo_sect(1,itype) = 3.90625e-6
          dhi_sect(nhi,itype) = 10.0e-4
    
          dum = log( dhi_sect(nhi,itype)/dlo_sect(1,itype) ) / nhi !replaced alog by log by Manish Shrivastava on 11/28/2011. alog denoted natural log in fortran 77. log(x) is natural log in fortran 90
          do n = 2, nhi
        dlo_sect(n,itype) = dlo_sect(1,itype) * exp( (n-1)*dum )
        dhi_sect(n-1,itype) = dlo_sect(n,itype)
          end do
          do n = 1, nhi
        dcen_sect(n,itype) = sqrt( dlo_sect(n,itype)*dhi_sect(n,itype) )
        volumlo_sect(n,itype) = (pi/6.) * (dlo_sect(n,itype)**3)
        volumhi_sect(n,itype) = (pi/6.) * (dhi_sect(n,itype)**3)
        volumcen_sect(n,itype) = (pi/6.) * (dcen_sect(n,itype)**3)
        sigmag_aer(n,itype) = (dhi_sect(n,itype)/dlo_sect(n,itype))**0.289
          end do
      end do
    
    ```

- 简化测试8bin的结果如下：

  - ```fortran
          program main 
          real dlo_sect(8)
          real dhi_sect(8)
          integer nhi
          nhi = 8
          dlo_sect(1) = 3.90625e-6
          dhi_sect(nhi) = 10.0e-4
          dum = log( dhi_sect(nhi)/dlo_sect(1) ) / nhi 
          do n = 2, nhi
            dlo_sect(n) = dlo_sect(1) * exp( (n-1)*dum )
            dhi_sect(n-1) = dlo_sect(n)
          end do
          print*,dlo_sect
          print*,dhi_sect
          end
          
    ! 输出为
    3.9062502E-06  7.8125004E-06  1.5625001E-05  3.1250001E-05  6.2500003E-05  1.2500001E-04  2.5000001E-04  5.0000002E-04
    7.8125004E-06  1.5625001E-05  3.1250001E-05  6.2500003E-05  1.2500001E-04  2.5000001E-04  5.0000002E-04  1.0000000E-03
    ```

  - 

  - ![image-20221213194516063](WRFChem/image-20221213194516063.png)

- saprc99_mosaic_8bin_vbs2_aq (chem_opt=203)说明

  - 参考文献```Shrivastava, M., Fast, J., Easter, R., Gustafson Jr, W. I., Zaveri, R. A., Jimenez, J. L., Saide, P., and Hodzic, A.: Modeling organic aerosols in a megacity: comparison of simple and complex representations of the volatility basis set approach, Atmos. Chem. Phys., 11, 6639-6662, 10.5194/acp-11-6639-2011, 2011.```

  - **SAPRC-99 mechanism** with **The 9-species VBS mechanism** / **highly condensed 2-species SOA mechanism**

  - **POA**的计算

    -  The POA species are segregated by two emissions sectors: biomass burning and anthropogenic (predominately fossil fuel)
    -  To allow calculating O:C ratios for the modeled OA, separate model species are used for the oxygen and non-oxygen (C, H, N) components of each species.
    - POA(a)*i,e,x,n* = aerosol-phase POA, where i is the volatility species (1–9), e is either the biomass or anthropogenic emission sector, x is either the oxygen or non-oxygen component, and n is the size bin (1–4).
    - 模式中的POA物种名如，pcg1_b_c、pcg2_f_c

  - **SOA**的计算

    - SOA包括两个部分

      - SI-SOA: formation from S/IVOC, 模式中物种名和POA对应，opcg1_b_c、opcg2_f_c
        - ![image-20221213202231961](WRFChem/image-20221213202231961.png)
      - V-SOA: formation from trantional VOCs,模式中物种名分别为ant1_c和biog1_c，分别表示来自人为排放VOC和自然排放VOC
        - 可以从KPP中的mechanism文件中看出saprc99_mosaic_8bin_vbs2_aq.eqn
        - ![image-20221213202420876](WRFChem/image-20221213202420876.png)

    - WRF-Chem中使用的实际上是**SAPRC-99 mechanism** with **highly condensed 2-species SOA mechanism**，只考虑了2种挥发性的POA，即 

      $C^∗\ values\ (at\ 298\ K\ and\ 1\ atm)\ of\ 10^{−2}\ and\ 10^5\ µg\ m^{−3} $

      -  there are 40 POA species (8 gas, 32 aerosol), 20 SI-SOA species (4 gas, 16 aerosol), and 10 V-SOA species (2 gas, 8 aerosol) （Shrivastava的文章中使用的是4bin的MOSAIC方案）
      - 其中POA中POA(a)*i* = 2*,e,x* would almost entirely remain in the gas phase under most atmospheric conditions due to its high volatility，因此模式只输出16个aerosol物种
      - **注意**： 在8binMOSAIC方案中，V-SOA只输出了4个bin（共8 aerosol，同4bin方案），**其他bin的变量没有输出，在register.chem中有定义**
      - 

      

# 气溶胶光学性质的计算

## 总体思路（以MOSAIC方案为例）

- 相同bin中内混，不同bin之间外混；对特定的bin[i]：
  1. 计算各组分质量$M_{i,j}$和数浓度$N_i$
  2. 根据质量和数浓度计算体积$V_{i,j}$和直径$D_{p,i}$
  3. 计算负折射指数(这里会根据混合方式的不同[表现在模式中即option_method]分别计算)：
     1. volume averaging approximation: 总的负折射率等于质量加权平均
     2. Maxwell-Garnett approximation：根据Maxwell-Garnett计算
     3. shell-core mixing：分别计算核和壳的负折射指数
  4. 调用mie散射算法计算总的吸收效率、散射效率和不对称因子（这里会根据option_mie选项有两种方法）：
     1. option_mie=1  切比雪夫多项式近似... Chebyshev economization 即简化了Mie散射计算过程
     2. option_mie=2 完整的Mie散射算法

## 程序中细节说明

- 气溶胶光学性质参数的计算程序

  - optical_driver.F 
  - module_optical_averaging.F 
  - 重要的子程序 

  1. subroutine optical_prep_sectional， 准备Mie散射计算需要的相关数据

     - 根据气溶胶组分数据计算下列变量：

     - ```fortran
       radius_core,radius_wet, number_bin,                     &
       swrefindx,swrefindx_core,swrefindx_shell,               &
       lwrefindx,lwrefindx_core,lwrefindx_shell
       
       !	number_bin_col(nbin_a,kmaxd) --   number density in layer, #/cm^3
       !	radius_wet_col(nbin_a,kmaxd) -- wet radius, shell, cm
       !       radius_core_col(nbin_a,kmaxd) -- core radius, cm; NOTE:
       ```

     - **如果需要修改Mie散射计算结果，可以在此处增加相关变量**

  2. subroutine optical_averaging ，其中，会调用

     - optical_prep_sectional 子程序： 计算*3-D arrays for refractive index, wet radius, and aerosol number then passed into mieaer_sectional*
     - mieaer程序： *produces vertical profiles of aerosol optical properties for 4 wavelengths that are put into 3-D arrays and passed back up to chem_driver.F*，其中

     - *tauaer\*, waer\*, gaer\* passed to module_ra_gsfcsw.F*

     - *tauaer\*, waer\*, gaer\*, l2-l7 passed to module_phot_fastj.F*

- 输出变量说明：

  - ```fortran
    ! OUTPUT: saved in module_fastj_cmnmie
    !   real tauaer  ! aerosol optical depth 气溶胶光学厚度
    !        waer    ! aerosol single scattering albedo 单次散射反照率
    !        gaer    ! aerosol asymmetery factor 不对称因子
    !        extaer  ! aerosol extinction 消光
    !        l2,l3,l4,l5,l6,l7 ! Legendre coefficients, numbered 0,1,2,......  勒让德系数
    !        sizeaer ! average wet radius 平均湿半径
    !	 bscoef ! aerosol backscatter coefficient with units km-1 * steradian -1  JCB 2007/02/01 气溶胶后向散射系数
    ```

- 问题：

  - **如果要修改MIE散射的计算结果，可以在 call mieaer 语句之后添加修改**

  - 怎么在wrfout中输出tauaer等参数？grid%tauaer1?改register文件， namelist.input中设置  **opt_pars_out             = 1,**

  - 计算AOD： To calculate AOD, you just need to integrate the extinction coef throughout the entire model column; i.e. multiple the extinction coef by the thickness of the model box and sum all the boxes:

    AOD = ∑ EXT(l) * dz(l)

    With these namelist options, you should have EXTCOEF55 variable in the output that represents the exticntion coefficient at 550 nm in the units of km^-1. You need to calculate layer thickness (can be done by using PH and PHB variables from the WRF-Chem output). The final step is to calculate AOD as multiplication of EXTCOEF55 with the layer thickness (make sure you have it in km). 

  - Alternatively, **you can add "tauaer1, tauaer2, tauaer3, and tauaer4" to the model output using runtime I/O option**. They represent layer AOD at 300, 400, 600, and 999 nm. The vertical integral of these quantities will give you total AOD at these wavelengths and then you can use the Angstrom power law to derive AOD at any wavelength. 

  - ```matlab
    ########################################### Calculation Using MATLAB ####################################################
    
    Height_in_Km = ((PH + PHB)/(9.81))*(1000) ##### This will calculate the height between WRF levels in Km where PH = Perturbation Geopotential and PHB = Base-State Geopotential.
    
    Then if you have number of levels in your output is suppose K
    
    for X=1:K
        Distance_Between_Levels (:,:,X) = Height_in_Km(:,:,X+1) - Height_in_Km(:,:,X);
        EXTCOF55_Converted(:,:,X) = (EXTCOF55(:,:,X).*Distance_Between_Levels(:,:,X));
    end
    
    AOD(:,:)= nansum(EXTCOF55_Converted,3);  ############## Mean of the Level Dimension #########
    
    #################################################### END ##############################################################
    ```

  - The way I calculate AOD is by using the variables named TAUAER1, TAUAER2, TAUAER3 and TAUAER4.

    These variables are the aerosol optical depth of each model layer at 300, 400, 600 and 1000 nm, respectively. To calculate the optical depth at another wavelength then you have to calculate the Angstrom exponent following the formula shown in https://en.wikipedia.org/wiki/Angstrom_exponent

    E.g. I calculate the total AOD in the column at 550 nm using TAUAER2 and TAUAER3 because the 550 nm wavelength falls between 400 and 600 nm:

    angstrom_exponent =  -( log(TAUAER2) - log(TAUAER3) ) / log(400/600) 

    AOD550_3D     =  TAUAER2 * ( (400/550) ^ angstrom_exponent )

    AOD550_2D     =  sum(AOD550_2D,3)

    AOD550_3D is the 3-dimensional AOD where you have an optical depth for each layer of your model. Then, the total column AOD is simply the sum across the third dimension.

    If the variables TAUAER<n> are not listed in your model outputs, then you need to modify the registry file for chemistry (Registry/registry.chem)and compile again:

    state   real  tauaer1     ikj   misc     1     -    rh    "TAUAER1"        "bin 1 layer optical thickness" "?"

    state   real  tauaer2     ikj   misc     1     -    rh    "TAUAER2"        "bin 2 layer optical thickness" "?"

    state   real  tauaer3     ikj   misc     1     -    rh    "TAUAER3"        "bin 3 layer optical thickness" "?"

    state   real  tauaer4     ikj   misc     1     -    rh    "TAUAER4"        "bin 4 layer optical thickness" "?"

```fortran
! key options for aerosol optical properities 
aer_op_opt  ! 1-5 设置光学性质计算方法
			! 根据aer_op_opt 设置 option_method 和 option_mie. 
			! option_method is used to set the method of aerosol optical properties 
			! 1.  volume averaging mixing
			! 2. Maxwell-Garnett mixing 
			! 3. shell-core
			! option_mie is used to set the method of mie routines
			
     aer_op_opt_select: SELECT CASE(config_flags%aer_op_opt)
     CASE (VOLUME_APPROX)
       option_method=1
       option_mie=1
     CASE (MAXWELL_APPROX)
       option_method=2
       option_mie=1
     CASE (VOLUME_EXACT)
       option_method=1
       option_mie=2
     CASE (MAXWELL_EXACT)
       option_method=2
       option_mie=2
     CASE (SHELL_EXACT)
       option_method=3
       option_mie=2
     CASE DEFAULT

! option_mie    选择怎么计算mie散射
! option_method 选择怎么计算光学性质

! namelist.input 选项
aer_op_opt  = 1 aerosol optical properties calculated based upon volume approximation
            = 2 aerosol optical properties calculated based upon Maxwell approximation
            = 3 aerosol optical properties calculated based upon exact volume approximation
            = 4 aerosol optical properties calculated based upon exact Maxwell approximation
            = 5 aerosol optical properties calculated based upon exact shell approximation

```

-  说明
  - **[Bond et al., 2013, JGR]** 提到: using the Bond and Bergstrom [2006] refractive index and density, **Adachi et al. [2010]** calculated $MAC_{BC}$ for uncoated spheres as 6.4 $m^2g^{-1}$, with increases for volume mixing (13.6 $m^2g^{-1}$), core shell (13.3 $m^2g^{-1}$), Maxwell-Garnet effective medium approximation (12.0 $m^2g^{-1}$), and realistic coated BC particles (9.9 $m^2g^{-1}$) at 550 nm wavelength.

  - Climate models assume soot particles are spherical and  

    (1) uncoated (**external‐mixing model**), 

    (2) concentrically coated (**core‐shell model**), 

    (3) **homogeneously mixed** with other materials on a molecular scale (**volume‐mixing model)**,

    (4) mixed with other aerosol particles according to rules such as the **Maxwell‐Garnet effective medium approximation** (MGEMA), which assumes that isolated soot spherules are suspended in an embedding material

  ![image-20221221145507232](WRFChem/image-20221221145507232.png)
---
title: 标准地图问题
date: 2022-10-25 09:18:14
toc: true
tags: visualization
categories:
  - map
excerpt: 审图号问题
---

# 标准地图审图号shp

- GS(2020)4619

```python
shpdir = "D:/云文件/数据/shpfiles/GS(2020)4619"
world_border  = Reader(f"{shpdir}/coastline_withoutchina.shp")
china_border  = Reader(f"{shpdir}/china.shp")
province_border  = Reader(f"{shpdir}/province.shp")
ax.add_geometries(china_border.geometries(),
                  ccrs.PlateCarree(),edgecolor='k', linewidth=0.5, zorder=3,
                  facecolor='none')
ax.add_geometries(province_border.geometries(),
                  ccrs.PlateCarree(),edgecolor='k', linewidth=0.2, zorder=3,ls='--',
                  facecolor='none')
ax.add_geometries(world_border.geometries(),
                  ccrs.PlateCarree(),edgecolor='k', linewidth=0.2, zorder=3,
                  facecolor='none')
```



# 南海地图

- 注意需要和主图一样的填充数据

```python
# 南海小地图
ax_n = inset_axes(ax, "100%", "100%", 
                  bbox_to_anchor=(0.88,0.01,0.15,0.3),
                  bbox_transform=ax.transAxes, #loc="center",
                  axes_class=GeoAxes, 
                  axes_kwargs=dict(map_projection=proj))
ax_n.pcolor(x, y, data2d, cmap=incmap,vmin=z_min, vmax=z_max,alpha=0.8)
ax_n.set_extent([105, 123, 0, 25], ccrs.PlateCarree())
ax_n.add_geometries(china_border.geometries(),
                    ccrs.PlateCarree(),edgecolor='k', linewidth=0.2, zorder=2,
                    facecolor='none')
```

- 实例脚本

```python
def csplot(ax,x,y,data2d,incmap=cmaps.GMT_seis_r,z_min=0,z_max=75,htitle=None,vtitle=None,addmax=0,final=False,proj=None):
    cs = ax.pcolor(x, y, data2d, cmap=incmap,vmin=z_min, vmax=z_max,edgecolors='face')
    if(final):
        ax.add_geometries(world_border.geometries(),
                    ccrs.PlateCarree(),edgecolor='k', linewidth=0.2, zorder=2,
                    facecolor='none')
        ax.add_geometries(china_border.geometries(),
                    ccrs.PlateCarree(),edgecolor='k', linewidth=0.5, zorder=2,
                    facecolor='none')
    # 设置 colorbar
    ax.spines['top'].set_visible(False)
    ax.spines['right'].set_visible(False)
    ax.spines['bottom'].set_visible(False)
    ax.spines['left'].set_visible(False)
    if (addmax==1):
      for key,values in regiondict.items():
        number = regionmean(data2d,values)
        # number = data2d.max()
        ax.text(0.02, 1.-values*0.1, '%s: %.1f' %(key,number),fontsize=12,transform=ax.transAxes, ha='left', va='center',
        color='k',zorder=3,) # bbox=dict(boxstyle="square",ec='w',fc='w',alpha=.8))
      xy2 = np.array([0.0,0.45])
      if(number>100):
        rect = mpathes.Rectangle(xy2,0.38,0.6,color='#8ecae6',transform=ax.transAxes,zorder=3)
      else:
        rect = mpathes.Rectangle(xy2,0.35,0.6,color='#8ecae6',transform=ax.transAxes,zorder=3)
      ax.add_patch(rect)
    elif(addmax==0):
        number = data2d.mean()
        ax.text(0.08, .1, '%.1f' %(number),fontsize=12,transform=ax.transAxes, ha='left', va='center',
        color='k',
        bbox=dict(boxstyle="square",ec='w',fc='w',alpha=.8)
        )
    else:
        pass
    if(htitle): ax.text(0.5, 1.1, htitle,fontsize=15,transform=ax.transAxes, ha='center', va='center')
    if(vtitle): ax.text(-0.1, 0.5, vtitle,fontsize=15,transform=ax.transAxes,rotation='vertical', ha='center', va='center')
###############################################################################################################
    # 南海小地图
    ax_n = inset_axes(ax, "100%", "100%", 
                        bbox_to_anchor=(0.88,0.01,0.15,0.3),
                        bbox_transform=ax.transAxes, #loc="center",
                        axes_class=GeoAxes, 
                 axes_kwargs=dict(map_projection=proj))
    ax_n.pcolor(x, y, data2d, cmap=incmap,vmin=z_min, vmax=z_max,alpha=0.8)
    ax_n.set_extent([105, 123, 0, 25], ccrs.PlateCarree())
    ax_n.add_geometries(china_border.geometries(),
                ccrs.PlateCarree(),edgecolor='k', linewidth=0.2, zorder=2,
                facecolor='none')
###############################################################################################################
    return cs, ax_n
```


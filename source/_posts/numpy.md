---
title: numpy
date: 2022-10-23 18:09:12
tags: data analysis
categories:
  - python
---



# 指定顺序排序

``` python
list_custom = ['337城市',"京津冀及周边","汾渭平原", '长三角', "成渝地区","珠三角"]
regiondata['Region'] = regiondata['Region'].astype('category')
regiondata['Region'].cat.reorder_categories(list_custom, inplace=True)
regiondata.sort_values('Region', inplace=True)
```

# 合并数组

```python
np.column_stack((a,b))  #作为列合并
np.row_stack((a,b))  # 作为行
```

# 日期index

```python
df = pd.read_csv(fname,index_col="datetime")
df.index = pd.to_datetime(df.index,format="%Y-%m-%d")

df.index.month # 获取月份
df.index.year # 获取年份

df.loc['2000-6-1':'2000-6-10'] # 切片

df[df.index.month.isin([5, 6, 7, 8, 9, 10])] # 按月份切片

# 时区转换
df.index = pd.to_datetime(df.time,format="%Y-%m-%dT%H:%M:%S", utc=True)
df.index.tz_convert('Asia/Shanghai')
```



# 常见

```python
# 经过pandas 读取后, 再写出,部分数据小数位数超长
# 在read_csv时, 加入参数float_precision=“round_trip”, 所有数据会当做string读取, 使用时再进行相应的转换为float
df = pd.read_csv('%s/%s_%s_%s.csv' %(outputdir, region, var, YYYYMM),float_precision="round_trip")

# 三维计算相关系数
def pearsonr_2D(x, y):
    """computes pearson correlation coefficient
       where x is a 1D and y a 2D array"""
    upper = np.sum((x - np.mean(x,axis=0)) * (y - np.mean(y, axis=0)), axis=0)
    lower = np.sqrt(np.sum(np.power(x - np.mean(x,axis=0), 2),axis=0) * np.sum(np.power(y - np.mean(y, axis=0), 2), axis=0))
    rho = upper / lower
    return rho
A = np.array([[[1,3,4],[4,6,5]],[[5,7,8],[1,2,23]],[[3,9,4],[5,7,5]]])
B = np.array([[[7,1,3],[14,4,2]],[[15,6,1],[6,7,91]],[[3,9,4],[5,7,5]]])
c = pearsonr_2D(A,B)

for i,region in enumerate(region):
    print(i,region)
# 创建dataframe追加行数据
df = pd.DataFrame(columns = ["ebayno", "p_sku", "sale", "sku"]) #创建一个空的dataframe 
df = df.append(dframe1.loc[dframe1.p_sku == ps], ignore_index=True)  #忽略索引,往dataframe中插入一行数据 

df = df.append([{'site':site,'MB':mb,'NMB':nmb,'NME':nme,'r2':r2}],ignore_index=True)
来自 <http://blog.csdn.net/zn505119020/article/details/77324029> 

合并
df = pd.concat([inputdf,classdf],axis=1)

根据分组统计，对不同行操作
gp_col = 'SITE'
df_max = obs.groupby(gp_col)[species].max()
df_min = obs.groupby(gp_col)[species2].min()

插入列 insert（位置，列名，数据）
df_max.insert(0,"站点",sitename)

DataFrame 切片操作，loc, iloc
print df.loc[1:3, ['total_bill', 'tip']]
print df.loc[1:3, 'tip': 'total_bill'] 
print df.iloc[1:3, [1, 2]] 
print df.iloc[1:3, 1: 3]

来自 <https://blog.csdn.net/ly_ysys629/article/details/55224284> 



读取csv
pd.read_csv("filename",sep=',',header=None,index=False)  # sep 分隔符
obs  = pd.read_csv("d04_chem.txt",delim_whitespace = True) 空格分割
csv编码问题： VS code先用正确编码打开（GB…?），然后保存为正常编码（能正确显示GB…），然后重新保存为UTF-8格式，读取正常。

获取数据框列名
print(df.columns)   
获取数据框列
df["column-name"]
选取行
df[df["column-name"]=="spec-name"]
选取数据
df.column[index]
df.ix[0,1]

列表转换成dataframe
df = pd.DataFrame([w1_p,w2_p,w3_p,w4_p],index=['w1_pre','w2_pre','w3_pre','w4_pre'])
修改dataframe的列名
df.columns=['site','time','aqi','so2','no2','pm10','co','o3','pm2_5']

```



#  查找

```python
cname = allsite.query("Area == @city").iloc[0,4]
cname = allsite.query("Area == '北京'").iloc[0,0]
```

# 日期处理

- 某个月的开始和结束日期

```python
from dateutil.relativedelta import relativedelta
start = datetime.datetime.strptime("2017-%s-01" %mon,"%Y-%m-%d")
end = start + relativedelta(months=+1) + datetime.timedelta(days=-1)
```

- 时区转换

```python
data.index = pd.to_datetime(data.index,format='%Y%j%H').tz_localize('UTC').tz_convert("Asia/Shanghai")
```

- 设置时间格式

```python
res.index = res.index.strftime('%Y-%m-%d')
```

# list 转 dictionary @ data frame

```python
keys = ["price","toalAmount"]
values = ["0.01","10000"] 
d = dict(zip(keys,values))

pd.DataFrame([dict(zip(columns,temp))],columns=columns,index=None)
```

# 等差数列

```python
cticks = np.linspace(zmin, zmax, num=11, dtype=int)
```

# mask

```python
def regionmean(lat,lon,data,corners):
    llcrnrlon,urcrnrlon,llcrnrlat, urcrnrlat = corners
    mask = np.where((lat<urcrnrlat)&(lat>llcrnrlat)&(lon<urcrnrlon)&(lon>llcrnrlon),1,0)
    if(len(data.shape)==3):
        mask3d = np.broadcast_arrays(mask[np.newaxis,:,:],data)[0]
        datamask = np.ma.masked_where(mask3d!=1, data)
        return np.nanmean(np.nanmean(datamask,axis=1),axis=1)
    else:
        datamask = np.ma.masked_where(mask!=1, data)
        return np.nanmean(datamask.flatten())
```

# 对应元素运算

```python
a = np.array([[1, 2]])
b = np.array([[2, 4]])
r1 = a + b  # [[3 6]]
r2 = a - b  # [[-1 -2]]
r3 = a * b  # [[2 8]]
r4 = a / b  # [[0.5 0.5]]
```

# 拼接两段数据

```python
data2 = f.loc['2017-06-01':'2017-08-30']
data3 = f.loc['2017-09-01':'2017-11-30']
data1 = f.loc['2017-03-01':'2017-05-31']
data4 = f.loc['2017-01-01':'2017-02-28']
data5 = f.loc['2017-12-01':'2017-12-31']
data0 = pd.concat([data4,data5])

>>> a=np.array([[1,2,3],[4,5,6]])
>>> b=np.array([[11,21,31],[7,8,9]])
>>> np.concatenate((a,b),axis=0)
array([[ 1,  2,  3],
       [ 4,  5,  6],
       [11, 21, 31],
       [ 7,  8,  9]])
>>> np.concatenate((a,b),axis=1)  #axis=1表示对应行的数组进行拼接
array([[ 1,  2,  3, 11, 21, 31],
       [ 4,  5,  6,  7,  8,  9]])

ax.plot(pm25[0:59],data[0:59,10],'.',c='k',alpha=0.5,label='Winter')
ax.plot(pm25[334:365],data[334:365,10],'.',c='k',alpha=0.5)
ax.plot(pm25[59:151],data[59:151,10],'.',c='b',alpha=0.5,label='Spring')
ax.plot(pm25[151:243],data[151:243,10],'.',c='g',alpha=0.5,label='Summer')
ax.plot(pm25[243:334],data[243:334,10],'.',c='r',alpha=0.5,label='Autumn')

winter = np.concatenate((arrs[0:59],arrs[334:365]),axis=0)
spring = arrs[59:151]
summer = arrs[151:243]
autumn = arrs[243:334]
```





```python
x = np.linspace(0,31,31);  print(x)
y = np.sum(data,axis=1)
x_new = np.linspace(0,31,100)
tck = interpolate.splrep(x,y)
y_bspline = interpolate.splev(x_new,tck)
ax2.plot(x,y,"wo",ms=3)
ax2.plot(x_new,y_bspline,'w-',linewidth=0.5)
```
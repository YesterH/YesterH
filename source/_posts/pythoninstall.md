---
title: python安装
date: 2022-10-25 09:09:45
toc: true
tags: software
categories:
  - python
excerpt: 安装python及相关配置
---

# requirements

```python
numpy
pandas
xarray
Cartopy
matplotlib==3.2.2
notebook
cmaps
cycler
scipy
seaborn
Shapely
jupyterlab


conda install --yes --file requirements.txt

pip install https://github.com/j08lue/pycpt/archive/master.zip
    

```

- pycpt 导入cpt配色方案

[GitHub - j08lue/pycpt: Python tools to load and handle cpt (GMT format) color maps for use with matplotlib, e.g. from cpt-city](https://github.com/j08lue/pycpt)

https://github.com/j08lue/pycpt



# jupyterlab 安装设置

- 安装miniconda
- conda install jupyterlab
- 复制一个快捷方式，修改成：
- C:\Software\conda\python.exe C:\Software\conda\cwp.py C:\Software\conda C:\Software\conda\python.exe C:\Software\conda\Scripts\jupyter-lab-script.py "D:\云文件\代码\python\jupyter"
- 并且起始位置改成最后的那句 D:\云文件\代码\python\jupyter

# 生成配置文件
- jupyter lab --generate-config
- 单独的浏览器窗口, 修改config文件中的
- c.NotebookApp.browser = 'C:/Program Files (x86)/Microsoft/Edge/Application/msedge.exe --app=%s'
- *c.NotebookApp.notebook_dir = ''*

# 安装拓展

- 打开jupyterlab，enable拓展
- 关闭jupyterlab，安装nodejs： conda install nodejs
- 安装拓展，如：jupyter labextension install @jupyterlab/toc

# 导出html 隐藏代码

```python
jupyter nbconvert Fig1_obs.vs.pred.ipynb --no-input --to html
```



# 字体设置成Helvetica

- 下载Helvetica.ttf 文件

- 找到matplotlib文件位置

  - ```python
    import matplotlib
    print(matplotlib.matplotlib_fname())
    ```

- 把字体文件移动到 matplotlib/mpl-data/fonts/ttf

- 修改 matplotlibrc文件

  - ```python
    # 改成这样 Helvetica 在最前面
    font.sans-serif : Helvetica, DejaVu Sans, Bitstream Vera Sans, Computer Modern Sans Serif, Lucida Grande, Verdana, Geneva, Lucid, Arial, Avant Garde, sans-serif
    ```

- 删除缓存文件
- ```bash
  $ cd ~/.matplotlib`
  `$ rm fontlist*
  ```

#  镜像设置

## 临时使用

pip install -i https://pypi.tuna.tsinghua.edu.cn/simple some-package
注意，simple 不能少, 是 https 而不是 http

## 设为默认

升级 pip 到最新的版本 (>=10.0.0) 后进行配置：

pip install pip -U
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple

如果您到 pip 默认源的网络连接较差，临时使用本镜像站来升级 pip：

pip install -i https://pypi.tuna.tsinghua.edu.cn/simple pip -U

## conda添加镜像

```
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud//pytorch/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/
conda config --set show_channel_urls yes
```
显示已经有的源
```
conda config --show channels
```
链接：https://www.jianshu.com/p/7e663bb0d904




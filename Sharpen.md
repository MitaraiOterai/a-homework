## 作业1
陶嘉威 502023070023 自然地理
[Data Access | Landsat Science (nasa.gov)](https://landsat.gsfc.nasa.gov/data/data-access/)数据获取
[Band Combinations (pdx.edu)](https://web.pdx.edu/~nauna/resources/10_BandCombinations.htm)
### 1. 研究区及数据
巢湖区域[Data Access | Landsat Science (nasa.gov)](https://landsat.gsfc.nasa.gov/data/data-access/)
|Data Set Attribute|Attribute Value|
|---|---|
|[Landsat Product Identifier L1](https://www.usgs.gov/centers/eros/science/landsat-collection-2-data-dictionary#landsat_product_id)|LE07_L1TP_121038_20230627_20230723_02_T1|
|[Landsat Scene Identifier](https://www.usgs.gov/centers/eros/science/landsat-collection-2-data-dictionary#landsat_scene_id)|LE71210382023178EDC00|
Landsat7 图像，包含R、G、B、IR、SWIR* 2（30m）和一个全波段（15m）
经裁剪得到东南部巢县部分。
### 2. 实验方法
#### 2.1 融合
选取三种方案：321、432、543进行RGB融合，效果如下：
![[Pasted image 20231028222053.png]]
<center>321组合，真彩色</center>
![[Pasted image 20231028222146.png]]
<center>432组合，假彩色方案一，植物呈现红色</center>
![[Pasted image 20231028222243.png]]
<center>543组合，假彩色方案二，植物呈现鲜绿色</center>
分别使用CN（Brovey）、HSV、NNDiffuse Pan三种方法进行融合（Envi内置的数个Sharpen工具）
#### 2.2 定量评价
肉眼观察，对不同通道，不同融合方式，不同插值模式进行肉眼分析，主要有亮度、色彩、辨明度三个维度。
#### 2.3 定性评价
使用python中的NumPy、OpenCV、Skimage库，计算均方误差（MSE）、结构相似性指数（SSIM）、互信息（Mutual Information）。
MSE计算两个图像之间像素差异的平方和，值越小，图像越相似，误差越小；
``` python
# 计算均方误差 (MSE)
    MSE_value = np.mean((original_image - fused_image) ** 2)
```
SSIM用于评估图像质量，包括亮度、对比度、结构相似性，越接近1，两个图像越接近；
``` python
# 计算结构相似性指数 (SSIM)
    SSIM_value = ssim(original_image, fused_image, win_size=3)
```
MI越高，二者表达的相同信息越多，融合效果越好。
```python
    def calculate_mutual_information(image1, image2):
        # 将图像转换为一维数组
        image1_flat = image1.ravel()
        image2_flat = image2.ravel()
# 计算互信息
        mi = mutual_info_score(image1_flat, image2_flat)
        return mi
    MI_value = calculate_mutual_information(original_image, fused_image)
```
### 3. 实验结果
输出多幅图幅。
定性评价三种波段组合下的HSV融合方式，相同波段组合波段下的HSV、NNSP、CN（B）融合方式；
定量评价HSV，432波段融合下，Nearest Neighbor，Bilinear，Cubic Convolution三种插值方式，并于NNPS融合方案作对比。
### 4. 结果评价 
#### 4.1 定性评价
1. 颜色波段选为432时，能很好地区分植物、建筑物和水体，543稍有失真，321颜色区分度最差。之后定量比较时仅仅考虑432波段。
	![[Pasted image 20231028231210.png]]
	<center>波段为321、432、543时，采用HSV的融合效果</center>
2. 定性：颜色可区分度，目测准确度、亮度等：HSV>NNPS>>CN(B)
	![[Pasted image 20231028230300.png]]<center>波段选为432时，HSV、NNPS、CN（B）的融合效果</center>
	![[Pasted image 20231028230616.png]]
	<center>波段选为543时，HSV、NNPS、CN（B）的融合效果</center>
	![[Pasted image 20231028231435.png]]
	<center>波段为321时HSV、NNPS的融合效果</center>
3. CN（B）和NNPS融合过程中，有三种插值模式可以选择：Nearest Neighbor，Bilinear，Cubic Convolution，三种方式融合效果肉眼难以鉴别，在定性评价部分略去，见定量评价。
#### 4.2 定量评价
输出表格结果如下：![[Pasted image 20231029130107.png]]文件名构成：融合方案、重采样方法、30m波段。
### 5. 主要结论
1. NPS融合方式较HSV，MSE更小，SSIM更接近1，但MI较低，原因可能为NNPS融合后亮度更低更接近全色图像，但是出现了些许信息丢失。
2. 在HSV融合中，MSE差距不大，波段组合为321时，三个融合评价指标最好，其次为432，最后为543。推测原因321波段组合为真彩色，更加接近全波段。
3. 采用的融合方案、使用的波段、重采样方法对融合效果的影像逐级下降：三种重采样方式的差别在1%左右。

